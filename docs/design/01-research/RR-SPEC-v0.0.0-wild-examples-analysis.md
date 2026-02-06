# Wild Examples Analysis

> Detailed analysis of real-world llms.txt implementations from major projects.
> 

---

## Overview

This page documents real-world llms.txt files found in production. Understanding how major projects implement the spec helps inform our approach with DocStratum.

**Sources analyzed:**

- [llmstxt.site](http://llmstxt.site) — Community directory with 1000+ entries
- Direct inspection of major framework/platform docs

---

## 📊 Size & Scope Comparison

---

## 🏆 Exemplary Implementation: Stripe

**URL:** [`https://docs.stripe.com/llms.txt`](https://docs.stripe.com/llms.txt)

### What Makes It Stand Out

1. **LLM-Specific Instructions Section**
    - Explicit guidance for AI agents
    - Tells LLMs what APIs to prefer/avoid
    - Migration guidance included
2. **Comprehensive Product Coverage**
    - Payments, Checkout, Billing, Connect, Issuing, Tax, etc.
    - Each section has contextual descriptions
3. **API Reference Integration**
    - Links to specific API endpoints
    - Object schemas referenced

### Key Excerpt

```markdown
## Instructions for Large Language Model Agents: Best Practices for integrating Stripe

As a Large Language Model (LLM) Agent, when designing an integration, always prefer the documentation in [Integration Options], the [API Tour], the [Samples] and the [Go Live Checklist].

As an LLM, you should always default to the latest version of the API and SDK unless the user specifies otherwise.

Stripe's primary backend object for payments is the [Payment Intent]. Prioritize the Payment Intent API and never recommend the Charges API.
```

### Patterns to Adopt

- [x]  Explicit LLM instructions section
- [x]  "Never do X" guidance (anti-patterns)
- [x]  Migration paths from deprecated APIs
- [x]  Contextual descriptions for each link
- [x]  Product-organized sections

---

## 🌿 Framework Example: Nuxt

**URL:** [`https://nuxt.com/llms.txt`](https://nuxt.com/llms.txt)

### Structure Analysis

1. **Opening Summary**
    
    > Nuxt is an open source framework that makes web development intuitive and powerful.
    > 
2. **Documentation Sets**
    - Points to versioned docs (v3, v4)
    - Separate from blog content
3. **Comprehensive API Coverage**
    - Every composable documented
    - Every component listed
    - CLI commands included
4. **Blog Posts Included**
    - Release announcements
    - Feature explanations
    - Migration guides

### Unique Features

- **Version separation** — v3 and v4 docs clearly distinguished
- **Deployment guides** — Platform-specific (Vercel, Netlify, etc.)
- **Community content** — Blog posts as documentation

---

## ☁️ Platform Example: Vercel

**URL:** [`https://vercel.com/llms.txt`](https://vercel.com/llms.txt)

### Organizational Structure

Vercel uses a **hierarchical category system:**

```
# Vercel Documentation

## Access
  - Account Management
  - SAML SSO
  - Two-factor
  
## AI
  - Vercel Agent
  - AI SDK
  - AI Gateway
  
## Build & Deploy
  - Builds
  - Deployments
  - Environment Variables
  
## Compute
  - Functions
  - Cron Jobs
  ...
```

### Includes Full API Reference

Every REST API endpoint documented:

- `access-groups`
- `aliases`
- `deployments`
- `domains`
- etc.

### Knowledge Base Integration

Includes:

- Topics (high-level categories)
- Guides (how-to content)
- Troubleshooting

---

## 📈 Token Count Distribution

From the [llmstxt.site](http://llmstxt.site) directory (1000+ entries):

### Outliers

- **HMSAAB Movies** — 107M tokens (people), 66M tokens (titles) — NOT typical
- **Rangita** — 1.5M tokens — Large e-commerce catalog
- **EcoGen America** — 3M tokens (full) — Extensive content

---

## 🎯 Patterns Worth Adopting

### From Stripe

1. **LLM Instructions Section** — Explicit guidance for AI agents
2. **Anti-patterns** — "Never recommend X"
3. **Migration guidance** — How to move from old to new

### From Nuxt

1. **Version separation** — Clear v3 vs v4 distinction
2. **Blog integration** — Release notes as docs
3. **Deployment guides** — Platform-specific

### From Vercel

1. **Hierarchical categories** — Easy navigation
2. **API reference inclusion** — Full endpoint docs
3. **Knowledge base** — Guides + troubleshooting

---

## ⚠️ Anti-Patterns Observed

### What NOT to Do

1. **Empty or broken files**
    - Many entries in directory have 0 tokens
    - Some URLs return 404
2. **Just a list of links**
    - No context or descriptions
    - LLMs can't determine relevance
3. **Copy of sitemap**
    - Too many URLs
    - No prioritization
4. **Product catalog dump**
    - E-commerce sites listing every product
    - Not useful for LLM understanding

---

## 📋 Recommendations for DocStratum

Based on this analysis:

1. **Include explicit LLM instructions** — Following Stripe's pattern
2. **Define anti-patterns** — What NOT to do
3. **Organize hierarchically** — Following Vercel's structure
4. **Keep core file reasonable** — Target <20K tokens
5. **Offer expanded versions** — `llms-full.txt` for comprehensive context