# MiniMunin for SmartOS / OmniOS

A lightweight, shell-based implementation of the Munin node client for illumos-based systems, primarily targeting SmartOS Global Zones and OmniOS.

## Overview

MiniMunin is a port of FreeBSD's [minimunin](http://erdgeist.org/arts/software/minimunin/) to illumos. It provides system monitoring capabilities using only shell scripts and standard system utilities—no Perl or Python required.

The implementation follows the Munin node protocol, allowing it to communicate with standard Munin master servers for centralized monitoring and graphing.

## Features

- **Zero Dependencies**: Uses only shell and built-in system utilities
- **Lightweight**: Minimal resource footprint
- **Native Integration**: Leverages illumos-specific tools (kstat, iostat, zfs utilities)
- **SMF Integration**: Managed via Service Management Facility
- **Extensible**: Easy to add custom plugins
- **SmartOS Optimized**: Special support for zones and ZFS monitoring

## Installation

### SmartOS Global Zone

1. Create the installation directory:
   ```bash
   mkdir -p /opt/custom/munin
   ```

2. Copy the repository contents:
   ```bash
   cd /opt/custom/munin
   # Copy all files from the git repository to this directory
   ```

3. Set up the SMF service:
   ```bash
   mkdir -p /opt/custom/smf
   cp /opt/custom/munin/smf/munin-tcp.xml /opt/custom/smf/
   svccfg import /opt/custom/smf/munin-tcp.xml
   ```

4. Verify the service is running:
   ```bash
   svcs | grep munin
   nc -v 127.0.0.1 4949
   ```

### OmniOS

1. Create the installation directory:
   ```bash
   mkdir -p /opt/custom/munin
   ```

2. Copy the repository contents:
   ```bash
   cd /opt/custom/munin
   # Copy all files from the git repository to this directory
   ```

3. Import the SMF manifest:
   ```bash
   svccfg import /opt/custom/munin/smf/munin-tcp.xml
   ```

4. Verify the service is running:
   ```bash
   svcs | grep munin
   nc -v 127.0.0.1 4949
   ```

## Architecture

### Main Components

- **`minimunin`**: Main daemon script that implements the Munin node protocol
- **`plugins/`**: Directory containing plugin scripts
- **`configs/`**: Configuration files for plugins
- **`template/`**: Plugin templates for creating new plugins
- **`smf/`**: Service Management Facility manifest

### Built-in Plugins

The following plugins are built directly into the main `minimunin` script:

- **load**: System load average (5-minute average)
- **iostat**: Disk I/O statistics from iostat
- **processes**: Number of processes and lightweight processes (LWPs)
- **uptime**: System uptime in days

### External Plugins

#### CPU & Memory
- **cpu**: CPU usage broken down by state (user, system, idle, etc.)
- **memstat**: Memory usage in various states
- **zone_memstat**: Memory statistics for zones

#### Storage & ZFS
- **zfs-arc-stats**: ZFS ARC (Adaptive Replacement Cache) statistics
- **zfs-arc-usage**: ZFS ARC usage states
- **zfs-iops**: ZFS I/O operations per second
- **zfs-vfs-throughput**: ZFS VFS throughput in bytes
- **zpool-iostat**: ZFS pool I/O statistics
- **vm-disk-pct-usage**: Zone and KVM disk usage percentage alongside zpool usage (SmartOS-specific)
- **vm-disk-zfs-usage**: Zone and KVM disk usage in bytes alongside zpool usage (SmartOS-specific)
- **zone_df**: Disk usage from inside zones (SmartOS-specific)

#### Network
- **tcp-states**: TCP connection states from netstat
- **if_**: Network interface throughput (template-based)
- **if_err_**: Network interface errors (template-based)

#### System
- **ntp_offset**: NTP daemon time offset
- **paging_in**: Pages swapped in
- **paging_out**: Pages swapped out

#### Templates
- **zfs_fs_**: Multi-filesystem ZFS graph template

