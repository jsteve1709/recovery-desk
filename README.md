# Recovery Desk

**A prioritised vehicle-recovery queue for an auto lender's collections team.**
Independent concept built on synthetic data. Not affiliated with any company.

## The problem
An auto lender's collections team holds hundreds of delinquent vehicles and can visit only a few dozen a day. Most teams work the list oldest-dues-first. That ignores the question that decides whether a visit succeeds: can we actually find this vehicle today? Connected-vehicle data answers that question, but a collections manager needs it as a short, ranked list with a reason for each row, not as a map of raw GPS pings.

I worked a version of this problem by hand. During my internship at an industrial equipment maker, I diagnosed Rs 1.49 crore of overdue receivables through 30+ finance and sales interviews, segmented 40+ accounts by risk and ageing, and set up escalation workflows. The team recovered Rs 52 lakh in 45 days. The lesson was that recovery is mostly a prioritisation problem. Recovery Desk is what I would want as a product for the same job with vehicles.

## User and job to be done
**Primary user:** collections manager at an auto or two-wheeler lender, planning tomorrow's field visits for 3 to 10 recovery agents.
**Job:** "Tell me which vehicles my agents should go to today, why, and what to do when they get there."
**Secondary user:** the recovery agent, who needs a one-line brief, not a dashboard.

## What "working" means
| Metric | Role | Definition |
|---|---|---|
| Rs recovered per agent-day | North star | Value of vehicles recovered divided by agent-days spent |
| Visit hit rate | Driver | Share of visits that end in a recovery |
| Wasted visits | Driver | Visits where the vehicle was not at the expected location |
| Complaints and escalations | Guardrail | Must not rise; visits only between 8 am and 7 pm, in line with RBI's conduct rules for recovery agents |

## What the prototype does
`recovery_desk.py` (plain Python, no dependencies):
1. Generates a **synthetic** portfolio of 500 delinquent vehicles with loan data (days past due, amount outstanding) and telematics signals (hours since last ping, km driven in 7 days, night-parking consistency, ignition events).
2. Scores each vehicle on **expected value recovered**: how findable it is (ping freshness, predictable parking) times how recoverable (dues age, mobility) times amount outstanding.
3. Writes today's ranked queue with an **agent brief** for each vehicle (`data/queue_today.csv`). Briefs are rule-based in this version; the next version generates them with an LLM.
4. Runs a **simulated 10-day pilot** comparing the Recovery Desk queue with the oldest-dues-first queue (`results.md`).

## Simulated pilot result
3 agents, 6 visits each per day, 10 days, same 500 vehicles:

| Queue | Hit rate | Rs lakh recovered | Rs lakh per agent-day |
|---|---|---|---|
| Oldest dues first | 38% | 342.5 | 11.42 |
| Recovery Desk | 49% | 578.7 | 19.29 |

**Read this honestly.** The simulated outcomes are driven by signals similar to the ones the score uses, so the lift shows how the pilot would be measured, not what a real lender would see. The real test is the pilot plan below.

## What I would build next (prioritised with RICE)
| Feature | Reach | Impact | Confidence | Effort | Why this rank |
|---|---|---|---|---|---|
| LLM-written agent briefs with local-language option | High | Medium | High | Low | Agents act on briefs, not dashboards |
| Feedback loop: agent marks found / not found / recovered | High | High | High | Medium | Turns every visit into training data for the score |
| GPS-dark alert (device tampering or removal) | Medium | High | Medium | Medium | Dark vehicles are the costliest misses |
| Route batching of nearby vehicles | Medium | Medium | Medium | High | Useful once ranking is trusted |

## Pilot plan with a real lender
1. **Discovery first:** interview 5 collections managers and 5 field agents on how they choose visits today and why visits fail.
2. **A/B by agent team for 4 weeks:** half the teams use the Recovery Desk queue, half keep the current list.
3. **Decide on the north star:** ship if Rs recovered per agent-day improves with no rise in complaints; otherwise study the misses and retune.

## Risks I would manage
- **Borrower dignity and compliance:** location data is used only to plan lawful, daytime visits; briefs never encourage pressure tactics.
- **Data access and consent:** use only telematics the lender is entitled to under the loan agreement.
- **Fairness:** monitor whether ranking concentrates visits on particular areas or groups.

## Run it
```
python recovery_desk.py
```

*Built by Joshua Steve J (MBA, IIM Indore) with AI pair-programming, which is how I like to prototype: fast, then measured.*
