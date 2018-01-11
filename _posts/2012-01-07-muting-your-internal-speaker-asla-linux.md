---
title: ALSA(HDA), Linux :Muting your internal speaker
date: 2012-01-07 12:57:58.000000000 +05:30
---
if ALSA sound drivers aren't automatically muting your internal sound when you plug in your headphones or for any other matter you need to do same. if you are in such a situation where you'll need to mute your internal speaker on linux system with alsa sound core, this's how you'll do it.

## Using HDA-Analyzer
* Download the HDA-Analyzer from here <a title="HDA analyzer" href="http://www.alsa-project.org/main/index.php/HDA_Analyzer">http://www.alsa-project.org/main/index.php/HDA_Analyzer</a>
* Run it with root privileges

On the left side select a card, and then a codec, then check through the entries tagged as PIN and try muting under "Output amplifier" section.

NOTE: you don't need to change, close and reopen program for the changes to apply.

## Muting from command line with hda-verb
if you need to do same from command-line then you can download, hda-verb from here [ <a href="ftp://ftp.suse.com/pub/people/tiwai/misc/hda-verb-0.4.tar.gz"><code>ftp://ftp.suse.com/pub/people/tiwai/misc/hda-verb-0.4.tar.gz</code></a>], compile it and use it as follows

### Muting internal speakers

```bash
hda-verb /dev/snd/hwC1D0 0x14 set_pin_wid 0xc0
hda-verb /dev/snd/hwC1D0 0x15 set_pin_wid 0x00
```

### Reverting settings
Just swap the pin numbers in both command lines.
Where `/dev/snd/hwC0D0` is your sound card, you can list sound cards on your system by following

`ls /dev/snd/hw*`

`0x15, 0x14` is you pin number of the internal speakers and headphone jack respectively, replace it with yours.
