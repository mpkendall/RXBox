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

## Power on the T113-i - 5 hours (2026-06-29)

As I mentioned before, I'll be using the Allwinner T113-i microprocessor. This chip is fairly ubiquitous and cheap (so all the cost can go to the RAM :yay:).

I spent most of this session working on providing the chip with adequate power rails. There are 6 different levels:
- VDD_CPU - powers the primary CPU core at 0.9V nominal. I'm actually going to be using a nifty DVFS implementation by injecting the feedback pin of the regulator using the VDD-CPUFB pin (PWM and goes through a low-pass filter to convert to analog). This way, the regulator can adjust the output voltage on command proportionally to the CPU load! More on this later.
- VDD_SYS - Power supply for system. 0.9V nominal
- VCC_DRAM - As you can probably guess, this powers the DRAM core. For a standard DDR3 implementation, this should sit at around 1.5V.
- VCC_RTC and VCC_LVDS - Power supply for internal RTC and LVDS core. These can be combined on a single 1.8V output.
- AVCC - Separate analog power supply. Will likely use a separate LDO for isolation.
- pretty much everything else is 3.3V (including VCC_IO, VCC for the GPIO pins, and some other peripherals).

One important caveat is that the start-up rate of each voltage rail needs to be controlled and timed. While this could be done using just RC filters on the EN pins, like the reference design shows, what happens if it experiences a voltage glitch or rapid reset? now they won't reset correctly. For this reason, I'm planning to implement some sort of power sequencing, using either a dedicated IC or implementing one on an MCU or FPGA (I'll need an FPGA for the SDR anyways, but I'd still like to keep these systems separate if possible).

![allwinner power sequencing docs](https://cdn.hackclub.com/019f0b6d-e290-71b9-97c6-fff889e2cad6/paste-1782603176823.png)

Here's what my first implementation of the power tree looks like:

![6 regulators with sequencing](https://cdn.hackclub.com/019f14dc-b99b-7655-a64d-fb5468d97619/paste-1782761436289.png)

The DVFS implementation on the VDD_CPU line is controlled by the CPU_PWM net. This is, well, a PWM signal modulated by the CPU to scale the output voltage on demand. Instead of using a dedicated regulator, to minimize the BOM, I'm just going to add a few extra passives. First, the PWM signal is converted into an analog average by the low-pass filter. When the PWM is high, it'll use the existing divider, which calculates to around 1.11V. The 130kR resistor sets the gain and how much the PWM alters the resistance seen at that point. This yields around 0.83V at the lowest setting.