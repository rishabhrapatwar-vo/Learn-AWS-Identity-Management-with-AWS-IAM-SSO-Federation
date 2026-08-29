# IAM Policy Structure

> Difficulty: Beginner–Intermediate
> Importance: High

`CORE`

## 1. The Four Core Elements

`CORE`

**Instructor Concept, exact structure**: an IAM policy is a JSON document consisting of one or more **statements**, each built from up to four elements:

| Element | Required? | Meaning |
|---|---|---|
| **Effect** | Required | `Allow` or `Deny` |
| **Action** | Required | The specific API action(s) the statement grants or denies permission for |
| **Resource** | Required | The specific resource (by ARN) the action applies to |
| **Condition** | Optional | Further restricts *when* the statement applies |

## 2. Four Worked Examples

`CORE` `EXAM FOCUS`

### Example 1 — Full Wildcard (AdministratorAccess)

```
Effect: Allow, Action: *, Resource: *
```

**Instructor Concept, exact reading**: allows any action on any resource — the simplest possible policy statement, and also the most privileged one possible.

### Example 2 — Specific Action + Condition (IP-Restricted Deny)

```
Effect: Deny, Action: ec2:TerminateInstances, Resource: *,
Condition: { NotIpAddress: { aws:SourceIp: [...] } }
```

**Instructor Concept, exact reading**: the action is scoped precisely (`ec2:TerminateInstances`, not a wildcard), the resource is `*`, but a **condition** checks the request context's source IP address — the effect *denies* the action unless the request originates from a specific, whitelisted IP range. **Deeper Explanation, why `NotIpAddress` rather than `IpAddress`**: the deny is written as "deny if NOT in the allowed range" — this is a common, deliberate pattern for restricting a dangerous action (terminating instances) to a known network (e.g., a corporate office) without needing a separate allow statement elsewhere; the condition itself does the restricting work directly within a deny.

### Example 3 — Resource-Based Policy With a Principal

```
Effect: Allow, Principal: *, Action: elasticfilesystem:[read/write actions],
Resource: <EFS file system ARN>, Condition: { Bool: { aws:SecureTransport: true } }
```

**Instructor Concept, exact reading**: the `Principal` element is what marks this as a **resource-based** policy (see [Identity-Based and Resource-Based Policies](identity-vs-resource-policies.md)) — here `Principal: *` grants read/write access to **all** principals, but the `Condition` requires `aws:SecureTransport` to be `true`, meaning the connection must be encrypted (SSL/TLS) — an unencrypted connection attempt is denied regardless of who's making it.

### Example 4 — Policy Variables (Per-User S3 Folder)

```
Effect: Allow, Action: s3:*, Resource: arn:aws:s3:::mybucket/${aws:username}/*
```

**Instructor Concept, exact mechanism**: the `${aws:username}` **policy variable** is substituted at evaluation time with the requesting principal's own friendly name — so one single policy, attached to a group or applied broadly, automatically scopes each user to *their own* folder within a shared bucket (e.g., the instructor's own account substitutes to `mybucket/Neal/*`), without needing a separate, hand-written policy per user.

**Deeper Explanation, why this matters at scale**: policy variables are what let a single reusable policy do per-identity scoping automatically — the alternative (a distinct, manually maintained policy statement per user) doesn't scale past a handful of users and is exactly the kind of maintenance burden ABAC and policy variables both exist to eliminate.

## 3. Common Misconfigurations

`EXAM FOCUS` `IMPORTANT`

- Using a wildcard `Resource: *` when a specific ARN would have scoped the statement correctly — broader than necessary, violating least privilege for no functional gain.
- Writing a condition using the wrong logical sense (e.g., `IpAddress` when `NotIpAddress` was actually needed, or vice versa), silently inverting the intended restriction.
- Forgetting the `Principal` element is what distinguishes a resource-based policy from an identity-based one — omitting it (or including it where it doesn't belong) produces a structurally invalid or misapplied policy.

## Related Topics

- [Identity-Based and Resource-Based Policies](identity-vs-resource-policies.md)
- [IAM Policy Evaluation](policy-evaluation.md)
- [Policy Tools](policy-tools.md)

## Try This

> Write a policy statement (by hand, not with the generator) granting a test user full S3 access only to a folder matching their own username inside one shared bucket, using the `${aws:username}` policy variable — then create two test users and confirm each can only reach their own folder.

## Progress

- [ ] I can name the four policy elements and which are required vs. optional
- [ ] I can read and explain a condition-based policy statement, including negated conditions like `NotIpAddress`
- [ ] I can explain what a policy variable does and why it scales better than per-user hand-written policies
