# DNS and Mail

**Scope:** mail is DNS's most demanding customer —
the records that decide whether your mail arrives,
and why a DNS failure is always also a mail
failure.

## Sending starts with a lookup

To deliver a message to `you@example.com`, a mail
server asks DNS one question: `example.com`'s **MX
records** (chapter 4). No MX, no delivery route —
the sender queues the message, retries for days,
and eventually bounces it. This is why "the website
works but mail is broken" and "everything is down"
are both DNS-shaped: web needs A records, mail
needs MX, and a broken delegation (chapter 2) takes
both at once.

Two MX rules that bite in practice:

- MX targets are **names, never IP addresses**, and
  the name must have its own A/AAAA records.
- An MX target **must not be a CNAME**. Some
  senders tolerate it; strict ones refuse. If your
  mail host asks you to point MX at a name they
  gave you, check what that name really is.

And the deliberate opposite: a domain that should
receive **no** mail can say so with the null MX —
`example.com`'s real MX answer is `0 .`, telling
the world "don't even queue it."

## The three proofs: SPF, DKIM, DMARC

Receiving servers don't just accept mail that
*claims* to be from your domain. Three DNS-based
systems let you publish who's allowed to claim it:

- **SPF** — a TXT record listing which servers may
  send for your domain. The lab's minimal version:
  `"v=spf1 mx -all"` — "my MX hosts may send;
  nobody else may" (the `-all` is the hard-fail
  ending; `~all` is the softer, learning-mode one).
- **DKIM** — your mail server signs messages with a
  private key; the matching **public key lives in
  DNS** as a TXT record at
  `<selector>._domainkey.example.com`, so receivers
  can verify the signature. (Key *generation* is
  your mail software's job — documented here, since
  the DNS side is just "publish this TXT record at
  this name.")
- **DMARC** — a TXT record at `_dmarc.example.com`
  saying what to do when SPF/DKIM don't line up
  (`v=DMARC1; p=none` just watches; `p=reject`
  refuses), and where to send reports about it.

All three are "just DNS records," which cuts both
ways: they're easy to publish, and they're
**invisible casualties of DNS edits** — nobody
misses the `_dmarc` TXT record in a zone cleanup
until the phishing reports start. Chapter 10's
inventory step exists largely because of this
section.

## Receiving has a DNS side too

Your own sending reputation is partly DNS:

- **PTR (reverse DNS)** on your sending address
  should resolve to a name that itself resolves
  back to the address. Big receivers check; a
  missing or generic reverse name quietly costs
  deliverability. On a hosted IP, your provider
  controls this record — ask them.
- Your **HELO name, MX names, and A records** should
  tell one consistent story. Mail forgives a lot;
  obvious incoherence isn't on the list.

## When DNS fails, mail fails *slowly*

One mercy: mail is a store-and-forward system, so
a DNS outage usually means *delayed* mail, not
lost mail — senders queue and retry, typically for
four to five days. Fix the DNS inside that window
and the backlog delivers itself. The dangerous
failures are the quiet wrong ones: an MX pointing
at a decommissioned host, an SPF record missing a
new sending service. Those don't bounce loudly;
they just leak mail, a little, for months.
