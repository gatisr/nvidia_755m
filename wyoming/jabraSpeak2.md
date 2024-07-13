# Jabra speak2 55 - whyoming satellite

## configure jabra (if device doesn't show up after install drivers with that comes wyoming)

1. Check devices

```bash
aplay -l
arecord -l
```

2. Edit alsa devices according to devices printed before, prefer devices with plughw prefix

```bash
sudo mousepad ~/.asoundrc
```

```json
pcm.jabra {
    type hw
    card UC
    device 0
    format S16_LE
    rate 48000
}

pcm.!default {
    type asym
    playback.pcm "plug:jabra"
    capture.pcm "dsnoop:UC,0"
}

pcm.!sysdefault {
    type asym
    playback.pcm "plug:jabra"
    capture.pcm "dsnoop:UC,0"
}
```

3. init alsa devices

```bash
sudo alsactl init
```

4. test play and record

```bash
arecord -f S16_LE -r 48000 -D dsnoop:UC,0 test.wav
aplay -D plughw:UC,0 test.wav
aplay -D plughw:UC,0 test.wav -f S16_LE -r 48000 -D
```

* Restart alsa service - if doesn't see changes

```sh
sudo systemctl unmask alsa-restore.service alsa-state.service
sudo systemctl enable alsa-restore.service alsa-state.service
sudo systemctl restart alsa-restore.service alsa-state.service
```
## Configure wyoming

1. follow to install steps in official repo - https://github.com/rhasspy/wyoming-satellite/blob/master/docs/tutorial_installer.md

2. edit config according your needs

```bash
sudo mousepad ~/projects/wyoming-satellite/local/settings.json
```

```json
{
  "satellite": {
    "name": "rhasspberry5",
    "type": "wake",
    "debug": true,
    "event_service_command": null
  },
  "mic": {
    "device": "plughw:CARD=UC,DEV=0",
    "noise_suppression": 3,
    "auto_gain": 20,
    "volume_multiplier": 1.0,
    "rate": 48000,
    "width": 2,
    "channels": 1,
    "samples_per_chunk": 1024
  },
  "snd": {
    "device": "plughw:CARD=UC,DEV=0",
    "volume_multiplier": 1.0,
    "feedback_sounds": [
		"awake",
		"done",
                "timer-finished"
	]
  },
  "wake": {
    "system": "openWakeWord",
    "openwakeword": {
      "wake_word": "alexa",
	  "threshold": 0.5,
	  "trigger_level": 1,
	  "debug_logging": true
    },
    "porcupine1": {
      "wake_word": "porcupine",
      "sensitivity": 0.5
    },
    "snowboy": {
      "wake_word": "snowboy",
      "sensitivity": 0.5
    }
  }
}
```

3. restart wyoming service to see changes

```bash
sudo systemctl restart wyoming-satellite
```

* Kill pipewire if it show that mic or speaker is already in use (also you can just try to run `aplay` command in terminal to use speaker, somehow, in some cases it helps)

```sh
sudo killall pipewire
```

