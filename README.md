MiniMunin For SmartOS / OmniOS
----------------------

minimunin - is a work in progress porting freebsd's minimunin to illumos  * 
It mainly targets SmartOS Global Zones and OmniOS to a lesser extent.
It uses only shell and installed programs from the SmartOS GZ. No perl
python required.

Since this is a port of miniunin for FreeBSD see http://erdgeist.org/arts/software/minimunin/
for details on how it works. 

o Installation


-SmartOS - GZ
  1. Create /opt/custom/munin and copy the contents from git into that directory.
  2. Copy /opt/custom/munin/smf/munin-tcp.xml into /opt/custom/smf
  3. Import the smf via `svccfg import /opt/custom/smf`
  4. Verify minimunin is running and accepting connections via `svcs |fgrep munin` and `nc -v 127.0.0.1 4949`
  
-OmniOS
  1. Create /opt/custom/munin and copy the contents from git into that directory.
  2. Import the smf via `svccfg import /opt/custom/munin/smf/munin-tcp.xml`
  3. Verify minimunin is running and accepting connections via `svcs |fgrep munin` and `nc -v 127.0.0.1 4949`

o Plugins

-Base Plugins
  * load: Basic system load
  * iostat: Output from iostat
  * processes: Number of heavy and lightweight processes
  * uptime: Uptime in days

-Included Plugins
  * zfs-arc-stats: Graphs Kernel ARC stats
  * zfs-arc-usage: Graphs Kernel ARC Usage states
  * cpu: Graphs CPU usage broken out by various stated
  * vm-disk-pct-usage: This is very SmartOS oriented. This plugin graphs the zones + kvm disk usage along side the zpool usage
  * vm-disk-zfs-usage: This is very SmartOS oriented. This plugin graphs the zones + kvm disk usage along side the zpool usage
  * zone_df: This is very SmartOS oriented. This plugin graphs a zone's df output from inside of the zone 
  * memstat: Graphs memory usage in its various states
  * ntp_offset: Graphs the ntpd offset
  * paging_in: Graphs pages swapped in
  * paging_out: Graphs pages swapped out. Should be merged with paging_in graph
  * tcp-states: Graphs the output from netstat's states
  * zfs-iops: Graphs the number of ZFS iops
  * zfs-vfs-throughput: Graphs the bitrate of ZFS operations
  * zpool-iostat: Graphs the iostat from zpool 

-Included Plugin Templates
  * if_: Graphs the throughput of a nic  * 
  * if_err_: Graphs the errors of a nic .
  * zfs_fs_: A multifs zfs graph

