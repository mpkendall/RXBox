## Starting fresh - 1 hour (2026-06-26)

I started this project several months ago. However, I really didn't like the direction the project was going. The initial concept was a Raspberry Pi Zero 2W, RTL-SDR, and some off-the-shelf sensors. However, this was artificially large and bulky, heavy, and not repeatable. It's also _not very cool_.

![old version of rxbox](https://cdn.hackclub.com/019f0650-f48b-78f2-8435-dc73028d62f9/paste-1782517395386.png)
_^ an old prototype!_

I had a distant vision at that time of a fully-integrated PCB, though I've procrastinated on making any real progress until now. I've started by moving all of the old files over the the `old` branch and clearing this one out!

I foresee the hardest part of this project being the SDR topology. While there's several options and implementations, some preliminary research I've done suggests they're all too expensive or not good enough™ (either due to frequency limitations or bad bandwidth).

For example, direct sampling needs crazy ADCs (aiming for decent efficiency at UHF). You could also use a Tayloe mixer, which used a quadrature multiplexer to sample 0, 90, 180, and 270 degrees of the signal (so inituitively, you'd need to run it at 4x the frequency). For now, a Zero-IF configuration looks the most promising. It uses IQ signals (in phase and quadrature [90 degrees offset]), and separates the RF energy into two separate tracks. This way, you can reconstruct both the phase and the magnitude of the signal!

![zero-if flowchart](https://cdn.hackclub.com/019f0949-5d1f-77c2-973d-a074041366b0/paste-1782567229248.png)

I'll probably use an FPGA to interface directly between the ADCs and the on-board processor.

I'm hoping to use an Allwinner chip, most likely the T113-i. From some preliminary research, this does have the capability to directly drive a 565 RGB display! This way I can hopefully save some GPIO while still keeping decent display quality.