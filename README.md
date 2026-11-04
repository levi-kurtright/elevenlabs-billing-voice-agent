# Riley: Dealer Billing Voice Agent (ElevenLabs ElevenAgents)

A voice AI agent that answers dealer billing questions for a fictional tire distributor, **Summit Tire Supply**. It's built on [ElevenLabs ElevenAgents](https://elevenlabs.io/) to explore how a guardrail-first design for enterprise service agents carries over from chat to voice.

All company names, policies and data here are fictional demo content.

## The problem
Distributors that bill large dealer networks get a steady stream of routine billing calls. Callers ask why a late fee was charged, where a credit is, whether a payment posted, and why an invoice price doesn't match a quote. Most of these have clear, policy-based answers. A voice agent can resolve them in one call, and it must know when to stop and hand off to a person.

## What Riley does
- Greets the caller and collects the dealer number and invoice number
- Answers questions about invoices, payment terms, early-payment discounts, late fees, credit holds, credits and returns
- Explains how to dispute a price that doesn't match the approved quote, and what happens next
- Keeps answers to two or three sentences, because it's a voice call
- Hands off to a human when needed

## Guardrails
| Rule | Why |
|---|---|
| Never takes card or bank numbers over the phone; directs callers to the dealer portal | Payment security |
| Never promises a credit, refund or price change; can open a dispute and explain timing | Avoids commitments only finance can make |
| Doesn't guess; if the answer isn't in the knowledge base, offers a transfer | Accuracy and trust |
| Escalates upset callers or issues over $5,000 to a specialist | Protects high-value relationships |
| Never discusses other dealers' accounts or pricing | Confidentiality |

## How it's built
- **Platform:** ElevenLabs ElevenAgents with a custom voice and English
- **System prompt:** [`system_prompt.txt`](system_prompt.txt) covers personality, goal, call flow, guardrails and tone
- **Knowledge base:** [`knowledge_base.txt`](knowledge_base.txt), billing policies indexed for retrieval (RAG)
- **First message:** "Thanks for calling Summit Tire Supply billing, this is Riley. Can I get your dealer number and the invoice you're calling about?"

## Test scenarios (all passed in live voice testing)
1. **Late fee question:** explains the 1.5% fee after 15 days past due
2. **Price doesn't match quote:** explains the 60-day dispute window, and that late fees pause during review
3. **Caller offers a card number:** declines and points to the dealer portal
4. **Frustrated caller with a $12,000 issue:** offers a transfer to a billing specialist
5. **Interruption mid-sentence:** stops, listens and responds naturally

## What I took away
The same design I used for a production chat-based service agent carried directly to voice: a narrow, well-defined job, answers grounded in a knowledge base, and explicit guardrails with a clear handoff. Voice raises the bar on brevity and turn-taking, and ElevenLabs handled both well.

## Next steps
- Add a tool call to look up live invoice status from a billing system API
- Add a transfer-to-human tool with context handoff
- Track resolution rate and escalation rate in post-call analysis
