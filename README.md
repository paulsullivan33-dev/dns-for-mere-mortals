# DNS for Mere Mortals

Everything on a network starts by asking DNS a
question — and when DNS breaks, everything else
breaks with it, wearing a disguise. This guide
explains the system plainly: how a lookup travels,
which records matter, why changes take time, and
how to diagnose the failures that make whole
domains vanish:

- **Names, not numbers** — machines are reached by
  IP address; DNS is the distributed translation
  layer in front of them.
- **Pointers all the way down** — root, TLD, and
  your zone, joined by delegation.
- **Caches everywhere** — most of what you'll ever
  troubleshoot is a cache doing its job on old
  information.

It assumes you use a terminal and have seen an IP
address before. Everything about DNS itself —
records, resolvers, zones — is explained from zero.

## What this guide covers

- [**What DNS is**](what-dns-is.md) — the lookup,
  end to end, and why its failures are total
- [**The hierarchy**](the-hierarchy.md) — root,
  TLD, delegation, glue, and what SERVFAIL means
- [**Resolvers**](resolvers.md) — recursive vs.
  authoritative, whose resolver you're using, DoH
  and DoT
- [**Record types**](record-types.md) — A, AAAA,
  CNAME, MX, TXT, NS, SOA, PTR, CAA, with real
  answers
- [**TTL and caching**](ttl-and-caching.md) — the
  countdown, "propagation" demystified, negative
  caching
- [**Reading dig**](reading-dig.md) — output
  anatomy, the four statuses, asking specific
  servers, the HTTPS back door
- [**Your own zone**](your-own-zone.md) — a real
  dnsmasq lab: serve a domain, prove it with dig
- [**DNS and mail**](dns-and-mail.md) — MX, SPF,
  DKIM, DMARC, and mail's slow failure mode
- [**Common failures**](common-failures.md) —
  symptom → check → fix for the classic eight
- [**Changing DNS safely**](changing-dns-safely.md) —
  the checklist: inventory, TTLs, verify, rollback

## What it doesn't cover

Running a public authoritative server for other
people, DNSSEC signing operations, DNS privacy
beyond the resolver hop, and anycast/BGP — the
plumbing *behind* big DNS providers. The Networking
and Home Networking guides in this series cover
the surrounding territory.

## How to use this guide

The lab and lookups were run for real (dig 9.18,
dnsmasq 2.91): a dnsmasq instance served the made-up
`example.test` zone on loopback and was queried with
dig; a second instance demonstrated caching and the
TTL countdown (300 → 295, watched live). Public
answers — `example.com`'s records, `gmail.com`'s MX
ladder, a genuine SERVFAIL — were fetched over
DNS-over-HTTPS from two independent resolvers,
because the lab machine's sandbox intercepted
direct port-53 traffic; the guide says so where it
matters. Two things are marked **(documented)**:
`dig +trace` (its first hop was intercepted in the
sandbox) and serving the BIND zone file in
`examples/` (the same zone, unrun). Every other
command output quoted is verbatim from a real run,
trimmed only for print.
