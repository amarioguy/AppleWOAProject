## Welcome to the M1 Windows project

Keeping things simple, the goal is to hopefully run Windows on Apple's M1 chip (and hopefully future Apple silicon chips too given tweaks to the foundation!).

### Why a whole project for this?

Contrary to how it may appear on the surface, Apple's chips are architecturally very different from standard ARM64 chips from companies like Qualcomm or MediaTek and a lot of hardware enablement needs to be done as a result. Thankfully a lot of work has already been done in this area for Linux through the Asahi Linux project (huge thanks to them for all the hard work they've done and made open source please show them some support!), though plenty of work remains to be done since Windows does some things differently from Linux and there are things the Asahi folks can ignore but I cannot.

(An example: The Asahi Linux folks can't use PSCI, the ARM standard for core bringup, for starting the other ARM cores since they plan to run Linux in EL2 and Apple does not implement EL3. Windows requires PSCI, and this combined with other circumstances means I'll need to get PSCI working on the Apple cores.)


### When will the project be done?

Whenever it's done.

This is not going to be a trivial project to finish, there's a lot of Apple specific oddities I must account for and things I need to do to ensure the M1 is able to boot Windows in a stable way. I'll need time to make sure everything works properly so there's no ETA on anything thus far.


### Disclaimer

This project is not guaranteed to be successful. I'll do my best to ensure it goes to completion but ultimately, there is zero guarantee that I will be able to get Windows working in a great way by the end of it all. But I'm going to try my absolute hardest. 


### What makes Windows on M1 hard?

The short answer: the interrupt controller and the IOMMU.

The detailed answer: Apple implements a non-standard interrupt controller on their SoCs, called the Apple Interrupt Controller (henceforth referred to as AIC). The ARM64 Windows kernel predictably does not implement support for the AIC (the overwhelming majority of ARM64 systems that run Windows use an interrupt controller standard called the Generic Interrupt Controller). Needless to say this is going to be the first major roadblock to getting Windows running since interrupts are fundamental to any multitasking OS.

However, there's a second problem too. While the Apple MMU supports both 16K and 4K pages, the IOMMU only supports 16K pages as far as we know, which presents challenges with hardware communicating with Windows, and this is something I also need to have working.

There is of course the consideration of essentially all the Apple specific hardware however most of this can be solved with drivers or with ACPI.

### Current game plan to deal with the interrupt controller issue

There's an somewhat obscure feature on M1 (and M1 Pro/Max/Ultra, henceforth referred to as M1 v2) chips where part of the GICv3 can be virtualized to guest OSes to enable faster interrupt handling. This does imply the use of a lightweight hypervisor in EL2 to handle the physical interrupts and routing them to Windows which is running in EL1, which does mean we need to give up Hyper-V support since neither M1 nor M1 v2 support nested virtualization. However this is not a huge loss in the grand scheme of things since the main point of the project is to get Windows running as close to the bare metal as feasible.

Thankfully, there's already a very lightweight, open-source hypervisor for Apple silicon platforms that works great. Meet m1n1, the bootloader the Asahi folks are using to bootstrap UBoot on their implementation. The bootloader also serves as a very lightweight hypervisor that I can tweak such that it can be used for launching our UEFI firmware in EL1 and getting Windows running. (Right now work is being done in a custom fork of m1n1, I will push these changes to mainline m1n1 once I've verified that UEFI boots successfully)

However, that's not the full story. This vGIC implementation has two oddities about it. The first is that it doesn't support hardware interrupt deactivation and it doesn't support the maintenance interrupt. This is simple enough to work around, since KVM encountered the same problem when using the vGIC in Linux it shouldn't be too hard to use their fix or a variation of their fix to work around this issue. The second is more pertinent, and that is when a guest OS writes invalid data to the vGIC registers, occasionally it'll fire an SError exception on the host which needless to say is *very bad*. Thankfully, since m1n1 is open source, I should be able to update the SError exception handler to handle this hardware bug. There is also some things to emulate but there won't be nearly as much performance penalty as if I had to emulate it fully.

Once this is complete, UEFI and Windows should be able to notice the vGIC and use it as if it were a physical GIC and they should see little issue with it (though they will probably see issue with the myriad of Apple specific hardware)

### Why not HAL Extensions to provide native AIC support?

Simply put, HAL Extensions do not provide enough functionality to make this possible. They're too limited in what they can do and in what they're allowed to use. If a future Windows version allows foreign interrupt controllers through HAL Extensions, I will revisit using HAL Extensions.

### Progress (current as of 6/4/2022)

I'm working on getting m1n1 and it's hypervisor updated to handle booting UEFI and Windows better, including setting up the vGIC in EL1. Concurrently also working on getting Project Mu compiled for M1 platforms.

### Tentative checklist of things still to be done/drivers to be made

- hypervisor/GIC/UEFI bringup
- Windows early boot
- ANS2
- IOMMUs
- power management
- display output (relevant to desktops only)
- Thunderbolt
- Ethernet
- Wireless
- GPU/DCP

### Credits

The Asahi Linux project and it's developers for paving the way for custom OSes on M1 Macs, for m1n1, for hardware, and for general help on certain aspects of the M1 architecture

Microsoft - Project Mu

All the people who helped me with figuring out how to start
