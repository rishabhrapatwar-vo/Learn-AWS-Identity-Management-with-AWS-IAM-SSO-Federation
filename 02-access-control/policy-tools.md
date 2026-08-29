# Policy Tools: AWS Policy Generator and IAM Policy Simulator

> Difficulty: Beginner
> Importance: Medium

`IMPORTANT`

Two tools make working with IAM policy JSON significantly less error-prone than hand-authoring it from scratch, and this course uses both directly.

## 1. AWS Policy Generator

`HANDS-ON LAB`

**Instructor Concept, exact purpose**: "a way that we can actually generate our own policies using a graphical interface" — rather than writing raw JSON by hand, you select a policy type (IAM policy, S3 bucket policy, and others), fill in effect/action/resource/principal through form fields, and the tool emits the corresponding JSON to copy directly into the console.

**Instructor Concept, exact demo walkthrough**, building the exact scenario used in [Identity-Based and Resource-Based Policies](identity-vs-resource-policies.md):

1. Choose **IAM policy** as the type (attaches to a user/group/role) vs. **S3 bucket policy** (attaches to the resource) — the tool explicitly separates these, reinforcing the distinction covered in that file.
2. For an identity-based policy: select the service (S3), the action(s), and supply the target resource's ARN (copied directly from the S3 console) — generate, then paste the resulting JSON into the user's inline policy JSON editor.
3. For a resource-based (bucket) policy: choosing this type in the generator surfaces a **Principal** field — confirming, tool-side, that this is what structurally distinguishes the two policy types, not just a naming convention.

**Deeper Explanation, the real value of this tool**: it's not about avoiding learning JSON — it's about avoiding the specific, easy-to-make syntax and structural mistakes (missing brackets, wrong element names, forgetting the `Principal` field on a resource-based policy) that come from typing raw policy JSON manually, especially early on.

## 2. IAM Policy Simulator

`HANDS-ON LAB` `IMPORTANT`

**Instructor Concept, exact purpose**: "a way that you can simulate the effects of IAM policies" — letting you check exactly what a given user, group, or role can and cannot do, **without logging in as that identity**. **Real-World Consideration, exact instructor framing**: this matters because, in a real organization, you frequently **can't** just log in as another user to test their permissions — the simulator gives you that visibility without needing their credentials at all.

**Instructor Concept, exact demo, using the Lindsay/permissions-boundary scenario from [Permissions Boundaries](permissions-boundaries.md)**:

1. Select a principal (user, group, or role) from the left-hand panel.
2. Select a service and either specific actions or "select all."
3. Run the simulation — results show **allow/deny per individual action**, computed against every applicable policy (identity-based, boundary, SCP, and so on) exactly as the real evaluation engine would.

**Instructor Concept, exact confirmed results**: simulating Lindsay (IAM full access + a permissions boundary excluding EC2) against IAM actions showed **all allowed**; simulating the same user against EC2 actions showed **all denied** — matching precisely what the hands-on demo in [Permissions Boundaries](permissions-boundaries.md) later confirmed by actually logging in as her and attempting to launch an instance. Simulating a different user with genuine EC2 permissions and no restricting boundary showed the expected allows.

**Deeper Explanation, why this tool complements rather than replaces the evaluation logic in [IAM Policy Evaluation](policy-evaluation.md)**: the simulator is a fast, safe way to *check the outcome* of the evaluation logic for a specific principal/action combination — but understanding *why* it produced that outcome (which policy type is doing the allowing, denying, or capping) still requires knowing the evaluation sequence and union/intersection rules that file covers. The simulator answers "what," not "why."

## 3. Common Misconfigurations

`IMPORTANT`

- Manually authoring complex policy JSON (especially resource-based policies with conditions) instead of using the generator, and introducing a structural error that's hard to spot by eye.
- Assuming a "denied" simulator result and a real login always agree without accounting for factors the simulator can't fully model (e.g., certain resource-based policy nuances or MFA-conditional statements) — the simulator is highly reliable but is still a simulation, not a substitute for a final real-world confirmation on anything genuinely sensitive.

## Related Topics

- [Identity-Based and Resource-Based Policies](identity-vs-resource-policies.md)
- [Permissions Boundaries](permissions-boundaries.md)
- [IAM Policy Evaluation](policy-evaluation.md)

## Try This

> Before applying any new policy to a real user in your own account, run it through the IAM Policy Simulator first against the affected principal and the specific actions you intend to grant or restrict — make this a standing habit rather than a one-off exercise.

## Progress

- [ ] I can explain what the Policy Generator produces and why it reduces hand-authoring errors
- [ ] I can use the Policy Simulator to check a principal's effective permissions without logging in as them
- [ ] I understand the simulator shows the evaluation *outcome*, not the *reasoning* behind it
