# MiniMunin For SmartOS / OmniOS

minimunin - is a work in progress porting freebsd's minimunin to illumos . 
It mainly targets SmartOS Global Zones and OmniOS to a lesser extent.

Since this is a port of miniunin for FreeBSD see http://erdgeist.org/arts/software/minimunin/
for details on how it works. 

Installation

-SmartOS - GZ
  1. Create /opt/custom/munin and copy the contents from git into that directory.
  2. Copy /opt/custom/munin/smf/munin-tcp.xml into /opt/custom/smf
  3. Import the smf via `svccfg import /opt/custom/smf`
  4. Verify minimunin is running and accepting connections via `svcs |fgrep munin` and `nc -v 127.0.0.1 4949`
  
-OmniOS
  1. Create /opt/custom/munin and copy the contents from git into that directory.
  2. Import the smf via `svccfg import /opt/custom/munin/smf/munin-tcp.xml`
  3. Verify minimunin is running and accepting connections via `svcs |fgrep munin` and `nc -v 127.0.0.1 4949`


-Included Plugins
  1. df: This is very SmartOS oriented. This plugin graphs the zones + kvm disk usage along side the zpool usage
  2. 
