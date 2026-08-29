# IAM Authentication Methods

> Difficulty: Beginner
> Importance: High

`CORE`

## 1. Console Access — Username, Password, Optional MFA

**Instructor Concept, exact flow**: a user (the instructor's example: `John`) authenticates to the Management Console with a **username and password**, optionally followed by an **MFA token** if configured (see [MFA and Account Security](mfa-and-account-security.md)). Success means the user is authenticated and can perform whatever operations their attached policies authorize in the console UI.

## 2. Programmatic Access — Access Keys

`CORE` `IMPORTANT`

**Instructor Concept, exact definition**: the CLI and the API are both **programmatic** access methods, and both authenticate using **access keys** rather than a username/password. An access key is composed of **two parts**: the **access key ID** and the **secret access key** — both are required together to authenticate. Access keys are associated with a specific user account, and **anyone holding both parts can act as that user**, performing anything that user is permitted to do.

**Exam/Real-World Note**: this is precisely why access keys are treated with the same seriousness as a password — a leaked access key ID *and* secret access key pair is a full account compromise for whatever that key's user can do, not a partial one.

## 3. Other, Less Common Authentication Methods

`IMPORTANT`

**Instructor Concept, exact list of additional configurable credential types on a user account**:

- **Signing certificate (X.509)** — used for legacy SOAP and CLI access to certain services (the instructor's example: Amazon EC2's older interfaces).
- **SSH and HTTPS Git credentials** — used to authenticate to **AWS CodeCommit** the way you would to any Git remote.
- **Amazon Keyspaces credentials** — used specifically for authenticating to Amazon Keyspaces.

**Deeper Explanation**: the common thread across every method here — password, access keys, signing certificate, Git credentials — is that they're all just different *proofs of identity* for the same underlying principal; once authenticated by any of them, the exact same IAM authorization logic ([How IAM Works](how-iam-works.md)) decides what that principal can actually do. The credential type never changes the permission model, only the mechanism of proving who's asking.

## 4. Hands-On: Creating a User, a Group, and Configuring the CLI

`HANDS-ON LAB`

**Steps, summarized from the demo**:

1. **Create a group first** (e.g., `Admin`), and attach the AWS-managed **`AdministratorAccess`** policy to it. **Instructor Concept, exact JSON observation**: this policy's core statement is `Effect: Allow`, `Action: *`, `Resource: *` — both wildcards, meaning "allow any action on any resource." The instructor is explicit that this is a huge amount of privilege, appropriate only for users who genuinely need it.
2. **Create a user** (e.g., named for yourself), choosing **console access** with a custom password (and deselecting "require password reset at next sign-in" for a lab account), then **add the user to the Admin group** created in step 1.
3. Confirm on the user's summary page that the `AdministratorAccess` permission shows as **inherited from the group**, not attached directly — proof the group-based model from [Users, Groups, Roles, and Policies](users-groups-roles-policies.md) is working as described.
4. **Generate an access key** from the user's Security Credentials tab, for CLI use. **Instructor Concept, exact warning**: the secret access key is shown **exactly once** — if you close the dialog without saving/downloading it, it cannot be retrieved again; you'd have to delete that key and create a new one.
5. **Configure the CLI**: run `aws configure`, supplying the access key ID, secret access key, and a default region. Confirm it worked by running a simple command (the demo used `aws s3 ls`) — before configuration this fails with a "unable to locate credentials" error; after, it succeeds (returning nothing if there are no buckets, but critically, no error).
6. **Log out of root and log back in as the new IAM user**, using the account ID or account alias plus the new username and password — from this point on, the new user is what gets used for the rest of the course, matching the "stop using root" guidance from [AWS Accounts](../00-getting-started/aws-accounts.md).

### The Local Credential Storage Warning

`IMPORTANT`

**Instructor Concept, exact demo and warning**: after running `aws configure`, the credentials are written to local files (`~/.aws/config` and `~/.aws/credentials`). The instructor explicitly `cat`s the credentials file to show the access key ID and secret access key sitting there **in plain text, unencrypted, on disk**. **Exact quote-level takeaway**: "anyone who compromises your machine or application can then compromise this information" — access keys configured locally (or, worse, hardcoded into application source code) are a direct, tangible compromise path, not a theoretical one. This is the practical, hands-on version of the reasoning behind preferring roles (short-lived, auto-expiring credentials) over long-lived user access keys wherever a workload can use one instead.

## 5. Common Misconfigurations

`EXAM FOCUS` `IMPORTANT`

- Attaching `AdministratorAccess` (or any policy) directly to individual users out of convenience instead of via a group.
- Hardcoding access keys into application source code or committing them to version control — the exact plaintext-on-disk risk demonstrated above, but persisted somewhere far more exposed.
- Never rotating access keys, leaving a long-lived credential valid indefinitely even if its security posture should have degraded over time.
- Losing a secret access key at creation time and being unable to retrieve it — requiring key deletion and recreation, which is avoidable simply by downloading/saving it immediately when first shown.

## Related Topics

- [How IAM Works](how-iam-works.md)
- [Users, Groups, Roles, and Policies](users-groups-roles-policies.md)
- [AWS Security Token Service (STS)](sts.md)
- [MFA and Account Security](mfa-and-account-security.md)

## Try This

> After configuring the CLI in your own account, locate and open `~/.aws/credentials` in a text editor to see the plaintext access key for yourself — then consider what happens to that file if your laptop is ever compromised. This single observation is the practical argument for preferring roles over long-lived keys wherever possible.

## Progress

- [ ] I can name the two components of an access key and explain why both are needed
- [ ] I can list at least two authentication methods beyond username/password and access keys
- [ ] I've created a user, a group, and configured the CLI in my own account, and confirmed the access key file is stored in plaintext locally
