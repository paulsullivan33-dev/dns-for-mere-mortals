# Glossary

**A / AAAA** — the address records: a name's IPv4
(A) and IPv6 (AAAA) addresses.

**Authoritative server** — a server that holds zone
data itself and answers from it, as opposed to a
resolver reporting what it was told. The `aa` flag
in dig marks its answers.

**CAA** — a record listing which certificate
authorities may issue certificates for the domain.

**Cache** — a resolver's memory of recent answers,
each kept for its TTL. Most "DNS weirdness" is a
cache doing exactly its job.

**CNAME** — an alias record: this name is that
name; go ask there. Cannot share its name with any
other record, and cannot sit at a zone's apex.

**Delegation** — the parent zone's NS records that
hand a domain to its nameservers. If the delegation
breaks, the domain vanishes even with a perfect
zone.

**dig** — the standard DNS query tool; shows status,
sections, TTLs, and exactly which server answered.

**DKIM** — mail signing whose public key is
published in DNS, letting receivers verify a
message really passed through an authorized server.

**DMARC** — a DNS-published policy for what to do
when SPF/DKIM don't line up, plus reporting.

**DNS-over-HTTPS (DoH)** — DNS queries carried
inside HTTPS (port 443). Browsers use it; `curl`
can ask resolvers with it too.

**DNS-over-TLS (DoT)** — DNS inside TLS on port
853; Android's "Private DNS" is this.

**Glue** — nameserver addresses published in the
*parent* zone, needed when the nameserver names
live inside the domain they serve.

**MX** — mail exchanger: the record naming a
domain's mail servers, with a preference number
(lower wins). Targets are names, never IPs, and
never CNAMEs.

**Negative caching** — resolvers remembering that a
name *doesn't* exist, for the SOA's negative TTL.

**NODATA** — a NOERROR answer with no records: the
name exists, that type doesn't. Not an error.

**NS** — nameserver record: who is authoritative
for a zone — held in the parent (the delegation)
and at the zone's apex.

**NXDOMAIN** — the status for "that name does not
exist," stated by the authority.

**Propagation** — the folk term for waiting out
other resolvers' caches. Nothing is pushed; TTLs
expire, one cache at a time.

**PTR** — the reverse record: an address's name,
living in the `in-addr.arpa` tree, written
backwards.

**Recursive resolver** — the server your device
asks, which walks the hierarchy and caches. Your
router, your ISP, 1.1.1.1, 8.8.8.8.

**REFUSED** — the server declines to answer you:
policy, or it only serves its own.

**Registrar** — who you pay for the name; holds the
delegation settings. Not necessarily your DNS host.

**SERVFAIL** — "server failure": the resolver
couldn't get an answer — broken delegation, DNSSEC
failure, dead servers. A machinery failure, not a
fact about the name.

**SOA** — start of authority: one per zone, holding
the primary server, admin contact, serial, and the
timers secondaries and negative caching obey.

**SPF** — a TXT record listing which servers may
send mail for the domain.

**TLD** — top-level domain: `.com`, `.org`,
`.test`, a country code — the registry's layer.

**TTL** — time to live, in seconds: how long a
resolver may cache a record. Counts down in the
answers you can watch with dig.

**TXT** — free-text records, adopted as DNS's
proof-and-policy drawer: verification tokens, SPF,
DKIM keys, DMARC.

**Zone** — one domain's subtree as administered
together: the unit a DNS host serves, a zone file
describes, and an SOA opens.
