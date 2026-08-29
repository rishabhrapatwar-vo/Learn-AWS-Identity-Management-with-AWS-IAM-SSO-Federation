# Multi-Factor Authentication and Initial Account Security

> Difficulty: Beginner
> Importance: Critical

`CORE`

## 1. The Three Factors of Authentication

`CORE`

**Instructor Concept, exact framework**: authentication factors fall into three categories:

1. **Something you know** — a password. Proves identity because (in theory) only you know it.
2. **Something you have** — a physical device. Proves identity because only you possess it.
3. **Something you are** — biometrics (retinal scan, fingerprint). **Instructor Concept, exact note**: AWS does not use this factor today; only the first two are in play.

**Instructor Concept, exact definition of MFA**: using **two of these factors together** — specifically, in AWS, a password (something you know) plus a code from a device (something you have). A username alone is "not particularly secret" and doesn't count as a factor; the password is what actually proves identity via "something you know."

## 2. Virtual vs. Physical MFA Devices

`IMPORTANT`

| Type | Mechanism | Cost |
|---|---|---|
| **Virtual MFA device** | An authenticator app (the instructor uses Google Authenticator) generating a time-based rotating code, one entry per AWS account registered | Free |
| **Physical MFA (hardware token / U2F security key)** | A physical device generating or holding the code | Requires purchase |

**Instructor Concept, exact best practice, stated directly**: "it is an AWS best practice to enable MFA" — reasoning given explicitly: passwords alone do get compromised sometimes, but the odds of *both* your password *and* your separate physical device being compromised together are far lower.

## 3. Hands-On: Enabling MFA on a User

`HANDS-ON LAB`

**Steps, summarized from the demo**: from the user's **Security Credentials** tab → **Assigned MFA device** → **Manage** → choose **Virtual MFA device** → scan the presented QR code with an authenticator app → enter **two consecutive codes** from the app (proving the pairing actually works, since a single code could be a fluke) → **Assign MFA**. From that point on, signing in as this user requires the password *and* the current authenticator code.

## 4. Access Key Hygiene

`IMPORTANT` `EXAM FOCUS`

**Instructor Concept, exact rules and demo observations**:

- A user can hold **up to two access keys at once**, no more. **Instructor Concept, exact reasoning this limit supports**: this is precisely what makes safe **key rotation** possible — create the second key, migrate everything using the first key over to it, then deactivate/delete the old one, with a brief overlap window rather than a hard cutover.
- If a key is suspected compromised, you can **deactivate it immediately** — once deactivated (and any STS-derived temporary credentials from it expire), it can no longer authenticate anything, without needing to delete it outright first.
- **Rotating access keys periodically is a stated security best practice** — the same logic as periodic password rotation, applied to programmatic credentials.

## 5. Password Policy — Configuring It for All Users

`IMPORTANT`

**Instructor Concept, exact configurable options, set at the account level under Account Settings**: minimum character length; requiring uppercase letters and special characters; **password expiration** (a maximum age before a forced change); whether users are **allowed to change their own password**; and **password reuse prevention** (how many previous passwords are remembered and blocked from reuse). **Real-World Consideration, exact framing**: this matters specifically once an account has many users signing in — a consistent, enforced password policy is what keeps "weak, easily guessed password" from being an individual user's unilateral choice.

## 6. The One Thing Easy to Forget: Root MFA

`CORE` `EXAM FOCUS`

**Instructor Concept, exact closing warning**: after walking through enabling MFA for the day-to-day IAM user, the instructor explicitly flags that **MFA was never enabled for the root user** in the demo — and calls this out as a mistake to correct: "it is a best practice that you do enable MFA for the root account as well." **Deeper Explanation, why this is easy to miss and important not to**: once you've switched to using an IAM user day-to-day (per [Users, Groups, Roles, and Policies](users-groups-roles-policies.md)), the root user is used so rarely it's easy to forget it even exists as an active login path — but it remains the single most privileged identity in the account, with no permission boundary possible on it at all, making its own credential security the highest-value target of all.

## 7. Common Misconfigurations

`EXAM FOCUS` `IMPORTANT`

- Enabling MFA on IAM users but forgetting the root user — leaving the account's single most powerful identity protected by password alone.
- Never setting an account-wide password policy, leaving password strength entirely up to individual user discretion.
- Letting access keys accumulate indefinitely without rotation, or without deactivating ones no longer in active use.
- Creating a third access key attempt and being confused by the two-key limit — the fix is always rotate-then-delete, not "why won't this let me create more."

## Related Topics

- [Authentication Methods](authentication-methods.md)
- [Users, Groups, Roles, and Policies](users-groups-roles-policies.md)
- [AWS Accounts](../00-getting-started/aws-accounts.md)

## Try This

> In your own account: enable MFA on your day-to-day IAM user first, confirm it's required on your next sign-in, then go back and enable MFA on the root user too — don't leave it for later, since it's the step this lesson explicitly flags as easy to forget.

## Progress

- [ ] I can name the three authentication factors and which two AWS actually uses
- [ ] I can explain why the two-access-key limit is a deliberate design choice supporting safe rotation, not an arbitrary restriction
- [ ] I've enabled MFA on both my day-to-day IAM user and my account's root user
