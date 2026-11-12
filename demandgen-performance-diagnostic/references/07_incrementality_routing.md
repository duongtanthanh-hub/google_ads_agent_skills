# 7. Incrementality routing ("Is Demand Gen working?")

Platform diagnostics explain **what** Demand Gen reports. They can't show
whether those sales would have happened anyway. When the question is about
incrementality, pausing or scaling Demand Gen as a channel, do this step and
hand over.

## 7.1 Triggers

Any of: "is Demand Gen incremental / worth it / working", "should we cut Demand
Gen", "Demand Gen is unproven", "these are people who'd buy anyway", or a plan
to pause all Demand Gen.

## 7.2 Grade the evidence already offered (MEAS-10)

Evidence offered                                                                                                    | Grade                                                    | Why
------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ---
Conversion Lift study (user- or geography-based) or a Google Ads campaign experiment with a holdout, run per CHG-05 | PASS (can support a decision)                            | Randomised control; reports incremental conversions with intervals
Advertiser-run geo experiment (matched markets, pre-registered KPI, power checked, ≥ CHG-05 duration)               | PASS if designed per `../test-design-and-power/SKILL.md` | Controlled at geo level
Pre/post with CausalImpact using unaffected control series, enough pre-period, placebo checks                       | WARN (supporting evidence only)                          | Quasi-experimental; sensitive to control choice and other changes
**Clickers vs non-clickers** (comparing buyers who clicked a Demand Gen ad with those who didn't)                   | BLOCK as evidence                                        | Selection bias: people who click are already more interested; the comparison measures who clicks, not what the ad caused
**Short on/off test** (e.g. 4 days on, 4 off)                                                                       | BLOCK as evidence                                        | Shorter than CHG-05; inside the ramp-up window; conversion lag pushes "on" sales into "off" days; weekday mix differs; carry-over from earlier exposure
**Tiny holdout** (e.g. 3 states or regions, no power calculation)                                                   | BLOCK as evidence                                        | Too few units to detect a realistic lift; regional noise dominates
Platform-attributed conversions or ROAS alone                                                                       | BLOCK as evidence of incrementality                      | Attribution isn't causation (this also cuts the other way: low attributed ROAS doesn't prove zero lift)

"BLOCK as evidence" means: don't base a pause, scale or "unproven" conclusion on
it. It doesn't block running Demand Gen.

## 7.3 What to report

-   Verdict for incrementality: **NOT_ASSESSED** unless PASS-grade evidence
    exists. "Unproven" means "not yet measured properly", not "no effect".
-   Say what the platform data does show (from steps 4–8), labelled as
    attributed, not incremental.
-   Don't recommend pausing all Demand Gen on weak evidence, and don't claim
    it's incremental from attributed conversions. A scale or stop decision
    justified only by an observational comparison is a BLOCK (DG-12).
-   Recommend a proper test and hand over to
    `../lift-and-geo-experiments/SKILL.md` (Conversion Lift user- or
    geography-based, campaign experiment, or a geo experiment), with
    `../test-design-and-power/SKILL.md` for sizing and duration, and
    `../causal-impact-analysis/SKILL.md` only when a randomised test isn't
    possible. Longer term, `../mmm-meridian/SKILL.md` can use the test result as
    a prior.

## 7.4 Pre-conditions to fix before a test (from this diagnostic)

A test on a broken setup wastes the holdout. Before handing over, list:

-   Measurement: purchase is the primary goal (MEA-01), values are real (MEA-02,
    MEAS-01), and the KPI (warehouse revenue or Ads value) is agreed.
-   Delivery: the campaign isn't starved (DG-02) and isn't in repeated learning
    (step 6).
-   Bidding: a click strategy on a sales goal makes the test answer "do cheap
    clicks drive sales?", not "does Demand Gen drive sales?". Say which question
    the test will answer.
-   Season: avoid starting a test in the week before Black Friday unless the
    question is specifically about the peak; peaks raise noise and the needed
    sample.