## Plugin Templates

Templates allow you to create multiple instances of a plugin for different resources. For example:

- `if_e1000g0` - Monitor e1000g0 network interface
- `if_net0` - Monitor net0 network interface
- `zfs_fs_zones` - Monitor zones filesystem

To use a template, create a symbolic link in the `plugins/` directory:

```bash
cd /opt/custom/munin/plugins
ln -s ../template/if_ if_e1000g0
ln -s ../template/zfs_fs_ zfs_fs_zones
```

## Configuration

Plugin configurations are stored in `/opt/custom/munin/configs/`. Configuration files use the following format:

```ini
[plugin_name]
env.VARIABLE_NAME value
user username
command /path/to/command %c
```

- **env.VARIABLE_NAME**: Set environment variables for the plugin
- **user**: Run the plugin as a specific user (requires root)
- **command**: Override the default command (%c is replaced with plugin path)

## Usage

### Testing Plugins

You can test plugins manually:

```bash
# Test plugin configuration
/opt/custom/munin/plugins/cpu config

# Test plugin data fetch
/opt/custom/munin/plugins/cpu fetch
```

### Connecting to the Node

The Munin node listens on port 4949 by default. You can connect manually:

```bash
nc localhost 4949
```

Available commands:
- `list` - List all available plugins
- `config <plugin>` - Get plugin configuration
- `fetch <plugin>` - Get plugin data
- `quit` - Close connection

### Service Management

```bash
# Check service status
svcs munin

# Enable service
svcadm enable munin

# Disable service
svcadm disable munin

# Restart service
svcadm restart munin

# View service logs
svcs -L munin
```

## Creating Custom Plugins

### Plugin Requirements

1. Must be executable
2. Must support two commands:
   - `config` - Output plugin configuration
   - `fetch` - Output current values

### Example Plugin

```bash
#!/bin/sh

case $1 in
config)
    cat <<EOF
graph_title My Custom Plugin
graph_vlabel value
graph_category system
myvalue.label My Value
EOF
    ;;
fetch)
    echo "myvalue.value 42"
    ;;
esac
```

### Installing Custom Plugins

1. Place the plugin in `/opt/custom/munin/plugins/`
2. Make it executable: `chmod +x /opt/custom/munin/plugins/myplugin`
3. Test it: `/opt/custom/munin/plugins/myplugin config`
4. The plugin will be automatically discovered by the main daemon

## Troubleshooting

### Service Won't Start

Check the SMF service logs:
```bash
svcs -L munin
tail -f /var/svc/log/network-munin-tcp:default.log
```

### Plugin Not Showing Up

1. Verify the plugin is executable
2. Check that it's in the correct directory
3. Test the plugin manually
4. Restart the service

### Connection Refused

1. Verify the service is running: `svcs munin`
2. Check if port 4949 is listening: `netstat -an | grep 4949`
3. Check firewall rules if connecting remotely

## License

This code is released under the Beer Ware License:

> This script was written by erdgeist@erdgeist.org and updated by nonesuch@longcount.org.
> 
> Do whatever you want with it, as long as you leave this notice along with the code.
> Should we meet some day and you find the code is worth it, let's enjoy a beer together.

## Credits

- Original FreeBSD minimunin: [erdgeist](http://erdgeist.org/arts/software/minimunin/)
- SmartOS/OmniOS port: nonesuch@longcount.org
- Additional contributions welcome!

## Contributing

Contributions are welcome! When adding new plugins:

1. Follow the existing plugin structure
2. Test on both SmartOS and OmniOS if possible
3. Document any special requirements or dependencies
4. Use only standard shell and system utilities

## Resources

- [Munin Documentation](http://munin-monitoring.org/)
- [SmartOS Documentation](https://wiki.smartos.org/)
- [OmniOS Documentation](https://omnios.org/documentation)
- [Original FreeBSD minimunin](http://erdgeist.org/arts/software/minimunin/)