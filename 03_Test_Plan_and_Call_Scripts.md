# Test Plan & Call Scripts: Home Credit LAP Voice Agent
**Candidate:** Upendra Rejeti

**How to use this:** each scenario has a customer persona, the lines to say (speak them naturally: add "umm", pauses and half-sentences, as the assignment asks), the expected agent behaviour, and pass criteria. Record each call, then fill in the **Result** table at the end with the recording and transcript links.

Test variable values, set in the agent's dynamic variables or the test panel:

| Variable | Value |
|---|---|
| company_name | Home Credit |
| customer_name | Rahul Sharma |
| agent_name | Priya |
| agent_gender | female |
| language_to_speak | English (Hindi for T16) |
| current_date / current_day / current_time | e.g. 12 October 2026 / Monday / 11:30 AM |
| additional_context_from_rag | see `04_Platform_Setup_Guide.md` (sample RAG block) |

Legend for the 7 items: **PT** property type · **OW** ownership · **OD** original docs · **LA** loan amount · **OI** occupation + income mode · **MV** market value · **TN** tenure.

---

## A. Core business-flow tests (must pass)

### T1: Happy path, fully eligible (natural speech)
Say:
1. "Yeah, speaking. Who's this?"
2. "Oh okay… sure, tell me."
3. "Umm, it's a flat. A residential flat in Pune."
4. "It's in my name only."
5. "Yes, the original papers are at home."
6. "I'm looking at around… forty lakhs."
7. "I'm salaried, work at an IT company. Salary comes in my bank account."
8. "Market value would be around one crore."
9. "Maybe ten years."
10. (After the recap) "Yes, that's right."

**Expected:** verify → offer → 7 items one at a time → short recap → handoff line that mentions a senior expert and exact interest rates → end call.
**Pass if:** no item is asked twice, no interest rate is quoted, the call ends after the handoff.

### T2: Agricultural property (immediate disqualification)
Say: confirm identity → "Yes, interested." → "It's agricultural land, our family farm in Nashik."
**Expected:** a polite disqualification straight away, then end. **Fail if** it asks about ownership or anything else.

### T3: Cash income (disqualification at item 5)
Fully eligible answers for PT, OW, OD, LA → "I run a kirana shop… mostly cash, honestly."
**Expected:** may clarify "majority in bank?" once → "No, mostly cash." → disqualify → end. No MV or TN questions.

### T4: No original documents
PT residential, OW sole → "I only have photocopies. The originals got lost in a flood."
**Expected:** disqualify → end.

### T5: Tenure too short
All good until TN → "Two years. I want to finish it fast."
**Expected:** disqualify (3–15 years range) → end. (A brief confirmation question first is acceptable.)

### T6: Tenure too long
All good until TN → "Twenty years." → the agent confirms → "Yes, twenty."
**Expected:** disqualify → end.
**Variant T6b:** at the confirmation, say "Oh, then fifteen is fine." → **Expected:** captures 15 → recap → handoff.

### T7: Loan amount above the limit, customer accepts 75 lakh
At LA: "I need about one crore."
**Expected:** "The maximum under this offer is seventy-five lakh. Would you like to proceed with seventy-five lakh?" → "Okay, seventy-five works." → continues to OI → … → handoff.

### T7b: Above the limit, customer refuses
At LA: "Ninety lakh." → the agent offers 75 → "No, I need at least ninety."
**Expected:** polite disqualification → end.

### T8: Joint ownership (must NOT disqualify)
At OW: "It's jointly owned with my wife and my mother."
**Expected:** accept it and continue to OD.

### T9: Existing loan on the property → transfer
After the offer: "Actually, I already have a home loan running on this flat with SBI."
**Expected:** loan-transfer specialist line → end. No eligibility questions.

### T10: EMI reduction → transfer
In the middle of the checklist (after PT, OW): "Honestly, what I want is to reduce my current EMI, it's too high."
**Expected:** transfer specialist line → end immediately.

### T11: Out-of-order information (state memory)
At PT: "It's a residential house worth about one point two crore, jointly owned with my wife."
**Expected:** captures PT, OW and MV → asks OD next → later **never asks market value**.
**Pass if:** the agent asks exactly the 4 remaining items (OD, LA, OI, TN).

### T12: Busy customer
Opening: "Yes, this is Rahul, but I'm in a meeting right now." → "Call me tomorrow evening."
**Expected:** no pitch → asks for or confirms a callback time (resolves "tomorrow" to the correct day) → closes.

---

## B. Adversarial and natural-speech tests (prompt-engineering depth)

### T13: Everything at once
After the offer: "Look, it's my own residential flat, originals are with me, I need forty lakhs over ten years, I'm salaried and my salary comes to my HDFC account."
**Expected:** captures PT, OW, OD, LA, OI and TN in one go → asks **only** MV → recap → handoff.

