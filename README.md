# Raspberry x PanaSonic x Spotify Connect x Radio Pomme d'API

## Goal

Having a Raspberry plugged on a Panasonic SA-PM92 and playing Radio Pomme d'API from Audio In
at startup and Spotify from DAC with Spotify Connect.

Easy to use for kids and parents :) 

Kept as README for the moment, I hope anytime I'll be able to provide something for flexible and automated.

## Overall configuration

Plug it like this:
- Raspberry USB-A to Panasonic DAC USB-B
- Raspberry Audio out to Panasonic Aux In (Unfortunately, only mono for this configuration)
- Raspberry Power to Panasonic USB-A In (starts the Pi where stereo is turned on - avoid having Pi turned on all the time)

Plug the power input of your Raspberry on the Panasonic SA-PM92 USB input and your Raspberry will start when you turn 
on the stereo saving bandwidth and power.

## Raspberry Setup

Tested ont Raspbian / Raspberry Pi OS.

### Install dependencies
    
Install MPC/MPD & Raspotify.

```bash
sudo apt install -y mpc mpd
sudo apt-get -y install curl && curl -sL https://dtcooper.github.io/raspotify/install.sh | sh
```

### Configure MPC/MPD

Configure explicitly line out as output for MPD to have DAC a default output.

Set default volume & autostart
```bash
mpc volume 100
sudo systemctl enable mpd
````

In file `/etc/mpd.conf`:
```bash
audio_output {
	type		"alsa"
	name		"My ALSA Device"
	device		"hw:0,0"	# optional
	mixer_type      "hardware"	# optional
	mixer_device	"default"	# optional
	mixer_control	"PCM"		# optional
	mixer_index	"0"		# optional
}
```

### Start radio on boot

In file `/etc/rc.local`:
```bash
# Wait for network
while ! ping -c 1 -W 1 8.8.8.8; do
    echo "[rc.local] Waiting for network"
    sleep 1
done

# Radio Pomme d'API
echo "[rc.local] Starting Radio Pomme d'API"
mpc clear
mpc repeat on
mpc add https://listen.radioking.com/radio/361753/stream/411358
mpc play

exit 0
```

Make file executable
```bash
sudo chmod +x /etc/rc.local
```

### Configure Alsa for DAC

I do not fully remember how I did it, but it took me a lot of time to make it work.

In file `/etc/asound.conf`:
```bash
pcm.dac {
    type dmix
    ipc_key {
        @func refer
        name defaults.pcm.ipc_key
    }
    ipc_gid {
        @func refer
        name defaults.pcm.ipc_gid
    }
    ipc_perm {
        @func refer
        name defaults.pcm.ipc_perm
    }
    tstamp_type {
        @func refer
        name defaults.pcm.tstamp_type
    }
    slave {
        pcm {
            type hw
            card P2
            device 0
            subdevice 0
        }
        channels 2
        rate 44100
        format S24_LE
        period_size 0
        buffer_size 0
        periods 0
        buffer_time 100000
        period_time 20000
    }
}

pcm.!default {
    type asym
    capture.pcm {
        type plug
        slave.pcm "null"
    }
    playback.pcm {
        type plug
        slave.pcm "dac"
    }
}

ctl.!default {
    type hw
    card P2
}
```

## Configure Raspotify

In file `/etc/raspotify/conf`:
```bash
LIBRESPOT_BITRATE="320"
LIBRESPOT_INITIAL_VOLUME="100"
LIBRESPOT_VOLUME_CTRL="fixed"
LIBRESPOT_FORMAT="S24"
```

## Bonus


