---
title: "Santa-Claude: using Claude Code to run Christmas for four kids"
permalink: /projects/santa-claude/
excerpt: "Wishlists, budgets, price watches and a shopping basket, all handled by Claude Code, so Christmas for four kids stays fair and on budget."
categories: [projects]
tags: [claude-code, ai, christmas, family]
header:
  teaser: /assets/images/projects/santa-claude/teaser.svg
---

Christmas with four kids is a logistics problem. Two teenagers and a pair of twins whose birthday lands just before
Christmas means four wishlists, two sets of birthday presents, a few dozen parcels from a dozen shops, and one rule that
matters more than any other: **it has to be fair**. Nobody should open noticeably less than their siblings.

In past years that meant a spreadsheet I kept by hand. This year I handed the admin to [Claude Code](https://claude.com/claude-code) and called it Santa-Claude. I still choose the
presents. Santa-Claude keeps track of them.

> This page deliberately doesn't say who is getting what. The kids know where this site lives.
{: .notice--info}

## Where the wishlists live

The kids keep their wishlists on [GiftList](https://giftlist.com/), in a family group. Each item has a price and a link
to the shop, and they add to it all year.

GiftList needs a sign-in, which Claude Code won't do for you, and quite rightly. So it opened GiftList in the
desktop app's built-in browser, I signed in myself, and it took over from there. It read all four lists and copied every
item into a tracker, about thirty items in all, each with its price and a link back to the original listing.

## The tracker

The tracker is a spreadsheet that Claude Code built as a private page on claude.ai. I can open it from anywhere, and Claude Code can update it while we talk.

- **Gifts tab.** One row per present: child, occasion (Christmas or birthday), status, price, where to buy it and notes.
  Status runs **Idea → To buy → Ordered → Bought → Wrapped**.
- **By child tab.** Each child's planned spend, what's left of their budget and how many presents they have. Christmas
  and the twins' birthday have separate tables, because their birthday budget is kept apart from Christmas.
- **Totals at the top.** The overall budget, how much is planned, how much is bought and how much is left.

The useful thing is that I never edit it by hand. I just say what I want:

- "Remove the Lego, he doesn't really use it anymore."
- "Move all the manga volumes to his birthday."
- "Raise everyone's budget a bit and fill the gaps."
- "That one's arrived." *(marks it Bought)*
- "Money-wise, how much has been bought and how much is left?"

Each change comes back with a little table of the four kids' totals, so I can see straight away whether it has knocked
the balance out.

## Keeping it fair

Fairness is the part a plain spreadsheet never helped with. The kids' wishlists are wildly uneven: one asks for a few
expensive things, another for lots of cheap ones. So the plan became:

1. **Start with their own lists.** Whatever they asked for goes in first.
2. **Add one bigger main present each.**
3. **Fill the rest of each budget** with ideas that fit that child's interests, until all four are within a few pounds of
   each other.

When I changed my mind about a main present (and I did, several times), Santa-Claude rebalanced everything else, pointed
out who was now over or under, and suggested what to add or take away. It also spotted problems I hadn't thought of. When
one present needed another present to work, and the two were due on different days, it pointed out that he'd be opening
something on his birthday with nothing to plug it into until Christmas.

## It remembers the house rules

Claude Code has a memory that carries between sessions, and that's what turned it from a clever spreadsheet into
something that actually knows the family. Over a week it picked up rules like these:

- We're an **all-digital family**, so no DVDs or Blu-rays. Suggest digital films or gift cards instead.
- **Christmas pyjamas** are a separate Christmas Eve present for everyone, so keep them off the lists.
- What devices each child already has, so it doesn't suggest a second one.
- Who has a cinema membership (so no cinema gift cards), who already owns a particular book, and who's small for their
  age, so check kids' sizes for clothes.
- One of them is a good badminton player, so suggest mid-range kit, not a beginner's racket.
- We're going skiing in February, so ski kit makes a good present.

Every one of those started as a correction: it suggested something they already had, or got a detail wrong, and I told
it. Once said, it stayed said. It even got the twins' age wrong once, by assuming they'd already had the birthday that's
coming up. It fixed the sheet and its notes as soon as I pointed it out.

## Watching the prices for Black Friday

The expensive presents were worth waiting for. Santa-Claude set up a **scheduled task** that checks the big-ticket items
twice a day, at 8am and 6pm, from mid-November until the day after Cyber Monday.

- It starts from the prices in the sheet.
- It only notifies me when something drops by **at least 10% or £15**, and is lower than any drop it has already told me
  about, so I don't get a stream of 50p alerts.
- Anything marked **Bought** comes off the watch.

When I asked "when is Black Friday?", it also pointed out that UK shops often start their deals a week or more early,
which is exactly what the watch was set up to catch.

## Shopping help, with me at the checkout

Santa-Claude does a fair amount of the legwork:

- **Comparing options.** For a badminton racket, it compared models and current prices across UK shops, explained the
  difference between head-heavy and head-light rackets, and picked one that suited a good player.
- **Checking delivery dates.** A T-shirt from a US print-on-demand shop looked perfect, until it read the shop's own
  shipping page: up to 30 working days to the UK, and import fees on arrival. It found an officially licensed UK
  alternative arriving that week instead.
- **Refurbished options.** For the bigger electronics it looked at refurbished sellers, compared grades and storage
  sizes, and kept both children's matching presents matching when I changed the spec.
- **Spotting rules.** When I added a pocket knife for one of the twins, it pointed out that UK shops won't sell knives to
  under-18s and what's legal for a child to carry.
- **Filling the basket.** Signed in to Amazon in the built-in browser, it can add items to the basket, using the exact
  listings from the kids' wishlists. It stops before checkout every time. **I place every order myself.**

It also says when it's guessing. Ideas it came up with from memory are labelled as rough prices, not checked live, and
anything it isn't sure of (a model number, whether a headset works with an older console) comes with a note saying so
and what to check.

## Spoiler-safe views

With four kids in the house, the tracker is a minefield. One evening I wanted one of the twins to help me check his
brother's list, so I asked for a view of that list **without prices and without the main present**. Santa-Claude made a
clean page with just the gift names, and even pointed out that one remaining line (a case for the main present) would
give the game away.

## What I learned

- **Let it do the bookkeeping, not the choosing.** It's great at suggesting ideas, but the best presents still came from
  knowing the kids. Where it shines is keeping thirty-odd items, four budgets and two occasions straight.
- **Correct it once and it remembers.** The memory is what makes it feel like a helper rather than a search engine. There's a
  lot of "no, she's already got that" at the start, and much less of it after.
- **Keep the money in your own hands.** Sign-ins, payments and checkouts stay with me. Claude Code is careful about that
  anyway, and it's the right split.
- **Ask for the totals often.** "How much has been bought vs how much is left?" is the question I ask most, and it's the
  one that used to take an evening with a calculator.

Santa-Claude hasn't made the presents any cheaper. But I know exactly where we are in late November, and that nobody is going to open noticeably less than their siblings.
