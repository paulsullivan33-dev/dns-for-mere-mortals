# What DNS Is

**Scope:** the one system underneath everything
else — what it does, and why its failures never
stay small.

## The phone book that isn't a book

Every machine on a network is reached by an **IP
address** — `192.0.2.80`, a number. Humans are bad
at numbers and good at names, so the **Domain Name
System (DNS)** exists to translate:

```
www.example.test  →  192.0.2.80
```

That translation is all DNS does. It holds no web
pages, carries no mail, moves no files. It answers
one kind of question — *what's the address (or the
pointer, or the proof) for this name?* — and yet
almost nothing works without it, because almost
every other system starts by asking it something.

The phone-book comparison is useful but wrong in
one important way: there is no book. DNS is
**distributed** — millions of servers each hold a
small piece of the whole, and a lookup is assembled
on the fly by asking several of them in turn. There
is no master copy to consult, and no single machine
whose failure empties the system. The pieces are
held by whoever owns them: your domain's records
live on *your* chosen DNS host's servers, and the
rest of the world finds them by following pointers
(see the Hierarchy chapter).

## One lookup, end to end

You type a name into a browser. Before any page
exists, roughly this happens:

1. The browser asks the operating system, which
   asks a **resolver** — usually your router, your
   ISP's DNS service, or a public one.
2. If the resolver answered this question recently,
   it replies from **cache** and the story ends in
   a millisecond.
3. Otherwise the resolver goes hunting: it asks a
   **root** server *"who handles `.test`?"*, then
   asks the `.test` servers *"who handles
   `example.test`?"*, then asks `example.test`'s
   own servers the real question.
4. The final answer comes back, the resolver caches
   it for a while (the **TTL**), and your browser
   connects to the address it contains.

Steps 2–4 are the whole subject of this guide: the
hierarchy, the resolvers, the records, the caching.

## Why it breaks everything

There's an old operations joke — *it's always
DNS* — and it survives because DNS outages wear a
disguise. When a domain's DNS fails,
the website is *up*. The mail server is *up*. The
certificates are fine. But nobody can find any of
it, so from the outside the failure is total, and
every error message blames something else: browsers
say the server doesn't exist, mail bounces with
lookup failures, apps report connection errors.

The practical consequence shapes this whole guide:
**when everything fails at once, suspect the map,
not the territory.** Check DNS early, check it with
real tools (`dig`, chapter 6), and check it from
more than one resolver — because thanks to caching,
two people can honestly get two different answers
to the same question, and both be "right."
