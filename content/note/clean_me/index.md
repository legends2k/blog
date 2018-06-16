+++
title = "Shutting the ubiquitous backdoor"
description = "a.k.a Intel ME with a programmer and some guts ;)"
date = "2018-05-12T15:00:46-07:00"
tags = ["tech", "security", "hardware", "tinker"]
+++

I believe [_resistance is futile_][Borg] if someone is bent on spying on you; there’re both serious and [creative ways][Side-channelling] to go about it.  Talking to your processor, behind your back, isn’t the primary or only means.  Ours is [golden age][Digital Age] where corporates and government agencies stoop low enough to pry into bedrooms.  Did you know that _almost_ all machines[^1] you _own_ have a processor aside from the one you paid for?  It controls your processor (host) and won’t listen to you.

- Unbeknownst to the host, it runs out-of-band, on a separate chip
- It has unfettered access to any memory region; it runs at [ring][Protection ring] level −3[^8]
- It can run even when your machine is in stand-by
- It runs an OS and a TCP/IP server on certain ports bypassing any firewall in the host
    - You would never know even if you’re spied on
- Can’t be terminated
    - It controls the hosts boot cycle

A processor inside your processor, having its own operating system[^7], running services, buried very deep within and can’t be switched completely off -- it’s needed to even bring up your processor during boot.  I’m talking about the [Intel ME][] --- a [backdoor][] that’s built into Intel processors; it’s inner workings known only to Intel who refuses divulging anything about it in the name of security[^4].  What’s that?  Yeah, I too went "I’m lucky! I own an AMD machine".  Well, look up [PSP][]; it’s AMD’s ME.  I think it’s [clear by now][NSA] such 

> back doors would exist even on mobile and server processors.  It is just matter of time before they’re exposed.

[^1]: the ones with micro-processor(s)
[^8]: Negative ring levels are below the kernel which is at 0.  Lesser is deeper.
[^7]: [MINIX][] OS, written by [the same guy][Tanenbaum] who [flamed Linus Torvalds][Tanenbaum-Torvalds-debate] for writing a monolithic kernel.  In some sense the microkernel design _won_ since it’s now [the most used OS][ME-Minix] 😉
[^4]: _Security by obscurity_ -- the worst form of security, as opined by security experts.

[Borg]: https://en.wikipedia.org/wiki/Borg_(Star_Trek)
[Side-channelling]: https://en.wikipedia.org/wiki/Side-channel_attack
[Digital Age]: https://en.wikipedia.org/wiki/Information_Age
[Protection ring]: https://en.wikipedia.org/wiki/Protection_ring
[Intel ME]: https://en.wikipedia.org/wiki/Intel_Management_Engine
[Backdoor]: https://en.wikipedia.org/wiki/Backdoor_(computing)
[PSP]: https://en.wikipedia.org/wiki/AMD_Platform_Security_Processor
[NSA]: https://www.bleepingcomputer.com/news/hardware/researchers-find-a-way-to-disable-much-hated-intel-me-component-courtesy-of-the-nsa/
[MINIX]: https://en.wikipedia.org/wiki/MINIX
[Tanenbaum]: https://en.wikipedia.org/wiki/Andrew_S._Tanenbaum
[Tanenbaum-Torvalds-debate]: https://en.wikipedia.org/wiki/Tanenbaum%E2%80%93Torvalds_debate
[ME-Minix]: https://www.networkworld.com/article/3236064/servers/minix-the-most-popular-os-in-the-world-thanks-to-intel.html


# Motive

[Some bright minds][corna] have found a way to neutralize Intel’s backdoor[^5] i.e. not completely kill it, but make it harmless.  Now neutralizing just your MEs isn’t going to stop your other machines from prying.  Security experts and the paranoids will rightfully suggest

> If you really want to be secure, go back to the cave.  Use pen and paper; burn it, when you’re done!

Knowing it’s beyond me, why am I still doing it?  Well definitely not for security[^2], but the hacker in me would like to own his toys -- a reasonable expectation.  A stranger shouldn’t tell you how to use your pen; even worse, use it behind your back routinely.  It’s as simple as that.  It’s probably one of the reasons[^3] I’m a [FOSS][] loyalist.

> I don’t want some processor, I didn’t pay for, to drain my laptop power, or steal data for some merchant who wants to sell stuff that I don’t want.

If this makes sense then you should clean your machine too, but at your own risk.  Make sure you read the [Clean ME guides][], check version and compatibilities and proceed cautiously.  Basically, we’re going to do a BIOS firmware update[^6].  Just that it’s not done internally using software but externally with a cheap [BIOS programmer][].  How come re-programming the BIOS firmware shuts the backdoor?  When you cold boot your processor, [it starts with an amnesia][BIOS boot] and so it always starts with the BIOS boot program.  In this program, there’s a hidden kill-switch for the back door: the [_HAP/AltMeDisable_ bit][HAP].  If set, ME will just boot the host and halt.

Let’s get on with it!

