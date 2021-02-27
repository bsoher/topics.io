---
sort: 9
---

# Scion Server Specifications

## About Scion's Hardware
Here's some random notes about the hardware in scion.

Aside from yanking open the physical box, the best way to get info about
what's in scion is from the command *`dmidecode`*. Note that you can
only run that as root.

Thanks to `dmidecode` I learned that scion's CPU is an 
[2.27GHz Intel Xeon E5520](http://ark.intel.com/products/40200/Intel-Xeon-Processor-E5520-%288M-Cache-2_26-GHz-5_86-GTs-Intel-QPI%29).
That's a quad-core chip.

The motherboard is a 
[Supermicro X8ST3](http://www.supermicro.com/products/motherboard/Xeon3000/X58/X8ST3-F.cfm). 
Specifically, I'm pretty sure it is an X8ST3-F although `dmidecode` 
doesn't say.

That motherboard has the following RAM specs --

 * Six DIMM slots
 * Accepts 1333Mhz DDR3 DIMMS
 * Max 24G of RAM
 * RAM can be ECC or non-ECC
 * RAM must be _unbuffered_ (the opposite of _registered_)
 
As of this writing three of the six DIMM slots are filled with ECC RAM for 
3x2Gb = 6Gb.

When buying RAM, ECC versus non-ECC and unbuffered versus registered matters.
ECC RAM is best for servers. Server memory is often ECC & registered but this
motherboard can't handle that combination. If you need to replace the RAM,
get ECC unbuffered memory.

