# IAM Identity Center (AWS SSO)

> Difficulty: Advanced
> Importance: Critical

`CORE` `EXAM FOCUS`

## 1. What It Is, and Its Relationship to "AWS SSO"

`CORE`

**Instructor Concept, exact framing**: "IAM Identity Center is the successor to the AWS Single Sign-On service — it's basically the same product but renamed." It provides **centralized permissions management and single sign-on** — not just to AWS services, but to **other AWS accounts and business applications** as well. **Instructor Concept, exact strategic note**: "AWS is steering us towards using Identity Center instead of IAM for many use cases" — this is presented as the direction AWS itself is pushing organizations, not merely one option among equals.

## 2. Identity Sources Identity Center Supports

`IMPORTANT`

- **Identity Center's own built-in directory** — you can create user accounts directly within Identity Center itself, making it a genuine identity source in its own right, not only a federation broker.
- **Active Directory** — via **AD Connector** or **AWS Managed Microsoft AD**, depending on whether the directory is on-premises or AWS-hosted (see [AWS Directory Services](aws-directory-services.md)).
- **Standard SAML 2.0 identity providers** more generally.

## 3. IAM Identity Center vs. IAM — A Direct Comparison

`CORE` `EXAM FOCUS`

**Instructor Concept, exact comparison, point by point**:

| Dimension | IAM | IAM Identity Center |
|---|---|---|
| **Core purpose** | Managing access to AWS services and resources | Centralized identity management and SSO — to AWS services, other accounts, *and* external business applications |
| **Federation** | Supports federation to external IdPs via SAML/OIDC ([Identity Federation](identity-federation.md)) — described as "more of a legacy configuration today" | Built-in federation with external IdPs, streamlined specifically for ease of setup |
| **Multi-account access** | Requires a more complex setup — assuming roles across accounts manually | Easier — a single login grants access to multiple accounts and applications directly |
| **Business application integration** | Limited; requires custom setup | Purpose-built — many pre-built SSO integrations (the instructor's examples: Salesforce, Office 365) |
| **Fine-grained control** | Very detailed, programmatic, policy-based access control | Centralized, user-experience-focused (a user portal), with permission sets doing the fine-grained work |

**Deeper Explanation, the practical decision rule**: reach for plain IAM federation when you specifically need programmatic, deeply customized policy control over AWS resources within a fairly contained scope. Reach for Identity Center when the actual requirement is **centralized SSO across many accounts and many applications, AWS and non-AWS alike** — which, per the instructor's own framing, is most enterprise scenarios today.

## 4. Permission Sets — How Access Actually Gets Granted

`CORE` `EXAM FOCUS`

**Instructor Concept, exact mechanism**: access levels in Identity Center are controlled by **permission sets** — essentially a named bundle of policy (an existing AWS-managed job function policy, or a custom one), which is then **assigned to a user (or group) against a specific AWS account**. **Instructor Concept, exact clarification on the assignment flow, called out because it's not obvious**: "how do we go about now assigning the permission set to the account? Well, you actually do it via a user... it's the user that's going to get the permissions" — you don't attach a permission set directly to an account in isolation; the actual grant happens by selecting the account, then the user (or group), and then the permission set together, as one combined assignment.

## 5. Hands-On: SSO Across Accounts, Backed by Active Directory

`HANDS-ON LAB` `EXAM FOCUS`

**Steps, summarized from the demo, building directly on the Managed Microsoft AD lab in [AWS Directory Services](aws-directory-services.md)**:

1. **Enable AWS SSO / Identity Center** (one region at a time — it's not multi-region by default).
2. **Change the identity source** from Identity Center's own built-in directory to the **existing Active Directory** already connected via Managed Microsoft AD — confirming Identity Center layers cleanly on top of a directory source already in place, rather than requiring identities to be recreated inside it.
3. Under **AWS Organizations accounts**, both the management and a second (production) account are visible — because the whole Organization is already set up (see [AWS Organizations](../03-organizations/aws-organizations.md)).
4. **Create a permission set** using an existing AWS-managed job function policy (the demo's example: **Data Scientist**, granting access to AWS data analytics services).
5. **Assign** that permission set to the target (production) account, selecting the Active-Directory-sourced user (`Jennifer`, the same user created in the directory-services lab) as the recipient.
6. **Confirmed result**: Jennifer, logged into her Amazon WorkSpaces desktop (authenticated via Active Directory, per [AWS Directory Services](aws-directory-services.md)), opens the Identity Center **user portal URL** (`<directory>.awsapps.com/start`), signs in, and sees exactly **one published application** — the production AWS account. Clicking through offers a choice of **Management Console** or **programmatic/CLI access**; choosing Management Console logs her directly into the **production account** (not the management account her directory lives in), with exactly the Data Scientist permission set's access — confirmed by successfully reaching data-analytics-oriented services like Amazon Athena.

**Deeper Explanation, why this demo is the payoff moment for the whole federation module**: this single login flow — one Active-Directory-authenticated user, reaching a *different AWS account entirely*, with precisely scoped permissions, via one URL and zero manual role-assumption steps on the user's part — is the concrete difference the comparison table in §3 is describing in the abstract. The user never ran `sts:AssumeRole`, never touched a role ARN, never needed an external ID; Identity Center handled all of that federation/assumption machinery behind the single sign-on experience.

## 6. Built-In Business Application Integrations

`IMPORTANT`

**Instructor Concept, exact detail**: Identity Center's application catalog includes many pre-built SSO integrations for common business applications (the instructor's example shown live: Cloud Conformity) — adding one typically just requires supplying account-specific configuration details for that application, rather than building a SAML/OIDC integration from scratch the way plain IAM federation would require.

## 7. Common Misconfigurations

`EXAM FOCUS` `IMPORTANT`

- Attempting to assign a permission set directly "to an account" without realizing the actual mechanism routes through selecting a user or group first.
- Forgetting Identity Center is enabled per-region, and being confused when it doesn't appear active elsewhere.
- Building a custom IAM SAML/OIDC federation setup for a straightforward multi-account SSO requirement, when Identity Center was purpose-built for exactly that and would have been meaningfully simpler to set up.
- Leaving the Managed Microsoft AD, WorkSpaces, and Identity Center configuration running after finishing labs — the same cost discipline flagged in [AWS Directory Services](aws-directory-services.md) applies here too, since this lab builds directly on that infrastructure.

## Related Topics

- [Identity Federation](identity-federation.md)
- [AWS Directory Services](aws-directory-services.md)
- [AWS Organizations](../03-organizations/aws-organizations.md)
- [AWS Security Token Service (STS)](../01-iam-fundamentals/sts.md)

## Try This

> If you completed the Managed Microsoft AD lab, continue directly into this one: enable Identity Center, switch its identity source to your existing directory, create a permission set, and assign it to your production account for the same test user — then log in through the SSO portal URL and confirm you land directly in the target account with exactly the expected access.

## Progress

- [ ] I can explain why Identity Center is described as AWS's currently preferred direction over plain IAM federation for most enterprise use cases
- [ ] I can complete the comparison table between IAM and IAM Identity Center from memory
- [ ] I can explain the permission-set assignment flow, including that it routes through a user/group, not directly to an account
- [ ] I can walk through the full hands-on flow from Active-Directory-authenticated login to landing in a different AWS account with scoped permissions
