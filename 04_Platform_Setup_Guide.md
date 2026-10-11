# Platform Setup Guide: Deploying and Recording on Retell AI (or Bolna)

These steps take you from the prompt file to recorded test calls with public links. Dashboard labels change from time to time, so if a button is named slightly differently, look for the closest equivalent.

## 1. Create the agent (Retell AI)
1. Sign up at retellai.com (the free trial includes test credits) and open the dashboard.
2. **Agents → Create agent → Single Prompt agent** (start blank).
3. **LLM:** choose a strong model (a GPT-4o / GPT-4.1-class or Claude-class model if listed). A stronger model follows the state rules much more reliably. Set temperature low (about 0.2–0.4) for consistent rule-following.
4. **Prompt:** paste the full contents of `02_System_Prompt.txt`.
5. **Voice:** pick a professional **Indian English female** voice so it matches `agent_gender = female`. For the Hindi test (T16), pick a voice that supports Hindi, and switch to a male voice when you test `agent_gender = male`.
6. **Language:** English (India) for English tests. For T16, set Hindi or multilingual if offered.
7. **Welcome message:** set it so the **agent speaks first**, either with a dynamic first line or leaving it to the prompt (the prompt already contains the opening line).
8. **Functions / Tools:** add the built-in **End Call** function (`end_call`). The prompt tells the agent to end the call after every closing line.
9. **Speech settings (recommended):** keep interruption sensitivity medium-high so the customer can cut in (tested in T21), keep backchannel off or low, and set end-of-call after about 20–30 s of silence.

## 2. Dynamic variables
Retell substitutes any `{{variable}}` in the prompt. Set test values in the dashboard's dynamic-variable / test panel:

```
company_name        = Home Credit
customer_name       = Rahul Sharma
agent_name          = Priya
agent_gender        = female
language_to_speak   = English
current_date        = 12 October 2026
current_day         = Monday
current_time        = 11:30 AM
additional_context_from_rag = (paste the sample block below)
```
`conversation_history` and `customer_utterance` are supplied automatically by the platform, because it sends the live transcript to the LLM every turn. They are kept in the prompt because the assignment's variable spec requires them; if the platform shows them as literal text, that's harmless.

### Sample RAG block (test content written by the candidate, not official Home Credit data)
```
Loan Against Property: secured loan against residential, commercial or industrial property.
Typical documents: original property title documents, KYC of applicants, last 6 months bank statements, income proof (salary slips or ITR / business financials).
Co-owners of the property generally join as co-applicants.
Interest rates are personalised and shared by the senior loan expert after preliminary eligibility.
Nearest branch: customers can use the branch locator on the official website or call customer care.
The company never asks for OTP, PIN or passwords over a call.
```

## 3. Post-call analysis
In **Post-Call Analysis**, add these custom extraction fields so each call log shows structured results:

| Field | Type | Description |
|---|---|---|
| call_outcome | enum | QUALIFIED_HANDOFF, DISQUALIFIED, TRANSFER_SPECIALIST, CALLBACK, NOT_INTERESTED, WRONG_PERSON |
| disqualification_reason | text | e.g. agricultural / cash income / tenure / docs / amount |
| property_type | text | |
| ownership | text | |
| original_documents | text | |
| loan_amount_lakh | number | |
| occupation | text | |
| income_mode | text | |
| market_value | text | |
| tenure_years | number | |
| callback_time | text | |
| any_question_repeated | boolean | Did the agent ask for an already-provided item? (quality check) |

This turns every test call into a measurable log entry and shows the evaluator that state tracking works.

## 4. Running the tests
1. Use **Test Audio / Web Call** in the dashboard (no phone number needed). If you'd rather use a phone call, the trial may allow a test number.
2. Run the scenarios in `03_Test_Plan_and_Call_Scripts.md`. **Speak naturally**: fillers, pauses, corrections and interruptions.
3. After each call, open **Call History** to see the recording, transcript, latency and post-call analysis.
4. If a test fails, note it, adjust the prompt, re-test, and record the fix in the "Notes / fix applied" column. Evaluators value an iteration log.

## 5. Public links for submission
1. Download each call's **recording** (audio) and copy its **transcript** from Call History.
2. Create a Google Drive folder named `Upendra_Rejeti_SalesAgents_Voice_Agent_Tests` containing:
   - `Recordings/T01_Happy_Path.wav` (and so on)
   - `Transcripts/T01_Happy_Path.txt`, or a single Google Doc with every transcript
   - `01_System_Prompt` (PDF or text)
   - `Approach_Document.pdf`
   - `Test_Results` (the filled results table)
3. Share the folder with **Anyone with the link → Viewer**. Open the link in a private or incognito window to confirm it works.
4. Retell call-detail pages may require a login, so the Drive folder is the reliable public link.

## 6. Alternative: Bolna
Bolna (bolna.ai / bolna.dev) also works well for Indian-language voice agents:
- Create an agent → paste the prompt into the agent prompt → choose an LLM, a voice (Indian English or Hindi) and a transcriber that supports Hindi.
- Variables use `{variable}` or `{{variable}}`, depending on the version. Match the syntax in the prompt if needed.
- Enable hang-up / end-call behaviour, then test with a web or phone call and download the recordings and transcripts from the execution logs.
