# Pricing Details

Detailed breakdown of ODIN pricing tiers.

## Trial Tier (Free Forever)

**Cost:** €0/month (never billed)

**Resources:**
- 1,440 CPU-hours per month
- 2,880 GB-RAM-hours per month
- Up to 25 concurrent voice users (CCU)

**Limitations:**
- Intended for evaluation only
- Not suitable for live production use
- Servers auto-shutdown when resources exhaust
- No resource rollover to next month

**Use Cases:**
- Development and testing
- Small-scale prototypes
- Educational projects

---

## Indie Starter Plan

**Cost:** €49/month (never exceeds this amount)

**Resources:**
- 14,400 CPU-hours per month
- 28,800 GB-RAM-hours per month
- Up to 500 concurrent voice users (CCU)
- 4 hours one-on-one support with engineer

**Grace Period:**
- If usage temporarily exceeds pool, servers scale up to 200% for 48 hours at no extra cost
- After grace period, servers stop (no surprise charges)

**Eligibility Requirements:**
- Studios with <€250,000 annual turnover
- ≤10 employees

**Billing:**
- Pro-rata billing for mid-month starts/switches
- Resources refilled monthly
- Never charges beyond €49/month - servers stop instead

---

## Pay-as-you-Go (Pro)

### Voice Pricing

Per Peak Concurrent User (PCU) per month:

| PCU Range | Price per PCU |
|-----------|---------------|
| 1–19,999 | €0.29 |
| 20,000–39,999 | €0.25 |
| 40,000–59,999 | €0.23 |
| 60,000–79,999 | €0.20 |
| ≥80,000 | €0.19 |

### Fleet Pricing

Daily rates (examples):

| Instance | Specs | Daily | Monthly (~) |
|----------|-------|-------|-------------|
| S-2 | 1 vCPU / 2GB RAM | €0.60 | €18 |
| M-4 | 2 vCPU / 4GB RAM | €0.93 | €27.90 |
| L-8 | 4 vCPU / 8GB RAM | €2.00 | €60 |

### Billing Details

- Monthly cycles run from first to last day of month
- Usage tracked daily
- Highest daily concurrent instance count determines monthly cost
- No hidden traffic fees
- No minimum commitment

### Volume Discounts

- Automatic tier-based pricing for voice users
- Significant savings at scale (€0.19 vs €0.29 per PCU)
