# PROPOSAL FOR A SOUNDFONT v3.00 STANDARD

## Disclaimer

This work is a proposal for updating the soundfont standard and is by no mean an official document on which you should base a development, at least as long as it's not adopted by major actors working with soundfonts if it's possible to list them (at least FluidSynth / FluidLight for the open-source community).


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

### Musical capabilities

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Round-robin sample playback**|At the instrument level, one division is linked to only one sample.|At the instrument level, one division can be linked to an undefined number of samples, played sequentially. For example, using one key triggers a first sample and reusing the same key triggers a second sample. In such a division, comprising several samples, all parameters are duplicated except two: key and velocity ranges.<br><br>The same can be done at the preset level: one division can be linked to several instruments, played sequentially.|
|**Multiple loops in samples**|At the sample level, it is possible to specify only one loop.|At the sample level, an unlimited number of loops can be specified. If several loops are correctly intricate, they will be played successively during playback.|
|**Filters**|At the instrument and preset level, one low-pass filter of the 2nd order (12 dB / octave) is automatically assigned and it is possible to configure it.|At the instrument or preset level, no low-pass filter are automatically assigned. It is then possible to add up to height configurable filters:<br><br><ul><li>low-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>high-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>band-pass filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave);</li><li>notch filter, 1st order (6 dB / octave) or 2nd order (12 dB / octave).</li></ul><br>*Note: the purpose of specifying a variable at the preset level is to modify the corresponding variable for the children at the instrument level. So if a high-pass filter is defined at the preset level and a low-pass filter is defined at the instrument level, the configuration of the high-pass filter will be ignored since there is no corresponding high-pass filter at the instrument level.*|
|**Playback modes**|At the instrument level, it is possible to choose between three modes for playing a sample:<br><br><ul><li>"no loop"<br>the sound is triggered when a key is triggered, no loop are played;</li><li>"loop"<br>the sound is triggered when a key is triggered, a loop is used indefinitely if defined at the sample level;</li><li>"loop + end"<br>the sound is triggered when a key is triggered, a loop is used as long as the key is help, then the sound is played to the end when the key is released.</li></ul>|The attack of a sound is important but so is the release. Thus, the following mode is added:<br><br><ul><li>"release"<br>the sound is triggered when a key is released.</li></ul><br>*Note: in this mode, the delay / attack / decay / release of the volume envelop will be used to compute the initial velocity of the sample.*|
|**Independent envelopes**|At the instrument or preset level, it's possible to use 2 envelopes:<br><br><ul><li>one for the volume (vol env);</li><li>one for the pitch and / or filter (mod env).</li></ul>|At the instrument or preset level, it's possible to use 3 independent envelopes:<br><br><ul><li>one for the volume (vol env);</li><li>one for the pitch (pitch env);</li><li>one for the filter (filter env).</li></ul><br>*Note: it will be possible to specify only one envelope for the pitch instead of 2. This could be seen as a regression but also as a clarification.*|
|**Independent LFOs**|Similarly to the envelopes, it's possible at the instrument or preset level to use 2 LFOs:<br><br><ul><li>one for the volume and / or pitch and / or volume (Mod LFO);</li><li>one for the pitch (Vib LFO).</li></ul><br>|At the instrument or preset level, it's possible to use 3 independent LFOs:<br><br><ul><li>one for the volume (vol mod);</li><li>one for the pitch (pitch mod);</li><li>one for the filter (filter mod).</li></ul><br>*Note: it will be possible to specify only one LFO for the pitch instead of 2. This could be seen as a regression but also as a clarification.*|
|**Default "velocity -> filter cutoff" modulator**|Three versions of the default modulator "velocity -> filter cutoff" have been used.<br><br>In v2.01, it included a switch (as second source) triggered between velocities 63 and 64, resulting in an ugly jump.<br><br>Creative Labs realized the error and in v2.04, the switch as been removed.<br><br>In the meantime, developers completely removed the default filter "velocity -> filter cutoff".|Default filter "velocity -> filter cutoff" is removed.|


### Packaging

|Before (v2.xx)|After (v3.00)|
|--|--|
|Soundfonts are all packed in the .sf2 format, containing raw sample data.|Three packaging will be possible, depending on the use case.<br><br>**Classical "sf2"**<br><br>All parameters and samples are contained in one sf2 file, the samples being either saved in a raw format or with the lossless compression FLAC. A sf2 file can then be reused and edited with the same quality. FLAC compression could become automatic.<br><br>**Compressed soundfont**<br><br>All parameters and samples are contained in one sf3 file, the samples being saved with the lossy compression OGG. A sf3 file can be then easily embedded but is not intended to be edited because of the lower quality.<br><br>**Multiple files**<br><br>After having improved the soundfont capabilities, saving the soundfont as an sfz can become straightforward. All parameters will be in a sfz file and all samples will be saved as .wav / .flac or .ogg files.<br><br>Note: saving a soundfont v3.00 as an sfz file can be viewed as an independent step (an export step) with no impact on the soundfont v3.00 standard. This is something than can however be kept in mind to bring communities together.|


### Miscellaneous

|  |Before (v2.xx)|After (v3.00)|
|--|--|--|
|**Metadata**|Soundfont metadata is made of:<br><br><ul><li>a soundfont title, with a maximum length of 256;</li><li>a date, with a maximum length of 32;</li><li>an author, with a maximum length of 256;</li><li>a copyright, with a maximum length of 256;</li><li>a description, with a maximum length of 65536;</li><li>a soundfont version;</li><li>a field for specifying a wave table sound engine, with a maximum length of 256;</li><li>a field for specifying a wave table ROM, with a maximum length of 256;</li><li>a field for specifying a product, with a maximum length of 8;</li><li>a field for specifying soundfont compatible tools, with a maximum length of 256.</li></ul>|Soundfont metadata is made of:<br><br><ul><li>a soundfont title;</li><li>a date written as a UTC timestamp so that it can be understood by everybody;</li><li>an author;</li></li>a copyright;</li><li>a description;</li><li>a soundfont version;</li><li>an unlimited number of possible additional fields.</li></ul><br>Length limits are all removed.|
|**Element names**|The maximum length of a sample, instrument or preset name is 20.|The length of a sample, instrument or preset name is not limited.|
|**Number of generators / modulators**|The number of generators and modulators is limited to 65536.|The number of generators and modulators is not limited.|
|**dB units**|Due to a historical code mistake, dB stored in a soundfont or not reel dB but 0.4 dB.|dB stored in a soundfont are real dB.|
|**Samples**|Raw data, either 16 or 24 bit, is used.|Raw data (.wav), FLAC or OGG can be used. The precision in bits is now sample-dependent, not fixed for the entire soundfont.|
|**Sample description**|No description can be added to a sample.|A description can be added to a sample. This could be useful for instance to specify a license associated to a sample, or describe the way a sample has been generated.|


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