[^5]: Sorry AMD owners; a consoling factor, however, is apparently PSP is much less capable than ME.
[^2]: Heck I wouldn’t be blogging about it if that’s the case, would I now?
[^3]: Another reason is, of course, its [superior engineering][UNIX], better science and execution that stands the test of time, _despite_ unpaid contributors not being paid engineers. Beat that corporates!
[^6]: Just that it’ll be a non-OEM update this time.

[corna]: https://github.com/corna/me_cleaner
[Clean ME Guides]: https://github.com/corna/me_cleaner/wiki
[BIOS programmer]: https://en.wikipedia.org/wiki/Programmer_(hardware)
[BIOS boot]: http://flint.cs.yale.edu/feng/cos/resources/BIOS/
[FOSS]: https://en.wikipedia.org/wiki/Free_and_open-source_software
[HAP]: https://github.com/corna/me_cleaner/wiki/HAP-AltMeDisable-bit
[UNIX]: https://en.wikipedia.org/wiki/Unix_philosophy

# Connect

You should just be able to physically access your motherboard; no soldering involved.

{{< figure src="images/hardware.jpg" title="Tools" alt="USB Programmer CH341A (left), SOP8 clip (bottom), SOP16/8-DIP8 board (top)" caption="USB Programmer CH341A (left), SOP8 clip (bottom), SOP16/8-DIP8 board (top)" >}}

CH341A (Black) is a cheap[^10] USB [SPI chip](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus) programmer.  The SOP8 clip -- that saves you from removing and re-soldering the BIOS chip -- is sold separately.  Make sure you buy the clip that comes with a SOP16/8-DIP8 board; this proxies as the BIOS chip going into the programmer's slot. Take note

1. The first bit is denoted on the BIOS chip-top with a circle marking.
2. Make sure the clip's red line, denoting the first bit, matches with this circle i.e. first on chip is connect to the first on clip when clipping.
3. Connect the wire end of the clip to the board such that the first bits match -- the board has all bit pins marked.
    + Usually a 10-bit bus cable is used as the connector; notice that bits 9 and 10 are cut out.
4. Look at the CH341A programmer board for a drawing about the 25 SPI BIOS chip layout. This should also have the circle marking.
5. Connect the board-end of the clip such that its first and the SPI's first bits match.

{{< figure src="images/setup.jpg" title="My Setup" >}}

Connect the programmer's USB to a Linux machine; I used a Xubuntu bootable USB to convert my old Windows 10 laptop.  You're all set hardware-wise.  Run `lsusb` on the terminal and make sure you see something like

{{< highlight cfg  "hl_lines=1" >}}
BUS 008 Device 012: ID 1a86:5512 QinHeng Electronics CH341 in EPP/MEM/I2C mode, EPP/I2C adapter
{{< /highlight >}}

This means your Linux machine successfully detects the programmer.

[^10]: Decent quality ones costed around $15 together

# Backup

Install `flashrom`; your distro’s package repository will definitely have this nifty tool.  Make sure the chip is recognized by it:

