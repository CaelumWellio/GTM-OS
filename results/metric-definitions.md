# Metric definitions

| Metric | Definition | HubSpot carrier | Measurable today? |
|---|---|---|---|
| Qualified reply | A human reply from a school contact to an outbound email or sequence that is not an auto-reply or unsubscribe | Email engagement, direction INCOMING_EMAIL, `hs_sequence_id` on the parent thread | Partly: replies yes, link to sequence only where enrolled via HubSpot |
| Meeting booked | A lead reaching a meeting-booked stage | Lead `date_first_entered_meeting_booked` | Yes |
| Meeting held | A booked meeting that took place | Meeting `hs_meeting_outcome` | No: outcomes not recorded (70% still SCHEDULED) |
| Pipeline created | A Sales Pipeline AU deal created from a lead | Deal `createdate`, `lead_ids` | Yes |
| Pipeline won | Deal `hs_is_closed_won = true` | Deal | Yes |
| Hook attribution | Which hook a school responded to | Not present. Requires `hook_id`, `persona`, `variant` properties on enrolment or deal | No: prospective stamp waits for the decision |

Source: ~/Documents/GTM Project/docs/revops/2026-09-03-sdr-funnel-leak-analysis.md, section 0; outputs/sdr_funnel field lists.
