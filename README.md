- Note from the Author

I wanted to see what Claudes new model Fabel 5 was capable of so i gave claude a project to see what it is capable of. The following is a discription of the project, why cloaude cose it, and how it created what it did. 

# MERIT ORDER

A live wholesale electricity market simulator for a fictional balancing authority, the Centennial Power Pool.

## Attribution

This project is vibe coded. Henry Adams asked Claude (Anthropic's AI model) to build something that demonstrates its own capabilities, gave it almost no direction beyond that, and Claude designed, wrote, and tested the entire thing in one sitting with no human writing or reviewing code line by line during the build. Henry did not author the code. What follows is Claude's own explanation of what it did and why, written in the first person because it is describing its own decisions, not Henry's.

## The prompt, as given

Henry offered two options: rebuild his existing portfolio work, or build something of Claude's own choosing from scratch to show off what it can do. He leaned toward the second option and said, essentially, just go, no further input needed. Everything below, the choice of project, the design, the modeling decisions, the testing approach, was Claude's call, not something Henry specified.

## Why I chose this project

I could have defaulted to something generic and safe to demo: a game, a data dashboard with placeholder numbers, a chat interface. Those look impressive to almost anyone but can't actually be evaluated for correctness by someone who doesn't already know the answer. That felt like a weak test of capability, closer to a magic trick than a demonstration.

Henry works as an Energy Scheduler at Xcel Energy in the WECC wholesale power markets, handling NERC tagging, OASIS transmission scheduling, and energy purchase and sale transactions. That meant I had access to something rare in a demo like this: an evaluator who actually knows the domain and can tell the difference between a simulator that looks like a power market and one that behaves like one. So I chose to build a wholesale electricity market simulator specifically because it is the one project where Henry could grade the substance, not just the surface polish. I picked the harder, more falsifiable test on purpose.

## What research and reasoning went into it

I did not have access to Henry's proprietary work systems or real WECC market data, and nothing here is drawn from Xcel Energy's actual operations or any real balancing authority. Instead I built the simulator from general, publicly known principles of how wholesale electricity markets are structured, which are well established in the industry:

- Wholesale markets clear through a merit order: generating units are dispatched from cheapest marginal cost to most expensive until supply meets demand, and in a uniform price auction, everyone gets paid the price of the last, most expensive unit needed.
- Real generating units cannot change output instantly. They have physical ramp rates, so a unit's usable capacity in the next few minutes depends on where it is now and how fast it can move, not just its nameplate size.
- When available supply gets close to demand, prices do not rise smoothly. Grid operators use scarcity pricing mechanisms so that prices spike as reserves thin out, which is what makes losing a big unit or hitting a heat wave actually expensive rather than a rounding change.
- Wind and solar are typically dispatched as price takers, often bidding at or below zero because of production tax credit economics, and get curtailed first when there is more renewable output than the grid can use.
- A carbon price changes the relative marginal cost of coal versus gas generation and, past some threshold, can flip their order in the dispatch stack. That crossover point is a real and well known dynamic in markets with a carbon price.

I used those principles, which reflect how power markets generally work rather than any specific real system, to design a fictional nine unit fleet (nuclear, coal, hydro, two gas combined cycle plants, three gas peaker plants, and an import tie), fictional cost curves, and a fictional load and weather model, all invented for this simulator and none of it copied or reverse engineered from a real utility.

## What it actually does

It is a single node, uniform price, real time electricity market that runs continuously in the browser once opened. Every simulated minute, it:

- Builds a supply stack from nine dispatchable generating units plus wind, solar, and a battery, each offering into the market at its own marginal cost.
- Restricts each unit's offer to what it can physically ramp to in time, so the market cannot instantly reshuffle around a shock.
- Clears the market by stacking offers from cheapest to most expensive until they meet demand, and sets the price at the last unit needed.
- Applies a scarcity price adder that escalates as reserves thin below ten percent, with a hard price cap and simulated firm load shedding if supply genuinely cannot keep up.
- Lets wind and solar bid in as negative priced, weather driven resources, with economically ordered curtailment when there is too much renewable output for demand to absorb, which can also push prices negative.
- Runs a battery that decides on its own when to charge and discharge based on price and reserve conditions, with no scripted schedule.
- Lets the carbon price be adjusted with a slider, which reprices coal against gas in real time and can flip their position in the dispatch order past roughly twelve dollars a ton in this model.
- Lets the user trigger grid stress events (a heat dome, a wind lull, a cloud front, a transmission tie derate) or manually trip a generating unit to watch the market respond.

All of this is rendered live across three hand drawn canvas charts (the bid stack, a twenty four hour price history, and a twenty four hour generation mix), in a single self contained HTML file with no external libraries and no build step.

## How I checked it was not just decorative

Before treating it as finished, I pulled the market engine code out of the page and ran it by itself, without the visuals, through a scripted set of test scenarios with a fixed random seed so the results are repeatable, and I checked the actual numeric outcomes rather than just watching the charts look plausible:

- A calm day clears near the cheapest generator's cost overnight and climbs into the peaker price range during the evening demand ramp, without triggering scarcity pricing or shedding any load.
- A heat dome event pushes reserves below the scarcity threshold and prices respond as designed.
- A heat dome combined with a wind lull pushes the market all the way to its price cap.
- Manually tripping a large unit during the evening ramp produces a real, several times larger price spike, not a small blip.
- Raising the carbon price to forty dollars a ton measurably shifts generation away from coal and toward gas.
- Overbuilding renewable capacity drives midday prices negative and produces real curtailment.

All of those checks passed before I called it done. I am including that detail because a simulator that only looks right when someone is casually clicking around is not actually proof of anything. Running it headless and asserting on outcomes is what separates a good looking dashboard from a market that behaves like a market.

## What I am least certain about

I would specifically want someone with real WECC experience, meaning Henry, to sanity check a few things I could not fully validate on my own:

- Whether the ramp rates and lookahead assumptions I chose produce price spike sizes that feel realistic for a system at this scale, since I set those values from general knowledge rather than calibrating them against real fleet data.
- Whether the scarcity pricing curve I used, which rises with the square of how far reserves are below ten percent and caps at two thousand dollars, is a reasonable simplified stand in for how real operator defined reserve demand curve pricing behaves, or whether it is a rough caricature of it.
- Whether the battery's charge and discharge price thresholds are dispatching at points that would actually make economic sense, or just happen to look right on the chart.

## One open question for Henry

Do you want this README kept in its current framing, aimed at someone evaluating what Claude can do and how it approached the problem, or would you rather it be rewritten more neutrally as plain documentation of the tool itself, with the vibe coded attribution kept but the explanatory tone toned down? I defaulted to the fuller explanation since that is what you asked for here, but I want to check before treating that as the final version.
