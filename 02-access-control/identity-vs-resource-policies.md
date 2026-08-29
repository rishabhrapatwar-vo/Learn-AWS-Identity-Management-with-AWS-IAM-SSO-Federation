# Identity-Based and Resource-Based Policies

> Difficulty: Intermediate
> Importance: Critical

`CORE`

## 1. Identity-Based Policies — Three Attachment Types

`CORE` `IMPORTANT`

**Instructor Concept, exact definition**: identity-based policies are JSON documents that "control the actions an identity can perform" — attached to a user, a group, or a role. There are two fundamentally different ways to attach one:

| Attachment Type | Relationship | Sharing | Deletion Behavior |
|---|---|---|---|
| **Inline policy** | One-to-one with a single user, group, or role | Cannot be shared or reused elsewhere | Deleted automatically if the user/group/role it's attached to is deleted |
| **Managed policy** (AWS managed or customer managed) | Standalone document, independently created | Can be attached to **multiple** users, groups, and roles simultaneously | Persists independently of any single identity it's attached to |

**Instructor Concept, exact distinction within managed policies**: **AWS managed** policies are created and maintained by AWS — usable in your account, but you cannot modify them. **Customer managed** policies are created and maintained by you, giving full control over the exact permission set.

**Deeper Explanation, when to choose which**: inline policies make sense for a genuinely one-off permission tightly coupled to a single identity's lifecycle; managed (especially customer-managed) policies make sense whenever the same permission set needs to apply to more than one identity, since only a managed policy can be attached to multiple users/groups/roles at once without duplicating the JSON.

## 2. Resource-Based Policies

`CORE` `IMPORTANT`

**Instructor Concept, exact definition**: a resource-based policy is a JSON document attached **directly to a resource** — an S3 bucket, a DynamoDB table, and others. Unlike an identity-based policy, it contains a **`Principal`** element specifying exactly *who* the policy grants (or denies) access to.

**Instructor Concept, exact worked example**: a bucket policy on an S3 bucket with `Effect: Allow`, `Principal: <user's ARN>`, `Action: S3:*`, `Resource: <bucket ARN>` — this grants that one specific user any S3 action, but scoped to that one bucket only. The user (`Paul`, in the demo) is then able to perform `S3:PutObject` because the *resource*, not the user's own identity policy, is what grants it.

**Instructor Concept, exact structural distinction to recognize a resource-based policy by sight**: the presence of a `Principal` element is the tell — identity-based policies never have one (they're already attached to the identity in question, so specifying "who" would be redundant); resource-based policies always need one, since the same policy document lives on the resource and must say who it applies to.

## 3. A Role's Two Policies, Revisited Through This Lens

`IMPORTANT`

**Instructor Concept, exact reframing**: recall from [STS](../01-iam-fundamentals/sts.md) that a role has a **trust policy** and a **permissions policy**. This file adds a precise classification: **the trust policy is itself an example of a resource-based policy** (it specifies *who* — which principal — is allowed to assume the role), while **the permissions policy is an example of an identity-based policy** (it defines what the role, once assumed, can actually do). A single IAM role is therefore governed by one of each policy type simultaneously — this is not a coincidence or a special case, it's the same two-policy-type system covered in this file, applied to one entity.

## 4. Hands-On: Both Policy Types Together

`HANDS-ON LAB`

**Steps, summarized from the demo, using the AWS Policy Generator tool** (see [Policy Tools](policy-tools.md) for the tool itself):

1. Create two S3 buckets, each with a couple of objects uploaded.
2. Create a user (`Paul`) with **no permissions at all**.
3. Generate and attach an **identity-based inline policy** to Paul: `Allow`, `S3:*`, scoped to one bucket's ARN. Logging in as Paul at this point: he can access objects inside that one bucket directly by URL/path, but **cannot list buckets at the account level** — a separate `S3:ListAllMyBuckets` action (with `Resource: *`) has to be added as a second statement before the bucket list view works at all. **Deeper Explanation**: this is a precise, real illustration that "full access to a specific bucket" and "the ability to see that the bucket exists in a list" are two entirely separate permissions — a common source of "why can't I see my bucket even though I have access to it" confusion.
4. Generate and attach a **resource-based policy** (an S3 bucket policy) directly to one bucket: `Effect: Deny`, `Principal: <Paul's ARN>`, `Action: S3:*`, `Resource: <bucket ARN>`.
5. Result: Paul can still **list** the bucket (the identity-side `ListAllMyBuckets` permission still applies), but gets **insufficient permissions** the moment he tries to open it — because the resource-based `Deny` on that specific bucket overrides the identity-based `Allow`, exactly per the evaluation logic in [IAM Policy Evaluation](policy-evaluation.md).

**Deeper Explanation, the takeaway from step 5**: this demo is a concrete, hands-on proof of the single most important evaluation rule in this course — **an explicit deny anywhere always wins**, regardless of how permissive any other applicable policy is. It doesn't matter that Paul's identity-based policy grants full S3 access; one resource-based deny on one specific bucket is enough to override it for that bucket alone.

## 5. Common Misconfigurations

`EXAM FOCUS` `IMPORTANT`

- Granting object-level S3 access without also granting `ListAllMyBuckets`/`ListBucket`, then being confused why the bucket doesn't appear in the console list view.
- Forgetting that a resource-based deny overrides every identity-based allow, and being surprised when a seemingly "full access" user is blocked from one specific resource.
- Using an inline policy for a permission set that's actually needed by multiple identities, duplicating the same JSON instead of creating one reusable customer-managed policy.

## Related Topics

- [How IAM Works](../01-iam-fundamentals/how-iam-works.md)
- [AWS Security Token Service (STS)](../01-iam-fundamentals/sts.md)
- [IAM Policy Evaluation](policy-evaluation.md)
- [IAM Policy Structure](policy-structure.md)
- [Policy Tools](policy-tools.md)

## Try This

> Reproduce the Paul/testbucket demo in your own account: grant a test user object-level access via an identity-based policy, confirm they can't see the bucket in the list view until you add `ListAllMyBuckets`, then add a resource-based `Deny` on the bucket and confirm it overrides the identity-based `Allow`.

## Progress

- [ ] I can distinguish inline from managed policies, and AWS-managed from customer-managed
- [ ] I can identify a resource-based policy by the presence of a `Principal` element
- [ ] I can explain why a role's trust policy is resource-based while its permissions policy is identity-based
- [ ] I can explain, from the hands-on demo, why a resource-based deny overrides an identity-based allow
