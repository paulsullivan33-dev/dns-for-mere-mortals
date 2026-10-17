# The Hierarchy

**Scope:** how the namespace is organized — root,
TLD, and your zone — and the one idea that explains
most catastrophic DNS failures: delegation.

## A tree, read right to left

A domain name is a path down a tree, written from
the leaf backwards:

```
www.example.com.
└┬┘ └─┬───┘ └┬┘ └ root (the trailing dot)
 │    │      └ top-level domain (TLD): .com
 │    └ domain: example.com
 └ subdomain: www
```

- The **root** (the silent trailing dot) is run by
  13 named server identities — hundreds of physical
  machines worldwide behind those names.
- Each **TLD** (`.com`, `.org`, `.test`, your
  country's code) is run by a registry.
- Each **domain** (`example.com`) is run by whoever
  registered it — meaning: by the DNS host they
  pointed it at.
- **Subdomains** (`www`, `mail`) are just more
  names inside the domain, managed in the same
  place.

Nobody in this chain holds a copy of your records
except the last step. Everyone else holds
*pointers*. That distinction is the chapter.

## Delegation: the handoff

When you register a domain, you tell the registrar
which servers are **authoritative** for it. The
registry then publishes **NS records** for your
domain in the TLD's zone — a referral that says
*"for anything under example.com, ask these
servers."* That referral is the **delegation**.
Following it is how every fresh lookup finds you.

Check: the real delegation for `example.com`,
asked over DNS-over-HTTPS (see chapter 6 for the
command):

```
"Answer": [
  {"name": "example.com.", "type": 2, "TTL": 21600,
   "data": "elliott.ns.cloudflare.com."},
  {"name": "example.com.", "type": 2, "TTL": 21600,
   "data": "hera.ns.cloudflare.com."}
]
```

Two NS records, a six-hour TTL, and a resolver
somewhere confirming: *"response from an
authoritative server for this zone."* Your domain
has exactly the same arrangement, with your DNS
host's names in place of these.

## Glue: the bootstrap problem

Delegation has a chicken-and-egg trap. If your
nameservers are *inside* your own domain —
`ns1.example.com` serving `example.com` — a
resolver can't look them up, because looking
anything up in `example.com` requires asking them
first. The fix is **glue records**: the registry
publishes the nameservers' IP addresses *in the
parent zone*, alongside the NS records, so the
resolver can reach the servers without resolving
them first. Glue lives at the registrar/registry,
not in your zone — remember that when we reach the
failures chapter. It only matters when nameserver
names are in-domain; servers named under somebody
else's domain need no glue.

## When the delegation breaks

If the delegation points at servers that don't
answer for your zone — the names were retired, the
zone was never loaded there, the registrar and the
DNS host disagree about who's in charge — resolvers
follow the referral into a wall. They can't fall
back: the delegation *is* the only path. The
result is usually **SERVFAIL** ("server failure"):
the resolver's honest report that the machinery
above it is broken. It is not "the domain doesn't
exist" (that's **NXDOMAIN**, chapter 6) and it's
not "your records are wrong" — it's "the handoff
itself failed," and from the outside the domain
simply vanishes while its zone file sits somewhere,
perfectly correct, on servers nobody is being sent
to.

A verified SERVFAIL, from a deliberately broken
test domain (both big public resolvers agree):

```
{"Status": 2, "Comment":
  "DNSSEC validation failure.",
 "extended_dns_errors": [{"info_code": 9,
  "extra_text": "No DNSKEY matches DS RRs"}]}
```

That one is a DNSSEC mismatch rather than a dead
delegation, but the status — 2, SERVFAIL — is the
same: *the lookup machinery failed*, not *the name
is absent*. Learning to read which of those two a
failure is, is half of DNS troubleshooting.
