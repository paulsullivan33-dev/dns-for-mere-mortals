# Your Own Zone

**Scope:** running a tiny authoritative zone
yourself — with dnsmasq, on a laptop, for a made-up
domain — and proving it works with the tools from
chapter 6.

## Why bother

You will probably host real DNS with a provider,
not a laptop. But one afternoon as the authority —
not a client, not a resolver, *the source* — teaches
the whole system faster than any reading. This lab
was run for real while writing this guide; the
config files are in the repo's `examples/` folder.

The domain is `example.test`. The `.test` TLD is
reserved for exactly this: it can never be a real
public domain, so nothing you do can collide with
anybody's production anything. The addresses are
from `192.0.2.0/24` (TEST-NET-1), reserved for
documentation for the same reason.

## The server: dnsmasq in authority mode

dnsmasq is famous as a home-router DHCP/DNS
forwarder, but it has an authoritative mode. The
lab config, in full (also `examples/dnsmasq-lab.conf`):

```
port=15353
listen-address=127.0.0.1
bind-interfaces
no-resolv
no-hosts
no-poll
local-ttl=300
addn-hosts=lab-hosts.txt
cname=blog.example.test,www.example.test
mx-host=example.test,mail.example.test,10
txt-record=example.test,"v=spf1 mx -all"
ptr-record=25.2.0.192.in-addr.arpa,mail.example.test
auth-server=ns.example.test
auth-zone=example.test
auth-ttl=300
```

And `lab-hosts.txt` — yes, a hosts file; in
authority mode dnsmasq serves these as zone data:

```
192.0.2.80 example.test www.example.test
192.0.2.25 mail.example.test
192.0.2.53 ns.example.test
```

Line by line, the interesting parts: a high
**port** and loopback-only **listen-address**, so
the lab touches nothing real; **no-resolv**, so it
can never accidentally forward your questions to
the internet; **auth-zone**, which flips dnsmasq
from forwarder to authority for `example.test` —
it builds the zone's SOA itself (check below) and
answers NXDOMAIN for names the zone doesn't have.
Start it with:

```
dnsmasq --conf-file=dnsmasq-lab.conf --no-daemon
```

## Check: interrogate your own zone

Every answer below is real lab output:

```
$ dig @127.0.0.1 -p 15353 +tcp example.test SOA +short
ns.example.test. hostmaster.ns.example.test. \
  1792269275 1200 180 1209600 300

$ dig @127.0.0.1 -p 15353 +tcp undefined.example.test
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN ...

$ dig @127.0.0.1 -p 15353 +tcp example.test MX +short
10 mail.example.test.
```

The serial in that SOA is generated from the
clock; refresh/retry/expire are dnsmasq's defaults.
Names not in the zone get NXDOMAIN — the authority
saying *that name is not mine to give*, which is a
different thing from a forwarder shrugging.

## Honest boundaries of the lab

- **The sandbox note.** This guide's lab machine
  blocked UDP DNS replies outright (a sandbox
  firewall rule), so every lab query here was run
  over TCP — that's the `+tcp` in the commands.
  DNS over TCP is real DNS, identical answers; on
  your machine, plain `dig @127.0.0.1 -p 15353`
  over UDP will work and match.
- **dnsmasq is a lightweight authority.** It builds
  SOA and answers the record types above, but ask
  it for the zone's own apex NS records and it
  returns an empty answer — a real authoritative
  server (BIND 9, Knot, your provider) serves
  those properly. For the full-fat version,
  `examples/example.test.zone` is the same zone as
  a classic zone file; serving it with BIND is
  **(documented)** here, not run.
- **A real zone has more in it.** Secondaries,
  DNSSEC signing, and being reachable by the whole
  internet are all deliberately out of scope —
  this lab exists to make chapters 2, 4, and 5
  stop being abstract.
