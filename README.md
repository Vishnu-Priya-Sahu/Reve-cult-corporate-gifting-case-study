# Corporate Gifting Lead Outreach Automation

**Built during my internship at REVE CULT (Triman Cult Tech Pvt. Ltd.) as AI Automation & Website Developer Intern**

## Problem

REVE CULT needed a way to reach corporate leads for gifting inquiries across email, WhatsApp, and LinkedIn without someone manually drafting and sending every message. But outreach automation on its own isn't enough — once leads start replying, you need a safe way to handle those replies too. Pricing questions, meeting requests, and anything ambiguous can't be left to an AI to answer on its own; they need a human in the loop. The challenge was building a system that could automate the repetitive parts of outreach while staying reliably cautious about anything it wasn't equipped to handle.

## Architecture

The workflow uses a Google Sheet as the lead database. Whenever a new lead is added, it triggers the automation, which validates the contact data and drafts personalized outreach content for email, WhatsApp, and LinkedIn. The email is sent automatically with the product catalogue attached, while the WhatsApp and LinkedIn drafts are logged for manual sending, since those channels don't allow the same kind of automated dispatch.

On the receiving end, incoming email replies are watched continuously via IMAP. Each reply goes through a classification step that decides whether it's something the AI can safely respond to on its own, or whether it needs to be escalated to a human reviewer.

![Lead Outreach Workflow](asset/Lead%20Outreach%20Workflow.jpeg)

## The Bug — and the Fix

This is the part of the system I'm proudest of, because it came from something breaking in a way I didn't expect, and fixing it meant rethinking the design rather than patching around the symptom.

The original human-review flagging logic expected replies to arrive in a fairly structured format. In practice, real replies didn't look like that at all — people replied with a single word like "interested," or in a language other than English, and the AI couldn't parse those into the structure it expected. The flagging step was silently failing, which meant replies that should have gone to a human reviewer were falling through the cracks instead.

Rather than trying to make the AI smarter at parsing every possible reply format, I narrowed its automated scope down to exactly one case it could handle reliably: a keyword match for catalogue requests. Everything else — pricing questions, meeting requests, short or ambiguous replies, non-English replies — now routes to human review by default. The system fails safe: when it isn't sure, it hands off instead of guessing.

This was a deliberate design choice, not a limitation. Pricing can vary and often involves negotiation, so hard-coding it into the AI risked it quoting something wrong or inconsistent, which damages trust with a corporate lead. Scheduling depends on the team's real availability, so having the AI attempt that would mean either guessing at a calendar or making promises it couldn't keep. And a short or ambiguous reply like "interested" genuinely could mean several different things — pricing, a meeting, general interest — so guessing wrong there risks losing the lead entirely. In each case, more automation meant more risk, not more intelligence, so I chose to keep the AI confident and narrow, and default everything uncertain to a human.

I also added a refinement on top of that: instead of just flagging a reply for review, the workflow writes an extracted reason into the Sheet and the Slack notification — "pricing inquiry," "meeting request," "short reply," "non-English reply," and so on. Reviewers no longer have to read a raw email and figure out why it needs their attention; they get pre-triaged context the moment it lands in front of them.

![AI Reply Classifier](asset/AI%20Classifier.jpeg)

![Flagged for human review in the activity log](asset/Activity%20Log.jpeg)

## Impact

Reliable escalation was restored — replies that used to slip through flagging now get caught and routed correctly. And because the review-reason labeling was added on top of the fix, human reviewers get labeled, pre-triaged context instead of sorting through raw inbox messages themselves, which makes the review step faster and less error-prone.

## Other Automations Built

Alongside this system, I also built a welcome-email automation that triggers on signup, and an AI-driven review-moderation workflow that checks incoming product reviews for appropriateness and generates a promo code reward when a review includes media.