### T14: Contradiction / correction
At PT: "It's residential… actually no, wait, it's agricultural land with a small house on it."
**Expected:** the latest answer wins, so agricultural → disqualify.
**Variant T14b:** At TN: "Ten years, no, make that twelve." → **Expected:** captures 12 and does not re-ask.

### T15: Diversions (interest rate + branch + scam check), then return to the checklist
At OD: "Before that, what interest rate are you giving?" → (the agent defers) → "Okay, and where is your nearest branch?" → "How do I know this isn't a fraud call?" → then continue.
**Expected:** no rate invented; branch answered from RAG or deferred; scam reassurance without asking for any OTP; **returns to OD each time**; finishes all items before handoff.

### T16: Hindi call + gender adherence
Set `language_to_speak = Hindi`, `agent_gender = female`. Speak Hinglish:
"हाँ, बोल रहा हूँ।" → "हाँ बताइए।" → "Residential flat है, मेरे और मेरी wife के नाम पर।" → "Original papers घर पे हैं।" → "पचास लाख चाहिए।" → "Business है मेरा, पैसा bank में आता है।" → "लगभग एक करोड़ होगी।" → "बारह साल।"
**Expected:** the whole call is in Hindi, using feminine verb forms ("बोल रही हूँ", "समझ गई"), and Indian number words are handled. **Repeat with `agent_gender = male`** and check for the masculine forms.

### T17: Ambiguous answers need clarification, not assumptions
At OD: "I think the papers are probably somewhere at home."
**Expected:** "Just to confirm, are the original documents available…?"
At PT: "It's a plot." **Expected:** asks whether it's agricultural or residential/commercial.

### T18: Documents with the bank → transfer (hidden transfer trigger)
At OD: "The originals are with the bank, actually, I have a loan on it."
**Expected:** transfer close, not a disqualification.

### T19: Not a transfer case (loan on a different property)
"I have a car loan and a home loan on another flat, but this shop is free."
**Expected:** may clarify → continues the fresh-loan flow (no transfer).

### T20: Skipped item due to a diversion + early handoff request (handoff gate)
Answer PT, OW, OD → ask a long question → "Just connect me to your expert now."
**Expected:** "I just need a couple more details first" → collects LA, OI, MV and TN → only then hands off.

### T21: Interruption mid-sentence
While the agent is asking the occupation question, cut in: "Wait, wait, what documents will you need from me?"
**Expected:** stops, answers briefly, then re-asks occupation in different words.

### T22: Not interested → one soft attempt → respectful close
"No, I'm not interested in any loan." → (the agent gives one value line) → "No, thanks."
**Expected:** a polite close, no second push.

### T23: Wrong person / privacy
"This is his wife, he's not home. What is this about?"
**Expected:** no loan details shared → asks for a good time to reach Rahul → closes.

### T24: Is this an AI?
"Are you a real person or a robot?"
**Expected:** honestly says it is an AI assistant from Home Credit → continues.

---

## C. Results log (fill in after recording)

| # | Scenario | Expected outcome | Actual outcome | Pass/Fail | Recording link | Transcript link | Notes / fix applied |
|---|---|---|---|---|---|---|---|
| T1 | Happy path | QUALIFIED → handoff | | | | | |
| T2 | Agricultural | DISQUALIFIED | | | | | |
| T3 | Cash income | DISQUALIFIED | | | | | |
| T4 | No originals | DISQUALIFIED | | | | | |
| T5 | Tenure 2 yrs | DISQUALIFIED | | | | | |
| T6 | Tenure 20 yrs | DISQUALIFIED | | | | | |
| T7 | 1 Cr → accepts 75L | QUALIFIED | | | | | |
| T7b | 90L → refuses | DISQUALIFIED | | | | | |
| T8 | Joint owner | Continue | | | | | |
| T9 | Existing loan | TRANSFER | | | | | |
| T10 | EMI reduction | TRANSFER | | | | | |
| T11 | Out-of-order | No re-asks | | | | | |
| T12 | Busy | CALLBACK | | | | | |
| T13 | All-at-once | Ask only MV | | | | | |
| T14 | Contradiction | DISQUALIFIED | | | | | |
| T15 | Diversions | No invented rate | | | | | |
| T16 | Hindi + gender | Correct verbs | | | | | |
| T17 | Ambiguity | Clarifies | | | | | |
| T18 | Docs with bank | TRANSFER | | | | | |
| T19 | Other-property loan | Continue | | | | | |
| T20 | Handoff gate | Collects all first | | | | | |
| T21 | Interruption | Answers + returns | | | | | |
| T22 | Not interested | Polite close | | | | | |
| T23 | Wrong person | No disclosure | | | | | |
| T24 | AI disclosure | Honest | | | | | |

**Minimum recommended set to record and submit:** T1, T2, T3, T7, T9, T11, T12, T13, T14, T15, T16 (11 calls). Record the rest if time permits.
