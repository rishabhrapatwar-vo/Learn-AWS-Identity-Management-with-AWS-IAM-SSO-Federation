# AWS Directory Services

> Difficulty: Advanced
> Importance: High

`CORE`

## 1. AWS Managed Microsoft AD

`CORE` `IMPORTANT`

**Instructor Concept, exact definition**: a fully managed implementation of Microsoft Active Directory, running on Windows Server 2012. Creating one automatically provisions a **highly available pair of domain controllers**, deployed across **multiple subnets in multiple Availability Zones** — AWS handles the HA topology; you don't design it yourself.

### Trust Relationships With an On-Premises AD

`IMPORTANT`

**Instructor Concept, exact mechanism**: if a company already runs its own on-premises Active Directory, it can connect to it over a **VPN**, then establish a **trust relationship** between the on-premises AD and the AWS Managed Microsoft AD. A trust can be **one-way** (identities in one directory can authenticate/authorize against the other, but not vice versa) or **two-way** (both directions work). **Deeper Explanation**: a trust doesn't merge or migrate the two directories — each keeps its own identities and its own domain controllers; the trust simply lets one directory vouch for the other's authenticated identities for authorization purposes.

### What Managed Microsoft AD Connects To

`IMPORTANT`

- **EC2 instances** (Windows or Linux) can be joined to the domain directly.
- **Amazon WorkSpaces** desktops (e.g., a Windows 10 desktop) can be joined, letting users log in with a domain account.
- **Azure Active Directory and Office 365** — via two additional services: **ADSync** (identity **synchronization**) and **ADFS**, Active Directory Federation Services (the actual **federation**/identity-provider layer between the AWS-hosted AD and Azure AD/Office 365). **Exam Note**: ADSync and ADFS solve two genuinely different problems — sync keeps identities consistent across directories; federation lets one directory's authentication be trusted by another system without duplicating the identity at all. Confusing the two is an easy mistake.
- **AWS applications/services** directly — the hands-on lab specifically uses Amazon WorkSpaces to authenticate against Managed Microsoft AD and then reach the AWS Management Console via delegated access.
- **Group Policy** — fully editable and applicable, same as on-premises AD.
- **MFA via RADIUS** — note this is a **separate MFA mechanism from IAM's own MFA** (covered in [MFA and Account Security](../01-iam-fundamentals/mfa-and-account-security.md)); it requires actual RADIUS infrastructure, not a virtual/hardware IAM MFA device.

## 2. AD Connector

`CORE` `IMPORTANT`

**Instructor Concept, exact use case, explicitly contrasted with Managed Microsoft AD**: for an organization that wants to **keep its identities entirely on-premises** (no AWS-hosted directory at all) but still needs those identities to reach AWS services. AD Connector is a **proxy**, not a directory itself — it connects (via VPN or Direct Connect) to your existing on-premises Active Directory and forwards authentication/authorization requests to it.

**Instructor Concept, exact services AD Connector enables access to**: Amazon WorkSpaces, WorkDocs, WorkMail, and — critically — the **AWS Management Console itself**, via a **federated sign-in that maps Active Directory identities to IAM roles**. On-premises AD users then operate in AWS with whatever permissions that mapped role grants. AD Connector can also **seamlessly join Windows EC2 instances to the on-premises domain** directly, without needing a locally-hosted AWS directory at all.

**Deeper Explanation, the core decision between the two services**: Managed Microsoft AD is the right choice when you want AWS to **host and run** the directory itself (new deployment, or extending an existing one via trust); AD Connector is the right choice when the directory must **stay entirely on-premises** and AWS only needs a proxy/pass-through into it. This decision — host a new/extended directory in AWS vs. proxy into an existing on-premises one — recurs across this course's federation material and is worth deciding deliberately rather than defaulting to whichever is more familiar.

## 3. Hands-On: Building a Managed Microsoft AD Environment

`HANDS-ON LAB` `💰 COST NOTE`

**Steps, summarized across the two-part demo**:

