# SkyWave Direct — NovaIVR South Pilot: Diagnostic Note

## Recommendation

**Do not approve national expansion. Keep the pilot in the South, treat the June ticket fall as unproven, and gate any rollout on (a) fixed authentication and (b) a measured cross-channel repeat-contact rate.**

The observed June improvement in CareDesk tickets is real but its cause is unproven, and roughly half the mechanism is recording, not resolution. Three observed facts drive this: (1) South network-fault exposure in June (8 events, 7,045 accounts affected) was *higher* than the pre-pilot April baseline (6 events, 5,723 accounts), yet recorded complaints fell by half; (2) case creation inside the IVR collapsed from 49.5% of sessions (LEGACY) to 3.3% (NOVA-S1), and 24.0% of NOVA sessions (650/2,709) end before authentication — a population the containment dashboard excludes by design per Farah Iqbal's 18 May email; (3) repeat calling inside the IVR nearly quadrupled versus legacy (57.1% vs 16.3% of subscribers with >1 session), and 208 of the 390 authentication-drop subscribers who called back failed authentication again on their next call. There is genuine self-service deflection for routine queries (balance_check 29.1%, pack_info 21.3% of sessions — stages with no legacy equivalent), but that cannot explain the fall in NO_SIGNAL tickets, which need field resolution, not IVR menus. One complete pilot month (June) is too thin a base for a national decision.

## Evidence Table

Observed facts are marked **[F]**, inferences **[I]**, causal claims **[C]**.

| # | Claim | Type | Source | Confidence |
|---|-------|------|--------|------------|
| 1 | The containment dashboard deliberately excludes sessions ending before authentication ("starts after a subscriber has been identified"), and Technology still had open "authentication and prompt-timing items" as of 18 May. | [F] | `email-ivr-pilot.eml`, "For clarity" paragraph and closing line | High |
| 2 | South CareDesk tickets: 175 (Jan) → 198 (Mar peak) → 184 (Apr) → 157 (May) → 104 (Jun). Control regions flat: E 101→102, N 168→170, W 176→169 (Jan→Jun). | [F] | `tickets.csv`, monthly counts by `region` (3,541 rows) | High |
| 3 | South NO_SIGNAL tickets: 62 (Apr) → 51 (May) → 28 (Jun); NO_SIGNAL also fell in E (33→23) in June. The South decline exceeds the region-wide trend by roughly 15–20 tickets. | [F]/[I] | `tickets.csv`, `issue_type=NO_SIGNAL` by month/region | High |
| 4 | South network faults did not fall: 6 events / 5,723 accounts affected (Apr, all legacy), 10 events / 9,409 (May, incl. 1 HIGH affecting 4,171), 8 events / 7,045 (Jun, full NOVA). June fault exposure ≥ April while recorded complaints fell ~45%. | [F] | `service_events.csv`, `region=S`, monthly `affected_accounts_est` | High |
| 5 | NovaIVR ran South-only from 10 May (May: 1,150 NOVA + 42 legacy sessions; Jun: 1,559 NOVA). 650 of 2,709 NOVA sessions (24.0%) end at `authenticate` — vs a 7.9% authenticate-drop rate under LEGACY-South. The rate did not improve: 23.3% (May) → 24.5% (Jun). | [F] | `ivr_interactions.jsonl`, `pilot_version`, `terminal_stage`, `started_at` | High |
| 6 | The 650 pre-auth drops are predominantly technical failures, not customer choice: TOKEN_MISMATCH 129, ANI_LOOKUP_TIMEOUT 86, PIN_RETRY_LIMIT 9, ERROR/none 78, ABANDONED 162, RESOLVED 186; median session duration 74s (callers attempted, then were blocked). | [F]/[I] | `ivr_interactions.jsonl`, `error_code`/`outcome`/`duration_seconds` at `terminal_stage=authenticate` | High |
| 7 | IVR case creation collapsed from 49.5% of LEGACY-South sessions to 3.3% of NOVA sessions (94 case_ids / 2,709), while all 21 NOVA `agent_queue` sessions ended ABANDONED. | [F] | `ivr_interactions.jsonl`, `case_id` presence; `terminal_stage=agent_queue` | High |
| 8 | The "above target" containment figure is 64.8% (`outcome=RESOLVED` over post-authentication sessions only); counting all sessions it is 56.1%. The gap is the 650 excluded sessions. | [F]/[I] | `ivr_interactions.jsonl`, `outcome` rates, both denominators; consistent with `email-ivr-pilot.eml` metric definition | High |
| 9 | Repeat contact inside the IVR rose sharply: 57.1% of NOVA subscribers had >1 session vs 16.3% under LEGACY; the 549 auth-drop subscribers generated 698 excess sessions; 208 of 390 who returned failed authentication again on their next session; 91 failed authentication 2+ times; only 13.3% of auth-drop subscribers have any May/June South CareDesk ticket. | [F] | `ivr_interactions.jsonl`, `subscriber_id` session sequences; cross-checked vs `tickets.csv` | High |
| 10 | South contact demand rose, not fell: IVR sessions 1,192 (May, mixed) → 1,559 (Jun) while tickets fell 157→104. | [F] | `ivr_interactions.jsonl` monthly counts; `tickets.csv` | High |
| 11 | Routine-query deflection is genuine: balance_check (29.1%) and pack_info (21.3%) have no LEGACY equivalent (0%), and June South IVR tickets resolve mostly as EXPLAINED 15 / PAYMENT_SYNC 12 / REMOTE_REFRESH 11 / TECH_VISIT 5 / PACK_UPDATED 4. | [F]/[I] | `ivr_interactions.jsonl`, `terminal_stage` value counts; `tickets.csv` June `resolution_code` | Medium-High |
| 12 | [C] The June ticket fall is driven substantially by channel filtering — pre-auth sessions create no case, and case-creation logic changed — with a smaller genuine-deflection component and some region-wide seasonality; it is not evidence that customer problems fell. | [C] | Inference from #4, #5, #7, #9, #10 | Medium-High |

