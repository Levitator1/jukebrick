# jukebrick
$40 Stereo Wifi Speaker appliance

This is a recipe for a wifi speaker appliance you can use to play music from your computer to some remote pair of speakers on the other side of the house.
I will be using it to listen to streaming Web music when I do chores in the kitchen. Requisite scripts and configuration files are included, and there's
not a lot to it. You will need a Linux desktop to host the streaming client, or some other way of capturing your streaming service's audio output
and retransmitting it on your LAN with the RTP protocol.

Prerequisites:
- A Wifi router. I used the classic Raspberry Pi Zero and I found it necessary to downgrade to 802.11n, but that works fine for me
- A Linux Desktop with Pulseaudio or some other system that will stream Internet music and output it via RTP protocol
- The Linux Desktop or other RTP source must be able to connect to the router and transmit to wireless clients, whether by cable or wireless
- A USB hub for connecting a temporary console to the Pi, for setup

BOM:
- One Raspberry Pi Zero W ($10)
- One >= 8GB SD Card ($7)
- Raspberry Pi Zero USB power supply ($8)
- Raspberry Pi Zero case ($6-$15)
- One "Sabrent" MMSA USB audio adapter ($8)
- Your choice of stereo speakers or even headphones with phono plug for plugging into the Sabrent
- (Cost $39-48 plus speakers)

# Client device configuration
The Raspberry Pi Zero does have built-in sound hardware, but its output is combined with the HDMI output. Since a splitter dongle
costs about the same or more than a Sabrent audio adapter, it's found better to add an audio device than to occupy the HDMI port.
Linux kernel module support is standard for the Sabrent device and it should be plug-and-play under Raspbian.

- Image the SD card with a copy of the Raspbian distribution. See raspbian.org for instructions.
- Install the card in the Pi
- Plug a mouse, keyboard, and Sabrent audio into the Pi using a USB hub
- Plug a monitor into the Pi via HDMI
- Apply power to the Pi and boot it up
- `passwd` to change the default password for 'pi', the default user (default password is 'raspberry')
- Run `sudo raspi-config`
- Configure the Pi...
- Set the hostname
- Set the locale and language. It will be British by default.
- Set the Wifi SSID and passphrase
- Save settings and reboot
- `sudo apt-get update`
- `sudo apt-get upgrade`
- `sudo apt-get install pulseaudio`
- `sudo update-rc.d pulseaudio-enable-autospawn remove`
- Copy the included pulseaudio-system script to /etc/init.d
- Set up the init script to manage the global/system instance of pulseaudio
- - `cd /etc/rc5.d`
- - `ln -s ../init.d/pulseaudio-system S99pulseaudio-system`
- Copy the file `system.pa` to `/etc/pulse/system.pa`
- You may need to edit the name of the ALSA device in the above file to reflect how it is detected on your system. It is long mess like: alsa_output.usb-C-Media_Electronics.... etc
- You can find out by what name the Sabrent device was detected with the command `sudu -u pulse pactl list sinks`. Look at the "Name:" field.
- Reboot the Pi
- It should now be ready to play RTP streams off the LAN. You can check /var/log/syslog for error reports from pulseaudio
- You should be able to test to verify that the Sabrent device works by doing `sudo -u pulse paplay somesound.wav`. You will need to download some random wav file to test with.
- At whatever point you decide that the Pi is properly configured, you can disconnect the USB hub and boot it headless with only the power supply, and the Sabrent device connected


# Host device configuration

Assuming you are using a Linux desktop with the very commonly used Pulseaudio sound server, this is really easy.

- `paprefs`
- Under the `Multicast/RTP` tab, select `Enable Multicast/RTP sender" and `Create separate audio device for Multicast/RTP`
- Close paprefs
- Launch your favorite audio application of any kind, a music player, or a Web music streaming app
- Run `pavucontrol`, which is commonly accessible from the system tray as the audio control panel
- Go to the `Playback` tab, find the entry for your music application, and assign it RTP as its output device

You should get sound out of your Raspberry Pi Zero, way on the other side of your residence. I configured a generous three second buffer, which should be ample to discourage
dropouts and gaps in the audio unless your wireless connection is exceedingly weak.

# Troubleshooting
- You should be able to SSH to the Pi for command-line access: `ssh pi@jukebrick`
- To see whether the RTP packets are arriving as they should do
- - `sudo apt-get install tcpdump`
- - `sudo tcpdump udp`
- You should see a continuous flow of UDP packets from the host machine. They were mostly the same size, slightly over a kilobyte on my setup.
- Again, you can watch /var/log/syslog for messages from pulseaudio. It is necessary for the Sabrent to be initialized at the ALSA level before pulseaudio starts.


If pulseaudio starts too early it won't see the Sabrent and you will get an error message about an ALSA device module failing to load. I hardcoded a ten second delay
into the init.d script because I'm super lazy, and you can adjust this to a longer delay if needed. You can also do `sudo service pulseaudio-system restart` to force a restart of pulseaudio once you are sure the ALSA device is initialized.

That's it.

Jose Batista
K3YAB