1. **Create the directory** (Directory Service console → Set up directory → AWS Managed Microsoft AD, Standard edition) — supply a **private** DNS name (not internet-facing) and an admin password. **Instructor Concept, exact cost detail**: roughly **$86/month** for the two domain controllers, though free-tier eligible with a 30-day limited trial — the instructor explicitly warns to **terminate the directory** after finishing the related labs to avoid ongoing charges. Creation takes roughly 20–45 minutes.
2. While waiting, create two supporting **IAM roles**: one (`EC2DomainJoin`) with `AmazonSSMDirectoryServiceAccess` + `AmazonSSMManagedInstanceCore`, used to join an EC2 instance to the domain at launch; another (`AD-PowerUser`), trusted by the *directory service itself* (not EC2), granting a delegated console-access user PowerUser-level permissions.
3. Once active, enable an **application access URL** (a public, internet-reachable endpoint for directory-integrated apps) and enable **AWS Management Console access** for the directory, associating it with the `AD-PowerUser` role created above.
4. Launch a **Windows Server EC2 instance**, selecting the directory and the `EC2DomainJoin` role under "domain join directory" at launch time — this joins the instance to the domain automatically as part of boot.
5. RDP into the instance as `<domain>\admin`, install the **Remote Server Administration Tools** (Server Manager → Add Roles and Features → AD DS and AD LDS tools) — these aren't present by default even though the instance is domain-joined.
6. Using **Active Directory Users and Computers**, create a user (`Jennifer`), assign an email address. **Instructor Concept, exact observation on managed-service limits**: default OUs like "Users" have **Create User/Group/OU grayed out** — you can't create new objects there directly; **full control is only available within the OU named after your own domain** (created specifically for you), reflecting that this is a genuinely managed service with deliberately locked-down defaults elsewhere in the tree.
7. **Delegate console access to Jennifer**: in the Directory Service console's "Delegate console access" setting, add Jennifer against the `AD-PowerUser` role created in step 2.
8. Register the directory with **Amazon WorkSpaces**, then launch a Windows 10 WorkSpace assigned to Jennifer.
9. Log into the WorkSpace as Jennifer (domain credentials), then open the directory's Management Console access URL from inside that WorkSpace session — **confirmed working**: Jennifer reaches the AWS Management Console, authenticated entirely through her Active Directory identity, authorized via the delegated `AD-PowerUser` role.

**Deeper Explanation, why this specific lab matters conceptually**: it's the fully manual, ground-level version of what [IAM Identity Center](iam-identity-center.md) later automates and generalizes — a directory-based identity reaching the AWS console through a role mapping, but done here with one directory, one role, and one account, before SSO's multi-account, multi-application layer is introduced on top of the exact same directory.

## 4. Common Misconfigurations

`IMPORTANT` `💰 COST NOTE`

- Leaving a Managed Microsoft AD directory (and its associated WorkSpaces/EC2 resources) running after finishing hands-on labs — this is explicitly **not** a fully free-tier-safe configuration to leave idle.
- Choosing AD Connector when the actual requirement is to host new identities in AWS (no existing on-premises AD to proxy into) — AD Connector only works when there's a real directory on the other end of the VPN/Direct Connect connection.
- Confusing ADSync (identity synchronization) with ADFS (identity federation) as if they solved the same problem — they solve genuinely different ones and are typically used together, not interchangeably.
- Expecting full administrative control over every OU in a Managed Microsoft AD tree — several default OUs are intentionally locked down as part of the managed-service model.

## Related Topics

- [Identity Federation](identity-federation.md)
- [IAM Identity Center](iam-identity-center.md)
- [MFA and Account Security](../01-iam-fundamentals/mfa-and-account-security.md)

## Try This

> If you have the budget/free-tier allowance for it, build the Managed Microsoft AD lab end to end in your own account — the "OU with full control vs. locked-down default OUs" observation and the delegated console-access flow are much clearer once seen directly rather than read about. Remember to terminate the directory afterward.

## Progress

- [ ] I can explain the difference between AWS Managed Microsoft AD (AWS-hosted) and AD Connector (proxy to on-premises)
- [ ] I can distinguish ADSync (synchronization) from ADFS (federation) and explain what each actually does
- [ ] I can walk through how a directory user ends up with delegated AWS Management Console access, from domain user to IAM role
- [ ] I know this directory service is a real, ongoing cost and must be explicitly terminated after use
