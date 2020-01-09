# PROPOSAL FOR A SOUNDFONT v3.00 STANDARD

## Disclaimer

This work is a proposal for updating the soundfont standard and is by no means an official document on which you should base a development, at least as long as it's not adopted by major actors working with soundfonts if it's possible to list them (at least FluidSynth / FluidLight for the open-source community).


## Introduction

The SoundFont standard has been designed by the Creative Labs company in 1998 ([version 2.01](https://github.com/davy7125/soundfont-standard-v3/raw/master/sfspec21.pdf)) and 2006 ([version 2.04](https://github.com/davy7125/soundfont-standard-v3/raw/master/sfspec24.pdf)). This standard describes how to gather musical samples and parameters in a .sf2 file so that it can be used by a synthesizer.

Technology and usage have evolved and there is a growing need for an update on this standard.

* The standard has primarily be designed so that .sf2 files can be read and played by a hardware system. The hardware capabilities of the time induced limitations that are now obsolete: soundfont files are now read by software synthesizers that are much more powerful. Limitations can thus be removed.

* Soundfonts are more and more used in mobile devices or as third party (for instance in MuseScore) and there is a need for decreasing the size of the soundfont so that it can be easily embedded.

* Due to these defects several formats based on soundfonts have been designed (the list may not be exhaustive):
  * to compress soundfonts: sfArk, SfPack, sf3, sf4;
  * to increase its capabilities: sfz (providing also another packaging), Viena - while still using the sf2 v2.xx format - extended some of the sf2 capabilities.

Providing an update to the soundfont standard could:
* limit the need for creating custom formats,
* increase the soundfont notoriety,
* result in a whole set of software working together around an up-to-date format.


## Requirements

### Metadata

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Soundfont description**|A soundfont description is made of:<br><br><ul><li>a soundfont title, with a maximum length of 256;</li><li>a date, with a maximum length of 32;</li><li>an author, with a maximum length of 256;</li><li>a copyright, with a maximum length of 256;</li><li>a description, with a maximum length of 65536;</li><li>a soundfont version;</li><li>a field for specifying a wave table sound engine, with a maximum length of 256;</li><li>a field for specifying a wave table ROM, with a maximum length of 256;</li><li>a field for specifying a product, with a maximum length of 8;</li><li>a field for specifying soundfont compatible tools, with a maximum length of 256.</li></ul>|A soundfont description is made of:<br><br><ul><li>a soundfont title;</li><li>a date written as a UTC timestamp so that it can be understood by everybody;</li><li>an author;</li></li>a copyright;</li><li>a description;</li><li>a soundfont version;</li><li>an unlimited number of possible additional fields.</li></ul><br>Length limits are all removed.|
|**Element names**|The maximum length of a sample, instrument or preset name is 20.|The length of a sample, instrument or preset name is not limited.|
|**Sample description**|No description can be added to a sample.|A description can be added to a sample. This could be useful for instance to specify a license associated to a sample, or describe the way a sample has been generated.|
|**CC value initialization**|MIDI controller values are not initialized when opening a soundfont, resulting in a possible undefined initial state for modulators using CC values.|Default MIDI controller values can be specified so that the initial state of modulators is known.|
|**Key switch area**|Key switch are not allowed.|A key switch area can be defined:<br><br><ul><li>lower key;</li><li>upper key;</li><li>default key.</li></ul><br>This step is mandatory if key switches are used in a "conditional start" below.|

### Samples

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Sample types**|Raw data, either 16 or 24 bit, is used.|Raw data (.wav), FLAC or OGG can be used. The precision in bits is now sample-dependent, not fixed for the entire soundfont.|
|**Stereo support**|Both sides of a stereo sample are processed as mono samples. All parameters are duplicated at the sample and instrument levels. Default panning for these samples are either -50 (left) or 50 (right).|Both sides of a stereo sample share the same configuration at the sample and instrument levels. Default panning become 0 at the instrument level.|
|**Multiple loops**|At the sample level, it is possible to specify only one loop.|At the sample level, an unlimited number of loops can be specified. If several loops are correctly intricate, they will be played successively during playback.|

### Instrument / preset configuration

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Round-robin sample playback**|At the instrument level, one division is linked to only one sample.|At the instrument level, one division can be linked to an undefined number of samples, played sequentially. For example, using one key triggers a first sample and reusing the same key triggers a second sample. In such a division, comprising several samples, all parameters are duplicated except two: key and velocity ranges.<br><br>The same can be done at the preset level: one division can be linked to several instruments, played sequentially.|
|**Filters**|At the instrument and preset level, one low-pass filter of the 2nd order (12 dB / octave) is automatically assigned and it is possible to configure it.|At the instrument or preset level, no low-pass filter are automatically assigned. It is then possible to add up to six configurable filters:<br><br><ul><li>low-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>high-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>band-pass filter, 2nd order (12 dB / octave);</li><li>notch filter, 2nd order (12 dB / octave).</li></ul><br>*Note: the purpose of specifying a variable at the preset level is to modify the corresponding variable for the children at the instrument level. So if a high-pass filter is defined at the preset level and a low-pass filter is defined at the instrument level, the configuration of the high-pass filter will be ignored since there is no corresponding high-pass filter at the instrument level.*|
|**Playback modes**|At the instrument level, it is possible to choose between three modes for playing a sample:<br><br><ul><li>"no loop"<br>The sound is triggered when a key is triggered, no loop are played.</li><li>"loop"<br>The sound is triggered when a key is triggered, a loop is used indefinitely if defined at the sample level.</li><li>"loop + end"<br>The sound is triggered when a key is triggered, a loop is used as long as the key is help, then the sound is played to the end when the key is released.</li></ul>|In addition to the 3 existing modes, 2 playing modes are added:<br><br><ul><li>"release"<br>The sound is triggered when a key is released. In this mode, the delay / attack / decay / sustain /release of the volume envelop will be used to compute the initial velocity of the sample.</li><li>"back and forth"<br>The sound is triggered on a key press, enters in the loop and go back in a reverse way to the loop start once the loop end has been reached.</li></ul><br>There is now 5 distinct playing modes and based on them, 5 more modes will be added considering a "reverse" option. It will be possible to read a sample in a reverse way from the end to the start.|
|**Independent envelopes**|At the instrument or preset level, it's possible to use 2 envelopes:<br><br><ul><li>one for the volume (vol env);</li><li>one for the pitch and / or filter (mod env).</li></ul>|At the instrument or preset level, it's possible to use 3 independent envelopes:<br><br><ul><li>one for the volume (vol env);</li><li>one for the pitch (pitch env);</li><li>one for the filter (filter env).</li></ul><br>*Note: it will be possible to specify only one envelope for the pitch instead of 2. This could be seen as a regression but also as a clarification.*|
|**Independent LFOs**|Similarly to the envelopes, it's possible at the instrument or preset level to use 2 LFOs:<br><br><ul><li>one for the volume and / or pitch and / or volume (Mod LFO);</li><li>one for the pitch (Vib LFO).</li></ul><br>|At the instrument or preset level, it's possible to use 3 independent LFOs:<br><br><ul><li>one for the volume (vol mod);</li><li>one for the pitch (pitch mod);</li><li>one for the filter (filter mod).</li></ul><br>*Note: it will be possible to specify only one LFO for the pitch instead of 2. This could be seen as a regression but also as a clarification.*|
|**Modulator sources**|In a modulator, the possible sources are:<br><br><ul><li>the value 1;</li><li>a general controller (note-on velocity, key number, aftertouch, ...);</li><li>a MIDI controller (CC016, CC072, ...);</li><li>the output of another modulator.</li></ul>|In addition to the 4 existing modulator sources, 3 other sources will be available:<br><br><ul><li>a LFO, defined by<ul><li>a shape (sine, square, triangle, ramp up, ramp down, random)</li><li>a polarity (values in the range [0;1] or in the range [-1;1])</li><li>a frequency</li></ul></li><li>an envelope, defined by<ul><li>a delay</li><li>an attack</li><li>a sustain</li><li>a decay</li><li>a release</li></ul></li><li>a random value, defined by<ul><li>a polarity (values in the range [0;1] or in the range [-1;1])</li></ul></li></ul>|
|**Conditional start**|A sample is triggered when a key is pressed, if the key and velocity match with the key and velocity ranges specified in the divisions triggering the sample. No other conditions are checked.|A sample is triggered when a key is pressed, if the key and velocity match with the key and velocity ranges specified in the divisions triggering the sample. In addition to this, it will be possible to add conditions that must be checked before triggering a sound. These conditions can be of three types.<br><br><ul><li>*MIDI controller value*<br>The value of a MIDI controller can be checked.</li><li>*Pitch bend value*<br>The value of the pitch bend wheel can be checked.</li><li>*Key switch*<br>After having specified a key switch area (as seen above), it will be possible to check if<br><ul><li>a key switch is activated,</li><li>a key switch is NOT activated,</li><li>the last key switch number is a specified value.</li></ul><br></li></ul>For all additional check, the logic can be: >, <, ≥, ≤, = or ≠.|
|**Exclusive class**|The activation of a division triggers a fast release of all other divisions sharing the same exclusive class (if specified).|In addition to the fast release, it will be possible to choose the normal release as specified in the volume envelope.|
|**Default "velocity → filter cutoff" modulator**|Three versions of the default modulator "velocity → filter cutoff" have been used.<br><br>In v2.01, it included a switch (as second source) triggered between velocities 63 and 64, resulting in an ugly jump.<br><br>Creative Labs realized the error and in v2.04, the switch as been removed.<br><br>In the meantime, developers completely removed the default filter "velocity → filter cutoff".|Default filter "velocity → filter cutoff" is removed.|
|**Attenuation**|Only a positive attenuation is allowed, resulting in an attenuation only.|A positive or negative attenuation is allowed, resulting in a possible amplification.|

### Packaging

|Before (v2.xx)|After (v3.00)|
|--|--|
|Soundfonts are all packed in the .sf2 format, containing raw sample data only.|Different packaging are possible depending on the compression inside it. Since the format itself would be the same and only the data storage changes, the ".sf2" could be kept while being prefixed by the compression extension. For example:<br><br><ul><li>"file.sf2"<br>This file contains raw data</li><li>"file.ogg.sf2"<br>This file contains OGG data (compression with loss: the actual .sf3)</li><li>"file.flac.sf2"<br>This file contains FLAC data (lossless compression)</li></ul><br>*Note 1: the compression prefix is just a hint so that the user knows what kind of data he is dealing with. If the compression prefix is missing, a software opening the soundfont would still be able to understand it.*<br><br>*Note 2: after having improved the soundfont capabilities, saving the soundfont v3.00 as an sfz can become straightforward. This is something than could be useful for interoperability.*<br><br>*Note 3: if the extension sf2 cannot be used because of a copyright infringement, the extension sf3 will be used.*|

### Miscellaneous

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Number of generators / modulators**|The number of generators and modulators is limited to 65536.|The number of generators and modulators is not limited.|
|**dB units**|Due to a historical code mistake, dB stored in a soundfont or not reel dB but 0.4 dB.|dB stored in a soundfont are real dB.|


## Description on the file format

*The modifications of the file format will be written when the requirements are completed.*


## Conclusion

If all requirements are solved:

* most of the nowadays needs regarding the sample-based synthesis will be added and it will give fresh air to the soundfont standard;
* soundfont v2.xx can be updated in the new format 3.00 with a loss only in a very specific case: when 2 LFOs or 2 envelopes target the pitch at the instrument or preset level;
* it will be possible to save a v3.00 soundfont as a v2.xx soundfont with a loss that will not prevent the soundfont to be played;
* it will be clearer to the user that .sf2 files contain lossless data while .sf3 files have a lower quality and file size;
* developers shouldn't have the need for another custom format in most use cases.

Then, time will be necessary to provide sound engines and VST compatible with the new version of the standard.
