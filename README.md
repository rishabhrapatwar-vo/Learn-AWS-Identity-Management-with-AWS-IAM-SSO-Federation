# AWS Identity Management: IAM, SSO & Federation

A self-learning reference built from the "Learn AWS Identity Management with AWS IAM, SSO & Federation" course. Each lesson is transformed into a deeper, first-principles explanation — grounded in what the instructor actually taught (marked as **Instructor Concept**), expanded with additional context, diagrams, and real-world framing where it helps understanding.

## How to Use This Repository

- Read folders in order (`00` → `05`) — later topics build on earlier ones (roles assume you understand policies; federation assumes you understand roles).
- Each file ends with a **Progress** checklist — use it to self-assess before moving on.
- **Try This** sections at the end of each file suggest a hands-on action in your own AWS free-tier account — identity management is not a spectator subject.

## Structure

| Folder | Covers |
|---|---|
| [00-getting-started](00-getting-started/) | AWS accounts, the root user, free tier vs. sandbox practice, initial account hardening |
| [01-iam-fundamentals](01-iam-fundamentals/) | How IAM works, principals (users/groups/roles), authentication methods, STS, MFA |
| [02-access-control](02-access-control/) | Identity vs. resource policies, RBAC vs. ABAC, permissions boundaries, policy evaluation logic, policy structure and tooling |
| [03-organizations](03-organizations/) | AWS Organizations, multi-account structure, Service Control Policies |
| [04-iam-roles](04-iam-roles/) | IAM role use cases — cross-account access, EC2 instance profiles |
| [05-directory-services-and-federation](05-directory-services-and-federation/) | AWS Directory Services, identity federation, IAM Identity Center (AWS SSO), Amazon Cognito |
| [reference](reference/) | Glossary and quick-reference material |

## A Note on Labels

- `CORE` — foundational, everything downstream depends on this
- `IMPORTANT` — commonly used in real environments
- `HANDS-ON LAB` — the lesson was a click-through demo in the source course; summarized as a repeatable set of steps here rather than transcribed screen-by-screen
- `💰 COST NOTE` — where a step can incur real charges outside the free tier
