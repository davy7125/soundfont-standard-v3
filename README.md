# PROPOSAL FOR A SOUNDFONT v3.00 STANDARD

## Disclaimer

This work is a proposal for updating the soundfont standard and is by no means an official document on which you should base a development, at least as long as it's not adopted by major actors working with soundfonts if it's possible to list them (at least FluidSynth / FluidLight for the open-source community).


## Introduction

The SoundFont standard has been designed by the Creative Labs company in 1998 ([version 2.01](https://github.com/davy7125/soundfont-standard-v3/raw/master/sfspec21.pdf)) and 2006 ([version 2.04](https://github.com/davy7125/soundfont-standard-v3/raw/master/sfspec24.pdf)). This standard describes how to gather musical samples and parameters in a .sf2 file so that it can be used by a synthesizer.

Technology and usage have evolved and there is a growing need for an update on this standard.

* The standard has primarily be designed so that .sf2 files can be read and played by a hardware system. The hardware capabilities of the time induced limitations that are now obsolete: soundfont files are now read by software synthesizers that are much more powerful. Limitations can thus be removed.

* Soundfonts are more and more used in mobile devices or as third party (for instance in MuseScore) and there is a need for decreasing the size of the soundfont so that it can be easily embedded.

* Due to these defects several formats based on soundfonts have been designed (the list may not be exhaustive):
  * to compress soundfonts: sfArk, SFPack, sf2pack, sf3, sf4;
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
|**Sample storage**|Samples are stored as raw data, either 16- or 24-bit.|Samples will be stored either as raw data, or can be stored after using the lossless compression FLAC or the lossy compression Opus. These choices have been taken with regards to the use cases of a soundfont.<br><br><ul><li>If the soundfont is intended to be shared, downloaded or embedded easily, then the Opus compression seems wise (Opus compression is free and provides a better quality than OGG).</li><li>If the soundfont must provide the best quality so that it can be reedited for instance, the lossless compression FLAC (also free) will be chosen.</li><li>If we need a fast load of the soundfont, raw data might be chosen.</li></ul><br>In addition to this, the precision in bits is now sample-dependent, not fixed for the entire soundfont.|
|**Stereo support**|Both sides of a stereo sample are processed as mono samples. All parameters are duplicated at the sample and instrument levels. Default panning for these samples are either -50 (left) or 50 (right).|Both sides of a stereo sample share the same configuration at the sample and instrument levels. Default panning become 0 at the instrument level.|
|**Multiple loops**|At the sample level, it is possible to specify only one loop.|At the sample level, an unlimited number of loops can be specified. If several loops are correctly intricate, they will be played successively during playback.|

### Instrument / preset configuration

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Round-robin sample playback**|At the instrument level, one division is linked to only one sample.|At the instrument level, one division can be linked to an undefined number of samples, played sequentially. For example, using one key triggers a first sample and reusing the same key triggers a second sample. In such a division, comprising several samples, all parameters are duplicated except two: key and velocity ranges.<br><br>The same can be done at the preset level: one division can be linked to several instruments, played sequentially.|
|**Filters**|At the instrument and preset level, one low-pass filter of the 2nd order (12 dB / octave) is automatically assigned and it is possible to configure it.|At the instrument or preset level, no low-pass filter are automatically assigned. It is then possible to add up to six configurable filters:<br><br><ul><li>low-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>high-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>band-pass filter, 2nd order (12 dB / octave);</li><li>notch filter, 2nd order (12 dB / octave).</li></ul><br>*Note: the purpose of specifying a variable at the preset level is to modify the corresponding variable for the children at the instrument level. So if a high-pass filter is defined at the preset level and a low-pass filter is defined at the instrument level, the configuration of the high-pass filter will be ignored since there is no corresponding high-pass filter at the instrument level.*|
|**Playback modes**|At the instrument level, it is possible to choose between three modes for playing a sample:<br><br><ul><li>"no loop"<br>The sound is triggered when a key is triggered, no loop are played.</li><li>"loop"<br>The sound is triggered when a key is triggered, a loop is used indefinitely if defined at the sample level.</li><li>"loop + end"<br>The sound is triggered when a key is triggered, a loop is used as long as the key is help, then the sound is played to the end when the key is released.</li></ul>|In addition to the 3 existing modes, 3 others are added:<br><br><ul><li>"release"<br>The sound is triggered when a key is released. In this mode, the delay / attack / decay / sustain of the volume envelop will be used to compute the initial velocity of the sample.</li><li>"one shot"<br>The sound is triggered on a key press and is read until the end with no loop and ignoring the note off event. This is useful for drums.</li><li>"back and forth"<br>The sound is triggered on a key press, enters in the loop and go back in a reverse way to the loop start once the loop end has been reached. Then the sample is read from the loop start to the loop end and so forth.</li></ul>.|
|**Independent envelopes**|At the instrument or preset level, it's possible to use 2 envelopes:<br><br><ul><li>one for the volume (vol env);</li><li>one for the pitch and / or filter (mod env).</li></ul>|At the instrument or preset level, it's possible to use 3 independent envelopes:<br><br><ul><li>one for the volume (vol env);</li><li>one for the pitch (pitch env);</li><li>one for the filter (filter env).</li></ul><br>*Note: it will be possible to specify only one envelope for the pitch instead of 2. This could be seen as a regression but also as a clarification.*|
|**Independent LFOs**|Similarly to the envelopes, it's possible at the instrument or preset level to use 2 LFOs:<br><br><ul><li>one for the volume and / or pitch and / or volume (Mod LFO);</li><li>one for the pitch (Vib LFO).</li></ul><br>|At the instrument or preset level, it's possible to use 3 independent LFOs:<br><br><ul><li>one for the volume (vol mod);</li><li>one for the pitch (pitch mod);</li><li>one for the filter (filter mod).</li></ul><br>*Note: it will be possible to specify only one LFO for the pitch instead of 2. This could be seen as a regression but also as a clarification.*|
|**Modulator sources**|In a modulator, the possible sources are:<br><br><ul><li>the value 1;</li><li>a general controller (note-on velocity, key number, aftertouch, ...);</li><li>a MIDI controller (CC016, CC072, ...);</li><li>the output of another modulator.</li></ul>|In addition to the 4 existing modulator sources, 3 other sources will be available:<br><br><ul><li>a LFO, defined by<ul><li>a shape (sine, square, triangle, ramp up, ramp down, random)</li><li>a polarity (values in the range [0;1] or in the range [-1;1])</li><li>a frequency</li></ul></li><li>an envelope, defined by<ul><li>a delay</li><li>an attack</li><li>a sustain</li><li>a decay</li><li>a release</li></ul></li><li>a random value, defined by<ul><li>a polarity (values in the range [0;1] or in the range [-1;1])</li></ul></li></ul>|
|**Conditional start**|A sample is triggered when a key is pressed, if the key and velocity match with the key and velocity ranges specified in the divisions triggering the sample. No other conditions are checked.|A sample is triggered when a key is pressed, if the key and velocity match with the key and velocity ranges specified in the divisions triggering the sample. In addition to this, it will be possible to add conditions that must be checked before triggering a sound. These conditions can be of different types.<br><br><ul><li>*MIDI controller value*<br>The value of a MIDI controller can be checked.</li><li>*Pitch bend value*<br>The value of the pitch bend wheel can be checked.</li><li>*Key switch*<br>After having specified a key switch area (as seen above), it will be possible to check if<br><ul><li>a key switch is activated,</li><li>a key switch is NOT activated,</li><li>the last key switch number is a specified value.</li></ul></li><li>*First*<br>No other key must be on for triggering the sample.</li><li>*Legato*<br>Another key must be on for triggering the sample.</li></ul>For all conditions that need to compare a value, the logic can be: >, <, ≥, ≤, = or ≠.|
|**Exclusive class**|The activation of a division triggers a fast release of all other divisions sharing the same exclusive class (if specified).|In addition to the fast release, it will be possible to choose the normal release as specified in the volume envelope.|
|**Default "velocity → filter cutoff" modulator**|Three versions of the default modulator "velocity → filter cutoff" have been used.<br><br>In v2.01, it included a switch (as second source) triggered between velocities 63 and 64, resulting in an ugly jump.<br><br>Creative Labs realized the error and in v2.04, the switch as been removed.<br><br>In the meantime, developers completely removed the default filter "velocity → filter cutoff".|Default filter "velocity → filter cutoff" is removed.|
|**Attenuation**|Only a positive attenuation is allowed, resulting in an attenuation only.|A positive or negative attenuation is allowed, resulting in a possible amplification.|

### Miscellaneous

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Number of generators / modulators**|The number of generators and modulators is limited to 65536.|The number of generators and modulators is not limited.|
|**dB units**|Due to a historical code mistake, dB stored in a soundfont or not reel dB but 0.4 dB.|dB stored in a soundfont are real dB.|
|**File size**|Maximum file size is 4 GB due to the 32-bit addresses and sizes stored in a sf2 file.|Addresses and sizes will be written using a 64-bit format, removing the 4 GB limitation.|
|**Number of banks**|128 banks are available.|Up to 16384 (128 × 128) banks are available.|


## Description of the file format

### File extension

File extension *sf2* could be kept if:

* there are no copyright infringement with the Creative company,
* it is considered that existing software opening sf2 files must include at least a check to the version number of the standard so that they either display a warning to the user (for incompatibility) or adapt the way they load the soundfont.

The alternative would be to use the extension *sf3*.

*Note:* a prefix to the extension could optionally be added for soundfonts containing samples compressed with the Opus algorithm - for example "soundfont.compressed.sf2" - giving a hint to the user that data have been compressed with a quality loss.

### Data structure

*Will be written when the requirements are completed.*

## Compatibility v2.xx ⟺ v3.00

### Reading the new format

Changes will be done so that the structure of a soundfont remains the same as much as possible. Software might not be able to fully use the new soundfont capabilities, but they will at least be able to easily read a soundfont v3.00 in a degraded mode.

### Conversions

Soundfont v2.xx can be updated in the new format 3.00 with a loss only in a very specific case when 2 LFOs or 2 envelopes target the pitch at the instrument or preset level.

It will be possible to save a v3.00 soundfont as a v2.xx soundfont with a loss that will not prevent the soundfont to be played.

## What about the sfz format?

The sfz format is already able to feature all the points mentioned above, with the exception of the packaging. So why an update to the soundfont standard would be worth? We could see these two main points:

* writing a clear standard will provide a clear documentation,
* the packaging offered by the sf2 allow soundfonts to be embedded more easily. 

The sfz format will still be ahead in the game regarding the audio synthesis possibilities but the sf2 format would still have a nice future. Its strength is indeed not in the multiplication of features but rather in that it does "just right" the job, in an almost minimalist way.

Also, if the new soundfont specifications meet the core features of the sfz format, we could then see two faces of the same coin!
