# Riley: Dealer Billing Voice Agent on Stripe

A voice AI agent that answers dealer billing questions from **live Stripe data** and a governed knowledge base for a fictional office products distributor, **Summit Office Supply**. Riley looks up invoices and plans, explains metered usage and failed payments line by line, issues **Stripe credit notes within policy**, gets declined card payments paid without ever taking a card number, and logs a tracked follow-up whenever a person is needed.

Riley's sister agent is [Quinn](https://github.com/levi-kurtright/voice-quoting-agent), who quotes, sells, and changes plans. The two hand callers to each other, and they share one Stripe Billing service, which lives in [Quinn's repo](https://github.com/levi-kurtright/voice-quoting-agent/tree/main/service).

All company names, dealers, policies and data are fictional. Stripe runs in **test mode**: real API calls, no real money.

## The problem
Distributors that bill large dealer networks field a steady stream of routine billing calls: is my invoice paid, what do I owe, why was I charged this, why did my payment fail. Most have clear answers in policy and in the billing system. A voice agent can resolve them in one call, but only if it answers from real data, never guesses, and knows exactly where its authority ends.

## What Riley does
| Caller asks | Tool | Source of truth |
|---|---|---|
| "Is invoice X paid? What do I owe?" | `check_invoice_status` | Stripe invoices |
| "What's on my plan? When's my next bill?" | `check_subscription` | Stripe subscriptions |
| "How much have we printed this month?" | `check_print_usage` | Stripe Billing meters + invoice preview |
| "Why is this invoice $103.20?" | `check_invoice_status` | Metered line items, explained in plain words |
| "Why are we past due?" | `check_invoice_status` | Decline reason + Stripe's next retry date |
| "I want to pay invoice X" | `send_payment_link` | Stripe emails its hosted payment page to the billing email on file |
| "The owner updated the card, run it again" | `retry_card_payment` | Stripe retries the charge on the call, after a yes |
| "How much print credit do we have?" | `check_print_credits` | Stripe Billing credit balance |
| "Two cartridges arrived cracked" | `preview_credit` → `issue_credit` | Stripe credit notes |
| Policy questions: late fees, disputes, credit holds | Knowledge base (RAG) | [`knowledge_base.txt`](knowledge_base.txt) |
| Orders, quotes, plan changes | `transfer_to_agent` | Hands off to Quinn |
| Anything needing a person | `log_follow_up` | Tracked ticket with a reference number |

## Guardrails
| Rule | Why |
|---|---|
| Reads back Stripe's amounts, dates and statuses verbatim; never guesses | Accuracy and trust |
| Credits: preview first, issue only on a clear yes, max $100 per invoice by phone (including earlier phone credits), never more than is owed, never a card refund | Small fixes on the call, finance controls the rest |
| Never takes card or bank numbers; Stripe emails the payment link only to the billing email already on file, never to an address read out on the call | Payment security and fraud prevention |
| A card retry reads back the amount first and runs only after a clear yes | No money moves without consent |
| Usage projections are always labeled as estimates | No promises about a bill that isn't final |
| Every "a specialist will follow up" is logged first, with a reference number | Nothing promised on a call gets lost |
| Escalates upset callers or issues over $5,000 | Protects high-value relationships |
| Never discusses other dealers' accounts | Confidentiality |

## Verified results (Stripe test mode)
- Dealer D-1042 → "1 open invoice, $332.50, due December 6"; CPGODWZY-0001 → "paid in full"
- $30 damaged-goods credit → previewed $74.69 → $44.69, issued only after "go ahead"; a retry didn't credit twice; $150 → specialist
- Print usage → 9,466 pages, 4,466 over the allowance, $53.59 overage, matching Stripe's invoice preview ($102.59) to the cent
- Test clock month-end → explains invoice SJ0HSXCP-0002: "13,600 pages: 5,000 included, and 8,600 at 1.2 cents a page, for $103.20"
- Card declined at renewal → explains the decline and Stripe's next retry; after the card was updated, Riley had Stripe retry on the call, it was paid, and a Stripe webhook closed the open follow-up automatically
- "I want to pay CPGODWZY-0002" → Stripe emailed the hosted payment page for $332.50 to the billing contact on file
- Prepaid print credit → $100 available; Stripe's next-invoice estimate drops to $0.00 "after $53.59 of your prepaid print credit"

## How it's built
- **Platform:** ElevenLabs ElevenAgents, Claude Haiku 5.5 as the LLM
- **System prompt:** [`system_prompt.txt`](system_prompt.txt), exported from the live agent
- **Tools:** [`tools/`](tools), ElevenLabs webhook tool definitions (replace `YOUR-STRIPE-VAL` with your service URL)
- **Knowledge base:** [`knowledge_base.txt`](knowledge_base.txt), billing policies indexed for retrieval
- **First message:** "Thanks for calling Summit Office Supply billing, this is Riley. Can I get your dealer number and the invoice you're calling about?"

## Testing
Eight automated ElevenLabs tests cover Riley: balance lookup, hand-off to Quinn, damaged-goods credit, print usage, failed payment with a logged follow-up, payment link instead of a card number, and card retry (reads back first, runs only after a yes). Every real call is also scored for "no money moves without a yes," "numbers only from tools," and correct routing. Details in [Quinn's TESTS.md](https://github.com/levi-kurtright/voice-quoting-agent/blob/main/TESTS.md).

## What I took away
The design that worked for a production chat service agent carries to voice and to money: a narrow job, answers grounded in a system of record, explicit limits, and a clean hand-off. Voice raises the bar on brevity. Billing raises it on verifiability. Every number has to trace back to Stripe.
