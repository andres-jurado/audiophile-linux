# audiophile-linux
This repository contains helpful information for audio enthusiasts who use Linux

# Hardware
My setup:
Ubuntu 18.04 64-bit > USB > Topping D10 > JDS Labs Atom (low gain) > Beyerdynamic DT-770 Pro 250 Ohm

# Setup
I listen to Tidal lossless through pulseaudio via Google Chrome (BTW, Tidal no longer requires Flash) and use pulseeffects to equalize my headphones. On occasion, I play bitperfect sound through Quodlibet.

# Pulseaudio
Here is the `git diff` of the changes I made from the default configuration of /etc/pulse/dameon.conf

```
diff --git a/daemon.conf.bak b/daemon.conf
index f66f7fe..6d7d7a6 100644
--- a/daemon.conf.bak
+++ b/daemon.conf
@@ -17,7 +17,7 @@
 ## more information. Default values are commented out.  Use either ; or # for
 ## commenting.
 
-; daemonize = no
+ daemonize = no
 ; fail = yes
 ; allow-module-loading = yes
 ; allow-exit = yes
@@ -30,11 +30,11 @@
 ; lock-memory = no
 ; cpu-limit = no
 
-; high-priority = yes
-; nice-level = -11
+ high-priority = yes
+ nice-level = -15
 
-; realtime-scheduling = yes
-; realtime-priority = 5
+ realtime-scheduling = yes
+ realtime-priority = 9
 
 ; exit-idle-time = 20
 ; scache-idle-time = 20
@@ -50,11 +50,11 @@
 ; log-time = no
 ; log-backtrace = 0
 
-; resample-method = speex-float-1
-; avoid-resampling = false
+ resample-method = speex-float-10
+ avoid-resampling = yes
 ; enable-remixing = yes
 ; remixing-use-all-sink-channels = yes
-; enable-lfe-remixing = no
+ enable-lfe-remixing = no
 ; lfe-crossover-freq = 0
 
 flat-volumes = no
@@ -72,17 +72,17 @@ flat-volumes = no
 ; rlimit-sigpending = -1
 ; rlimit-msgqueue = -1
 ; rlimit-nice = 31
-; rlimit-rtprio = 9
+ rlimit-rtprio = 9
 ; rlimit-rttime = 200000
 
-; default-sample-format = s16le
-; default-sample-rate = 44100
-; alternate-sample-rate = 48000
-; default-sample-channels = 2
-; default-channel-map = front-left,front-right
+ default-sample-format = float32le
+ default-sample-rate = 44100
+ alternate-sample-rate = 96000
+ default-sample-channels = 2
+ default-channel-map = front-left,front-right
 
-; default-fragments = 4
-; default-fragment-size-msec = 25
+ default-fragments = 2
+ default-fragment-size-msec = 125
 
 ; enable-deferred-volume = yes
 deferred-volume-safety-margin-usec = 1
```

I drew from these sources:

1. https://medium.com/@gamunu/enable-high-quality-audio-on-linux-6f16f3fe7e1f
2. https://www.systutorials.com/docs/linux/man/1-pulseaudio/#lbAI
3. https://wiki.archlinux.org/index.php/PulseAudio/Troubleshooting
4. https://forums.linuxmint.com/viewtopic.php?t=253225

# ALSA
I created the file ~.asoundrc/asound.conf with the following contents

```
# Use PulseAudio plugin hw
pcm.!default {
   type plug
   slave.pcm hw
}
```
Source: https://medium.com/@gamunu/enable-high-quality-audio-on-linux-6f16f3fe7e1f

# Pulseeffects
I installed the Flatpak version of pulseeffects. On the global menu I set:
```
Priority Type = Real Time
Priority = 9
```

# QuodLibet

I like this application because it's actively under development, allows easily to play bitperfect and runs natively on Wayland. On Ubuntu you can install it through APT and flatpak. The flatpak version cannot run bitperfect audio at this moment due to flatpak's security model. Thus, I use the APT executable.

The settings I use (which you will have to modify for your use case), File > Preferences > Output pipeline:

```
alsasink device=hw:Uber,0
```

You can figure out the name of your sound card with
```
aplay -l
```
