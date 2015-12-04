# debian-netns

Some simple scripts to simplify configuring network namespaces on
Debian-like systems.  Copy them into the corresponding directories under
`/etc/network`.

To configure a veth pair into a namespace:
```
auto ns0
iface ns0 inet manual
    peer-netns myns
    peer-iface eth0
```
Running `ifup ns0` will then create the `myns` network namespace and a veth
pair which joins `ns0` in the "real world" to `eth0` inside the namespace.

There are two more things you can specify:
```
auto ns0
iface ns0 inet manual
    peer-netns myns
    peer-iface eth0
    veth-mode l3
    configure-interfaces yes
```
`veth-mode l3` will configure the veth pair as a L3 point-to-point link
(meaning that you can then add routes with the next-hop set to the
interface, e.g., `ip route add 1.2.3.0/24 dev ns0`).

`configure-interfaces yes` will run `ifup -a` (or `ifdown -a`) inside the
`myns` namespace using interface config from `/etc/network/interfaces.myns`.

### Thoughts

Ideally, the namespace interface config would be under
`/etc/netns/myns/network/interfaces`, but thanks to this feature of `ip
netns exec`:

"ip netns exec automates handling of this configuration, file convention for
network namespace unaware applications, by creating a mount namespace and
bind mounting all of the per network namespace configure files into their
traditional location in /etc."

this means that when `ifup` runs, it won't see the `/etc/network/if-*.d`
scripts (at least if they're not mirrored in `/etc/netns/myns`).

It would be nice to have some better way to run `ifup` inside a namespace,
but this would require hacking `ifupdown`.  If you want to know how to run it
by hand, look in `if-up.d/netns`.  It's not as simple as you might think.

