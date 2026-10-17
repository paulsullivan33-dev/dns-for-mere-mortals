# Changing DNS Safely

**Scope:** the checklist. DNS changes are easy to
make and slow to take back — the whole discipline
is doing the thinking *before* the edit.

## Know the two different moves

"Changing DNS" confuses two operations that share
no steps:

- **Changing records** — same DNS host, new values.
  The checklist below.
- **Changing the DNS host** — moving the zone to a
  different provider. The safe order is: build the
  *complete* zone at the new host first, verify it
  answers, *then* change the delegation at the
  registrar. Resolvers keep using the old host
  until caches expire; both hosts serve the same
  data; nobody falls into the gap. Changing the
  delegation first and building the zone after is
  how domains vanish (chapter 9, first entry).

Changing **registrar** is a third thing: it moves
who you pay for the name. Done alone, with
nameservers untouched, it changes no DNS behavior
at all — the confusion of registrar with DNS host
causes real outages when someone "tidies up" the
nameserver fields mid-transfer.

## The checklist

1. **Inventory first.** Export or write down every
   record — all types, including TXT, CAA, and the
   underscore names (`_dmarc`, `selector._domainkey`,
   verification tokens). If the provider has no
   export, walk the types with dig (chapter 6) and
   screenshot the panel. This inventory *is* your
   rollback; there is no other.
2. **Lower TTLs ahead of time.** For records you're
   about to change: TTL to 300, at least one old
   TTL before the real edit — days ahead for a
   86400 record (chapter 5 explains why the wait is
   the step everyone skips).
3. **One change at a time.** A records today, MX
   Thursday. When something breaks at step two of
   five simultaneous edits, you'll wish each had
   its own afternoon.
4. **Make the edit, then verify at the authority.**
   Query the nameserver directly. If it's right
   there, your part is done — the rest is caches,
   and caches are weather, not workmanship.
5. **Verify from at least two other resolvers** —
   `dig @1.1.1.1`, `dig @8.8.8.8`, or the DoH pair
   from chapter 6. Agreement between two big
   independent caches plus the authority is as
   verified as DNS gets.
6. **Verify the dependents, not just the record.**
   Changed an A? Load the site *and* check mail
   still flows. Changed anything at all? Assume a
   TXT you forgot is involved until mail
   authentication (SPF/DKIM/DMARC) checks out —
   sending yourself a message to a big provider and
   viewing its authentication results is a real,
   mortal-scale test.
7. **Write down the rollback before you need it.**
   The old values, from step 1, where your future
   tired self can find them. Rolling back is just
   another change — with the same TTL physics, so
   a rollback decided late lands late.
8. **Raise TTLs back when it's stable** (if you
   lowered them). Optional, tidy, and the step
   whose omission costs nothing but a few extra
   queries — the only forgiving step on this list.

## A note on timing

Make changes when you can watch them: morning of a
day you're around, not the Friday-evening before a
trip. Most DNS edits are five minutes of work and
an hour of verification; the failures that hurt
are the ones discovered by somebody else, three
days later, in a different time zone, in the mail
queue.
