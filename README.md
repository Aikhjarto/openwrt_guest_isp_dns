# ISP DNS servers for the OpenWrt guest network

By default, guests get the router itself as their DNS server, so their queries go through the
router's dnsmasq (and its local names, blocklists and rebind protection). This hotplug script
instead hands out the DNS servers the ISP assigned on `wan` to the guest network (DHCP option 6).
That way guests resolve directly at the ISP and never see local names. (A firewall rule
allowing the guest zone to reach port 53 on the router is then only needed by clients with
hardcoded DNS settings.)

A fixed `dhcp_option '6,…'` would break when the ISP changes its servers, so the script updates it
on every `ifup` / `ifupdate` of `wan`:

1. It reads `dns-server` from `ifstatus wan`.
2. If the list differs from `dhcp.guest.dhcp_option`, it writes `6,<dns1>,<dns2>`, commits and
   reloads dnsmasq.
3. It logs the change with tag `guest-dns`.

Clients pick up new servers when they renew their lease.

## Install

```sh
scp -O etc/hotplug.d/iface/30-guest-dns root@router:/etc/hotplug.d/iface/
ssh root@router 'echo /etc/hotplug.d/iface/30-guest-dns >> /etc/sysupgrade.conf; ifup wan'
```

The interface names `wan` and `guest` (the `config dhcp 'guest'` section) are hardcoded. Edit
the script for other names. The script takes over the whole `dhcp_option` list of the guest
section, so other DHCP options for guests are removed.
