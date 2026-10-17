# Reading Dig

**Scope:** the diagnostic tool — what its output is
actually saying, the four statuses that matter, and
the three ways to ask (direct, specific resolver,
over HTTPS).

`dig` comes from the BIND tools (`dnsutils` /
`bind-utils`); `nslookup` ships everywhere including
Windows. Everything below was run against the
guide's lab or live resolvers unless marked
(documented).

## Anatomy of an answer

```
$ dig @127.0.0.1 -p 15353 +tcp www.example.test

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28943
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;www.example.test.        IN      A

;; ANSWER SECTION:
www.example.test.   300   IN      A       192.0.2.80

;; Query time: 1 msec
;; SERVER: 127.0.0.1#15353(127.0.0.1) (TCP)
```

Read it in this order:

1. **status** — the verdict. The four you'll meet:
   - `NOERROR` — a real answer (which may still be
     *empty*: NODATA, chapter 4).
   - `NXDOMAIN` — the name does not exist. Definitive,
     from the authority.
   - `SERVFAIL` — the machinery failed: bad
     delegation, broken DNSSEC, dead servers. The
     resolver is reporting *its* failure to get an
     answer, not a fact about the name.
   - `REFUSED` — the server declines to answer you
     (policy, or you're asking a server that only
     serves its own zones/customers).
2. **flags** — `aa` means *authoritative answer*:
   the reply came from the zone's own server, not a
   cache. Its absence on an otherwise good answer
   usually means "served from cache" (chapter 5).
3. **ANSWER SECTION** — the records themselves:
   name, TTL remaining, class (`IN` = internet),
   type, payload.
4. **SERVER** — *who actually answered you.* In a
   surprising share of mysteries, it's not who you
   assumed.

## The everyday forms

Ask about a specific type, get the short version,
ask a *specific server* — this is the whole daily
vocabulary:

```
$ dig example.test MX +short
10 mail.example.test.

$ dig @127.0.0.1 -p 15353 +tcp blog.example.test +short
www.example.test.
192.0.2.80
```

(`+short` prints just the payload — note how it
shows the CNAME hop and the final address as two
plain lines.)

`@server` is the single most powerful idea in DNS
debugging. `dig example.com` asks *your configured
resolver*; `dig @1.1.1.1 example.com` asks
Cloudflare's. Different answers? You're looking at
caching or a split view, and chapter 5 explains
both. Same wrong answer everywhere? The problem is
at the authority, and your resolver is innocent.

`nslookup` does the same job with less detail —
fine for a first glance:

```
$ nslookup www.example.test 127.0.0.1
Server:         127.0.0.1
Address:        127.0.0.1#15353

Name:   www.example.test
Address: 192.0.2.80
```

(The lab's nslookup used its `-vc` flag to ask over
TCP; your machine won't need it — see the lab note
in chapter 7.)

## dig +trace: watch the walk (documented)

`dig +trace example.com` makes dig do the
resolver's job itself, printing each referral down
the hierarchy — root, TLD, authority — as it goes.
It's the best way to *see* delegation, and the
fastest way to find the level where a broken one
goes wrong. It's marked documented because this
guide's lab sandbox intercepted the very first
(root) hop and the walk couldn't run there; on any
normal machine it's one command, and everything up
to the actual trace output here was verified.

## The HTTPS back door

When port 53 is blocked, filtered, or simply not
to be trusted — hotel Wi-Fi, a locked-down office,
a machine with no DNS tools — ask over HTTPS
instead (chapter 3). Both big resolvers expose the
same JSON shape:

```
$ curl -s -H 'accept: application/dns-json' \
  'https://cloudflare-dns.com/dns-query?name=example.com&type=A'
{"Status": 0, "Answer": [
  {"name": "example.com", "type": 1, "TTL": 105,
   "data": "172.66.147.243"}, ... ]}
```

`Status` is the same code dig prints: 0 NOERROR,
2 SERVFAIL, 3 NXDOMAIN. Two endpoints also give you
the multi-resolver comparison for free — Google's
and Cloudflare's caches will disagree about TTLs
exactly as chapter 5 showed, and agreement between
them is strong evidence about where a problem is
*not*.