## Rejected Hypotheses

- **"Fewer tickets = fewer customer problems."** Rejected: June fault exposure (7,045 accounts) exceeded April's (5,723) while recorded complaints halved (#4); the recording threshold changed (#7) and 24% of demand is invisible to CareDesk (#5, #1).
- **"Containment dashboard above target proves self-service works."** Rejected: the metric excludes the 650 pre-auth sessions by definition (#1, #8); 21.8% of all sessions end ABANDONED; every agent_queue session was abandoned (#7).
- **"The NO_SIGNAL fall shows the IVR resolved technical issues."** Rejected: self-service stages answer balance/pack queries; NO_SIGNAL requires remote refresh or field visits (June South TECH_VISIT = 5), and faults persisted at similar or higher levels (#4, #11).
- **"The whole South ticket decline is pilot-caused."** Rejected as overstated: NO_SIGNAL fell in all regions in June; the pilot-specific excess is ~15–20 tickets, superimposed on a region-wide dip (#3).

## Material Unknowns

1. **Cross-channel 7-day repeat contact for authentication failures.** Within-IVR repeat is observed and high (#9), but whether the 549 auth-drop subscribers then called agents, used the app, or visited dealers is unmeasured — pre-auth sessions carry no verified customer context (#1). High repeat would confirm suppression; low repeat would support genuine containment. Decision-changing.
2. **Fate of the 650 pre-auth sessions.** The email says they are "monitored in the technical pilot report" — whether anyone actions that report, and what it contains, is unknown. Decision-changing: an existing recovery queue justifies continuing the pilot; silent drop-off requires fixing authentication first.
3. **Whether the authentication defects are being fixed.** The email lists them as open items, and the drop rate was flat May→June (#5). Unknown: fix ETA and expected post-fix auth-drop rate.
4. **Durability and seasonality.** Only one complete NOVA month exists (June; the pilot began 10 May). Whether the pattern holds for a second complete month, and how much of the June dip is seasonal (visible in E), is unknown.

## Safe Next Action

**One read-only, 7-day cross-channel repeat-contact trace:** reconcile the 549 authentication-drop subscribers' phone numbers (ANI is captured in the IVR logs) against agent call logs, app, and dealer records for the 7 days after each failed session; run a sampled-call review of 30–50 of those sessions in parallel. It writes no customer record, changes no routing, and is fully reversible. Hold the national decision until the result: elevated cross-channel repeat → fix authentication before any expansion; low repeat with fixed auth → extend the pilot one more complete month, not a national rollout.

## Person to Question and Five Questions

**Farah Iqbal, Director, Customer Care Operations** — she owns the IVR journey stages, the case-creation conditions, and the containment metric definition (all three sit inside her remit; Dev Khanna's billing domain does not touch this question).

1. What are the exact numerator and denominator of the case-containment KPI on the dashboard — specifically, does "starts after a subscriber has been identified" exclude the 650 sessions that ended at `authenticate` from the denominator, and who approved that definition?
2. What follow-up, if any, does the technical pilot report trigger for sessions that end before authentication — can I get that report for 10 May–30 June, and who reviews it?
3. What exactly changed in the case-creation conditions between LEGACY (49.5% of sessions created a case) and NOVA-S1 (3.3%) — which rule, whose decision, and on what date?
4. What do the sampled-call reviews show for the 650 authentication failures and the 21 abandoned agent-queue transfers — why did subscribers hang up, and where are those sampling records?
5. What written thresholds, metrics, and observation window were agreed for the national go/no-go decision, and is completion of the authentication fix an explicit gate before that decision?

---
*Observed facts: items 1–11 in the evidence table and the numbers cited above. Inferences: the channel-filtering mechanism, the ~15–20 ticket pilot-specific excess, and the technical-failure interpretation of auth drops. Causal claim: #12 only. Unknowns: the four listed above.*
