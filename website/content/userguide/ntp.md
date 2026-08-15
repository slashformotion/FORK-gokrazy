---
title: "NTP Client"
menuTitle: "NTP Client"
weight: 22
---

Each gokrazy instance comes with a built-in NTP client (see [Instance Config →
GokrazyPackages](/userguide/instance-config/#gokrazypackages) for more details
on system packages) which sets the system clock once the network is up. The
client drops root privileges after start-up and only keeps the
`CAP_SYS_TIME` capability required for setting the clock.

## NTP servers

The NTP client determines which servers to query in the following order:

1. Servers specified on the [command line](/userguide/package-config/#flags),
   if any
2. NTP servers provided by the network’s DHCP server, if any (see below)
3. The default `*.gokrazy.pool.ntp.org` server pool

### DHCP-provided NTP servers

The [gokrazy DHCP client](/userguide/dhcp/) requests DHCP option 42 (NTP
servers) with each lease. If your DHCP server provides NTP servers, the DHCP
client writes them to `/tmp/ntp-servers`, and the NTP client uses these
addresses instead of the default pool. The file is re-read whenever the clock
is set, so changes from lease renewals are picked up without a restart.

If your DHCP server does not provide NTP servers, the file is removed and the
NTP client automatically falls back to the default server pool.

### Manually specifying NTP servers

To use your own NTP servers, specify them as positional [command-line
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

Servers specified on the command line take precedence over DHCP-provided
servers.

## Real-time clock

If your device has a real-time clock (RTC) at `/dev/rtc0`, the NTP client
will set it whenever the system clock is set, so that the correct time is
available directly after a reboot.

In addition, the current time is saved to the `ntp-time-at-last-shutdown` file
in the NTP client’s home directory (`/perm/home/ntp`) at shutdown, and
restored at boot, so that devices without an RTC start with an approximately
correct clock before the network is up.
