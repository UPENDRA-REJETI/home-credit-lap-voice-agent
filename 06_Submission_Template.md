**Subject:** AI Intern (Prompt Engineer) Assignment Submission: Upendra Rejeti

Hi Team,

Please find my submission for the Home Credit LAP Qualification Voice Agent assignment.

1. **System Prompt:** [link to 01_System_Prompt.pdf]. The paste-ready text is also in the folder.
2. **Call recordings and call logs (audio + transcripts):** [public Google Drive folder link]
   - Platform: Retell AI, single-prompt agent with end-call and post-call analysis fields
   - [N] recorded scenarios, including: happy path, agricultural disqualification, cash-income disqualification, loan above 75 lakh (accepted and refused), existing loan / EMI reduction → transfer, out-of-order and all-at-once answers, corrections, busy-customer callback, interest-rate and scam-check diversions, and a Hindi call with gender-correct verb forms.
3. **Approach document (optional):** [link to 05_Approach_Document.pdf]. It covers the architecture, the per-turn Extract → Validate → Update → Route → Respond loop, the eligibility decision table, my documented assumptions, and requirement-to-test traceability.

Summary of the approach: the agent rebuilds a silent 7-slot state from the conversation every turn and always asks the first missing item. This keeps the order sequential but never repeats a question. Disqualification and transfer checks run on every utterance, so they trigger immediately even when the information comes up out of order. A strict handoff gate (with a recap) ensures no lead is passed on before all 7 points are confirmed. Nothing outside the brief is invented: no interest rates and no market-value thresholds.

Thank you for the opportunity. I look forward to your feedback.

Regards,
Upendra Rejeti
[Phone] · [Email]
