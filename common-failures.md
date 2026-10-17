# Common Failures

**Scope:** the failures you'll actually meet,
symptom → check → fix. Statuses and behaviors are
from this guide's verified runs unless marked
(documented).

## The whole domain vanished — SERVFAIL everywhere

**Symptom:** every lookup fails, every resolver,
web and mail together, while your servers are
provably up.

**Check:** ask the *parent* who it delegates to,
and ask the nameservers themselves — DoH makes
both easy:

```
curl -s -H 'accept: application/dns-json' \
  'https://dns.google/resolve?name=example.com&type=NS'
dig @1.1.1.1 example.com        (on your machine)
```

Look for `Status: 2` (SERVFAIL) and for NS names
that are retired, misspelled, or not servers you
recognize.

**Fix:** at the **registrar**, point the delegation
at nameservers that actually serve your zone. The
zone file was probably never wrong — nobody was
being sent to it (chapter 2). Then wait out TTLs;
SERVFAILs are cached too, briefly.

## New site for you, old site for everyone else

**Symptom:** after an address change, results
differ by person, office, or resolver.

**Check:** compare resolvers — `dig @1.1.1.1`,
`dig @8.8.8.8`, and your default — and read the
**TTL on each answer**. Different TTLs remaining
mean different caches at different ages (chapter 5
verified 300 vs 105 on the same real records).

**Fix:** none exists, and that's the lesson: wait
out the old TTL, and next time lower it days ahead
(chapter 10). Flushing *your* resolver helps only
you.

## A brand-new name "doesn't exist" — for one person

**Symptom:** you created the record an hour ago;
most people reach it; one resolver still says
NXDOMAIN.

**Check:** ask the authority directly (your DNS
host's server, or the lab trick: query the
nameserver itself with `dig @<ns>`). If the
authority has it, the complainer's resolver
**negatively cached** the earlier miss — the SOA's
last field is how long that lasts (300 in the lab,
1800 on example.com).

**Fix:** wait out the negative TTL. Prevention:
create records *before* announcing names, not
after the first failed lookup has already been
cached by half the internet.

## "CNAME at the apex" — the bare domain broke

**Symptom:** `www` works; the bare domain doesn't,
or the provider's panel refused your record with a
cryptic error.

**Check:** did something put a CNAME on the apex?
The apex must hold SOA and NS; a CNAME claims the
whole name and collides (chapter 4).

**Fix:** put A/AAAA records on the apex pointing at
the same host, or use the provider's
flattening/ALIAS feature if they offer one. Never
"fix" it by deleting the SOA/NS — the panel won't
let you, and that's it working as intended.

## Web fine, mail bouncing

**Symptom:** senders get "domain not found" or
their queues silently expire; the website never
blinked.

**Check:** `dig <domain> MX` — then, for whatever
name comes back, `dig <that-name> A`. And
`dig <domain> MX` returning `0 .` means the domain
*declares* no mail service (chapter 8).

**Fix:** the usual three — MX missing entirely,
MX pointing at a CNAME (repoint it at the real
host name), or MX pointing at a decommissioned
server whose A record went away in a cleanup.

## Half the nameservers serve the old zone

**Symptom:** answers alternate between old and new
depending on which server a resolver happens to
ask. Truly intermittent — the worst kind.

**Check:** ask each authoritative server for the
SOA and compare **serials**:

```
dig @ns1.example.com example.com SOA +short
dig @ns2.example.com example.com SOA +short
```

(documented form — the lab has a single server.)

**Fix:** a lagging secondary heals itself once it
refreshes or is nudged; a serial that went
*backwards* (restored from an old backup) never
heals — secondaries ignore lower serials. Bump
the serial past the highest value ever served.

## In-domain nameservers unreachable — glue

**Symptom:** delegation lists `ns1.yourdomain`
style nameservers, and resolvers report they can't
reach *any* of them, though the servers answer fine
when asked by IP.

**Check:** ask the parent zone for the nameserver
names' addresses. In-domain nameservers need
**glue** A/AAAA records *at the registry* (chapter
2) — if they're absent, the bootstrap is
impossible.

**Fix:** add the glue at the **registrar** (it
lives in the parent, not your zone), with the
servers' real addresses. Zone-side A records for
the nameservers are good practice too, but they
don't substitute.

## Everything broke after a tidy "DNS cleanup"

**Symptom:** logins, mail authentication, or a
SaaS integration died a day after someone removed
"unused" records.

**Check:** the deleted were probably TXT records —
a verification token, SPF, DMARC — which look like
debris and are load-bearing (chapter 8). Compare
against any zone export or screenshot from before.

**Fix:** restore from the inventory you took
(chapter 10 makes taking one a rule). Absent one,
each service's admin panel will happily re-issue
its verification record — one slow rediscovery at
a time.
