---
ID: 36769
post_title: Audio troubleshooting
author: Kathy Reid
post_excerpt: ""
layout: page
permalink: >
  http://mycroft.ai/documentation/troubleshooting/audio-troubleshooting/
published: true
post_date: 2018-03-27 08:51:07
---
# Audio troubleshooting

  * [Troubleshooting Audio](#troubleshooting-audio)
    + [Missing pulseaudio](#missing-pulseaudio)
    + [Microphone can't hear me or CLI show no change in meter while speaking](#mic-cant-hear-me)
    + [Dev instance audio test](#dev-instance-audio-test)
    + [Test your mic](#test-your-mic)
  * [Pulseaudio settings](#pulseaudio-settings)
    + [Show current settings and info](#show-current-settings-and-info)
    + [List available input devices](#list-available-input-devices)
    + [List available output devices](#list-available-output-devices)
    + [Changing pulseaudio input and output](#changing-pulseaudio-input-and-output)
    + [Echo cancellation](#echo-cancellation)
    + [Pulseaudio modules](#pulseaudio-modules)
  * [Other useful commands](#other-useful-commands)
  * [Will my device work](#will-my-device-work)

## Troubleshooting Audio

Mycroft utilizes pulseaudio for sound input and output. Mark 1 and Picroft devices have pulseaudio set up correctly for the mycroft user. On picroft, any mic or speakers you add to it may need to be configured as Mycroft uses the default input and output from pulse. Systems without pulseaudio installed will likely also not function as expected.  Systems with both jack and pulseaudio may need additional configuration to work correctly. 

#### Missing pulseaudio

If you see an issue in the logs with `Popen(play_wav_cmd)` and `OSError: [Errno 2] No such file or directory` this usually indicates that mycroft can't find pulseaudio. Make sure it's installed and mycroft is able to access it.

#### Microphone can't hear me or CLI show no change in meter while speaking

If no audio is picked up by mycroft, check the mic to verify it's working.  If the mictest is successful, verify that pulse has your device as the default source.

##### Dev instance audio test

`start-mycroft.sh audiotest` will attempt to record, then play back a short audio clip using the default source and sink. 

##### Test your speakers

To test a WAV file, you can use: 
`aplay $WAVfile`

For an MP3/OGG format:
`mpg123 $mp3file`

##### Test your mic

The following command will start recording for ten seconds on the default input device when run:
`arecord -d 10 -o test.wav`

You can play it back to hear what is recorded with
`aplay test.wav`

## Pulseaudio settings


##### Show current settings and info

`pactl info`

For mycroft, the lines most relevant are the Default Source: and Default Sink:.

```
  picroft:~$ pactl info
  Server String: unix:/run/user/1000/pulse/native
  Library Protocol Version: 30
  Server Protocol Version: 30
  Is Local: yes
  Client Index: 813
  Tile Size: 65472
  User Name: pi
  Host Name: picroft
  Server Name: pulseaudio
  Server Version: 8.0
  Default Sample Specification: s16le 2ch 44100Hz
  Default Channel Map: front-left,front-right
  Default Sink: alsa_output.pci-0000_00_1b.0.analog-stereo
  Default Source: alsa_input.pci-0000_00_1b.0.analog-stereo
  Cookie: ab8a:0b4d
```

##### List available input devices

Use `pactl list sinks short` to list devices available to the current user:

```
picroft:~$ pactl list sinks short
1       alsa_output.pci-0000_00_1b.0.analog-stereo      module-alsa-card.c      s16le 2ch 44100Hz       SUSPENDED
```



##### List available output devices

`pactl list sources short` will list outputs, or sinks, available to the current user:

```
picroft:~$ pactl list sources short
1       alsa_output.pci-0000_00_1b.0.analog-stereo.monitor      module-alsa-card.c      s16le 2ch 44100Hz       SUSPENDED
2       alsa_input.pci-0000_00_1b.0.analog-stereo       module-alsa-card.c      s16le 2ch 44100Hz       SUSPENDED
```

##### Changing pulseaudio input and output 

If you need to adjust the device you're using for input or output, first determine the number of the source or sink you wish to set it to.  Then use `pactl set-default-source` (for input) or `pactl set-default-sink` (for output) to update:
```
$ pactl set-default-source 3
$ pactl set-default-sink 1
```

This would set the default input to be device 3 and the default output device to be 1.  Your numbers will vary.  A succsessful change will not have any additional output listed.  Verify with `pactl info`

##### Echo cancellation

Pulseaudio has an echo cancellation module that can be loaded.  
```
$ pactl load-module echo-cancellation
```
This is system wide. If not previously enabled, you will need to restart any applications using pulse.
For documentation, see https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#index45h3
Additional usage and tips can be found https://wiki.archlinux.org/index.php/PulseAudio/Troubleshooting

##### Pulseaudio modules

Find out what modules are installed:
```
$ pactl list modules short
```
For more information like usage counts and properties, remove the ```short```.  

### Other useful commands

`lsusb` can be used to see what USB devices are connected to the system.  `lsusb -v` can be also used, but produces significantly more output.

`groups` will list all the system groups the current user is a part of.  In dev instances of mycroft, can be used to see if the mycroft user is part of the audio group.  If running list sinks or list sources you see null or no devices for the user, use this command to verify if you're part of the `audio` group.

`alsamixer` is a command-line utility to adjust volume and muting of both inputs and outputs. 

`pulseaudio -k` kills the current instance of pulseaudio. 

`pulseaudio --start` will start pulseaudio.

`pavucontrol` is a GUI mixer client for X.  

`pacmd` is an interactive shell version of `pactl`. Use `help` to see more once in the shell. 

### Will my device work

Maybe. 

Most USB mics and speakers are usable with pulseaudio. In general, if your OS can recognize the device as an audio endpoint pulse will be able to connect it.  A variety of mics including the PS3 Eye, Blue snowballs, Jabra 410, various web cams, the movo mc1000, gaming headsets, and even the AIY hat have been used successfully. If you have the device on hand, try it out and see. If the information above isn't able to get it going, try asking on the chat server or the forum.  Always check the volume levels if everything else seems to be set correctly.