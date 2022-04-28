DPVS 8 "Jun 2016" Linux "Linux Administrator's Guide"
==========================================

NAME
----

dpvs - Data Plan Virtual Server

SYNOPSIS
----

`dpvs` &lt;EAL PARAMS&gt -- &lt;APP PARAMS&gt;

DESCRIPTION
----

`dpvs(8)` is virtual server base on dpdk


COMMANDS
----
####EAL common options:

`-c` COREMASK
  Hexadecimal bitmask of cores to run on

`-l` CORELIST
  List of cores to run on The argument format is &lt;c1&gt;[-c2][,c3[-c4],...] where c1, c2, etc are core indexes between 0 and 128

`--lcores` COREMAP
  Map lcore set to physical cpu set The argument format is '&lt;lcores[@cpus]&gt;[&lt;,lcores[@cpus]&gt;...]' lcores and cpus list are grouped by '(' and ')' Within the group, '-' is used for range separator, ',' is used for single number separator.  '( )' can be omitted for single element group, '@' can be omitted if cpus and lcores have the same value

`--master-lcore` ID
  Core ID that is used as master

`-n CHANNELS`
  Number of memory channels

`-m MB`
  Memory to allocate (see also --socket-mem)

`-r` RANKS
  Force number of memory ranks (don't detect)

`-b, --pci-blacklist`
  Add a PCI device in black list.  Prevent EAL from using this PCI device. The argument format is <domain:bus:devid.func>.

`-w, --pci-whitelist`
  Add a PCI device in white list.  Only use the specified PCI devices. The argument format is <[domain:]bus:devid.func>. This option can be present several times (once per device).  [NOTE: PCI whitelist cannot be used with -b option]

`--vdev`
  Add a virtual device.  The argument format is &lt;driver&gt;&lt;id&gt;[,key=val,...] (ex: --vdev=eth_pcap0,iface=eth2).

`-d` LIB.so|DIR
  Add a driver or driver directory (can be used multiple times) --vmware-tsc-map    Use VMware TSC map instead of native RDTSC

`--proc-type`
  Type of this process (primary|secondary|auto)

`--syslog`
  Set syslog facility

`--log-level`
  Set default log level

`-v`
  Display version information on startup

`-h, --help`
  This help

#### EAL options for DEBUG use only:


`--huge-unlink`
  Unlink hugepage files after init

`--no-huge`
  Use malloc instead of hugetlbfs

`--no-pci`
  Disable PCI

`--no-hpet`
  Disable HPET

`--no-shconf`
  No shared config (mmap'd files)

EAL Linux options:

`--socket-mem`
  Memory to allocate on sockets (comma separated values)

`--huge-dir`
  Directory where hugetlbfs is mounted

`--file-prefix'
  Prefix for hugepage filenames

`--base-virtaddr'
  Base virtual address

`--create-uio-dev`
  Create /dev/uioX (usually done by hotplug)

`--vfio-intr`
  Interrupt mode for VFIO (legacy|msi|msix)

`--xen-dom0`
  Support running on Xen dom0 without hugetlbfs

PARAMETERS
----

####Application manadatory parameters:                                             

`--rx` "(PORT, QUEUE, LCORE), ..." 
  List of NIC RX ports and queues handled by the I/O RX lcores                                        

`--tx` "(PORT, LCORE), ..." 
  List of NIC TX ports handled by the I/O TX lcores                                                              

`--w` "LCORE, ..." 
  List of the worker lcores                             

`--c` LCORE 
  The control lcore                                              

`--lpm` "IP/PREFIX => NEXTHOP; ..." 
  List of LPM rules used by the worker lcores for packet forwarding                                 

`--via` "HOP/HOP_MAC => PORT; ..." 
  List of VIA rules used by the worker lcores for packet forwarding                                 
                                                                               
####Application optional parameters:                                               

`--rsz` "A, B, C, D" : Ring sizes`
  A = Size (in number of buffer descriptors) of each of the NIC RX  
      rings read by the I/O RX lcores  
  B = Size (in number of elements) of each of the SW rings used by the  
      I/O RX lcores to send packets to worker lcores  
  C = Size (in number of elements) of each of the SW rings used by the  
      worker lcores to send packets to I/O TX lcores  
  D = Size (in number of buffer descriptors) of each of the NIC TX  
      rings written by I/O TX lcores  

`--bsz` "(A, B), (C, D), (E, F)" : Burst sizes
   A = I/O RX lcore read burst size from NIC RX
   B = I/O RX lcore write burst size to output SW rings
   C = Worker lcore read burst size from input SW rings
   D = Worker lcore write burst size to output SW rings
   E = I/O TX lcore read burst size from input SW rings
   F = I/O TX lcore write burst size to NIC TX

NOTES
----

SEE ALSO
--
The DPVS web site (https://github.com/yubo/doc/blob/master/docs/dpvs.md) for more documentation about DPVS.

  *ipvsadm(8), keepalived(8), dpdk, jsonrpc*

AUTHORS
----

Yu Bo <yubo@xiaomi.com>  
