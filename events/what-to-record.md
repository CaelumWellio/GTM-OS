# What to record at the next event

Five fields, filled in on the event row in `events/data/events.csv` within a week of the event. This is the cheapest thing in the whole build and the only part that compounds: every field below is one line of typing that makes the next year's answer better than this year's. Everything else in this folder is a reading of what already happened and cannot be improved by working harder at it.

Source: `events/data/events.csv`; `events/returns.md`.

## The five fields

**Who attended from Wellio, by role and by day.** Not names, roles and a count, plus how many days each was on the stand. It matters because the true cost of an event is the sponsorship fee plus the people, and the workbook holds only the fee, so every cost per win in `events/efficiency.md` is understated by an amount nobody knows. What it would have told us this year: whether the $25,000 event that returned 0.77 times was under-worked or fully staffed, which is the difference between a room that does not convert and a room nobody worked.

**Whether the delegate list arrived, when it arrived, and how many names it carried against the expected room size.** Three numbers, one line. It matters because all five delegate list policies found in the research are opt-in, so a list is the share of the room that consented and never the room. What it would have told us this year: what "Delegate List: Yes" actually meant on the 7 measured events that had one. Forty names out of 400 and 380 out of 400 are the same entry in the workbook today, and they are not the same purchase.

**How many conversations were had, counted as named schools spoken to.** A tally kept at the stand, not a recollection written up afterwards. It matters because it is the only number that sits between the size of the room and the deals that came out of it, so without it every event with a poor return has two possible explanations and no way to choose between them. What it would have told us this year: why one event produced 14 deals and no wins. Fourteen conversations converted badly is a message problem; four hundred conversations that yielded fourteen deals is a room problem, and today those look identical.

**The campaign name used, written on the event row at the moment the campaign is created.** Not reconstructed later. It matters because the whole join between an event and its money runs through this one string. What it would have told us this year: the workbook's performance column overstates wins by about 24 across nine events, one event claims 8 wins and holds none, and two more claim deals with no campaign behind them at all, so there is nothing to check the claim against. Every one of those is a naming problem, not a measurement problem.

**One campaign naming convention, used without exception.** Use `<market> <year> - <event key>`, with an optional ` - <variant>` on the end, and take the event key verbatim from the `event_key` column in `events/data/events.csv`. The portal is already close to this shape in places, which is why it is the shape to standardise on rather than something new. It matters because a convention is what lets a name be matched by a rule instead of by a person. What it would have told us this year: `events/data/campaign-aliases.csv` would not need to exist, and the two campaigns that look like an event but match no event row would have matched on their own.

Source: `events/data/events.csv`; `events/data/campaign-aliases.csv`; `events/returns.md`; `events/data/research.csv`.

## What this buys

Twelve of 151 events can be measured at all today, and eight of those carry a return you can divide. Every event recorded this way adds one to both counts. At around a dozen more, the two dimensions in `events/scoring.md` that currently rest on logic rather than evidence, access and voice, become testable for the first time, and the rubric stops needing to say so out loud.

Source: `events/scoring.md`; `events/returns.md`.
