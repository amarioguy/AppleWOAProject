## Welcome to the M1 Windows project

Keeping things simple, the goal is to hopefully run Windows on Apple's M1 chip (and hopefully future Apple silicon chips too given tweaks to the foundation!).

### Why a whole project for this?

Contrary to how it may appear on the surface, Apple's chips are architecturally very different from standard ARM64 chips from companies like Qualcomm or MediaTek and a lot of hardware enablement needs to be done as a result. Thankfully a lot of work has already been done in this area for Linux through the Asahi Linux project (huge thanks to them for all the hard work they've done and made open source please show them some support!), though plenty of work remains to be done since Windows does some things differently from Linux and there are things the Asahi folks can ignore but I cannot.

(An example: The Asahi Linux folks can't use PSCI, the ARM standard for core bringup, for starting the other ARM cores since they plan to run Linux in EL2 and Apple does not implement EL3. Windows requires PSCI, and this combined with other circumstances means I'll need to get PSCI working on the Apple cores.)


### What makes Windows on M1 hard?

The short answer: the interrupt controller and the IOMMU.

The detailed answer: Apple implements a non-standard interrupt controller on their SoCs, called the Apple Interrupt Controller (henceforth referred to as AIC). The ARM64 Windows kernel predictably does not implement support for the AIC (the overwhelming majority of ARM64 systems that run Windows use an interrupt controller standard called the Generic Interrupt Controller). Needless to say this is going to be the first major roadblock to getting Windows running since interrupts are fairly fundamental. RIght now there are three solutions I'm considering to solve the AIC problem (given the user controllable bootchain, all are plausible.)

However, there's a second problem too. While the Apple MMU supports both 16K and 4K pages, the IOMMU only supports 16K pages as far as we know, which presents challenges with hardware communicating with Windows, and this is something I also need to have working.

There is of course the consideration of essentially all the Apple specific hardware however most of this can be solved with drivers or with ACPI.

### Possible solutions to AIC issue

Question: Does Windows have a way of supporting non-standard interrupt controllers? If so can this be extended?

### Current potential solutions

1) Emulating a GIC fully in EL2. This would be the most obvious solution, however due to interrupt handling having to go through hypervisor code, there will be a performance penalty, the extent of which is not fully understood yet.

2) Writing a HAL extension. This would arguably be the best solution since if the HAL extension allows us to add native AIC support along with handling the 16k IOMMU pages, the only issue left would be the drivers and application compatibility testing. However, I'm still not fully sure if this is a viable path since the extent of how much HAL Extensions permit non-standard hardware is something I still need to answer.

3) Patching the Windows kernel directly to enable AIC support. The messiest solution, however it should be mentioned regardless. This solution would break Windows update so I'd prefer to avoid this if at all possible.

### Progress

Initial start has been fairly slow however I can confirm that the UEFI for the M1 will be using Project Mu (a Microsoft spinoff of EDK2) as the basis. The next step is getting Project Mu running on the M1

### Credits

The Asahi Linux project and it's developers for paving the way for custom OSes on M1 Macs, for m1n1, for hardware, and for general help on certain aspects of the M1 architecture

Microsoft - Project Mu

All the people who helped me with figuring out how to start
