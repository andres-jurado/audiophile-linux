# Tanu Kaskinen

This is Tanu Kaskinen's feedback on the options that we were using (reproduced with his permission)

```
Here are my comments:

daemonize = no

-- That's the default value anyway, and it's never necessary to change it in daemon.conf - the best practice is to control this option on the command line when starting pulseaudio. This has no relevance to quality or performance.

high-priority = yes
nice-level = -15
realtime-scheduling = yes
realtime-priority = 9
rlimit-rtprio = 9

-- Tweaking the priorities seems like possible snake oil, but I don't expect these changes to cause problems either. If there is evidence that the default priorities aren't good, then I would recommend submitting a patch to improve the defaults.

-- Regarding the realtime priority: the jack modules will actually use a priority of "realtime-priority" + 4 for the jack client thread, which is why the default rlimit-rtprio is set to 9. The logic here is that we assume that the default priority of jackd is 10, and we want a priority less than that, but more than other realtime threads in PulseAudio. Now if you increase "realtime-priority" to 9, other realtime threads will have priority 9 and the jack client thread will try to get priority 14, but it will be limited to 9, because the "rlimit-rtprio" option sets the upper limit to 9. The overall result is that the priorities won't have the relationships that they were designed to have, and they could behave worse than the default, but I don't expect this to have significant effect. By the way, just in case you didn't know it, the absolute numbers don't matter, the only thing that matters is what priorities threads have in relation to each other. If there are two threads, it doesn't matter if one has priority 10 or 20, if the other thread has priority 21.

resample-method = speex-float-10

-- Very likely overkill. Alexander Patrakov has studied the resamplers, and according to him humans can't detect distortions in speex-float-5 and higher. A quick search found this email related to this: pulseaudio-discuss.freedesktop.narkive.com/KVOBRkZO/patch-2-4-enabled-libsoxr-resampler-backend#post5

-- In my experience audiophiles often ignore "humans can't detect" arguments (maybe you're one of them, I hope I'm not offending you here), so they will always opt for "the best". So maybe this makes sense (if you don't mind the increased CPU use), at least you won't have to argue with people demanding "the best".

avoid-resampling = yes

-- This makes sense.

default-sample-format = float32le
default-sample-rate = 44100
alternate-sample-rate = 96000

-- I think there should be a comment to adapt these to the user's particular hardware.

default-sample-channels = 2
default-channel-map = front-left,front-right

-- I don't see much use for these settings (doubly so since they are the same as the default). They aren't really relevant to quality.

# Use PulseAudio plugin hw
pcm.!default {
   type plug
   slave.pcm hw
}

-- This is strongly recommended against if you're going to use pulseaudio! This makes alsa applications access the hardware directly by default, causing conflicts with pulseaudio. The comment is inconsistent with the actual configuration - pulseaudio won't be used with this configuration. A good distribution will install alsa configuration together with pulseaudio that will make pulseaudio the default alsa device, but if this has to be manually configured, then the correct contents of the file are:

pcm.!default {
    type pulse
}
```

```
One more thing: I said "If there is evidence that the default priorities aren't good, then I would recommend submitting a patch to improve the defaults." - I probably shouldn't have said that. If you manage to prove that changing the nice-level setting improves something for you, that probably won't be enough evidence to change the upstream default. I recall someone saying at some point that setting high-priority to no fixed glitches for them... The default nice value will probably stay the same forever, because changing it can have varying results.
```

```
Oh, I forgot one thing about the nice-level: changing it from -11 to -15 does absolutely nothing, if you don't also increase rlimit-nice too. The relationship between nice-level and rlimit-nice is a bit weird: the lowest allowed nice-level value is calculated as 20 minus "rlimit-nice", so with the default rlimit-nice value of 31 the lowest nice value is -11. But even changing rlimit-nice in daemon.conf may not help, if you haven't given your user permission to use lower nice values. I don't actually know where the lowest possible nice value is configured. I know it can be configured in /etc/security/limits.conf, but on my machine PulseAudio is running with nice value of -11 even though my user is not allowed negative nice values in limits.conf. I'm pretty sure rtkit is setting the nice value at PulseAudio's request, but I don't know where rtkit gets its configuration.
```

```
I looked a bit more into this... Setting any rlimits in daemon.conf is useless in a normal setup, unless you're running in the system mode or you have configured special permissions for your user in /etc/security/limits.conf. The normal setup is such that it's solely rtkit that decides whether to allow a particular realtime priority or nice value. The rlimits in daemon.conf don't affect rtkit in any way.
```



