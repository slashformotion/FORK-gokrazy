---
title: "NTP Client"
menuTitle: "NTP Client"
weight: 22
---

Each gokrazy instance comes with a built-in NTP client (see [Instance Config →
GokrazyPackages](/userguide/instance-config/#gokrazypackages) for more details
on system packages) which sets the system clock once the network is up.

## NTP servers

The NTP client determines which servers to query in the following order:

1. Servers specified on the [command line](/userguide/package-config/#flags),
   if any
2. NTP servers provided by the network’s DHCP server, if any
3. The default `*.gokrazy.pool.ntp.org` server pool

If your DHCP server is configured to provide NTP servers (DHCP option 42),
they are used automatically — no configuration required. If they cannot be
reached, the default pool is used temporarily.

To use your own NTP servers instead, specify them as positional [command-line
flags](/userguide/package-config/):

{{< highlight json "hl_lines=9-16" >}}
{
    "Hostname": "dynamic",
    "Packages": [
        "github.com/gokrazy/fbstatus",
        "github.com/gokrazy/hello",
        "github.com/gokrazy/serial-busybox",
        "github.com/gokrazy/breakglass",
    ],
    "PackageConfig": {
        "github.com/gokrazy/gokrazy/cmd/ntp": {
            "CommandLineFlags": [
                "ntp.example.org",
                "192.168.178.1"
            ]
        }
    }
}
{{< /highlight >}}

## Real-time clock

If your device has a real-time clock (RTC), the NTP client keeps it updated, so
that the correct time is available directly after a reboot. Devices without an
RTC start with an approximately correct clock restored from the last shutdown
until the network is up. This requires a writable
[perm partition](/userguide/permanent-data/), where the time of the last
shutdown is stored.
