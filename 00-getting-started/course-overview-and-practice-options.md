# Course Overview and Practice Options

> Difficulty: Beginner
> Importance: Foundational

`CORE`

## 1. What "Identity Management" Actually Covers

**Instructor Concept**: identity management is "the processes and technologies used to provide the appropriate levels of access to our resources." In AWS terms, this spans several distinct pieces that this repository covers in order:

- **AWS IAM** — an identity *source* in its own right (you can create user accounts directly inside IAM).
- **Other identity sources** — AWS Directory Services, an on-premises Active Directory domain controller, or social identity providers (Amazon, Facebook, Google).
- **The connecting layer** — federation and single sign-on, which is what lets an identity from any of those sources actually reach AWS resources without needing a separate, duplicate identity created inside every account.

**Deeper Explanation**: the throughline across this entire course is a single question asked repeatedly in different contexts — *given an identity, wherever it actually lives, how does it get the right level of access to the right AWS resources, and nothing more?* IAM users/groups/roles, Organizations, and federation are all different answers to variations of that same question, at different scales (single account vs. many accounts) and for different identity sources (native AWS vs. external).

## 2. Two Ways to Get Hands-On Practice

`IMPORTANT`

**Instructor Concept, exact framing**: "whatever technology you're learning, there's no substitute for getting hands-on practice — it really is the best way to learn." Two practical options exist, with real tradeoffs:

| Option | You Control | Billing | Best For |
|---|---|---|---|
| **Your own free-tier AWS account** | Full control — your account, do anything with it | You're responsible for any charges; a credit card is required at signup | Following along with hands-on labs exactly as instructed, especially anything involving multiple accounts (Organizations, cross-account roles) |
| **A sandbox / challenge lab** | Limited — the account is hosted by a third-party provider, not yours | No bills at all; pay a flat fee upfront instead | Low-risk scenario practice and skills validation; **not suitable for multi-account exercises** since you're typically given only one account with limited control |

**Instructor Concept, exact limitation called out explicitly**: a sandbox specifically **cannot** be used for exercises requiring cross-account access, because you only have the one account and limited control over it — [AWS Organizations](../03-organizations/aws-organizations.md) and cross-account IAM roles genuinely need a real, multi-account setup you control.

**Real-World Consideration**: this repository's hands-on steps assume your own free-tier account, matching the instructor's own recommendation — a sandbox is a reasonable supplement for extra scenario-based practice, but not a substitute for the account you'll actually build the multi-account exercises in later ([Organizations](../03-organizations/aws-organizations.md), [Directory Services and Federation](../05-directory-services-and-federation/)).

## 3. Common Misconceptions

`IMPORTANT`

- Assuming "free tier" means zero risk of ever being charged — it means a *generous allowance* of free usage, not an absolute ceiling; anything beyond it, or any service outside the free tier's scope, bills to the card on file. [AWS Accounts](aws-accounts.md) covers the specific billing-alarm safety net this course sets up to catch that.
- Assuming a sandbox account behaves identically to a real account for every exercise — it structurally can't support multi-account scenarios, regardless of how it's configured.

## Related Topics

- [AWS Accounts](aws-accounts.md)
- [AWS Organizations](../03-organizations/aws-organizations.md)

## Try This

> Before continuing, decide which practice option you're using for this course. If it's your own account, the next file walks through creating it and setting up a billing safety net before you touch anything else.

## Progress

- [ ] I can explain what identity management covers, in my own words, across identity sources and the AWS side
- [ ] I understand why a sandbox account can't be used for cross-account exercises
