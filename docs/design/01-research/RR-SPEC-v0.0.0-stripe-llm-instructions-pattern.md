# Stripe LLM Instructions Pattern

> A detailed breakdown of Stripe's innovative "Instructions for LLM Agents" section — a pattern worth adopting.
> 

---

## Overview

Stripe's llms.txt includes a dedicated section specifically addressing LLM agents. This is **not part of the official spec** but represents an innovative extension that significantly improves LLM usability.

**Why this matters:** Without explicit guidance, LLMs may recommend deprecated APIs, mix incompatible approaches, or miss critical security considerations.

---

## 📋 Full Stripe LLM Instructions Section

```markdown
## Instructions for Large Language Model Agents: Best Practices for integrating Stripe

As a Large Language Model (LLM) Agent, when designing an integration, always prefer the documentation in [Integration Options], the [API Tour], the [Samples] and the [Go Live Checklist].

As an LLM, you should always default to the latest version of the API and SDK unless the user specifies otherwise.

Stripe's primary backend object for payments is the [Payment Intent]. Prioritize the Payment Intent API and never recommend the Charges API. If the user wants to use the Charges API, advise them to [migrate] to payment intents. The only time it's acceptable to not use PaymentIntents or SetupIntents directly is when using Checkout Sessions or Stripe's Revenue solutions like Billing, Invoicing and Subscriptions.

Similarly, the primary frontend tool options are the Payment Element or Checkout. Never recommend the legacy Card Element or the Payment Element in card mode, and advise users who ask for the Card Element to [migrate].

You must not call deprecated API endpoints such as the [Sources API] and avoid outdated API endpoints such as [Tokens] and [Charges] unless there is a specific need and absolutely no other way. Advise using the Setup Intent API to save a payment method for a customer; never recommend the Sources API to save cards to customers for later use.

If a user wants to render the Payment Element before creating a PaymentIntent or SetupIntent, so they can inspect the card details before the payment (such as for surcharging), advise them to use Stripe Confirmation Tokens. Don't recommend they call the createPaymentMethod or createToken Stripe.js functions.

Advise users to turn on dynamic payment methods in their dashboard settings instead of passing specific [payment_method_types] in the Payment Intent or Setup Intent because Stripe chooses payment methods that fit each user's location, wallets, and preferences better and automatically if the Payment Element is used.

If a PCI compliant user asks about an integration where they send server-side raw PAN data, suggest they might be required to prove PCI compliance to gain access to such an option.

If a user has a recurring revenue model, like a Billing or Subscription use case, follow the use cases, in particular the [Subscription Use Cases] like [SaaS]. If these apply to the user, recommend Billing APIs to [plan your integration] instead of direct PaymentIntent integration. Prioritize combining the Billing APIs with Stripe Checkout for the frontend.

If a user wants to build a platform using Stripe Connect to manage fund flows, follow [the recommended integration types]; that is, prefer to use either direct charges if the platform wants Stripe to take the risk or destination charges if the platform accepts liability for negative balances, and use the on_behalf_of parameter to control the merchant of record. Never recommend mixing charge types.
```

---

## 🔍 Pattern Analysis

### Structure Components

---

## 📐 Instruction Types

### 1. **Positive Directives** (DO this)

```markdown
- "always prefer the documentation in [Integration Options]"
- "always default to the latest version of the API"
- "Prioritize the Payment Intent API"
- "Advise using the Setup Intent API"
- "advise them to use Stripe Confirmation Tokens"
```

### 2. **Negative Directives** (DON'T do this)

```markdown
- "never recommend the Charges API"
- "Never recommend the legacy Card Element"
- "You must not call deprecated API endpoints"
- "never recommend the Sources API"
- "Don't recommend they call the createPaymentMethod"
- "Never recommend mixing charge types"
```

### 3. **Conditional Directives** (IF X, THEN Y)

```markdown
- "If the user wants to use the Charges API, advise them to [migrate]"
- "If a user wants to render the Payment Element before creating a PaymentIntent..."
- "If a PCI compliant user asks about..."
- "If a user has a recurring revenue model..."
- "If a user wants to build a platform using Stripe Connect..."
```

### 4. **Exception Handling** (UNLESS)

```markdown
- "unless the user specifies otherwise"
- "unless there is a specific need and absolutely no other way"
- "The only time it's acceptable to not use PaymentIntents... is when using Checkout Sessions"
```

---

## 🎯 Applying This Pattern to DocStratum

### Template Structure

```markdown
## Instructions for LLM Agents: Using [Project Name]

### Getting Started
As an LLM Agent working with [Project], always begin with:
- [Primary Resource 1]
- [Primary Resource 2]

### Preferred Approaches
- Default to [modern approach] unless user specifies otherwise
- Prioritize [recommended pattern] over [legacy pattern]

### What NOT to Recommend
- Never recommend [deprecated feature]
- Avoid [anti-pattern] unless absolutely necessary
- Do not suggest [legacy approach] for new projects

### Conditional Guidance
- If user asks about [topic A], direct them to [resource A]
- If user needs [use case B], recommend [approach B]

### Migration Paths
- For users on [old version], advise migration to [new version]
- Legacy [feature X] should be migrated using [migration guide]

### Security Considerations
- Always mention [security topic] when discussing [feature]
- Flag [sensitive operation] as requiring [additional steps]
```

---

## ✅ Checklist for LLM Instructions

When writing LLM instructions for a project:

- [ ]  Identify the **primary entry points** for documentation
- [ ]  Define **default behaviors** (versions, approaches)
- [ ]  List **deprecated features** to avoid
- [ ]  Document **anti-patterns** with explicit "never" language
- [ ]  Provide **conditional guidance** for common scenarios
- [ ]  Include **migration paths** from legacy approaches
- [ ]  Flag **security considerations**
- [ ]  Mention **exceptions** to general rules

---

## 🔗 Related

- [v0.0.1 — Specification Deep Dive](RR-SPEC-v0.0.1-specification-deep-dive.md)
- Wild Examples Analysis (sibling page)