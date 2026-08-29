# IAM Roles: Use Cases in Practice

> Difficulty: Advanced
> Importance: Critical

`CORE` `EXAM FOCUS`

This file assumes the mechanics of roles and STS from [Users, Groups, Roles, and Policies](../01-iam-fundamentals/users-groups-roles-policies.md#4-roles) and [AWS Security Token Service (STS)](../01-iam-fundamentals/sts.md) — it walks through the three concrete use cases this course builds hands-on, in full detail.

## 1. Use Case: Cross-Account Access (Same Owner)

`CORE`

**Instructor Concept, exact scenario**: a user in Account B needs to access an S3 bucket that lives in Account A. The pattern:

1. The user has an **identity-based policy** granting `sts:AssumeRole` — this is what lets them attempt to assume anything at all.
2. A role exists in Account A with a **trust policy** naming Account B (or a specific principal within it) as allowed to assume it.
3. The role also has a **permissions policy** granting the actual bucket access (read/write, whatever's needed).
4. The user calls `sts:AssumeRole`, receives temporary credentials, and accesses the bucket **as the role**, not as themselves.

**Deeper Explanation**: this is structurally identical to what [AWS Organizations](../03-organizations/aws-organizations.md#4-what-happens-when-organizations-creates-a-new-account) already set up automatically via `OrganizationAccountAccessRole` — the difference here is **deliberately scoping the role's permissions policy narrowly** (just the one bucket) rather than granting full account access, which is what you'd actually want for anything beyond pure administrative cross-account access.

## 2. Use Case: Cross-Account Access (Third Party) — Adding an External ID

`CORE` `EXAM FOCUS`

**Instructor Concept, exact problem this solves**: when the two accounts belong to **different organizations** (a third party, not accounts you both own), you have to share the role's ARN with the external party so they can assume it. **Instructor Concept, exact risk this creates**: sharing an ARN alone isn't enough of a secret — anyone else who obtains that same ARN (e.g., if it leaks, or if another AWS customer guesses/discovers it through some other channel) could also attempt to assume the role, since ARNs aren't meant to be treated as secrets.

**Instructor Concept, exact fix**: add a **`Condition`** to the role's trust policy requiring `sts:ExternalID` to match a specific, secret value shared privately with the trusted third party — the trust policy's principal is still `AWS: <third-party account ID>`, but the condition adds `StringEquals: { sts:ExternalId: <shared secret> }`. Calling `sts:AssumeRole` without also supplying the correct external ID fails, even from an otherwise-trusted account. **Instructor Concept, exact caution on choosing the value**: "you wouldn't use one, two, three, four, five, you'd create something complex" — the external ID needs genuine entropy to function as the extra security layer it's meant to be, not a guessable placeholder.

### Hands-On: Building This End to End

`HANDS-ON LAB` `EXAM FOCUS`

**Steps, summarized from the demo** (Account A = management, Account B = production; user `Jack` in Account A needs S3 access in Account B):

1. In Account A: create user `Jack`; create and attach a policy granting him `sts:AssumeRole` (and role-listing) permission only — nothing else.
2. Generate access keys for Jack and configure a separate named CLI **profile** for him (`aws configure --profile jack`), distinct from the admin's own default profile — this lets both identities be used from the same terminal without conflict.
3. In Account B: create a role (e.g., `cross-account-s3-access`) with a trust policy specifying **Account A's account ID as principal** and **requiring an external ID** — the console explicitly labels this "a best practice when a third party will assume this role," even here where both accounts happen to be under the same person's control, as good habit-forming practice. Attach an S3 permissions policy to the role.
4. **Instructor Concept, exact detail on reading the resulting trust policy**: the principal ARN ends in `:root` — the instructor explicitly clarifies this "doesn't mean the root account... it actually means all accounts" within that account number, i.e., any identity in Account A satisfying the external-ID condition may assume the role, not literally only Account A's root user.
5. From the CLI, running `aws sts assume-role` with Jack's profile, the role's ARN, a session name, and the external ID returns temporary credentials: an **access key ID, secret access key, session token, and expiration**.
6. Export those three values as environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) — from that point, CLI commands run **without specifying any profile** automatically use the assumed role's temporary credentials instead of any configured user profile.
7. Confirm identity with `aws sts get-caller-identity` before and after — before, it reports Jack's own user in Account A; after exporting the temporary credentials, it reports the **assumed role** in Account B.
8. Cleanup: delete any test resources created, then `unset` the exported environment variables to return the CLI to using the default/named profiles again.

## 3. Use Case: Delegation to AWS Services — EC2 Instance Profiles

`CORE` `EXAM FOCUS`

**Instructor Concept, exact scenario**: an application running on an EC2 instance needs to access another AWS service (S3 in the demo, but the pattern is identical for DynamoDB, RDS, or anything else). The mechanism:

1. Create an **instance profile** and associate an **IAM role** with it.
2. EC2 assumes the role via an API call, governed by a **trust policy** whose principal is the *service* `ec2.amazonaws.com` — not a user or account, a service.
3. STS issues temporary credentials (access key, secret key, session token, expiration) to the instance.
4. The application on the instance uses those credentials to access S3, scoped exactly by the role's **permissions policy**.

### `iam:PassRole` — The Permission Everyone Forgets

`CORE` `EXAM FOCUS`

**Instructor Concept, exact and critical clarification**: a user attaching an existing role to an EC2 instance does **not** need the permissions *contained within* that role — Jack doesn't need S3 access himself just to attach an S3-Read-Only role to an instance. What he needs instead is **`iam:GetRole`** (to view/reference the role) and, critically, **`iam:PassRole`** (to actually assign/attach that role to the EC2 instance). **Deeper Explanation, why this distinction is genuinely important**: `iam:PassRole` is a narrowly-scoped, least-privilege way to let a user configure infrastructure with a role that's more powerful than the user's own permissions — without ever granting the user that power directly for themselves. This is precisely the mechanism [Permissions Boundaries](../02-access-control/permissions-boundaries.md) exists to guard against being abused for privilege escalation — an unrestricted `iam:PassRole` combined with the ability to launch compute is a real, recognized escalation path if the passed role is more privileged than intended and nothing constrains what can be passed.

### Hands-On: Building This End to End

`HANDS-ON LAB`

**Steps, summarized from the demo**:

1. Attach a policy to Jack granting exactly: create/modify/delete an instance profile, associate/disassociate an instance profile with an EC2 instance, plus `iam:GetRole` and `iam:PassRole` — **a deliberately narrow, task-scoped permission set**, explicitly called out as an illustration of least privilege in action.
2. Create a role (e.g., `S3-Read-Only`) with EC2 as the trusted service (trust policy) and the AWS-managed `AmazonS3ReadOnlyAccess` policy attached (permissions policy).
3. Launch an EC2 instance **without** assigning an IAM role at launch time (deliberately, to walk through every step manually rather than letting the console auto-create the instance profile).
4. From the CLI (as Jack): create an instance profile, add the role to it, then associate the instance profile with the running instance — the last step needs the instance's **region explicitly specified** if it differs from the CLI's configured default region (a real, easy-to-hit error in the demo).
5. Confirm in the console (**EC2 → Actions → Security → Modify IAM role**) that the instance profile is now attached.
6. SSH into the instance and run `aws s3 ls` — succeeds, using the instance's own temporarily-assumed role credentials, with **no access keys ever configured on the instance itself**.
7. **Instructor Concept, exact demonstration of the payoff**: detaching the role from the instance (via the same console path) and re-running `aws s3 ls` immediately fails, prompting for `aws configure` — proving the access genuinely depended on the attached role and not on any locally stored credential. **Exact instructor framing of why this matters**: "we don't want to store credentials in code... it is stored in clear text. So much more secure and a best practice to use IAM roles" — this is the EC2-specific version of the same plaintext-credential risk demonstrated hands-on back in [Authentication Methods](../01-iam-fundamentals/authentication-methods.md#the-local-credential-storage-warning).

## 4. Common Misconfigurations

`EXAM FOCUS` `IMPORTANT`

- Granting a user the permissions contained in a role directly, when only `iam:PassRole`/`iam:GetRole` were actually needed to attach that role to infrastructure.
- Omitting an external ID on a third-party cross-account trust policy, leaving the connection secured by ARN secrecy alone — which isn't real secrecy.
- Hardcoding access keys into an application running on EC2 instead of using an instance profile — reintroducing exactly the plaintext-credential risk roles exist to eliminate.
- Forgetting to specify `--region` on CLI commands targeting a resource outside the CLI's default configured region, producing confusing "not found" or association errors that are actually just a region mismatch.

## Related Topics

- [Users, Groups, Roles, and Policies](../01-iam-fundamentals/users-groups-roles-policies.md)
- [AWS Security Token Service (STS)](../01-iam-fundamentals/sts.md)
- [AWS Organizations](../03-organizations/aws-organizations.md)
- [Permissions Boundaries](../02-access-control/permissions-boundaries.md)

## Try This

> In your own two-account setup, reproduce the third-party cross-account access pattern with an external ID end to end via the CLI, then separately build the EC2 instance profile lab and prove — by detaching the role and re-running a command — that no credentials were ever stored on the instance itself.

## Progress

- [ ] I can explain why an external ID is added to a third-party cross-account trust policy, and what specific risk it closes
- [ ] I can explain the difference between needing a role's own permissions versus needing `iam:PassRole` to attach it to something
- [ ] I can walk through the full EC2 instance profile flow from role creation to an application accessing S3 with zero stored credentials
- [ ] I can explain why `iam:PassRole` combined with unrestricted compute-launch permission is a recognized privilege-escalation path
