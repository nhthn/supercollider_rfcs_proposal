- Title: Improve querying for available audio devices
- Date proposed: June 25, 2019
- RFC PR: https://github.com/snappizz/rfcs/pull/0000
- SuperCollider issue: https://github.com/supercollider/supercollider/issues/1557, https://github.com/supercollider/supercollider/issues/271

# Summary

I'd like to propose reworking of the way we can query audio devices available in the system.

# Motivation

Currently querying system devices is available through `ServerOptions.devices`.

There are following issues with the current approach:
1. it works only on macOS (CoreAudio)
2. it does not list any details, e.g.:
    - whether the device provides inputs or outputs
    - how many inputs/outputs are available
3. it duplicades code between sclang and scsynth, as the audio backend code needs to be also included in the language

Arguably, the most pressing issue is that this method is not cross-platform. However, in the process of reworking this, I think we should consider other issues as well.

I think there are two ways we could resolve issues 1. and 2.:
1. we can add PortAudio functinality to sclang in order to list devices using `ServerOptions.devices`
2. we can add a command line switch to the server to list available devices and parse this in the language

I would like to propose that we go with option #2 for the following reasons:
- we won't be adding more dependencies to sclang and will avoid more code duplication
- we will be able to provide more details about devices without breaking backwards compatibility on macOS
- we will enable non-sclang clients to query for audio devices

This would mostly affect windows users, who don't have a way to query devices currently. For macOS, it would improve the functionality with the invormation on available number of inputs/outputs

The request for being able to list devices on Windows comes up semi-regularly on the list and in github issues when setting input/output devices is discussed, e.g. [#4395](https://github.com/supercollider/supercollider/issues/4395)

# Specification

I propose the following implementation:

- add new command line switch to the server
  - since command line switches -d, -D, -l, -L are used, I suggest `-g` as in `[g]etAudioDevices`
- this would return the information in a parsable format and then the server quits
  - returned information: device name, number of inputs, number of outputs, (possibly) list of supported samplerates
  - format TBD (YAML vs JSON vs XML?)
- add sclang method to obtain and parse the values
  - since we are using server executable directly, I suggest usign a new class method to `Server`: `Server.getAudioDevices`
  - this method would return an array of dictionaries:
  ```supercollider
  Server.getAudioDevices;
  // returns
  [
    Dictionary[ ('name' -> "microphone"), ('outputs' -> 0), ('inputs' -> 2), ('availableSampleRates' -> [ 44100, 48000, 96000 ]) ],
    Dictionary[ ('name' -> "speakers"), ('outputs' -> 2), ('inputs' -> 0), ('availableSampleRates' -> [ 44100, 48000, 96000 ]) ]
  ]
  ```
# Drawbacks

I imagine that having to run the server command might be more heavy than calling a primitive from the sclang.

# Unresolved Questions

- which format to use?
  - XML is very popular/easy, but we don't have built-in parsing (and we can't rely on a Quark for this)
  - YAML vs JSON: sclang can parse either. What are the advantages of one vs the other?
- return options
  - should we list all available samplerate options? should these be ints or floats?
- sclang method 
  - I propose to use `Server.getAudioDevices`, but the existing macOS functionality is in `ServerOptions`. Would `ServerOptions` be a better place to implement this?
- sclang return format
  - is an array of Dictionaries a good return format?
- should this be also implemented in supernova?
