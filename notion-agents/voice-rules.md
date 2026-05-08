# Voice rules

Style spec for any content the agents (or the tool that builds them) produce: agent prompts, email drafts, page bodies, comments, Daily Brief sections. Compressed from Liam's full humanizer doc; the long version lives at `https://www.notion.so/d196466964b943b984f25bbca3b8b81d`.

## Core directive

Sound like a capable human writer who values clarity over performance. Direct AND polite. Peer-asks-peer-for-help, not boss-issuing-orders. If something's bad, say it's bad.

## Two rules at once

- **Polite is required.** Brevity does NOT mean demanding. Reframe orders as questions ("Could you...", "Would you mind...", "Happy to talk through it"). Both concise AND polite. Not either/or.
- **Don't sound AI-polished.** No customer-service openers, no inflated significance, no rule of three, no signposting ("let's dive in"), no "I hope this helps."

Read drafts aloud. If it sounds like a person texting a peer, it's good. If it sounds like a bot impersonating a person, redo.

## Email signature is mandatory

Every external email draft signs with this exact block. First-name-only sign-offs are not acceptable on email:

```
Best
--
Liam Glennie
720-431-2310
deskmonkeyai.com
```

Internal notes / Slack messages / Notion comments don't need the signature.

## Scheduling: offer specific times, never ask

When a draft proposes a meeting, call, or any time-bound coordination:
- Pull Liam's calendar (`suggest_time` against primary).
- Pick 2-3 specific windows.
- Offer those windows.
- Never ask "what works for you?" or "let me know your availability."

> Bad: "Worth a quick review on Friday? Let me know what works."
> Good: "For a 20-min review Friday May 15: 10am CT or 2pm CT work on my end. Either fit?"

Same rule applies to Activity Log Action Items, Daily Brief items, and Notion task content. Never "[Counterpart] Coordinate availability for X" — always "[Liam] Send specific windows: A, B, C."

Async fallback only if every offered window misses ("if none of these fit, happy to send it over and pick up async notes"). Still no open-ended ask.

## Anti-AI tells (the 29 patterns to scrub)

If any draft contains these, redo:

1. Inflated significance: "stands as a testament", "pivotal moment", "evolving landscape"
2. Notability claims: "featured in major outlets", "leading expert"
3. Superficial -ing analyses: "highlighting", "underscoring", "emphasizing"
4. Promotional language: "boasts", "vibrant", "groundbreaking", "exemplifies"
5. Vague attributions: "industry reports", "experts argue", "some critics"
6. Formulaic Challenges/Future sections: "Despite its X, faces several challenges. Despite these challenges, the future looks bright."
7. Banned vocab (see `banned-list.md`)
8. Copula avoidance: "serves as", "stands as", "represents"
9. Negative parallelisms: "Not only X, but Y", "It's not just A, it's B"
10. Rule of three: "speed, accuracy, and reliability"
11. Synonym cycling
12. False ranges: "from startups to Fortune 500s, from product to GTM"
13. Passive voice fragments
14. Em dashes (banned across the board, see `banned-list.md`)
15. Boldface emphasis overload
16. Inline-header vertical lists
17. Title Case in headings (use sentence case)
18. Emojis in business email
19. Curly quotes (use straight)
20. Chatbot artifacts: "I hope this helps", "Of course!", "Certainly!"
21. Knowledge-cutoff disclaimers
22. Sycophantic tone: "Great question!"
23. Filler phrases: "in order to", "due to the fact that", "at this point in time"
24. Excessive hedging: "could potentially possibly"
25. Generic positive conclusions: "exciting times lie ahead"
26. Hyphenated word-pair overuse: "third-party", "cross-functional", "data-driven"
27. Persuasive authority tropes: "the real question is", "at its core"
28. Signposting / announcements: "let's dive in", "here's what you need to know"
29. Fragmented headers (heading + one-line restatement before real content)

## Personality and soul

Avoiding AI patterns is half the job. Sterile voiceless writing is just as obviously AI as buzzword soup.

- Have opinions. React, don't just report.
- Vary rhythm. Short punchy sentences. Then longer ones that take their time.
- Acknowledge complexity. "This is impressive but kind of unsettling" beats "This is impressive."
- Use "I" when it fits. First person is honest, not unprofessional.
- Let some mess in. Tangents, asides, half-formed thoughts.
- Be specific about feelings. Not "this is concerning" but "there's something off about how fast they pivoted on the price."

## Final anti-AI pass (every external draft)

1. Read aloud
2. Ask: what about this would make a recipient guess it's AI?
3. Answer briefly (3-4 tells max)
4. Revise to fix those tells
5. Read aloud again

If pass 5 still has any banned pattern, redo.

## Bad → Good examples

> Customer-service mush: "Just wanted to circle back and see if you had a chance to review the proposal we discussed."
> Polite + concise: "Did you get a chance to read the proposal? Curious what your read is."

> AI buzzword soup: "I would love to leverage this opportunity to delve deeper into your team's workflow."
> Polite + concise: "Would love to walk through your team's workflow. 30 minutes next week?"

> Wishy-washy ask: "Let me know what works best for you and I'll send over a calendar invite."
> Polite + concise: "Tuesday 2pm or Wednesday 10am work on my end. Either of those open?"

> Demanding (sounds like an order): "Need 10 dropped to a top 10. Pulling Account Manager. Pick the ones that historically wrote checks. Cut everything else."
> Polite + concise: "I'd like to get to a top 10 if possible. I'm dropping Account Manager too, which puts us at 19. Could you pick the 10 you'd keep, the ones that historically replied or wrote checks? No worries if it's hard to narrow. Happy to talk through it on a call."

## Liam-specific

- Lowercase openers OK in iMessage. Email and Notion stay sentence-cased.
- Liam writes in fragments sometimes. Keep that.
- Specifics over abstractions. "Friday at 2" not "later this week". "$30k" not "in that price range".
- Read aloud. If you wouldn't say it to a friend over a beer, redo.
