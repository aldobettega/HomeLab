# Finding hardware

To start a homelab on a budget, you have to search for the right hardware on the right sites for the right prices. I didn't have much experience in the hardware market, and in 2026 the prices are high, so you have to be thoughtful about what you purchase.

Here are a few tips I can share after my hardware research:

- An initial setup consists of about one mini PC, one switch, and some patch cables to connect the mini PC and additional devices to the switch.
- The mini PC is the brain of the server. You have to search for at least 16GB of DDR4 RAM and a CPU with at least 6 cores and 6 threads since you have to virtualize several services. I chose an i5-8500T (the 'T' indicates that the processor has a reduced Thermal Design Power (TDP) of 35 watts, an essential spec for a machine that has to stay up 24/7).
- Buy used; give a shot to private sellers on eBay. I found an Acer Veriton N4660G for 160€, a very good price for this PC.
- Search by CPU (for ex. i5-8500T) instead of by the PC model (Acer Tiny), so you can skim a lot faster through the substantial specs of your PC.
- For the switch, I chose a TL-SG608E 8-port (30€ new on Amazon); the E means that it is managed. I found out later that some devices like access points need PoE (Power over Ethernet), so I had to buy a PoE injector because the model of this switch doesn't have this feature. Consider buying one with both managed and PoE features, or plan to buy a PoE injector (8-20€).

So I started with:

- Acer Veriton N4660G (160€ used):
    - 16GB RAM
    - 512GB SSD
    - Intel Core i5-8500T
- TP-Link Managed Switch TL-SG608E (30€ new)
- 5 patch cables (0.25m) CAT6 (10€ new)

# Testing hardware

It is recommended to test your used hardware to know what you have actually bought. The Acer had Windows 10 installed, so I used this OS to run the checkups.

- RAM: mdsched.exe, a Windows program to detect the status of the RAM.
- SSD: CrystalDiskInfo, a program you have to download and install from a browser.
- CPU: Prime95 with CPUID HWMonitor. You run Prime95 to stress all the cores and monitor the test with HWMonitor; if the temperatures go over 80°C, consider changing the thermal paste.

It is also recommended to open the case and clean the dust.

# Troubleshooting

## BIOS problems

After I opened the case, the PC could power on but couldn't boot. I'm not a HW expert, but I did some checks, and after those, the PC magically booted:

- Switch the RAM or try booting with one RAM stick at a time.
- Change the BIOS battery (maybe it was dead).
- Reset the BIOS with the power button.
- Check that if you changed the thermal paste, it didn't spread too much; in that case, clean it.
- Pray.

After those checks and prayers, the computer booted with a warning:
 `THE CHASSIS WAS OPENED`
So maybe this was one of the problems. This is a security message that blocks the boot of the BIOS. You have to disable it from the BIOS settings, so every time the system boots you don't have trouble.