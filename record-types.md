# Record Types

**Scope:** the records you'll actually meet, what
each is for, and the two rules that surprise
everyone. All examples are real unless noted: from
the guide's lab zone (`example.test`) or live
public answers.

A **record** is one row of zone data: a name, a
type, a TTL, and a payload. The types that matter:

## The address records: A and AAAA

**A** maps a name to an IPv4 address, **AAAA** to
an IPv6 address. One name can hold several of each
(resolvers rotate through them):

```
www.example.test.   300   IN   A   192.0.2.80
```

Asking for a type a name doesn't have is *not* an
error — the server returns NOERROR with an empty
answer (**NODATA**). Verified against the lab:
`www.example.test` has an A record but no AAAA, and
the AAAA query came back `status: NOERROR`,
`ANSWER: 0`. Browsers ask for both, every time —
someone with a broken AAAA setup and a working A
record has a site that's slow or dead for some
visitors and fine for others.

## CNAME: the alias

**CNAME** says "this name is another name — go ask
there." Verified chain from the lab:

```
blog.example.test.  300  IN  CNAME  www.example.test.
www.example.test.   300  IN  A      192.0.2.80
```

One query, both rows in the answer: the resolver
followed the alias and kept going to the address.
Two restrictions, both enforced by real servers
and both common beginner wounds:

- A CNAME **cannot coexist** with any other record
  at the same name — an alias claims the whole
  name, all types.
- A CNAME **cannot sit at the zone apex** (the bare
  domain), because the apex must hold SOA and NS
  records — which would collide with the rule
  above. Providers offering "CNAME flattening" or
  ANAME/ALIAS records are working around this
  behind the scenes.

## MX: where mail goes

**MX** names the servers that accept mail for the
domain, with a preference number (lower wins).
Real example, `gmail.com`:

```
gmail.com.  IN  MX  5  gmail-smtp-in.l.google.com.
gmail.com.  IN  MX  10 alt1.gmail-smtp-in.l.google.com.
gmail.com.  IN  MX  20 alt2.gmail-smtp-in.l.google.com.
```

Note what MX is *not*: it never contains an IP
address. It points at another *name*, which has its
own A/AAAA records. A domain can also declare it
accepts **no mail at all** — `example.com` really
does this: its MX answer is `0 .`, the "null MX,"
which tells senders to stop trying instead of
queueing. Chapter 8 is mail's full story.

## TXT: the sticky-note record

**TXT** holds arbitrary text, and the internet has
adopted it as the standard place to *prove things*
and *publish policy*: domain-ownership challenges,
SPF sender policy, DKIM public keys, DMARC rules.
The lab's:

```
example.test.  TXT  "v=spf1 mx -all"
```

TXT payloads are one or more quoted strings; long
keys arrive split into chunks that the reading
software rejoins. Nobody reads TXT for fun — it's
the record other systems check about you.

## NS and SOA: the zone's own paperwork

**NS** records at the apex list the zone's
authoritative servers — the in-zone copy of the
delegation from chapter 2. The two copies (parent's
and zone's own) must agree; the parent's is the
one resolvers follow first.

**SOA** ("start of authority") is exactly one per
zone: the zone's birth certificate and rulebook.
A real one, `example.com`'s, field by field:

```
elliott.ns.cloudflare.com.   ← primary nameserver
dns.cloudflare.com.          ← admin email (first dot = @)
2416374680                   ← serial: version of the zone
10000                        ← refresh: secondaries check in
2400                         ← retry if a check fails
604800                       ← expire: secondaries give up
1800                         ← negative TTL: cache "no" this long
```

The guide's lab generates the same shape
automatically: `ns.example.test. hostmaster... 1200
180 1209600 300`. The **serial** deserves respect:
secondary servers only pick up changes with a
higher serial, so "I edited the zone but the other
server still serves the old one" is often a serial
someone forgot to bump — or, on modern hosted DNS,
a propagation delay wearing the same costume.

## PTR and CAA: the specialists

**PTR** is the reverse lookup — an address asking
"who am I?" It lives in a separate tree
(`in-addr.arpa`), written backwards. Verified:

```
$ dig @127.0.0.1 -p 15353 +tcp -x 192.0.2.25
25.2.0.192.in-addr.arpa.  300  IN  PTR  mail.example.test.
```

PTR matters mostly for mail (receiving servers
side-eye senders whose addresses have no sane
reverse name) and for readable logs. Crucially,
you control PTR only for addresses *you* own the
block for — for a single hosted IP, the provider
sets it, or offers a panel to.

**CAA** lists which certificate authorities may
issue certificates for the domain — a small record
that turns "any CA on earth can mint a cert for
me" into a short named list. Optional, cheap, and
increasingly checked; if a certificate issuance
mysteriously fails, a stale CAA is a real suspect.
