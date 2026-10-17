# Resolvers

**Scope:** the two kinds of DNS server, whose
resolver you're actually using right now, and the
encrypted variants in plain terms.

## Two jobs, often confused

DNS servers come in two species that do opposite
work:

| | Recursive resolver | Authoritative server |
|---|---|---|
| Serves | you (clients) | one slice of the namespace |
| On a question | goes and *finds* the answer, chasing referrals | answers from its own zone data, or says the name isn't there |
| Caches | yes — that's most of its value | no need; it *is* the source |
| Examples | your router, ISP DNS, 1.1.1.1, 8.8.8.8 | your DNS host's nameservers |

When this guide says "the delegation points at
servers that don't answer," it means authoritative
servers. When it says "my change hasn't propagated,"
it means recursive resolvers and their caches. Most
DNS confusion is these two being mistaken for each
other.

## Whose resolver are you using?

By default, a chain of defaults decides for you:
your device asks the router (handed out by DHCP),
the router asks the ISP's resolvers. You can
override it per-device or per-network by naming a
**public resolver** instead — Cloudflare's
`1.1.1.1`, Google's `8.8.8.8`, Quad9's `9.9.9.9`
are the usual ones. Browsers add a twist: Chrome,
Firefox, and Edge can bypass the OS setting and use
their own resolver over HTTPS (next section), which
is why a name can fail in one browser and work in
another on the same machine.

None of these is "the DNS." They're all just
caches with different populations: an answer your
ISP's resolver has never been asked for takes the
full hierarchy walk; an answer 1.1.1.1 cached a
minute ago comes back instantly — with whatever
TTL remains on it (chapter 5).

## Encrypted DNS: DoH and DoT

Classic DNS is plaintext UDP on port 53: anyone on
the path can read your lookups and, worse, tamper
with answers. The two fixes encrypt the *client →
resolver* hop:

- **DNS-over-TLS (DoT)** — DNS inside TLS, on its
  own port (853). Easy to spot and to block, which
  is why it's mostly used between devices and a
  chosen resolver (Android's "Private DNS" is DoT).
- **DNS-over-HTTPS (DoH)** — DNS inside ordinary
  HTTPS on port 443, indistinguishable from web
  traffic. This is what browsers adopted, and why
  it's controversial with network admins: it works,
  and it routes around their resolver on purpose.

Neither encrypts the resolver's own work — the
resolver still walks the hierarchy in the clear and
still sees every name you ask for. Encrypted DNS
changes *who* you trust, not *whether* you trust.

DoH has a side benefit this guide uses constantly:
it's scriptable. Any machine with `curl` can ask a
public resolver a real question and get JSON back —
no DNS tools installed, no port 53 required:

```
$ curl -s -H 'accept: application/dns-json' \
    'https://dns.google/resolve?name=example.com&type=A'
{"Status": 0, "Question": [{"name": "example.com.", "type": 1}],
 "Answer": [
   {"name": "example.com.", "type": 1, "TTL": 300,
    "data": "172.66.147.243"},
   {"name": "example.com.", "type": 1, "TTL": 300,
    "data": "104.20.23.154"}]}
```

(Real output from writing this guide; the JSON is
trimmed for print.) `Status: 0` is a healthy answer.
Keep that command — chapter 6 builds the whole
diagnostic kit around it and around `dig`.
