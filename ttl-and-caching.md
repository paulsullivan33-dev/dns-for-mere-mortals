# TTL and Caching

**Scope:** why DNS changes feel slow, why
"propagation" is the wrong mental model, and the
one trick that makes planned changes painless.

## TTL: every record carries a fuse

Each record's **TTL** (time to live, in seconds)
tells resolvers how long they may cache it. When a
resolver fetches a record it starts counting down
from the full TTL; while the count runs, it answers
from cache without asking anyone. When it hits
zero, the next question triggers a fresh fetch.

That countdown is visible, and watching it once is
worth a chapter of theory. The guide's lab ran a
second dnsmasq as a caching resolver in front of
the lab authority. Two identical questions, five
seconds apart:

```
;; ANSWER SECTION:            (first answer)
www.example.test.   300   IN   A   192.0.2.80

;; ANSWER SECTION:            (five seconds later)
www.example.test.   295   IN   A   192.0.2.80
```

300 became 295 — the answer aged in the cache.
Notice also the second answer's flags lost the `aa`
(authoritative answer) mark: the first came from
the authority, the second from a cache honestly
saying "this is what I was told, and here's how
much longer I trust it."

## "Propagation" is cache expiry

Nothing is pushed out to the internet when you edit
a zone. No announcement, no sync. Your change is
live *at the authority* the moment it loads — and
invisible to everyone whose resolver is still
holding the old record, until that resolver's
countdown ends. "Waiting for DNS to propagate"
means *waiting for the world's caches to expire*,
one resolver at a time, on schedules set by your
old TTL.

Verified against the real internet while writing
this guide: `example.com`'s A records carry a TTL
of 300. Asked at the same moment, Google's resolver
reported the full **300** and Cloudflare's reported
**105** — same records, same addresses, different
remaining lifetimes, because Cloudflare's cache had
fetched them 195 seconds earlier. Neither is wrong.
That is the entire mystery of "my colleague sees the
new site and I don't," solved.

Consequences worth internalizing:

- **Different people see different answers** during
  any change window, and both answers are "live."
- **You cannot hurry a stranger's cache.** There is
  no flush-the-internet button; a few big resolvers
  offer purge pages for their own caches, nothing
  more.
- **Long TTLs are a trade, not a default.** 86400
  (a day) means fewer queries and faster answers —
  and a full day of stale answers the one time you
  need to move in a hurry.

## The migration trick: lower first, days ahead

Since the old TTL governs how long the old answer
survives, the time to shorten it is *before* you
need the change:

1. **A few days before** the move (at least one old
   TTL's worth), set the record's TTL low — 300 is
   the usual choice.
2. Wait out the *old* TTL. Anyone who cached the
   record just before your edit still holds it with
   the old, long fuse; that wait is not optional.
3. Make the change. Now every cache holding it
   expires within 300 seconds, and the new answer
   takes over world-wide within minutes.
4. Once stable, raise the TTL back up if you like.

Skip step 2 and the low TTL accomplishes nothing —
the classic mistake, and the reason "I set TTL to
300 and it still took a day" is a sentence spoken
weekly, somewhere, with feeling.

## Negative caching: even "no" has a TTL

Resolvers also cache *absences*: NXDOMAIN and
NODATA answers, for the duration named in the
zone's SOA (the last SOA field — `example.com`'s is
1800, the lab's is 300). Practical sting: create a
record *after* someone has already looked for it,
and their resolver may keep insisting it doesn't
exist for up to that negative TTL. When a brand-new
subdomain "doesn't work" for exactly one person,
this is the first suspect — chapter 9 returns to
it.