{{< highlight cfg  "hl_lines=1" >}}
$ sudo flashrom --programmer ch341a_spi -r BIOS_org.bin
flashrom v0.9.9-r1954 on Linux 4.15.0-20-generic (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Calibrating delay loop... OK.
Found GigaDevice flash chip "GD25Q64(B)" (8192 kB, SPI) on ch341a_spi.
Reading flash... done.
{{< /highlight >}}

It should be able to auto-detect the chip and give you a full dump of the chip contents.  If it says no devices were found, re-connect the clip; loose-contacts are common with these clips.  Get two dumps and binary-compare them to make sure the connection was fine and the images are valid.

If there're multiple SPI chips on your motherboard, make sure you get the right one i.e. the BIOS image you get from your OEM should have around the same size as the dump you just made.

{{< figure src="images/bios_chip.jpg" title="Eeny, meeny, miny, moe" alt="1 MiB Winbond EEPROM chip (left-bottom) – also an 8-pin SPI chip – was a red herring.  8 MiB GigaDevice GD25Q64(B) BIOS chip (centre) was the one." caption="1 MiB Winbond EEPROM chip (left-bottom) – also an 8-pin SPI chip – was a red herring.  8 MiB GigaDevice GD25Q64(B) BIOS chip (centre) was the one." >}}

Run `me_cleaner` on the dump to verify it:

{{< highlight cfg >}}
$ ./me_cleaner.py -c BIOS_org.bin
{{< /highlight >}}

You should get a valid output.  You might also check it with the `ifdtool` from the [coreboot][] repo: `ifdtool -d BIOS_org.bin`.

[coreboot]: https://www.coreboot.org/

# Clean

Run `me_cleaner` with the soft-clean option.  It's the safest; others wipe out the Intel ME module regions from the firmware; this one simply sets _the_ bit.

{{< highlight cfg "hl_lines=1" >}}
$ ./me_cleaner.py -s -O BIOS_clean.bin BIOS_org.bin
Full image detected
The ME/TXE region goes from 0x1000 to 0x200000
Found FPT header at 0x1010
Found 11 partition(s)
Found FTPR header: FTPR partition spans from 0x1000 to 0xa8000
Found FTPR manifest at 0x1448
ME/TXE firmware version 11.6.10.1196
Public key match: Intel ME, firmware versions 11.x.x.x
The HAP bit is NOT SET
Setting the HAP bit in PCHSTRP0 to disable Intel ME...
Checking the FTPR RSA signature... VALID
Done! Good luck!
{{< /highlight>}}

It should set the _AltMeDisable_ (or HAP) bit and wish you luck!

# Flash

Flash the cleaned image back on to the SPI chip

{{< highlight cfg "hl_lines=1" >}}
$ sudo flashrom --programmer ch341a_spi -w BIOS_clean.bin
flashrom v0.9.9-r1954 on Linux 4.15.0-20-generic (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Calibrating delay loop... OK.
Found GigaDevice flash chip "GD25Q64(B)" (8192 kB, SPI) on ch341a_spi.
Reading old flash chip contents... done.
Erasing and writing flash chip... Erase/write done.
Verifying flash... VERIFIED.
{{< /highlight >}}

It will first take a back-up for disaster-recovery.  It'll also see if it could optimize by not writing portions that don't differ.  Then it'll write and verify if the image and the chip contents match!  Quite a thorough piece of software; nice :)

# Verify

Now for the moment of truth!  Disconnet the clip, restart your machine.  The machine should boot in to the OS normally.  That isn't all: make sure you use it for more than 40 mins and if nothing goes wrong, yay!  You've successfully neutralized the backdoor! 🖖

To verify it properly, get the right version of [Intel ME System Tools][ME-Tools]

{{< highlight cfg "hl_lines=1" >}}
$ MEInfoWin64.exe -FWSTS

Intel(R) MEInfo Version: 11.8.50.3460
Copyright(C) 2005 - 2017, Intel Corporation. All rights reserved.

FW Status Register1: 0x80022004
FW Status Register2: 0x304D0116
FW Status Register3: 0x00000020
FW Status Register4: 0x00086000
FW Status Register5: 0x00000000
FW Status Register6: 0x40000004

  CurrentState:                               Disabled
  ManufacturingMode:                          Disabled
  FlashPartition:                             Valid
  OperationalState:                           Transitioning
  InitComplete:                               Initializing
  BUPLoadState:                               Success
  ErrorCode:                                  Disabled
  ModeOfOperation:                            Alt Disable Mode
  SPI Flash Log:                              Not Present
  FPF HW Source value:                        FPF HW Not Set
  ME FPF Fusing Patch Status:                 ME FPF Fusing patch NOT supported in this FW Version
  Phase:                                      BringUp
  ICC:                                        Valid OEM data, ICC programmed
  ME File System Corrupted:                   No
  PhaseStatus:                                UNKNOWN
  FPF and ME Config Status:                   Match
{{< /highlight >}}

Make sure you uninstall any ME-related software[^12]. If you're on Windows, you can also verify that in the _Device Manager_, under _System devices_, the _Intel(R) Management Engine Interface_ no longer shows up.  On [*nix][Unix-like] the MEI entry should no longer show up for `lspci`.

Finally, be a good team player and [report your success][CleanLog]!

[^12]: There are reports of ME software resetting the bit and resurructing the parasite

[ME-Tools]: https://www.win-raid.com/t596f39-Intel-Management-Engine-Drivers-Firmware-amp-System-Tools.html
[Unix-like]: https://en.wikipedia.org/wiki/Unix-like
[CleanLog]: https://github.com/corna/me_cleaner/issues/3#issuecomment-391503079

# References

1. [Intel x86s hide another CPU that can take over your machine](https://boingboing.net/2016/06/15/intel-x86-processors-ship-with.html)
2. [The Intel Management Engine: an attack on computer users’ freedom](https://www.fsf.org/blogs/sysadmin/the-management-engine-an-attack-on-computer-users-freedom)
3. [Deep dive into Intel Management Engine Disablement](https://puri.sm/posts/deep-dive-into-intel-me-disablement/)
4. [Disabling Intel ME 11 via undocumented mode](http://blog.ptsecurity.com/2017/08/disabling-intel-me.html)
5. [Sakaki’s EFI Install Guide/Disabling the Intel Management Engine](https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Disabling_the_Intel_Management_Engine)
6. [How to become the sole owner of your PC](https://github.com/ptresearch/me-disablement/blob/master/How%20to%20become%20the%20sole%20owner%20of%20your%20PC.pdf)
7. [Intel ME Myths and Reality](https://media.ccc.de/v/34c3-8782-intel_me_myths_and_reality)

# P.S.

Newer versions of ME[^13] are tougher to clean since they've protection mechanisms.  My older laptop having ME 8 was simpler; I'd to jump through a [few hoops][Dell Clean] logistically.

[^13]: This one was CSME 11, if you'd not noticed thus far.

[Dell Clean]: https://github.com/corna/me_cleaner/issues/3#issuecomment-385616918
