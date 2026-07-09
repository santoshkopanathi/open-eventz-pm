# Competitive & Channel Analysis — Open Eventz

The real competition for Open Eventz isn't just other apps — it's every existing way a parent currently finds this information, including ones that aren't apps at all. This doc looks at five of those channels: Macaroni Kid, ParentPass, individual city websites, library websites, and the physical bulletin boards/flyers posted at venues themselves. (Read "bulletin boards" below as covering the flyer-on-the-wall, sign-on-the-door category generally — flag if a different physical channel was intended.)

## At a glance

| Channel | Good at | Not good / helpful at | Structural challenge |
|---|---|---|---|
| Macaroni Kid | Personal, locally-written voice; free; consistent weekly cadence | No discoverable native app right now; weekly, not real-time; not really searchable or filterable | Quality depends entirely on how engaged the local franchise owner is — wildly inconsistent metro to metro |
| ParentPass | Real native app on iOS and Android; push notifications; built-in parent community/chat | Coverage visibly thins the further you get from its home base | Built outward from one community (Tarrant County) — depth doesn't travel evenly across a multi-county metro |
| City websites (parks & rec, city calendars) | Authoritative and official; accurate when current | Each city is its own separate domain and platform; seasonal program catalogs are often flipbook-style PDFs; no "free" filter; no cross-city search | Every city government runs its own site and process independently — nothing ties neighboring cities together |
| Library websites | Generally accurate; programs are usually genuinely free; decent internal event search | No shared platform across library systems even in the same metro; a library's own policy documents can disagree with each other | Each library system independently picked its own software and writes its own policy docs — no cross-library standard |
| Bulletin boards / flyers at venues | Often the freshest, most hyperlocal source; surfaces things with zero online footprint at all | Not searchable, not remote, easy to miss, impossible to plan around in advance, no reminder mechanism | The information only exists in the physical world — this is the literal "last mile" of event discovery |

## Macaroni Kid

What it's good at: it reads like a real local parent wrote it, because one did — each metro's edition is run by a local franchisee, which gives it a kind of trust and texture an algorithm-driven feed doesn't have. It's free to read, arrives on a predictable weekly cadence, and the Frisco-area edition is actively publishing seasonal content (we saw a 2026 summer camp guide from it directly).

What it's not good at: there's no evidence of a currently maintained, store-listed mobile app — the only mobile-app reference we found was a single old franchise blog post from around 2017, with nothing suggesting it's still active. So the actual product is a website plus an email newsletter, which means no push notifications, no in-the-moment "what's happening today" lookup, and no real filtering by free/age/distance — you're reading a curated list, not querying a database.

Structural challenge: because each metro is run by an independent franchise owner, quality and update frequency aren't standardized — one city's edition might be excellent and another's barely maintained, and there's no way to know which you're getting until you're already reading it.

## ParentPass

What it's good at: this one's a real, live, well-built app — it's actually on the App Store and Google Play right now, has push notifications, and layers in a community/chat feature so parents can connect with each other, not just browse a list.

What it's not good at: its actual depth is uneven across geography. It markets itself as built for North Texas broadly, but it was developed with families in Tarrant County specifically, and its own FAQ says explicitly that Tarrant County users are its top priority. Pulling its public reviews backs this up — the overwhelming majority of glowing testimonials name Fort Worth specifically.

Structural challenge: Tarrant County (Fort Worth) and Collin County (Plano/Frisco) are roughly 30-40 miles apart with entirely separate city governments, library systems, and school districts. An app that grew organically from one community doesn't automatically inherit equal depth in another, even within the same loosely-defined "North Texas" umbrella — coverage strength tends to radiate outward from wherever the founding community actually was, and thins out the further you get from it.

## Individual city websites (parks & rec departments, city event calendars)

What they're good at: this is about as authoritative as it gets — if a city's own parks department publishes it, it's coming from the source, not a secondhand aggregator, and it's generally accurate while current.

What they're not good at: we ran into this directly researching Plano and Frisco, two adjacent cities. Frisco splits its presence across two separate domains (friscotexas.gov and a separate townoffrisco.com calendar), and Plano's seasonal recreation catalog is published as a flipbook-style PDF — readable, but not something you can search, filter, or pull structured data from easily. Neither city's site has any concept of a "free" filter, and obviously neither links to the other, so a parent checking both cities is manually visiting two unrelated websites built on two different platforms with two different conventions.

Structural challenge: there is no shared standard across city governments — each city runs its own website, on its own platform (Frisco and Plano are both on different setups), updated on its own schedule, with no obligation or incentive to make their data easy for an outside party to consume. The fragmentation isn't a bug, it's just how independent municipal websites work.

## Library websites

What they're good at: library-run events are about as close to "verified free" as you can get — there's essentially no hidden cost model here — and the events themselves are usually well-organized and searchable within the library's own site.

What they're not good at: here's the part that's easy to miss until you actually compare two systems side by side — Frisco's library runs on the BiblioCommons platform, while Plano's runs on a completely different one (Communico). Two adjacent cities, two unrelated calendar systems, no shared format. And it's not just a platform problem: when we looked into Frisco library's own supervision policy for kids, we found the city's own legal code says one age threshold and the library's own published Summer FAQ states a different one — meaning even a single library's own documentation isn't always internally consistent, let alone consistent with the library three towns over.

Structural challenge: library systems are run independently by city or county, and each one separately chose its own software vendor and writes its own policy documentation. There's no industry-wide push toward a shared, queryable format the way there might eventually be in other sectors — so cross-library consistency would have to be built by something external to the libraries themselves (which is, not coincidentally, exactly the role an aggregator like Open Eventz could play).

## Physical bulletin boards and flyers at venues

What they're good at: this is honestly an underrated channel — a flyer taped to a library door or a sign posted at a community center is often the single freshest, most hyperlocal source there is, and it sometimes surfaces things that never make it online at all (a one-off church group activity, a pop-up event a small nonprofit didn't bother to post digitally).

What they're not good at: everything that makes it useful in person makes it useless remotely. It's not searchable, you can't filter it, you can't plan around it in advance, there's no reminder, and it's trivially easy to miss simply by not walking past that particular wall that week. There's also a freshness-in-reverse problem: an outdated flyer can sit there for weeks after the event it advertises has already happened, with nothing forcing it to be taken down.

Structural challenge: this information has never existed anywhere except physically, on that one wall, in that one building. It's the most extreme version of the fragmentation problem every other channel has — there's no website to scrape, no platform to integrate with, nothing but a person walking by and happening to look up. Digitizing even a fraction of this is genuinely hard, but it's also exactly the kind of thing that compounds in value the longer Open Eventz exists, since it's the one category competitors structurally can't touch with a scraper.

## What this adds up to

Every channel above is good at exactly one thing and structurally incapable of the rest. Macaroni Kid has voice and trust but no real-time product. ParentPass has a real app but uneven geographic depth. City and library websites have authority and accuracy but zero cross-source consistency, even between neighboring cities. Bulletin boards have freshness and hyperlocal reach but no way to travel beyond the room they're posted in. None of them stitch together, none of them agree on what "free" means or who needs to be there to supervise, and none of them are accountable to anyone for getting better at any of this over time. That gap between five different partial solutions and one coherent answer is, in plain terms, the actual product opportunity.

---
*Feeds into: Open Eventz PM OS, Section 2 (Market & Competitive Landscape) and Section 3 (Data Sourcing Strategy).*
