# Azure Hands-On Lab Notebook

**Path:** AZ-900 → AZ-104 → AI-200 → AZ-305
**Credit window:** ₹19,109.25 remaining · expires **18 Oct 2026** (29 days from today, 19 Sep 2026)
**Status:** Living document — new modules get added as you clear each one. Ask questions inline; I'll expand or correct sections as we go.

---

## Table of Contents

- [Part 0 — How to Use This Notebook](#part-0)
- [Part 1 — Your Certification Path, Mapped to the Labs](#part-1)
- [Part 2 — Ground Rules for Spending ₹19,109 in 29 Days](#part-2)
- [Module 1 — Foundations: Subscriptions, RGs, Regions, Tags, CLI](#module-1)
- [Module 2 — Identity & RBAC](#module-2)
- [Extension Lab A — Governance: Policy, Locks & Budgets](#ext-a)
- [Module 3 — Networking](#module-3)
- [Extension Lab B — Private Endpoint, Private DNS & Network Watcher](#ext-b)
- [Module 4 — Virtual Machines](#module-4)
- [Extension Lab C — Availability Zones & VM Scale Sets](#ext-c)
- [Module 5 — Storage](#module-5)
- [Extension Lab D — Storage Firewall, AzCopy & Data Protection](#ext-d)
- [Extension Lab G — Application Security Groups, Effective Rules & Stored Access Policies](#ext-g)
- [Module 6 — Load Balancing](#module-6)
- [Module 7 — App Hosting & Containers (AI-200 bridge)](#module-7)
- [Extension Lab E — App Service Slots, Scaling & ACI](#ext-e)
- [Module 8 — Security (Key Vault, Managed Identity)](#module-8)
- [Module 9 — Monitoring (Azure Monitor, KQL)](#module-9)
- [Module 10 — Backup & Recovery](#module-10)
- [Extension Lab F — Site Recovery: Region-Level DR](#ext-f)
- [Module 11 — Infrastructure as Code: ARM Templates & Bicep](#module-11)
- [Module 12 — Capstone](#module-12)
- [Appendix A — CLI Cheat Sheet (growing)](#appendix-a)
- [Appendix B — Naming & Tagging Convention](#appendix-b)
- [Appendix C — Glossary](#appendix-c)
- [Appendix D — Cost Tracking Log](#appendix-d)
- [Appendix E — Industry Best Practices: What's Real, What's Simplified](#appendix-e)

---

<a id="part-0"></a>
## Part 0 — How to Use This Notebook

1. Work top to bottom. Each module has: **concepts → portal steps → CLI steps → self-check → cleanup.**
2. Do the portal steps *and* the CLI steps for the same resource, not just one. AZ-104 exams and real jobs expect both.
3. Answer the self-check questions before looking at the answers. If you get one wrong, that's the thing to ask me about — not the whole module.
4. Never skip the cleanup step at the end of a module. That's what keeps this whole plan inside your credit.
5. Bring doubts back here as you hit them — "why did the peering not connect" is a better question mid-lab than a generic "explain VNets again."

---

<a id="part-1"></a>
## Part 1 — Your Certification Path, Mapped to the Labs

### 1.1 AZ-900 — Azure Fundamentals
Not lab-heavy. If you haven't already, skim it in parallel with Module 1 — it's vocabulary (IaaS/PaaS/SaaS, regions, SLAs, pricing models), not hands-on skill. Don't spend credit "practicing" AZ-900 — nothing in it needs a live resource.

### 1.2 AZ-104 — Azure Administrator Associate
**This is where ~80% of your 29 days and credit go.** Modules 1–10 below are built directly off the AZ-104 skills-measured outline: identity, governance, storage, compute, networking, and monitoring. This is the exam where hands-on Azure time has the highest payoff per rupee.

### 1.3 AI-200 — verified, and it's not what the name suggests
I checked this against Microsoft's current certification pages before writing this notebook, because "AI-200" isn't the exam I'd have guessed from memory. Confirmed facts:

- Official title: **"AI-200: Developing AI Cloud Solutions on Azure"**
- Certification: **Microsoft Certified: Azure AI Cloud Developer Associate**
- It **replaces AZ-204** (the old "Developing Solutions for Microsoft Azure" exam) — Microsoft's own certification page frames this as simplifying the developer path.
- It is **not** the Cognitive Services / Azure OpenAI exam — that's still **AI-102**, a separate credential.
- Skills measured: containerized app hosting (Azure Container Registry, Container Apps, AKS, KEDA), AI-oriented data services (Cosmos DB for NoSQL, Azure Database for PostgreSQL with pgvector, Azure Managed Redis), backend integration (Service Bus, Event Grid, Azure Functions), and security/observability (Key Vault, App Configuration, OpenTelemetry, KQL) — all via Python and Azure SDKs.

So your actual path is **Admin (AZ-104) → AI-flavored Developer (AI-200) → Architect (AZ-305)**, which is a coherent, real progression — just not the "Cognitive Services in the middle" path the name might suggest. Module 7 below is deliberately built as a bridge into AI-200 territory (Container Apps + ACR) so you're not starting that exam completely cold.

### 1.4 AZ-305 — Solutions Architect Expert
Mostly design judgment on top of AZ-104 knowledge, not new hands-on surface area. Once Modules 1–11 are done, the Capstone (Module 12) doubles as your AZ-305 warm-up: you'll be making the same trade-off decisions (which SKU, which tier, single-region vs. multi-region, cost vs. resilience) that AZ-305 tests.

---

<a id="part-2"></a>
## Part 2 — Ground Rules for Spending ₹19,109 in 29 Days

**The rule: spend on breadth of experience, not uptime.**

```
Create → configure → experiment → break → troubleshoot → delete
```

There's no educational difference between running something for 2 hours vs. 10 days — the second option just burns credit.

**Daily spend bands** (rough mental limits):

| Day type | Spend |
|---|---|
| Normal lab day | ₹0 – 300 |
| Heavy lab day | ₹300 – 800 |
| Expensive-service experiment (Firewall, Bastion, App Gateway) | ₹500 – 1,500, same day teardown |
| Keep as buffer until final week | ₹3,000 – 5,000 |

**Resource groups are your cleanup unit.** One RG per module/lab, named so you know what's in it, deleted whole when you're done:

```bash
az group delete --name rg-<module>-lab --yes
```

**Check spend daily** — Portal → Cost Management + Billing → Cost analysis. Microsoft notes usage data can lag 1–2 days, so a ₹0 reading doesn't guarantee a resource is free.

**Don't leave running between sessions:** Azure Firewall, VPN Gateway, Bastion, Application Gateway, larger VMs, AKS worker nodes, SQL Managed Instance, paid DB tiers. These are hourly-billed regardless of use.

**What this notebook covers, and what it doesn't.** Modules 1–12 give you a real, hands-on core across all five AZ-104 skill areas (identity/governance, compute, storage, networking, monitoring/recovery), and the six **Extension Labs (A–F)**, slotted between the modules, close several of the biggest remaining gaps: Azure Policy, resource locks, and budgets (Extension Lab A); private endpoints, private DNS zones, and Network Watcher's topology view (Extension Lab B — its IP Flow Verify and Effective Security Rules tools still need a real VM as the test source, so come back and run those against Module 4's `vm-app` once it exists); Availability Zones and VM Scale Sets (Extension Lab C); the storage account firewall, AzCopy, and blob versioning/soft delete (Extension Lab D); App Service deployment slots, scaling, and Azure Container Instances (Extension Lab E); and Azure Site Recovery (Extension Lab F). Extension Lab G closes two more: Application Security Groups plus effective security rules (the Network Watcher tool Extension Lab B had to defer until a VM existed), and stored access policies as a third SAS revocation model alongside Module 5's account-key and User Delegation SAS. Even with all of that, this still isn't full AZ-104 exam coverage. Topics the current outline includes that this notebook doesn't give hands-on time to: management groups (needs a multi-subscription setup this notebook doesn't have); wiring a budget's alert to an actual automated response (Action Groups + Automation to cap spend, not just notify — deliberately left out given the risk of a misconfigured auto-shutdown runbook); Advisor (a recommendations tab you read, not something you configure); object replication (adjacent to the replication/versioning concepts already covered, and pricier to lab properly); hands-on Azure Bastion (hourly-billed regardless of use — a deliberate cost tradeoff); and public Azure DNS zones (needs an actual domain delegated to Azure, which a personal lab subscription usually doesn't have). Treat this notebook — modules plus extension labs — as the spine of your AZ-104 prep, not the whole syllabus; plan dedicated (usually free or near-free) time on the topics above before sitting the exam.

---

<a id="module-1"></a>
## Module 1 — Foundations: Subscriptions, Resource Groups, Regions, Tags, Azure CLI & Cloud Shell

### 1.0 Learning objectives
By the end of this module you should be able to explain, and do, the following without looking anything up:
- The hierarchy: Tenant → Management Group → Subscription → Resource Group → Resource
- Why resource groups exist and how to use them as a lifecycle boundary
- How to create/list/inspect resources with both the Portal and Azure CLI
- How to tag resources meaningfully, and why that matters for cost tracking
- How to fully tear down a lab so nothing keeps billing

### 1.1 Concepts before you touch the portal

- **Tenant (Entra ID)** — your organization's identity boundary. One tenant can have many subscriptions.
- **Subscription** — the billing and access-management boundary. Your Free Trial subscription is one subscription inside your tenant.
- **Resource Group (RG)** — a logical container with **no cost of its own**. Everything in it typically shares a lifecycle: create together, delete together. This is the single most important habit for this month — see Part 2.
- **Region** — the physical Azure datacenter location (e.g., `centralindia`, `eastus`). Matters for latency, data residency, and which services/SKUs are even available.
- **Tags** — key/value metadata (`project=azure-learning`, `module=01-foundations`) attached to resources or RGs. Used for cost filtering, ownership, and cleanup — not just labeling.
- **ARM (Azure Resource Manager)** — the API layer underneath both the Portal and the CLI. Every action you take either way ends up as an ARM API call — that's why CLI and Portal always stay in sync.

**Region note for you specifically:** you're in Hyderabad, so **Central India (Pune)** is the closest full-featured Indian region and a sensible default for these labs — lower latency for anything you access interactively, and it keeps you in the habit of choosing region deliberately rather than accepting a default. Some services occasionally have capacity or availability caveats in specific regions, so if a resource type isn't offered in Central India, check availability in the portal's region dropdown before assuming something is broken.

### 1.2 Hands-on Lab A — Portal walkthrough

1. Sign in to [portal.azure.com](https://portal.azure.com).
2. Search **"Resource groups"** → **+ Create**.
3. Subscription: your Free Trial. Resource group name: `rg-foundations-lab`. Region: `Central India`.
4. Under **Tags**, add:
   - `project` = `azure-learning`
   - `module` = `01-foundations`
5. Review + Create.
6. Once created, open the RG → **Overview** — note the Resource group ID (this is the ARM resource ID format you'll see everywhere: `/subscriptions/<sub-id>/resourceGroups/rg-foundations-lab`).
7. Go to **Cost Management + Billing** (search bar) → **Cost analysis** → filter by this resource group's tag. It'll show ₹0 for now — that's expected, an empty RG costs nothing.

### 1.3 Hands-on Lab B — Same thing via Azure CLI / Cloud Shell

Open **Cloud Shell** (the `>_` icon in the top bar of the portal, or [shell.azure.com](https://shell.azure.com)). First time you open it, Azure will ask to create a storage account for Cloud Shell itself — accept it; it's a trivial, near-zero-cost persistent resource outside your lab RGs (worth knowing it exists, in case you ever look for "where did this extra storage account come from").

```bash
# Confirm who/what you're logged in as
az account show --output table

# List every subscription you have access to
az account list --output table

# If you have more than one subscription, pin the CLI to the Free Trial one
az account set --subscription "<subscription-name-or-id>"

# Create the same resource group as above, but from the CLI
az group create \
  --name rg-foundations-lab-cli \
  --location centralindia \
  --tags project=azure-learning module=01-foundations

# List all resource groups to see both (portal + CLI) side by side
az group list --output table

# Inspect one in detail (this is the same JSON the Portal is built on top of)
az group show --name rg-foundations-lab-cli

# Filter resource groups by tag — this is how you'll track "what belongs to this course"
az group list --tag project=azure-learning --output table
```

**Useful CLI quality-of-life command** — set defaults so you stop retyping `--resource-group` and `--location` in every command for the rest of this module:

```bash
az configure --defaults group=rg-foundations-lab-cli location=centralindia
```

### 1.4 Self-check questions

Answer these yourself first, then expand.

1. What's the actual difference between a subscription and a resource group?
2. Why does Azure recommend grouping resources by *lifecycle* rather than by *resource type* (e.g., "all my VMs" as one RG)?
3. What does the `--tags` flag do at creation time, versus updating tags on an existing group?
4. If `az group list --tag project=azure-learning` returns nothing after you just created a tagged group, what are two likely explanations?
5. Why does region choice matter for you specifically, beyond "pick whatever's default"?

<details>
<summary>Answers (click to expand)</summary>

1. A **subscription** is the billing + access boundary — it's what you pay for and what RBAC ultimately scopes against at the top. A **resource group** is a free, logical container *inside* a subscription for organizing resources that share a lifecycle. You can have many RGs per subscription.
2. Because the main operational use of an RG is "delete this and everything in it goes away together." Grouping by resource type means a VNet, a VM, and a storage account that belong to the *same* lab end up scattered across different RGs — so tearing down one experiment means hunting across groups instead of running one `az group delete`.
3. `--tags` at creation sets the initial tag set in the same API call. To change tags later you'd use `az group update --tags ...` (which **replaces** the tag set, so include existing tags you want to keep) or `az tag create`/`az resource tag` for finer-grained control.
4. Either (a) the RG doesn't have that exact tag key/value — tags are case-sensitive and exact-match — or (b) Cost/Resource Graph indexing hasn't caught up yet; `az group list` itself should be near-instant, so this usually means a typo in the tag rather than lag.
5. Latency to services you'll interact with interactively (SSH, RDP, web apps) is lower from a nearby region; some data residency/compliance requirements mandate specific regions; and not every service or VM SKU is available in every region, so your region choice can silently limit what you're able to provision later in the course.

</details>

### 1.5 Common mistakes / gotchas

- **Region name format** — the Portal shows "Central India" but the CLI wants `centralindia` (lowercase, no space). Run `az account list-locations --output table` if you're ever unsure of the slug.
- **Forgetting to tag** — untagged resources are the #1 reason people can't tell "is this mine, and can I delete it" three days later.
- **The Cloud Shell storage account** — it lives outside whatever RG you're working in. It's not a cost risk, but don't be surprised when you see an extra storage account you didn't explicitly create.
- **Assuming Portal and CLI are separate systems** — they're not. A resource you create in the Portal is fully visible and manageable from the CLI immediately, and vice versa. If you don't see something, it's a subscription/filter issue, not a sync issue.

### 1.6 Cost check + cleanup

Before deleting, glance at Cost Management → Cost analysis filtered to `project=azure-learning` — it should still read ₹0, since empty resource groups have no cost.

```bash
# Delete both lab resource groups from this module
az group delete --name rg-foundations-lab-cli --yes
```

(Delete `rg-foundations-lab`, the one you made via Portal, the same way — either via CLI or the "Delete resource group" button in the Portal.)

### 1.7 What's next

**Module 2 — Identity & RBAC** picks up right where this leaves off: you'll create a resource group again, but this time hand out scoped access to it using Azure roles instead of ad-hoc permissions — the foundation for everything AZ-104 tests about access control, and the pattern (`Managed Identity → RBAC → Resource`) that shows up again in AI-200.

Let me know once you've run through Module 1 and I'll write Module 2 next — or ask now if anything above didn't behave the way you expected.

---

<a id="module-2"></a>
## Module 2 — Identity & RBAC: Users, Groups, Roles, Scope, Managed Identities

### 2.0 Learning objectives
- Explain the difference between **Entra ID roles** (directory-level admin) and **Azure RBAC roles** (resource-level access)
- Explain **scope** and its four levels, and why inheritance flows downward
- Assign roles to both a user and a group, at the resource-group scope
- Create a **user-assigned managed identity** and explain the password-free access pattern it enables
- Know when to use Owner vs. Contributor vs. Reader

### 2.1 Concepts before you touch the portal

- **Microsoft Entra ID** (formerly Azure AD) is the identity store for your whole tenant — users, groups, service principals, managed identities all live here. It's tenant-wide, not per-subscription.
- **Azure RBAC** is a separate system from Entra ID admin roles. It controls what an identity (a user, group, service principal, or managed identity) can *do to Azure resources*. Don't confuse "Global Administrator" (an Entra ID directory role) with "Owner" (an Azure RBAC role) — they answer different questions: "can this person manage the directory itself" vs. "can this identity manage this resource."
- **Scope**, broadest to narrowest: `Management Group → Subscription → Resource Group → Resource`. A role assigned at a broader scope is inherited by everything underneath it. Assign at the narrowest scope that gets the job done.
- **Role definition** = a bundle of permissions. **Role assignment** = binding (principal + role definition + scope) together. Three parts, always.
- **Built-in roles you'll use constantly:**
  - `Owner` — full access, *and* can grant access to others.
  - `Contributor` — full access to manage resources, but **cannot** grant access to others.
  - `Reader` — read-only, everywhere in scope.
  - Most other built-in roles are service-specific (`Storage Blob Data Contributor`, `Virtual Machine Contributor`, etc.) — narrower and usually the better real-world choice than Contributor.
- **Groups over individuals** — assigning a role to a group (and managing membership) scales; assigning directly to 20 individual users doesn't. This is standard practice, not just a lab convention.
- **Managed identity** — an identity Azure manages for you, with no password or secret you ever see or rotate.
  - **System-assigned**: created and destroyed with the resource it's attached to (e.g., a VM). One-to-one.
  - **User-assigned**: created as its own standalone resource, independent lifecycle, can be attached to multiple resources at once. This is what you'll create below — you don't need a VM or App Service yet to make one, because the identity object itself exists independently of anything using it.
  - The pattern this replaces: `App → password/connection string → Storage`. The pattern it enables: `App → Managed Identity → RBAC → Storage`. You'll wire this up end-to-end once you have a VM/App Service (Modules 4, 7, 8) — today's lab just creates the identity and explains why it matters.

### 2.2 Hands-on Lab A — Portal walkthrough

1. Create a new resource group: `rg-identity-lab`, region `Central India`, tags `project=azure-learning`, `module=02-identity-rbac`.
2. **Entra ID → Groups → New group.** Group type: Security. Name: `az-learning-readers`. No members needed yet.
3. Open `rg-identity-lab` → **Access control (IAM)** → **Add role assignment**.
   - Role: `Reader`. Assign access to: **User, group, or service principal**. Select `az-learning-readers`. Review + assign.
4. Add a second role assignment on the same RG: Role `Contributor`, assigned to **your own account** — even though you're likely already Owner at the subscription level. This demonstrates that a narrower, explicit assignment can sit alongside inherited access.
5. Go to **Access control (IAM) → Check access**, search your own account, and look at the combined effective roles at this scope. Note both the inherited (subscription-level) and directly-assigned (RG-level) roles show up.
6. **Managed Identities** (search bar) → **+ Create** → User-assigned. Resource group `rg-identity-lab`, name `id-learning-uami`, region `Central India`. Create it, then open it and note the **Client ID** and **Object (principal) ID** — these are what you'll reference later when attaching it to a resource and granting it RBAC.

### 2.3 Hands-on Lab B — Same thing via Azure CLI

```bash
az group create \
  --name rg-identity-lab \
  --location centralindia \
  --tags project=azure-learning module=02-identity-rbac

# Find your own signed-in identity (you'll need this as --assignee below)
az ad signed-in-user show --query "{id:id, upn:userPrincipalName}" --output table

# Get the resource group's full ARM resource ID (needed for --scope)
RG_ID=$(az group show --name rg-identity-lab --query id --output tsv)
echo $RG_ID

# Assign yourself Contributor, scoped ONLY to this resource group
az role assignment create \
  --assignee "<your-upn-or-object-id>" \
  --role "Contributor" \
  --scope "$RG_ID"

# Create a Microsoft Entra security group
az ad group create \
  --display-name az-learning-readers \
  --mail-nickname az-learning-readers

# Get that group's object id
GROUP_ID=$(az ad group show --group az-learning-readers --query id --output tsv)

# Assign Reader to the group, scoped to the resource group
az role assignment create \
  --assignee-object-id "$GROUP_ID" \
  --assignee-principal-type Group \
  --role "Reader" \
  --scope "$RG_ID"

# List every role assignment at this scope — you should see both above
az role assignment list --scope "$RG_ID" --output table

# Create a user-assigned managed identity (standalone resource, no compute needed yet)
az identity create \
  --name id-learning-uami \
  --resource-group rg-identity-lab

# Inspect it — note clientId and principalId, you'll need these later
az identity show \
  --name id-learning-uami \
  --resource-group rg-identity-lab \
  --query "{clientId:clientId, principalId:principalId}" \
  --output table
```

### 2.4 Self-check questions

<details>
<summary>Question 1 — Entra ID roles vs. Azure RBAC roles: what's the actual difference?</summary>

Entra ID roles (e.g., Global Administrator, User Administrator) control the **directory itself** — who can create users, reset passwords, manage tenant-wide settings. Azure RBAC roles (e.g., Owner, Contributor, Reader) control **access to Azure resources** (VMs, storage, resource groups). A Global Administrator doesn't automatically get Contributor on your subscription, and an Owner on a subscription doesn't automatically get any Entra ID directory role. They're two independent permission systems that happen to reference the same underlying identities.
</details>

<details>
<summary>Question 2 — What are the four RBAC scope levels, and which direction does inheritance flow?</summary>

Management Group → Subscription → Resource Group → Resource, broadest to narrowest. A role assigned at a broader scope is inherited by everything below it — so Contributor at the subscription level means Contributor on every resource group and resource inside that subscription, automatically.
</details>

<details>
<summary>Question 3 — Why assign a role to a group instead of directly to individual users?</summary>

Because membership management (adding/removing people from the group) is decoupled from access management (the role assignment itself doesn't change). Onboarding or offboarding someone becomes a group-membership edit instead of hunting down every resource they had direct access to.
</details>

<details>
<summary>Question 4 — What's the practical difference between Owner and Contributor?</summary>

Both can fully manage resources in scope. The difference is that **Owner can also grant access to others** (create/modify role assignments), while **Contributor cannot**. This is why handing out Owner casually is riskier than it looks — it's not just "more access to resources," it's "can also change who else has access."
</details>

<details>
<summary>Question 5 — System-assigned vs. user-assigned managed identity: what's the difference, and when would you pick each?</summary>

A **system-assigned** identity is created and deleted together with the one resource it's attached to (e.g., delete the VM, the identity is gone too) — pick it when the identity's lifecycle should exactly match one resource. A **user-assigned** identity is its own standalone object with an independent lifecycle, and can be attached to multiple resources at once — pick it when several resources (e.g., a Function App and a VM) need to share the same identity and permissions, or when you want the identity to outlive any single resource.
</details>

### 2.5 Common mistakes / gotchas

- **Treating "Global Administrator" and "Owner" as the same kind of thing** — they're not interchangeable, and confusing them is a common real-world misconfiguration, not just a lab trap.
- **Defaulting to Owner or Contributor everywhere** instead of a narrower built-in role (`Storage Blob Data Reader`, `Virtual Machine Contributor`, etc.) — convenient now, a real problem in production.
- **Assigning at subscription scope "just to be safe"** — this is the opposite of least privilege and means every future resource group inherits that access whether you meant it to or not.
- **Creating a user-assigned identity and forgetting about it** — it's free sitting idle, but it's also an unused credential-equivalent object; clean it up with the rest of the resource group if you're not using it in a later module.

### 2.6 Cost check + cleanup

RBAC assignments and Entra ID groups have no cost. The only thing to actually delete is the resource group (which also removes the managed identity, since it's a resource inside it):

```bash
az group delete --name rg-identity-lab --yes

# Optional — remove the security group too, if you want a clean tenant afterward
az ad group delete --group az-learning-readers
```

### 2.7 What's next

**Module 3 — Networking** is the biggest module in this notebook, and the one AZ-104 (and real Azure jobs) weight most heavily: VNets, subnets, CIDR ranges, NSGs, peering, DNS, and route tables. You'll build a two-VNet, three-subnet lab and spend real time on it — this is worth doing carefully rather than rushing.

Run through Module 2, and tell me when you're ready for Module 3 — or ask now if any RBAC behavior above surprised you.

---

<a id="ext-a"></a>
## Extension Lab A — Governance: Azure Policy, Resource Locks & Budgets

*A compact detour before Module 3 — closes an AZ-104 governance gap. Pairs with Module 2's RBAC: RBAC controls **who** can act, this controls **what's allowed** regardless of who's doing it.*

**Objectives**
- Explain what Azure Policy does that RBAC doesn't
- Assign a built-in policy at resource-group scope and watch it block a non-compliant deployment
- Apply and remove a resource lock — and recognize when a lock, not RBAC, is the actual blocker
- Set a cost alert budget and understand what it does (and doesn't) do

**Concepts**

Azure Policy: a **definition** describes a rule plus an **effect**; an **assignment** applies a definition at a scope; an **initiative** groups multiple definitions (e.g., a compliance standard). Effects: **Deny** stops the request before creation, **Audit** lets it through but flags it as noncompliant, **Append/Modify** inject or change properties at deployment time, **DeployIfNotExists** auto-remediates by deploying a companion resource (e.g., auto-enabling diagnostic settings on anything missing them). The key distinction from RBAC: Policy blocks actions **regardless of who's performing them** — even a subscription Owner can be denied by a Deny policy, because Policy operates at the deployment layer, not the authorization layer.

Resource locks: **CanNotDelete** blocks deletion only (updates still work); **ReadOnly** blocks deletion *and* updates, freezing the resource entirely — even for Owners. Locks cascade: one applied at a resource group applies to everything inside it. "I have Owner but `az group delete` just fails" is a very common real-world ticket, and the fix is almost always removing a lock, not fixing permissions.

Budgets: Cost Management budgets send **alerts** (email or an Action Group) when spend crosses a threshold. On their own they do not stop spending, throttle anything, or delete resources — actually capping spend means wiring the budget's alert to something like an Automation runbook or Logic App that shuts resources down. This distinction trips a lot of people up.

**Hands-on — Portal**

1. Resource groups → **+ Create** → `rg-ext-a`, tag `project=azure-learning`, `module=ext-a-governance`.
2. Inside `rg-ext-a` → **Policies** → **Assignments** → **Assign policy**. Scope: `rg-ext-a`. Policy definition: search "**Require a tag on resources**" (built-in). Parameter: tag name = `costCenter`. Review + create.
3. Try creating a resource without that tag (e.g., a storage account, skip tags) → the deployment fails, citing the policy assignment. That's Deny doing its job.
4. Retry with the `costCenter` tag added → succeeds.
5. `rg-ext-a` → **Locks** → **+ Add**. Name `lock-ext-a`, type **CanNotDelete**. Try **Delete resource group** → fails with a lock-related error.
6. Delete the lock, and you'll be able to delete the group afterward (don't confirm the delete yet if you're doing the CLI steps below too).
7. **Cost Management + Billing** → **Budgets** → **+ Add**. Scope: your subscription. Amount: ₹500 (deliberately low so it's likely to actually trigger against real spend this month). Alert condition: 80% of forecast. Recipient: your email.

**Hands-on — CLI**

```bash
az group create --name rg-ext-a --location centralindia \
  --tags project=azure-learning module=ext-a-governance

# Find the built-in "Require a tag on resources" policy definition
az policy definition list --query "[?displayName=='Require a tag on resources'].{name:name, id:id}" -o table

RG_ID=$(az group show --name rg-ext-a --query id -o tsv)
POLICY_ID=$(az policy definition list --query "[?displayName=='Require a tag on resources'].id" -o tsv)

az policy assignment create --name require-costcenter-tag \
  --scope "$RG_ID" --policy "$POLICY_ID" \
  --params '{"tagName": {"value": "costCenter"}}'

# This should fail — no costCenter tag
az storage account create --name stnotagbvj --resource-group rg-ext-a \
  --location centralindia --sku Standard_LRS

# This should succeed
az storage account create --name sttaggedbvj --resource-group rg-ext-a \
  --location centralindia --sku Standard_LRS --tags costCenter=learning

# Lock the resource group, then prove it blocks deletion
az lock create --name lock-ext-a --resource-group rg-ext-a --lock-type CanNotDelete
az group delete --name rg-ext-a --yes --no-wait   # fails — lock is in the way
az lock delete --name lock-ext-a --resource-group rg-ext-a

# Optional: the same budget, from the CLI
az consumption budget create --budget-name budget-ext-a --amount 500 \
  --category cost --time-grain monthly \
  --start-date $(date +%Y-%m-01) --end-date 2027-01-01 \
  --notifications '{"Actual_GreaterThan_80_Percent":{"enabled":true,"operator":"GreaterThan","threshold":80,"contactEmails":["<your-email>"]}}'
```

**Self-check questions**

<details><summary>1. You're the subscription Owner, and you still can't create a resource in <code>rg-ext-a</code> without the <code>costCenter</code> tag. Why doesn't Owner override this?</summary>

Policy's Deny effect blocks the action for everyone, regardless of RBAC role, including Owners — Policy sits at the deployment layer, not the authorization layer.
</details>

<details><summary>2. A teammate says "I have Contributor on this resource group but <code>az group delete</code> just fails with no useful reason." First thing to check?</summary>

Whether a lock (CanNotDelete or ReadOnly) is attached to the resource group or an ancestor scope — check the Locks blade or <code>az lock list</code>. Lock errors can read as opaque permission failures.
</details>

<details><summary>3. You set a ₹500 budget alert. Spend hits ₹600. What actually happens?</summary>

An alert notification fires (email or Action Group). Nothing is stopped, throttled, or deleted automatically — enforcement has to be built separately.
</details>

<details><summary>4. What's the difference between an Audit and a Deny effect on the same underlying rule?</summary>

Audit lets the noncompliant resource get created but flags it in Policy's compliance view; Deny blocks the request outright before the resource is created.
</details>

**Common mistakes / gotchas**

- Trying to delete a locked resource group and assuming you lack permissions, when the real blocker is the lock.
- Forgetting a lock at RG scope cascades to *every* resource inside it, not just the group object.
- Treating a budget alert as a hard spending cap — it's a notification hook, not a limiter.
- Assigning a Deny policy at subscription scope while testing, then losing track of why an unrelated deployment elsewhere started failing — scope test policies to a throwaway RG first.

**Cost check + cleanup**

Policy assignments, locks, and budgets cost nothing by themselves; the two test storage accounts are the only billable objects here (negligible on Standard_LRS). **Delete the lock before deleting the resource group** — that ordering catches people in real jobs too:

```bash
az lock delete --name lock-ext-a --resource-group rg-ext-a 2>/dev/null
az group delete --name rg-ext-a --yes --no-wait
az consumption budget delete --budget-name budget-ext-a 2>/dev/null   # if you created it
```

**What's next:** Back to Module 3 — Networking.

---

<a id="module-3"></a>
## Module 3 — Networking: VNets, Subnets, CIDR, NSGs, Peering, Route Tables, DNS

This is the biggest module in the notebook and the most heavily weighted topic on AZ-104 — take your time here. Note up front: this module builds the network **structure**; real connectivity testing (pinging between subnets, hitting a web server) needs actual VMs, which is Module 4. Think of this module as wiring the building before the tenants move in.

### 3.0 Learning objectives
- Plan non-overlapping CIDR address spaces across VNets
- Build a VNet with multiple subnets, and a second VNet, in the same resource group
- Write NSG rules, understand priority evaluation, and attach NSGs to subnets
- Peer two VNets and verify the peering state from both sides
- Create a route table and understand what a custom (user-defined) route actually does to traffic
- Explain Azure-provided DNS name resolution within a VNet

### 3.1 Concepts before you touch the portal

- **VNet** — an isolated, private address space inside a region. Nothing outside it can reach into it unless you explicitly allow it (peering, VPN, public IP + NSG rule, etc.).
- **CIDR notation** — `10.0.0.0/16` means the first 16 bits are fixed (network portion), the remaining 16 bits are host addresses → 65,536 addresses. `10.0.1.0/24` fixes 24 bits → 256 addresses. Smaller number after the `/` = bigger range.
- **Azure reserves 5 addresses per subnet**: the network address (`.0`), the default gateway (`.1`), two reserved for Azure's internal DNS mapping (`.2`, `.3`), and the broadcast address (`.255` in a /24). So a /24 "256 addresses" subnet only has **251 usable** for your resources.
- **Subnet** — a subdivision of a VNet's address space. Resources get a private IP from whichever subnet their NIC is placed in.
- **NSG (Network Security Group)** — a stateful allow/deny filter, evaluated by **priority number, lowest first** (100 is evaluated before 200; lowest wins on a match). Can attach to a subnet, a NIC, or both. Every NSG ships with default rules around priority 65000–65500 (`AllowVNetInBound`, `AllowAzureLoadBalancerInBound`, `DenyAllInBound`, and outbound equivalents) that apply underneath anything you add — "no rule matches" still means "the default deny applies," not "everything is allowed."
- **VNet Peering** — connects two VNets so resources reach each other over private IPs, at (near) line-rate, without any gateway. Critically, **peering is non-transitive**: if A↔B and B↔C are peered, A cannot reach C through B. Each pair needs its own explicit peering.
- **Route Table / User-Defined Route (UDR)** — overrides Azure's default system routes for a subnet. The classic use: force all outbound traffic (`0.0.0.0/0`) through a firewall or network virtual appliance instead of straight to the internet.
- **DNS** — by default, Azure provides name resolution so VMs in the *same* VNet can resolve each other by name automatically. Cross-VNet or custom domain name resolution needs Azure Private DNS zones or a custom DNS server — out of scope for this module, flagged for later if you need it.

### 3.2 The lab you're building

```
rg-networking-lab
│
├── vnet-a  (10.0.0.0/16)
│   ├── snet-web   (10.0.1.0/24)  → nsg-web  (allow 80/443 from Internet)
│   ├── snet-app   (10.0.2.0/24)  → nsg-app  (allow 8080 from snet-web only)
│   └── snet-db    (10.0.3.0/24)  → nsg-db   (allow 5432 from snet-app only)
│
├── vnet-b  (10.1.0.0/16)
│   └── snet-mgmt  (10.1.0.0/24)  → rt-mgmt-lab (demo route table)
│
└── peering: vnet-a ↔ vnet-b (both directions)
```

Notice the segmentation pattern: web can reach app, app can reach db, but **web cannot reach db directly** — there's no rule allowing it. This three-tier pattern (and the deliberate absence of a shortcut rule) is exactly what AZ-104 network-segmentation questions test.

### 3.3 Hands-on Lab A — Portal walkthrough

1. Create resource group `rg-networking-lab` (Central India, tags `project=azure-learning`, `module=03-networking`).
2. **Virtual networks → + Create** → name `vnet-a`, address space `10.0.0.0/16`. In the Subnets step, add all three: `snet-web` (`10.0.1.0/24`), `snet-app` (`10.0.2.0/24`), `snet-db` (`10.0.3.0/24`). Create.
3. Create a second VNet `vnet-b`, address space `10.1.0.0/16`, one subnet `snet-mgmt` (`10.1.0.0/24`).
4. **Network security groups → + Create** three times: `nsg-web`, `nsg-app`, `nsg-db` (all in `rg-networking-lab`, Central India).
   - `nsg-web` → Inbound security rules → Add: Source `Any`/`Internet`, Destination port ranges `80,443`, Protocol `TCP`, Action `Allow`, Priority `100`.
   - `nsg-app` → Add: Source `IP Addresses` → `10.0.1.0/24`, Destination port `8080`, Protocol `TCP`, Allow, Priority `100`.
   - `nsg-db` → Add: Source `IP Addresses` → `10.0.2.0/24`, Destination port `5432`, Protocol `TCP`, Allow, Priority `100`.
5. Associate each NSG to its matching subnet: open the NSG → **Subnets** → **Associate** → pick `vnet-a` and the matching subnet.
6. **vnet-a → Peerings → + Add** → link to `vnet-b`, allow traffic both ways (this creates the local side). Then confirm from `vnet-b → Peerings` that the remote side shows **Connected** too (Azure Portal can create both sides in one flow if you leave the "add matching peering to remote vnet" box checked — verify both regardless).
7. **Route tables → + Create** → `rt-mgmt-lab` (Central India). Open it → **Routes → + Add**: Address prefix `0.0.0.0/0`, Next hop type `Virtual appliance`, Next hop address `10.1.0.10` (a placeholder — nothing is actually listening there, which is fine since `snet-mgmt` has no live resources yet).
8. Associate the route table: `rt-mgmt-lab → Subnets → Associate` → `vnet-b` / `snet-mgmt`.

### 3.4 Hands-on Lab B — Same thing via Azure CLI

```bash
az group create --name rg-networking-lab --location centralindia \
  --tags project=azure-learning module=03-networking

# VNet-A with all three subnets
az network vnet create \
  --resource-group rg-networking-lab --name vnet-a --location centralindia \
  --address-prefix 10.0.0.0/16 \
  --subnet-name snet-web --subnet-prefix 10.0.1.0/24

az network vnet subnet create --resource-group rg-networking-lab --vnet-name vnet-a \
  --name snet-app --address-prefix 10.0.2.0/24
az network vnet subnet create --resource-group rg-networking-lab --vnet-name vnet-a \
  --name snet-db --address-prefix 10.0.3.0/24

# VNet-B
az network vnet create \
  --resource-group rg-networking-lab --name vnet-b --location centralindia \
  --address-prefix 10.1.0.0/16 \
  --subnet-name snet-mgmt --subnet-prefix 10.1.0.0/24

# NSGs + rules
az network nsg create --resource-group rg-networking-lab --name nsg-web --location centralindia
az network nsg rule create --resource-group rg-networking-lab --nsg-name nsg-web \
  --name Allow-Web --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes Internet --destination-port-ranges 80 443

az network nsg create --resource-group rg-networking-lab --name nsg-app --location centralindia
az network nsg rule create --resource-group rg-networking-lab --nsg-name nsg-app \
  --name Allow-From-Web --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes 10.0.1.0/24 --destination-port-ranges 8080

az network nsg create --resource-group rg-networking-lab --name nsg-db --location centralindia
az network nsg rule create --resource-group rg-networking-lab --nsg-name nsg-db \
  --name Allow-From-App --priority 100 --direction Inbound --access Allow --protocol Tcp \
  --source-address-prefixes 10.0.2.0/24 --destination-port-ranges 5432

# Associate NSGs to subnets
az network vnet subnet update --resource-group rg-networking-lab --vnet-name vnet-a \
  --name snet-web --network-security-group nsg-web
az network vnet subnet update --resource-group rg-networking-lab --vnet-name vnet-a \
  --name snet-app --network-security-group nsg-app
az network vnet subnet update --resource-group rg-networking-lab --vnet-name vnet-a \
  --name snet-db --network-security-group nsg-db

# Peering — BOTH directions are separate commands
az network vnet peering create --resource-group rg-networking-lab \
  --name peer-a-to-b --vnet-name vnet-a --remote-vnet vnet-b --allow-vnet-access true
az network vnet peering create --resource-group rg-networking-lab \
  --name peer-b-to-a --vnet-name vnet-b --remote-vnet vnet-a --allow-vnet-access true

# Verify both sides show Connected
az network vnet peering list --resource-group rg-networking-lab --vnet-name vnet-a --output table
az network vnet peering list --resource-group rg-networking-lab --vnet-name vnet-b --output table

# Route table + demo route + association
az network route-table create --resource-group rg-networking-lab --name rt-mgmt-lab --location centralindia
az network route-table route create --resource-group rg-networking-lab \
  --route-table-name rt-mgmt-lab --name demo-default-route \
  --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address 10.1.0.10
az network vnet subnet update --resource-group rg-networking-lab --vnet-name vnet-b \
  --name snet-mgmt --route-table rt-mgmt-lab
```

Sanity-check commands worth running just to read the output:

```bash
az network vnet subnet list --resource-group rg-networking-lab --vnet-name vnet-a --output table
az network nsg rule list --resource-group rg-networking-lab --nsg-name nsg-app --output table
```

### 3.5 Self-check questions

<details>
<summary>Question 1 — Why is VNet peering non-transitive, and what does that mean for a 3-VNet design?</summary>

Peering only creates a direct relationship between the two VNets in that specific peering. If A↔B and B↔C are both peered, traffic from A cannot reach C via B — Azure does not forward across peerings. For a hub-and-spoke design where spokes need to reach each other, you either peer every spoke pair directly, or route spoke-to-spoke traffic through a network virtual appliance in the hub using UDRs — peering alone won't do it.
</details>

<details>
<summary>Question 2 — A /24 subnet has 256 addresses. Why only 251 usable?</summary>

Azure reserves 5 per subnet: the network address (first), the default gateway address, two for Azure's internal DNS, and the broadcast address (last). This is true for every Azure subnet regardless of size, and it's a common "off by 5" mistake when sizing subnets for a known number of VMs.
</details>

<details>
<summary>Question 3 — NSG rule at priority 100 denies traffic; another rule at priority 200 allows the same traffic. Which wins?</summary>

Priority 100 wins — NSG rules are evaluated in ascending priority order (lowest number first) and the **first match wins**, stopping evaluation. So the deny at 100 is applied before the allow at 200 is ever considered.
</details>

<details>
<summary>Question 4 — NSG on a subnet vs. on a NIC — what's the difference, and which is generally the better default?</summary>

A subnet-level NSG applies to every NIC in that subnet uniformly; a NIC-level NSG applies to just that one network interface, regardless of which subnet it's in. Subnet-level is generally the better default for consistent segmentation (everything in "the db subnet" gets the same rules by construction); NIC-level is for exceptions to that rule on a specific machine. Using both together means traffic must pass both to be allowed.
</details>

<details>
<summary>Question 5 — What does a 0.0.0.0/0 route to a virtual appliance actually do, and why is forgetting about it dangerous?</summary>

It overrides the subnet's default route to the internet, forcing **all** outbound traffic from that subnet through whatever IP is named as the next hop — normally a firewall or NVA. If that IP doesn't correspond to a real, listening device (as in this demo), traffic sent there is silently dropped — a "black hole" route. This is a genuinely common real-world outage: someone adds resources to a subnet months after a route table was set up for a since-decommissioned appliance, and everything in that subnet loses connectivity with no obvious error message pointing at the route table.
</details>

### 3.6 Common mistakes / gotchas

- **Overlapping address spaces** — if `vnet-a` and `vnet-b` share any overlapping CIDR range, peering creation will fail outright. Always plan non-overlapping ranges before creating VNets, even in a throwaway lab.
- **One-sided peering** — creating the peering from `vnet-a`'s side only leaves `vnet-b` showing no peering at all. Always create (or verify) both directions.
- **"Lower number wins" confusion** — it's easy to assume higher priority number = higher priority. It's the opposite: lower number = evaluated first = wins on a match.
- **Assuming "no rule" means "allowed"** — the default `DenyAllInBound` rule (priority ~65500) catches anything your custom rules didn't explicitly allow.
- **Leaving a route table pointed at a placeholder IP attached to a subnet that later gets real workloads** — exactly the scenario in self-check Q5. If you build on top of `rt-mgmt-lab` later, either point it at a real appliance or remove the route first.

### 3.7 Cost check + cleanup

Unlike every other module so far: **VNets, subnets, NSGs, peerings, and route tables have no hourly cost.** This is one of the few resource types you can leave running for the rest of the month without it touching your credit.

Because of that, **I'd recommend leaving `rg-networking-lab` standing** rather than deleting it — Module 4 (Virtual Machines) plugs directly into this exact network, so you'd just be rebuilding the same thing next session. If you'd rather tear it down and rebuild fresh, that's fine too:

```bash
az group delete --name rg-networking-lab --yes
```

### 3.8 What's next

**Module 4 — Virtual Machines** drops real VMs into `snet-web` and `snet-app`, and this is where the network you just built stops being diagrams and becomes something you can actually break: SSH in, install Nginx, toggle the NSG rule on `nsg-web` live, and watch the site go from reachable to unreachable and back.

Run through Module 3, and say the word when you're ready for Module 4 — or ask now if peering status, NSG behavior, or the route table didn't match what you expected.

---

<a id="ext-b"></a>
## Extension Lab B — Private Endpoint, Private DNS Zone & Network Watcher

*Sits between Modules 3 and 4 — builds on Module 3's network before Module 4 puts VMs into it.*

**Objectives**
- Explain the difference between a Service Endpoint and a Private Endpoint
- Deploy a private endpoint into `vnet-a` from Module 3 and prove — via the DNS record, not guesswork — that it got a private IP
- Understand why a private endpoint needs a Private DNS Zone to be usable transparently
- Use Network Watcher's topology view to see Module 3's network the way Azure sees it

**Concepts**

A **Service Endpoint** extends a subnet's identity onto a Microsoft-backbone route to a service's *public* endpoint — traffic leaves your VNet's address space but stays on Microsoft's network, and the service's firewall can filter it down to "only this subnet." The service still has a public IP.

A **Private Endpoint** is a NIC with a private IP, sitting in your subnet, mapped to one specific PaaS resource (a specific storage account, not "storage in general"). From your VNet's perspective, traffic to that resource never touches a public IP at all.

The catch: the resource's normal DNS name (`<account>.blob.core.windows.net`) still resolves publicly by default. For clients on your VNet to transparently get the *private* IP when resolving that name, you need a **Private DNS Zone** (`privatelink.blob.core.windows.net`) linked to your VNet, with an A record pointing at the private endpoint's IP. Skip that zone, and a private endpoint with no matching DNS record quietly keeps routing over the public path — a common "why is this still going over the internet" bug.

**Network Watcher** is enabled automatically per region once you have networked resources there. Its **Topology** view visualizes what's actually deployed — VNets, subnets, NICs, NSGs, peerings — without needing a VM to test from. Two of its other tools, **IP Flow Verify** and **Effective Security Rules**, need a real VM NIC as the test source, so those wait until Module 4 gives you one — come back and run them against `vm-web`/`vm-app` once they exist.

**Hands-on — Portal**

1. Reuse `vnet-a` from Module 3 if `rg-networking-lab` still exists — otherwise this lab works fine standalone with its own small VNet; the private-endpoint concept doesn't depend on Module 3's specific address space.
2. **Storage accounts → + Create** → `stextpebvj` in a fresh `rg-ext-b`, Standard_LRS.
3. Storage account → **Networking → Private endpoint connections → + Private endpoint**. Target sub-resource: `blob`. Virtual network: `vnet-a`, subnet `snet-app`. Private DNS integration: Yes — let Portal create/link `privatelink.blob.core.windows.net` automatically.
4. Open the new Private DNS zone resource → an A record for `stextpebvj` should already be sitting there, pointing at a `10.0.x.x` address from `snet-app`'s range. That record is your proof: a name that used to resolve publicly now has a private-IP record for anything attached to the zone.
5. Storage account → **Networking → Public network access** → **Disabled** → Save. This is the "for real" version: from outside the VNet, the account is now unreachable — exactly why Module 4's Bastion/VPN/peering discussion matters for anyone who still needs access.
6. Search "**Network Watcher**" in the Portal → **Topology** → select `rg-ext-b` → watch the VNet/subnet/NIC diagram render from what's actually deployed.

**Hands-on — CLI**

```bash
az group create --name rg-ext-b --location centralindia \
  --tags project=azure-learning module=ext-b-private-link

az storage account create --name stextpebvj --resource-group rg-ext-b \
  --location centralindia --sku Standard_LRS --kind StorageV2

STG_ID=$(az storage account show --name stextpebvj --resource-group rg-ext-b --query id -o tsv)

# Reuse vnet-a/snet-app from Module 3 — it lives in a different resource
# group (rg-networking-lab) from this lab's own rg-ext-b
SUBNET_ID=$(az network vnet subnet show \
  --resource-group rg-networking-lab \
  --vnet-name vnet-a --name snet-app --query id -o tsv)

# When --subnet is given as a full resource ID (as it is here, for a subnet
# living in a different resource group than this command's --resource-group),
# omit --vnet-name — supplying both is redundant and can conflict.
az network private-endpoint create \
  --name pe-stextpe --resource-group rg-ext-b \
  --subnet "$SUBNET_ID" \
  --private-connection-resource-id "$STG_ID" \
  --group-id blob --connection-name pe-conn-stextpe

az network private-dns zone create --resource-group rg-ext-b \
  --name "privatelink.blob.core.windows.net"

# The VNet is also in rg-networking-lab, not rg-ext-b — pass its full
# resource ID rather than a bare name so the command can find it across RGs
VNET_ID=$(az network vnet show --resource-group rg-networking-lab \
  --name vnet-a --query id -o tsv)

az network private-dns link vnet create --resource-group rg-ext-b \
  --zone-name "privatelink.blob.core.windows.net" \
  --name link-vnet-a --virtual-network "$VNET_ID" --registration-enabled false

az network private-endpoint dns-zone-group create \
  --resource-group rg-ext-b --endpoint-name pe-stextpe \
  --name default-zone-group \
  --private-dns-zone "privatelink.blob.core.windows.net" \
  --zone-name blob

# Proof: an A record now exists, pointing at a private IP
az network private-dns record-set a list \
  --resource-group rg-ext-b --zone-name "privatelink.blob.core.windows.net" -o table

# Lock the account down to private-only access
az storage account update --name stextpebvj --resource-group rg-ext-b \
  --public-network-access Disabled

# Topology view — no VM required
az network watcher show-topology --resource-group rg-ext-b -o table
```

**Self-check questions**

<details><summary>1. You created a private endpoint but skipped the Private DNS Zone step. Does the app connecting to the storage account break?</summary>

Not immediately — it keeps resolving the account's public DNS name and routes over the public endpoint (unless you've also disabled public network access, in which case it does break). The private endpoint existing doesn't force traffic through it; DNS resolution does.
</details>

<details><summary>2. What's the practical difference between a Service Endpoint and a Private Endpoint if both "keep traffic off the public internet"?</summary>

A Service Endpoint still targets the resource's public IP, just via Microsoft's backbone and filterable by source subnet. A Private Endpoint gives the resource an actual private IP inside your VNet, tied to that one specific resource instance.
</details>

<details><summary>3. Why did Network Watcher's Topology view work without a VM, but IP Flow Verify wouldn't?</summary>

Topology just renders what's deployed (VNets, subnets, NICs, peerings) from the resource graph. IP Flow Verify actually needs a live NIC to test a real packet flow through NSG rules.
</details>

**Common mistakes / gotchas**

- Assuming NSGs never apply to private endpoint NICs — subnet-level private endpoint network policies control this, and the defaults have changed across Azure's history; check current behavior rather than assuming either way.
- Creating the private endpoint but skipping the `dns-zone-group` step — the zone exists but never gets the A record, so nothing actually resolves privately.
- Disabling public network access before confirming the private path works — verify the A record and connectivity first, lock the door second.

**Cost check + cleanup**

Private endpoints bill a small hourly charge plus data processing; Private DNS zones bill a small monthly charge (prorated) plus per-query charges — negligible for a same-day lab, but don't leave them running for weeks.

```bash
az network private-endpoint delete --name pe-stextpe --resource-group rg-ext-b
az network private-dns link vnet delete --resource-group rg-ext-b \
  --zone-name "privatelink.blob.core.windows.net" --name link-vnet-a --yes
az network private-dns zone delete --resource-group rg-ext-b \
  --name "privatelink.blob.core.windows.net" --yes
az group delete --name rg-ext-b --yes --no-wait
```

**What's next:** Back to Module 4 — Virtual Machines. Once `vm-app` exists, come back and run `az network watcher test-ip-flow` against it — the one Network Watcher tool this lab had to defer.

---

<a id="module-4"></a>
## Module 4 — Virtual Machines: Build, SSH In, Break the NSG, Fix It

This module plugs real VMs into the exact network you built in Module 3 (`rg-networking-lab`), then deliberately breaks and fixes connectivity so NSG priority evaluation stops being theory.

### 4.0 Learning objectives
- Explain the VM → NIC → Subnet → VNet chain and the VM → OS Disk → Managed Disk chain
- Create VMs explicitly resource-by-resource (not via one auto-generating command), so every piece has a name you control
- SSH into a public-facing VM, then reach a private-only VM through it via **SSH agent forwarding** (never copying your private key onto another machine)
- Install and test Nginx, then live break/fix a security rule and watch the effect
- Know exactly which VM-related resources keep billing after you "delete" a VM, and clean up all of them

### 4.1 Concepts before you touch the portal

- **Network side:** `VM → NIC → Subnet → VNet`. The NIC is the actual resource holding a private IP; it's what "puts" a VM into a subnet.
- **Storage side:** `VM → OS Disk → Managed Disk`. The OS disk is itself a standalone Managed Disk resource with its own lifecycle — it can outlive the VM if you're not careful during cleanup.
- **Public IP is a separate, explicit resource.** A VM doesn't get internet-reachable by default — you attach a Public IP resource to its NIC. Not attaching one (as with `vm-app` below) is a deliberate security choice, not an oversight.
- **SSH keys over passwords** — Azure's default and recommended Linux auth method. You keep the private key; only the public key ever goes on the VM.
- **Free-tier VM sizes**: `B1s`, `B2pts v2` (Arm), `B2ats v2` (AMD) are the burstable "B-series" sizes covered by 750 free hours/month *each* for 12 months on a new account. 750 hours ≈ one VM running continuously for a month — so two VMs for a focused lab session is fine, but don't leave both running 24/7 for weeks.
- **Standard SKU Public IPs are now the only option** — Microsoft retired the old Basic SKU in September 2025. Standard public IPs are "secure by default" (all inbound denied unless an NSG explicitly allows it), which is exactly what your `nsg-web` rule from Module 3 already does.

### 4.2 The lab you're building

```
rg-networking-lab  (from Module 3 — keep it)
│
├── vnet-a
│   ├── snet-web  → vm-web  (public IP + SSH, runs Nginx on :80)
│   └── snet-app  → vm-app  (NO public IP, reachable only from snet-web, runs a test server on :8080)
```

`vm-web` is your jump box into the private network — mirroring how a real bastion/jump host pattern works, minus the cost of actual Azure Bastion.

### 4.3 Hands-on Lab — build both VMs via CLI (resource by resource, on purpose)

Creating each network piece explicitly — rather than letting one `az vm create` auto-generate everything — means every resource has a name *you* chose, which makes cleanup deterministic later.

```bash
# --- vm-web: public-facing, in snet-web ---
az network public-ip create --resource-group rg-networking-lab --name pip-vm-web \
  --sku Standard --allocation-method Static

az network nic create --resource-group rg-networking-lab --name nic-vm-web \
  --vnet-name vnet-a --subnet snet-web --public-ip-address pip-vm-web

az vm create --resource-group rg-networking-lab --name vm-web \
  --nics nic-vm-web --image Ubuntu2204 --size Standard_B1s \
  --admin-username azureuser --generate-ssh-keys \
  --os-disk-name disk-vm-web

# --- vm-app: private only, in snet-app ---
az network nic create --resource-group rg-networking-lab --name nic-vm-app \
  --vnet-name vnet-a --subnet snet-app

az vm create --resource-group rg-networking-lab --name vm-app \
  --nics nic-vm-app --image Ubuntu2204 --size Standard_B1s \
  --admin-username azureuser --generate-ssh-keys \
  --os-disk-name disk-vm-app
```

Grab both IPs before you SSH anywhere:

```bash
# vm-web's public IP
az vm show --resource-group rg-networking-lab --name vm-web -d --query publicIps -o tsv

# vm-app's private IP
az network nic show --resource-group rg-networking-lab --name nic-vm-app \
  --query "ipConfigurations[0].privateIPAddress" -o tsv
```

### 4.4 SSH in — with agent forwarding, not a copied key

```bash
# Once, locally: load your key into the SSH agent
ssh-add ~/.ssh/id_rsa

# -A forwards your agent so vm-web can use your key to reach vm-app,
# without your private key ever being copied onto vm-web
ssh -A azureuser@<vm-web-public-ip>

# From inside vm-web, jump to vm-app using the forwarded agent
ssh azureuser@<vm-app-private-ip>
# exit back to vm-web when done: exit
```

### 4.5 Install and test Nginx on vm-web

```bash
# on vm-web
sudo apt-get update && sudo apt-get install -y nginx
curl localhost   # confirms Nginx is serving locally
```

Now open `http://<vm-web-public-ip>` in your browser. It should load immediately — because `nsg-web`'s Module 3 rule (allow 80/443 from Internet, priority 100) is already in place. This is the payoff of building the network first: the VM just works the moment it lands in it.

### 4.6 Break it, then fix it

From **your own machine** (not SSH'd in), add a higher-priority deny rule:

```bash
az network nsg rule create --resource-group rg-networking-lab --nsg-name nsg-web \
  --name Temp-Block-HTTP --priority 90 --direction Inbound --access Deny --protocol Tcp \
  --source-address-prefixes Internet --destination-port-ranges 80 443
```

Refresh the browser — it should now time out. Priority 90 is evaluated before priority 100, so the deny wins even though the original allow rule is untouched. This is Module 3's priority lesson, now visible as a broken web page instead of a sentence.

Fix it:

```bash
az network nsg rule delete --resource-group rg-networking-lab --nsg-name nsg-web --name Temp-Block-HTTP
```

Refresh again — back to working.

### 4.7 Prove the segmentation: reach vm-app only from inside the VNet

On `vm-app` (SSH there via agent forwarding as above), start a throwaway server:

```bash
python3 -m http.server 8080 &
```

Back on `vm-web`:

```bash
curl http://<vm-app-private-ip>:8080
```

This succeeds — `nsg-app` allows port 8080 from `snet-web`'s range, and `vm-web` lives there. Since `vm-app` has no public IP at all, it has **no direct endpoint reachable from the public internet** — the segmentation from Module 3 is now a real, testable fact rather than a diagram. That's not the same as "unreachable by any means, period": approved private-management paths — Azure Bastion (connects over the VM's *private* IP, no public IP needed on the target), a VPN/ExpressRoute connection, or another peered/private network — can still reach it deliberately. What no public IP removes is the *casual, direct* internet path, which is exactly the thing this lab is testing. Stop the test server when done: back on `vm-app`, `kill %1`.

### 4.8 Self-check questions

<details>
<summary>Question 1 — Why shouldn't vm-app get a public IP, given the Module 3 design?</summary>

The three-tier segmentation deliberately keeps the app/db tiers unreachable from the internet — only the web tier is meant to be internet-facing. Attaching a public IP to `vm-app` would open a **direct internet-facing endpoint** on it, which is exactly what this design avoids — regardless of intent, it puts an internet path onto a tier that's supposed to have none. To be precise: this isn't because a public IP "bypasses" the NSG — the NSG still filters traffic to that IP just as it would any other. The problem is architectural, not a permissions hole: the whole point of the segmentation is that the app tier has no internet-facing surface at all to defend in the first place.
</details>

<details>
<summary>Question 2 — Trace the full chain from "VM" to "physical bits," and separately from "VM" to "the internet."</summary>

Storage side: VM → OS Disk → Managed Disk (backed by Azure Storage infrastructure). Network side: VM → NIC → Subnet → VNet → (optionally) Public IP → Internet. Both chains are separate resources with independent lifecycles — deleting the VM doesn't automatically remove the disk, NIC, or public IP.
</details>

<details>
<summary>Question 3 — Why use SSH agent forwarding (-A) instead of copying your private key onto vm-web?</summary>

Copying your private key onto any remote machine means that machine (and anyone who compromises it) now holds a credential that can authenticate as you to anything else that trusts that key — including production systems, if you reuse keys. Agent forwarding lets vm-web *use* your local key to authenticate onward without the key material ever leaving your own machine.
</details>

<details>
<summary>Question 4 — Why did the browser stop working after adding Temp-Block-HTTP at priority 90, even though the priority-100 Allow rule was untouched?</summary>

NSG rules are evaluated lowest-priority-number-first, and the first match wins. Priority 90 is evaluated before priority 100, so the new Deny rule matches the traffic first and stops evaluation right there — the Allow rule at 100 never gets a chance to apply.
</details>

<details>
<summary>Question 5 — Why doesn't `az vm delete` alone fully stop billing for that VM?</summary>

`az vm delete` removes the VM resource itself, but the OS disk (a Managed Disk), the NIC, and any attached Public IP are separate Azure resources with independent lifecycles — they are not automatically deleted with the VM. A Managed Disk keeps incurring storage cost, and a Standard Public IP keeps incurring its small hourly cost, until you delete them explicitly.
</details>

### 4.9 Common mistakes / gotchas

- **Forgetting disks and public IPs bill independently of the VM** — a "deleted" VM can still be quietly costing you if the disk and IP are left behind.
- **Attaching a public IP to vm-app "just to make testing easier"** — defeats the entire point of the lab and the Module 3 design.
- **Leaving `Temp-Block-HTTP` in place** after the break/fix exercise — clean up lab-only NSG rules, not just lab-only resources.
- **Copying your private key onto vm-web** instead of using `-A` — a habit that's genuinely dangerous outside a disposable lab.
- **Resizing a running VM to an incompatible size** — most resizes require deallocation first (`az vm deallocate`), since the new size may live on different underlying hardware. Plan for brief downtime, don't assume it's live.

### 4.10 Cost check + cleanup

What's actually billable from this module: two `B1s` VMs (free-tier eligible, watch the combined 750 hr/month pool), one Standard Public IP (small hourly cost, **not** covered by the free VM hours), and two OS disks (small ongoing storage cost even when the VM is stopped).

Clean up just the VM-related resources — leave `vnet-a`, `vnet-b`, the NSGs, peering, and route table standing, since those are free and Module 5 doesn't need new networking:

```bash
az vm delete --resource-group rg-networking-lab --name vm-web --yes
az vm delete --resource-group rg-networking-lab --name vm-app --yes

az disk delete --resource-group rg-networking-lab --name disk-vm-web --yes
az disk delete --resource-group rg-networking-lab --name disk-vm-app --yes

az network nic delete --resource-group rg-networking-lab --name nic-vm-web
az network nic delete --resource-group rg-networking-lab --name nic-vm-app

az network public-ip delete --resource-group rg-networking-lab --name pip-vm-web

# In case it was forgotten earlier
az network nsg rule delete --resource-group rg-networking-lab --nsg-name nsg-web --name Temp-Block-HTTP
```

Confirm nothing billable is left:

```bash
az resource list --resource-group rg-networking-lab --output table
```

You should see only VNets/subnets, NSGs, the peerings, and the route table — all free.

### 4.11 What's next

**Module 5 — Storage** moves to Blob/Files/Queue/Table, SAS tokens, access tiers, and replication — cheap to experiment with and a good change of pace after two networking-heavy modules.

Run through Module 4, and say the word for Module 5 — or ask now if SSH agent forwarding, the NSG break/fix, or cleanup didn't go as expected.

---

<a id="ext-c"></a>
## Extension Lab C — Availability Zones & VM Scale Sets

*Sits between Modules 4 and 5 — builds directly on the VM concepts Module 4 just covered.*

**Objectives**
- Explain Availability Zone vs. Availability Set, and why they protect against different-sized failures
- Deploy a VM Scale Set spread across zones and watch autoscale add/remove instances
- Know the Flexible vs. Uniform orchestration distinction well enough for AZ-104

**Concepts**

An **Availability Set** is a logical grouping *within a single datacenter*, spreading VMs across fault domains (separate power/network) and update domains (separate maintenance timing). It protects against a rack failure or a maintenance reboot taking every instance down at once — not against the whole datacenter failing.

An **Availability Zone** is a physically separate datacenter within the same region, each with independent power, cooling, and networking. `centralindia` — the region used throughout this notebook — has **3 availability zones**. Spreading VMs, or better, a Scale Set, across zones protects against a full datacenter-level outage: a strictly bigger blast radius than an Availability Set covers.

**Zonal** vs. **zone-redundant**: a zonal resource is pinned to one specific zone you choose (`--zone 1`) and doesn't survive that zone failing. A zone-redundant resource (Standard Load Balancer, ZRS storage) is spread across zones automatically by Azure, without you picking one. A Scale Set with `--zones 1 2 3` is the VM-level version of zone-redundancy.

**VM Scale Sets** manage a group of near-identical VM instances behind autoscale rules. **Flexible** orchestration (the current default and recommended mode) treats instances much like standalone VMs under shared management, supports mixed VM sizes, and integrates cleanly with per-instance zone placement. **Uniform** orchestration is the older model, still around for compatibility with tooling built against it. Autoscale rules watch a metric (commonly CPU%) over a window and add (scale out) or remove (scale in) instances when thresholds cross — horizontal scaling, as opposed to the vertical scaling ("bigger machine") Extension Lab E covers for App Service.

**Hands-on — Portal**

1. **Virtual machine scale sets → + Create**. Resource group `rg-ext-c`. Name `vmss-ext-c`. Orchestration: **Flexible**. Region `centralindia`. Availability zones: select **1, 2, and 3**. Image: Ubuntu Server 24.04 LTS. Size: `Standard_B1s`. Instance count: **2**. Authentication: SSH public key.
2. After creation: **Instances** blade — each instance lists which zone it landed in.
3. **Scaling** blade → Custom autoscale. Minimum 1, scale-out rule (CPU% > 70 over 5 min → add 1), scale-in rule (CPU% < 30 over 5 min → remove 1), maximum 3.
4. Optional: SSH or `az vm run-command` into an instance and run a short CPU-burn loop for a couple of minutes to watch autoscale actually add an instance, then stop the load and watch it scale back in.

**Hands-on — CLI**

```bash
az group create --name rg-ext-c --location centralindia \
  --tags project=azure-learning module=ext-c-availability

az vmss create \
  --resource-group rg-ext-c --name vmss-ext-c \
  --orchestration-mode Flexible \
  --image Ubuntu2404 \
  --vm-sku Standard_B1s \
  --instance-count 2 \
  --zones 1 2 3 \
  --admin-username azureuser \
  --generate-ssh-keys

# Which zone did each instance land in?
az vmss list-instances --resource-group rg-ext-c --name vmss-ext-c \
  --query "[].{name:name, zone:zones[0], power:provisioningState}" -o table

# Autoscale: out above 70% CPU, in below 30%, capped 1-3 instances
az monitor autoscale create \
  --resource-group rg-ext-c --resource vmss-ext-c \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-ext-c --min-count 1 --max-count 3 --count 2

az monitor autoscale rule create \
  --resource-group rg-ext-c --autoscale-name autoscale-ext-c \
  --condition "Percentage CPU > 70 avg 5m" --scale out 1

az monitor autoscale rule create \
  --resource-group rg-ext-c --autoscale-name autoscale-ext-c \
  --condition "Percentage CPU < 30 avg 5m" --scale in 1

# Compare: a single zonal VM, pinned to one specific zone
az vm create --resource-group rg-ext-c --name vm-zonal-ext-c \
  --image Ubuntu2404 --size Standard_B1s --zone 1 \
  --admin-username azureuser --generate-ssh-keys
```

**Self-check questions**

<details><summary>1. Your app runs on a single VM in an Availability Set with 3 fault domains. The datacenter housing it loses power entirely. Does the Availability Set help?</summary>

No — an Availability Set only spreads risk *within* one datacenter. It has no concept of a second, physically separate datacenter, which is what Availability Zones add.
</details>

<details><summary>2. You set <code>--zones 1 2 3</code> on a Scale Set with 2 instances. Are both instances guaranteed to land in different zones?</summary>

Not guaranteed for 2 instances across 3 zones — Azure spreads them, but with an instance count lower than the zone count, don't assume a specific layout; check <code>list-instances</code> rather than assuming.
</details>

<details><summary>3. Why does Flexible orchestration mode matter for AZ-104 specifically, beyond "it's newer"?</summary>

It's the current default/recommended mode and integrates cleanly with per-instance zone placement and mixed VM sizes — Uniform mode's constraints (identical instances, a different scaling model) are exactly the kind of distinction exam questions probe.
</details>

<details><summary>4. Does spreading instances across 3 zones instead of 1 change what Azure bills you for the VMs themselves?</summary>

No extra charge for the zone assignment itself — you pay for the compute you provision either way. Cross-zone bandwidth for chatty inter-instance traffic is a separate, usually small, consideration.
</details>

**Common mistakes / gotchas**

- Treating Availability Sets and Availability Zones as interchangeable — they protect against different-sized failures, and a VM can't be in both at once.
- Assuming Uniform mode is required for "real" Scale Sets — Flexible is the current default for new deployments.
- Forgetting a Scale Set creates *multiple* VMs (and disks) — cleanup has to account for every instance, not one.
- Leaving a CPU-burn test running and forgetting about it — it costs compute time and can trigger scale-out charges you didn't mean to pay for.

**Cost check + cleanup**

Each B1s instance draws from the same 750-free-hours/month pool as any other burstable B-series VM in your subscription — running 2–3 instances at once burns through that pool 2–3x faster than a single VM. Don't leave this running overnight.

```bash
az group delete --name rg-ext-c --yes --no-wait
```

Deleting the resource group takes the Scale Set, its instances, their disks, the autoscale setting, and the standalone comparison VM with it in one shot — the reason this notebook keeps steering you toward one resource group per lab.

**What's next:** Back to Module 5 — Storage.

---

<a id="module-5"></a>
## Module 5 — Storage: Blob/Files/Queue/Table, Access Tiers, Replication, SAS

A change of pace after two networking-heavy modules — storage is cheap to experiment with, and this is where the "RBAC over passwords" idea from Module 2 gets its first real payoff.

### 5.0 Learning objectives
- Create a storage account and understand its four services: Blob, Files, Queue, Table
- Explain replication options (LRS/ZRS/GRS/RA-GRS) and what specific failure each protects against
- Explain access tiers (Hot/Cool/Cold/Archive) and the storage-cost-vs-retrieval-cost trade-off
- Access blob data using **Entra ID + RBAC** instead of account keys, and generate a **User Delegation SAS**
- Set up a basic lifecycle management policy

### 5.1 Concepts before you touch the portal

- **Storage account** — a globally unique namespace (it becomes part of a DNS name, e.g. `<name>.blob.core.windows.net`) hosting one or more of: **Blob** (unstructured objects — files, images, backups), **Files** (SMB/NFS shares), **Queue** (simple messaging), **Table** (NoSQL key-value).
- **Replication options** — each protects against a different failure scope:
  - `LRS` (Locally Redundant) — 3 copies in one datacenter. Protects against disk/node failure only. Cheapest.
  - `ZRS` (Zone Redundant) — 3 copies across availability zones in the same region. Protects against a whole datacenter/zone going down.
  - `GRS` (Geo-Redundant) — LRS in your region + async-replicated copy in the paired region. The secondary copy isn't readable unless Microsoft fails over (or you use `RA-GRS`, which makes it readable).
  - `RA-GRS` — GRS with the secondary made readable at all times.
  - More redundancy = more durability/availability = more cost. Pick based on what failure you're actually protecting against, not by default.
- **Access tiers** (Blob only) — the same trade-off in a different dimension: cheaper to *store*, more expensive to *retrieve*, as you move down:
  - `Hot` — frequent access, cheapest to read, priciest to store.
  - `Cool` — infrequent (30+ day) access, cheaper storage, pricier retrieval, minimum-duration charges apply.
  - `Cold` — even less frequent (90+ days), cheaper still.
  - `Archive` — rarely accessed, cheapest storage by far, but **offline** — reading it requires "rehydration" that takes hours, not milliseconds.
- **Account keys vs. Entra ID/RBAC for data access** — every storage account has two master keys that grant **full, unscoped access to everything in it**. They don't expire and aren't tied to a specific identity — closer to a shared password than a credential. The modern, recommended pattern is Entra ID + RBAC roles like `Storage Blob Data Contributor` / `Storage Blob Data Reader`, assigned to actual identities — including the user-assigned managed identity you created back in Module 2. This is exactly the RBAC target that identity was built for.
- **Important AZ-104 nuance:** `Contributor` on the storage account's resource group is a **control-plane** role (manage the account itself — create it, delete it, change its settings). It does **not** automatically grant **data-plane** access (read/write actual blobs) — those are separate permission systems layered on top of each other. You'll hit this directly in the lab below.
- **SAS (Shared Access Signature)** — a signed URL granting time-limited, scoped access without sharing a master key. A **User Delegation SAS** (signed using your Entra ID credentials rather than an account key) is the secure, recommended form — it inherits an expiry and can be tied to your own RBAC permissions, unlike an account-key SAS which is only as revocable as rotating the whole key.

### 5.2 Hands-on Lab (storage account names must be globally unique, lowercase, no hyphens, 3–24 chars — swap `stazlearnbvj` below for your own)

**Portal walkthrough:**

1. **Storage accounts → + Create.** Resource group `rg-storage-lab` (tags `project=azure-learning`, `module=05-storage`), name `stazlearnbvj`, region Central India, Performance **Standard**, Redundancy **LRS**. Review + Create.
2. Open it → **Access control (IAM) → Add role assignment → Storage Blob Data Contributor** → assign to yourself. This is the control-plane-vs-data-plane split from 5.1, visible directly in the Portal: you can already manage this account (you created it), but until this step you cannot read or write the data inside it.
3. **Data storage → Containers → + Container** → name `demo-container`.
4. Open the container → **Upload** → pick any local file → Upload.
5. Click the uploaded blob → **Generate SAS** (top toolbar) → Permissions `Read`, Expiry `1 hour` → **Generate SAS token and URL** → copy the **Blob SAS URL**. Paste it into a new private/incognito browser tab — it downloads without you being signed into Azure there.
6. On the same blob's details pane, use **Change tier** to move it to `Cool`.
7. **Data storage → File shares / Queues / Tables** — each has its own `+ File share` / `+ Queue` / `+ Table` button, same pattern as containers.
8. **Data management → Lifecycle management → + Add a rule** — the wizard (Rule scope → Blob type → Base blob conditions → Actions) builds the exact same JSON policy you'd otherwise hand-author below; worth running through once to see how the two map to each other.

**CLI walkthrough:**

```bash
az group create --name rg-storage-lab --location centralindia \
  --tags project=azure-learning module=05-storage

az storage account create \
  --name stazlearnbvj --resource-group rg-storage-lab --location centralindia \
  --sku Standard_LRS --kind StorageV2 --access-tier Hot

# Grant YOURSELF data-plane access — this is the control-plane vs. data-plane
# split in action. Being Owner/Contributor on the RG does not do this for you.
STG_ID=$(az storage account show --name stazlearnbvj --resource-group rg-storage-lab --query id -o tsv)
az role assignment create \
  --assignee "<your-upn-or-object-id>" \
  --role "Storage Blob Data Contributor" \
  --scope "$STG_ID"

# Create a container and upload a test file — using Entra ID auth, not an account key
echo "hello from the free-credit lab" > hello.txt

az storage container create \
  --account-name stazlearnbvj --name demo-container --auth-mode login

az storage blob upload \
  --account-name stazlearnbvj --container-name demo-container \
  --name hello.txt --file ./hello.txt --auth-mode login
```

If the role assignment above hasn't propagated yet (can take a minute), the upload will fail with an authorization error — that's expected and worth noticing, not a bug.

**Generate a User Delegation SAS and prove it works without any Azure login:**

```bash
EXPIRY=$(date -u -d "+1 hour" '+%Y-%m-%dT%H:%MZ')   # 1 hour from now, UTC

SAS_URL=$(az storage blob generate-sas \
  --account-name stazlearnbvj --container-name demo-container --name hello.txt \
  --permissions r --expiry "$EXPIRY" --auth-mode login --as-user \
  --full-uri --output tsv)

echo "$SAS_URL"
curl "$SAS_URL"   # should print the file contents — no Azure credentials involved in this curl at all
```

**Change access tier, and touch the other three services:**

```bash
az storage blob set-tier \
  --account-name stazlearnbvj --container-name demo-container --name hello.txt \
  --tier Cool --auth-mode login

az storage share create --account-name stazlearnbvj --name demo-share
az storage queue create --account-name stazlearnbvj --name demo-queue
az storage table create --account-name stazlearnbvj --name demotable
```

(If Files/Queue/Table commands complain about auth mode, fall back to a connection string for just those: `az storage account show-connection-string --name stazlearnbvj --resource-group rg-storage-lab` — Entra ID data-plane support varies slightly by service.)

**A basic lifecycle management policy** (move to Cool at 30 days, delete at 365) — save as `policy.json`:

```json
{
  "rules": [
    {
      "name": "demo-lifecycle-rule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": { "blobTypes": ["blockBlob"], "prefixMatch": ["demo-container/"] },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

```bash
az storage account management-policy create \
  --account-name stazlearnbvj --resource-group rg-storage-lab --policy @policy.json
```

### 5.3 Self-check questions

<details>
<summary>Question 1 — LRS vs. ZRS: what specific failure does each protect against that the other doesn't?</summary>

LRS keeps 3 copies within a single datacenter — it survives a disk or node failure, but not the loss of that whole datacenter. ZRS spreads 3 copies across separate availability zones in the same region — it survives a full datacenter/zone outage, which LRS cannot.
</details>

<details>
<summary>Question 2 — Why would you deliberately choose Cool or Archive despite the higher retrieval cost?</summary>

When data is accessed rarely (backups, compliance archives, old logs), the storage-cost savings over months or years vastly outweigh the occasional higher retrieval cost. The tier is a bet on *access frequency*, not a universal "cheaper is better" choice — picking Archive for data you read daily would cost far more overall due to retrieval charges and rehydration delay.
</details>

<details>
<summary>Question 3 — Security difference between an account-key SAS and a User Delegation SAS?</summary>

An account-key SAS is signed with one of the account's master keys — it's valid until it expires *or* until you rotate that key (which invalidates every SAS signed with it, including unrelated ones). A User Delegation SAS is signed with your Entra ID credentials, scoped to your actual RBAC permissions, and can't be revoked by rotating an account key — it's tied to identity, not a shared secret.
</details>

<details>
<summary>Question 4 — Why did you need to explicitly grant yourself "Storage Blob Data Contributor" despite already being Owner/Contributor on the resource group?</summary>

Azure Storage has two separate permission layers: control-plane (manage the storage account resource itself — Owner/Contributor covers this) and data-plane (read/write the actual blobs, queues, tables inside it — this needs its own RBAC roles like `Storage Blob Data Contributor`). Being able to manage the account doesn't automatically grant access to the data inside it — a deliberate security boundary, not an oversight.
</details>

<details>
<summary>Question 5 — Where does a lifecycle policy fit into real-world cost optimization, and what's the risk of an aggressive delete rule?</summary>

It's how teams stop manually managing tiering/expiry on millions of objects — data ages into cheaper tiers automatically as it becomes less likely to be accessed, and old data expires without anyone remembering to clean it up. The risk of an aggressive "delete after N days" rule is deleting something that turns out to still be needed — for compliance, an audit, or a support case — with no easy way back once it's gone. Real policies are usually paired with a retention/legal-hold review before the delete action is added, not just the tiering actions.
</details>

### 5.4 Common mistakes / gotchas

- **Using account keys out of habit** instead of Entra ID/RBAC or a User Delegation SAS — keys are all-or-nothing, don't expire, and rotating one breaks every SAS token signed with it, not just the one you meant to revoke.
- **Storage account naming** — must be globally unique across *all* of Azure, lowercase letters and numbers only, no hyphens, 3–24 characters. A "name already taken" error is common and not a sign anything's wrong.
- **Assuming Contributor covers data access** — see self-check Q4. This trips people up in real environments constantly, not just in labs.
- **Setting Archive on something you'll need soon** — rehydration is measured in hours, not seconds; it's the wrong tier for anything on an active workflow.
- **Ignoring minimum storage duration charges** — moving a blob out of Cool/Cold/Archive before its minimum duration can still bill as if it stayed the full minimum period.

### 5.5 Cost check + cleanup

A few text files across four services should cost single-digit rupees at most for the lab's duration.

```bash
az group delete --name rg-storage-lab --yes
```

### 5.6 What's next

**Module 6 — Load Balancing** is the first module where you need to be genuinely careful: Load Balancer is cheap, but **Application Gateway bills hourly regardless of use** and is exactly the kind of service Part 2's ground rules warned about. Plan to create it, study it, and tear it down the same day — not something to leave running "to come back to later."

Run through Module 5, and say the word for Module 6 — or ask now if the RBAC data-plane step or the SAS generation didn't behave as expected.

---

<a id="ext-d"></a>
## Extension Lab D — Storage Firewall, AzCopy & Data Protection

*Sits between Modules 5 and 6 — a compact follow-on to Module 5's storage lab.*

**Objectives**
- Restrict a storage account to specific networks with the storage firewall, and see how it differs from the Private Endpoint approach in Extension Lab B
- Turn on blob versioning and soft delete, then actually lose and recover a blob
- Move data with AzCopy instead of one-file-at-a-time uploads

**Concepts**

The **storage firewall** (Networking blade → "Enabled from selected virtual networks and IP addresses") is a coarse allow-list at the account level — "only these IPs / only these VNet subnets may reach this account at all," with a default-deny fallback. It's cheaper and simpler than a Private Endpoint but coarser: it's still the account's *public* endpoint being filtered, not a private IP. The two aren't mutually exclusive — production setups often combine both, the firewall as the outer gate and a private endpoint as the actual path for trusted VNets.

Data protection is layered, and each layer catches a different mistake. **Versioning** automatically keeps every prior version of a blob when it's overwritten — protects against "an application overwrote the file with bad data." **Soft delete** keeps a deleted blob recoverable for a retention window — protects against "something deleted the file." A **snapshot** is a manual, point-in-time copy you trigger yourself. None substitutes for the others: versioning does nothing if a blob is *deleted* rather than overwritten, and soft delete does nothing if it's *overwritten* rather than deleted.

**AzCopy** is a purpose-built CLI utility for high-throughput, parallelized, resumable data transfer — the right tool once you're moving more than a handful of files, where `az storage blob upload` starts to feel slow one file at a time.

**Hands-on — Portal**

1. **Storage accounts → + Create** → `stextdbvj` in a fresh `rg-ext-d`, Standard_LRS.
2. **Networking** blade → **Enabled from selected virtual networks and IP addresses** → use the "Add your client IP address" button → **Save**.
3. Browse a container under **Storage browser** — it works, because your IP is now allow-listed. Notice that *without* adding your IP first, this same Portal blade would itself get denied, since the Portal's storage browser calls the data plane from your browser's IP.
4. **Data protection** blade → enable **"Enable versioning for blobs"** and **"Enable soft delete for blobs"** (retention: 7 days) → **Save**.
5. Storage browser → create a container → upload a small text file. Overwrite it with different content (same name). Click the blob → **Version history** — the original content is recoverable there.
6. Delete the blob entirely → toggle **Show deleted blobs** in the container view → the deleted blob appears grayed out → **Undelete**.

**Hands-on — CLI**

```bash
az group create --name rg-ext-d --location centralindia \
  --tags project=azure-learning module=ext-d-storage-hardening

az storage account create --name stextdbvj --resource-group rg-ext-d \
  --location centralindia --sku Standard_LRS --kind StorageV2

MY_IP=$(curl -s ifconfig.me)
az storage account update --name stextdbvj --resource-group rg-ext-d \
  --default-action Deny
az storage account network-rule add --account-name stextdbvj \
  --resource-group rg-ext-d --ip-address "$MY_IP"

# Versioning and soft delete are both properties of the blob service — one call
az storage account blob-service-properties update --account-name stextdbvj \
  --resource-group rg-ext-d --enable-versioning true \
  --enable-delete-retention true --delete-retention-days 7

# Being Owner/Contributor on the account does NOT grant blob data access —
# Entra-based data operations (--auth-mode login) need an explicit data-plane
# role. Grant yourself Storage Blob Data Contributor before touching blobs.
STG_ID=$(az storage account show --name stextdbvj --resource-group rg-ext-d --query id -o tsv)
MY_ID=$(az ad signed-in-user show --query id -o tsv)
az role assignment create --assignee-object-id "$MY_ID" --assignee-principal-type User \
  --role "Storage Blob Data Contributor" --scope "$STG_ID"
# RBAC can take a minute or two to propagate — if the next command fails
# with an authorization error, wait briefly and retry rather than assuming
# something's misconfigured.

az storage container create --account-name stextdbvj --name demo-container --auth-mode login

echo "version one" > file.txt
az storage blob upload --account-name stextdbvj --container-name demo-container \
  --name file.txt --file file.txt --auth-mode login

echo "version two" > file.txt
az storage blob upload --account-name stextdbvj --container-name demo-container \
  --name file.txt --file file.txt --auth-mode login --overwrite

# "version one" content is still retrievable by version ID
az storage blob list --account-name stextdbvj --container-name demo-container \
  --include v -o table

az storage blob delete --account-name stextdbvj --container-name demo-container \
  --name file.txt --auth-mode login

# Soft-deleted, not gone — undelete it
az storage blob undelete --account-name stextdbvj --container-name demo-container \
  --name file.txt --auth-mode login

# AzCopy — a short-lived SAS, then a parallelized copy of a local folder
END=$(date -u -d "1 hour" '+%Y-%m-%dT%H:%MZ')
SAS=$(az storage container generate-sas --account-name stextdbvj --name demo-container \
  --permissions racwdl --expiry "$END" --auth-mode login --as-user -o tsv)
mkdir sample-folder && echo "hello" > sample-folder/a.txt && echo "world" > sample-folder/b.txt
azcopy copy "sample-folder/*" "https://stextdbvj.blob.core.windows.net/demo-container?$SAS"
```

**Self-check questions**

<details><summary>1. You set the storage account's default action to Deny and forgot to allow-list your own IP first. What happens the next time you open the storage account's Storage Browser in the Portal?</summary>

It gets denied too — the Portal's browser makes data-plane calls from your browser's own IP, so a Deny-by-default firewall locks you out of the Portal view the same way it locks out any other caller.
</details>

<details><summary>2. A script accidentally deleted 200 blobs from a container. Soft delete is off, versioning is on. Are the blobs recoverable?</summary>

No — versioning only helps when a blob is *overwritten*; it captures nothing for a *deleted* blob. Only soft delete (or a separate backup) recovers from a deletion.
</details>

<details><summary>3. What's the actual difference between the storage firewall and a Private Endpoint for "keep this account off the internet"?</summary>

The firewall filters access to the account's still-public endpoint by source IP/VNet; a Private Endpoint gives the account a private IP with no public endpoint involved once public access is disabled. Firewall is coarser and cheaper; Private Endpoint is the stronger guarantee.
</details>

**Common mistakes / gotchas**

- Turning on `--default-action Deny` before adding any allow rule — instant self-lockout, including from the Portal.
- Enabling versioning *after* the data you cared about was already overwritten — it only protects overwrites that happen after it's turned on.
- Assuming AzCopy authenticates the same way as `az storage` commands automatically — it needs its own SAS token, connection string, or `azcopy login`, not your `az login` session.
- Assuming Owner/Contributor on the account is enough to run `--auth-mode login` commands — that's a control-plane role; Entra-based blob data access needs its own data-plane role (`Storage Blob Data Contributor` or similar), same lesson as Module 5's RBAC section.

**Cost check + cleanup**

Versioning keeps old blob content around, so storage costs creep up the longer test overwrites sit there — not significant for a same-day lab, but worth remembering for real workloads.

```bash
az group delete --name rg-ext-d --yes --no-wait
```

**What's next:** Back to Module 6 — Load Balancing.

---

<a id="ext-g"></a>
## Extension Lab G — Application Security Groups, Effective Security Rules & Stored Access Policies

*Sits between Extension Lab D and Module 6. Two unrelated but genuinely lab-shaped, cheap AZ-104 gaps in one compact lab: the networking half closes the "come back once a VM exists" promise made in Extension Lab B; the storage half adds a third SAS revocation model to Module 5's two.*

**Objectives**
- Group VMs into an Application Security Group and reference it in an NSG rule instead of a hardcoded IP range
- Read a NIC's actual effective security rules — the resolved combination of every NSG in play — instead of guessing from the rules you wrote
- Issue a SAS tied to a stored access policy, then revoke just that SAS without rotating account keys or touching anything else

**Concepts**

An **Application Security Group (ASG)** is a logical grouping you attach to NICs, then reference *as the source or destination in an NSG rule* instead of an IP range or CIDR block — "allow `asg-web` to reach `asg-app` on port 8080" keeps working automatically as VMs are added or removed from the group, rather than requiring a rule update every time your IP layout changes. One constraint worth knowing: when two ASGs are used together in the same rule (one as source, one as destination), **both must belong to the same virtual network** — a rule referencing ASGs from different VNets won't behave the way you'd expect.

**Effective security rules** matter because NSGs can be attached at the subnet level *and* the NIC level simultaneously (Module 3/4 already built exactly this: `nsg-app` at the subnet, potentially another at the NIC), plus every NSG layers its own default rules underneath anything you add. "Effective rules" is the actual resolved rule set Azure applies to a specific NIC after merging all of that — the tool to reach for when "I wrote a rule that should allow this and it's still blocked," instead of re-reading rules by eye and guessing which one wins.

A **stored access policy** moves a SAS's permissions and expiry *off* the signed token and onto a named policy object stored server-side on the container. An ad-hoc account-key SAS bakes its permissions/expiry into the token itself — the only way to kill it early is rotating the account key, which kills every other SAS signed with that key too. A SAS issued against a stored access policy instead references the policy by name; change or delete the policy, and every SAS tied to it dies immediately, without touching the account key or any unrelated token. One nuance worth being precise about: stored access policies work with an **account-key-signed (service) SAS** — they do **not** apply to a **User Delegation SAS** (Module 5's Entra-signed option), which is revoked through a completely separate trust chain (the user delegation key / your own RBAC access), not through container-level policies.

**Hands-on — Portal**

1. Module 4's cleanup already deleted `vm-web`/`vm-app`, but `rg-networking-lab`'s VNets and NSGs are still standing. Create one small VM — `vm-asg-demo`, Ubuntu, `Standard_B1s` — into `snet-app` of that existing network.
2. On the VM's **Networking** blade → **Application security groups** tab → **Configure the application security groups** → create `asg-app-demo` and attach it.
3. `nsg-app` → **Inbound security rules → + Add**. Source: **Application Security Group** → `asg-app-demo`. Destination port: `8080`. Action: Allow. Save.
4. VM → **Networking → Effective security rules** — Azure renders the fully resolved rule set for this NIC directly, combining the subnet-level NSG, anything at the NIC level, your new ASG-based rule, and the default rules underneath all of it.
5. Storage: on a container (reuse one from an earlier lab, or a fresh account) → **Access policy** blade → **+ Add policy** → name, permissions, expiry → **Save**. Then **Generate SAS** blade → choose the saved access policy from the dropdown instead of setting permissions/expiry by hand → generate → confirm the URL works. Go back to **Access policy**, delete it → try the same URL again — it now fails, even though its own listed expiry hasn't passed yet.

**Hands-on — CLI**

```bash
# --- Part 1: Application Security Groups + Effective Security Rules ---
# rg-networking-lab's VNets/NSGs are still standing after Module 4's cleanup;
# just the VMs were deleted. Rebuild one small VM to demonstrate against.

az network nic create --resource-group rg-networking-lab --name nic-vm-asg-demo \
  --vnet-name vnet-a --subnet snet-app

az vm create --resource-group rg-networking-lab --name vm-asg-demo \
  --nics nic-vm-asg-demo --image Ubuntu2404 --size Standard_B1s \
  --admin-username azureuser --generate-ssh-keys

az network asg create --resource-group rg-networking-lab --name asg-app-demo

az network nic update --resource-group rg-networking-lab --name nic-vm-asg-demo \
  --app-security-groups asg-app-demo

# Reference the ASG as a source, instead of a CIDR range
az network nsg rule create --resource-group rg-networking-lab --nsg-name nsg-app \
  --name Allow-From-ASG-Demo --priority 200 --direction Inbound --access Allow \
  --protocol Tcp --source-asgs asg-app-demo --destination-port-ranges 8080

# The actual resolved rule set for this NIC — subnet NSG + NIC-level config +
# defaults, all merged
az network nic list-effective-nsg --resource-group rg-networking-lab \
  --name nic-vm-asg-demo -o table

# --- Part 2: Stored Access Policies ---
az group create --name rg-ext-g --location centralindia \
  --tags project=azure-learning module=ext-g-asg-sap

az storage account create --name stextgbvj --resource-group rg-ext-g \
  --location centralindia --sku Standard_LRS --kind StorageV2

MY_ID=$(az ad signed-in-user show --query id -o tsv)
STG_ID=$(az storage account show --name stextgbvj --resource-group rg-ext-g --query id -o tsv)
az role assignment create --assignee-object-id "$MY_ID" --assignee-principal-type User \
  --role "Storage Blob Data Contributor" --scope "$STG_ID"

az storage container create --account-name stextgbvj --name policy-demo --auth-mode login
echo "hello from a stored access policy" > policy-file.txt
az storage blob upload --account-name stextgbvj --container-name policy-demo \
  --name policy-file.txt --file policy-file.txt --auth-mode login

# The policy holds the permissions + expiry — not the token
az storage container policy create --account-name stextgbvj --container-name policy-demo \
  --name read-only-week --permissions r --expiry $(date -u -d "+7 days" '+%Y-%m-%dT%H:%MZ')

# Issue a SAS against the policy — no --permissions/--expiry here, they're
# inherited. NOTE: no --as-user — a stored access policy needs an account-key
# (service) SAS, not a User Delegation SAS.
SAS_URL=$(az storage blob generate-sas --account-name stextgbvj --container-name policy-demo \
  --name policy-file.txt --policy-name read-only-week --auth-mode login \
  --full-uri --output tsv)

curl "$SAS_URL"   # should print the file contents

# Revoke — without rotating the account key or touching any other SAS/policy
az storage container policy update --account-name stextgbvj --container-name policy-demo \
  --name read-only-week --expiry $(date -u '+%Y-%m-%dT%H:%MZ')

curl "$SAS_URL"   # same token as before — now dead, because the policy backing it changed
```

**Self-check questions**

<details><summary>1. You reference <code>asg-web</code> and <code>asg-app</code> together as source and destination in one NSG rule, and it doesn't behave the way you expected. What's the likely cause?</summary>

Two ASGs used together in the same rule must belong to the same virtual network. If they were created against different VNets, the rule won't work the way you'd expect.
</details>

<details><summary>2. Effective security rules for your NIC shows a <code>DenyAllInBound</code> rule at priority 65500 that you never created. Why is it there, and does it matter?</summary>

Every NSG ships with default rules around priority 65000–65500 that apply underneath anything you add. It's not something you created, but it's exactly why "no rule matches" means deny, not allow — effective rules exist to prove that's actually being enforced, not just documented.
</details>

<details><summary>3. You deleted the stored access policy backing an already-issued SAS URL. The SAS's own listed expiry timestamp hasn't passed yet. Does it still work?</summary>

No — once the policy a SAS references is deleted (or its expiry is changed to the past), every SAS tied to that policy dies immediately, regardless of the expiry originally written into the signed URL. That's the entire point of using a policy instead of an ad-hoc SAS.
</details>

<details><summary>4. Why can't you tie a User Delegation SAS to a stored access policy the way you just did with an account-key SAS?</summary>

A stored access policy is a property of the container, checked against tokens signed by an account key. A User Delegation SAS is signed with a user delegation key derived from your Entra ID credentials — a separate trust chain entirely that doesn't reference container-level policies at all.
</details>

**Common mistakes / gotchas**

- Trying to use two ASGs together in a rule when they live in different VNets — it silently doesn't work the way you'd expect, and the error isn't always obvious about why.
- Adding `--as-user` out of habit when generating a policy-backed SAS — that flag switches you to a User Delegation SAS, which ignores `--policy-name` entirely.
- Assuming effective security rules only reflects NIC-level NSG rules — it merges the subnet-level NSG too, which is usually the one people forget about when debugging "why is this blocked."

**Cost check + cleanup**

```bash
# Part 1 — remove the demo VM from rg-networking-lab, leave the network standing
DISK_ID=$(az vm show --resource-group rg-networking-lab --name vm-asg-demo \
  --query "storageProfile.osDisk.managedDisk.id" -o tsv)
az vm delete --resource-group rg-networking-lab --name vm-asg-demo --yes
az disk delete --ids "$DISK_ID" --yes
az network nic delete --resource-group rg-networking-lab --name nic-vm-asg-demo
az network asg delete --resource-group rg-networking-lab --name asg-app-demo
az network nsg rule delete --resource-group rg-networking-lab --nsg-name nsg-app \
  --name Allow-From-ASG-Demo

# Part 2
az group delete --name rg-ext-g --yes --no-wait
```

**What's next:** Back to Module 6 — Load Balancing.

---

<a id="module-6"></a>
## Module 6 — Load Balancing: Load Balancer vs. Application Gateway

**Read this module's cost note before starting.** This is the first genuinely expensive-if-left-running service in the notebook — plan to build, test, and tear down Application Gateway in the same session, per Part 2's ground rules.

I checked current SKU status before writing this, since it changes what's even offered: **Basic Load Balancer was retired on 30 September 2025**, and **Application Gateway v1 was retired on 28 April 2026** — both are already gone as of today. So Standard Load Balancer and Application Gateway v2 aren't just "recommended," they're the only options that exist now.

### 6.0 Learning objectives
- Explain the fundamental difference between L4 (Load Balancer) and L7 (Application Gateway) load balancing
- Deploy a Standard Load Balancer across two backend VMs with a health probe and rule
- Watch a health probe detect a failed backend and remove it from rotation, live
- Deploy an Application Gateway v2 pointed at the same backends, and see how it treats the request differently
- Practice the "create → study → delete same day" discipline this service specifically demands

### 6.1 Concepts before you touch the portal

- **Load Balancer = Layer 4.** It distributes TCP/UDP traffic by IP/port without looking at HTTP content at all. Extremely low latency, cheap, and by default **preserves the original client IP** all the way to the backend.
- **Application Gateway = Layer 7.** It understands HTTP/HTTPS — URLs, headers, cookies — enabling path-based routing, SSL termination, cookie-based session affinity, and an optional Web Application Firewall (WAF). It acts as a genuine reverse proxy: **the backend sees the Gateway's own IP as the source**, with the real client IP passed along in an `X-Forwarded-For` header instead.
- **Health probes** — periodic checks (HTTP/HTTPS/TCP) against each backend. An instance that fails enough consecutive probes is automatically pulled out of rotation, and put back once it passes again. This — not anything more exotic — is the actual mechanism behind "high availability" for a load-balanced tier.
- **Billing shape matters here**: Standard Load Balancer has small per-rule and data-processing charges — cheap to run for a study session. **Application Gateway v2 bills for its minimum instance capacity every hour it exists, whether or not any traffic hits it.** This is exactly the always-on cost trap Part 2 warned about.
- **Application Gateway needs its own dedicated subnet** that can contain *only* Application Gateway resources — you can't drop it into `snet-web` or `snet-app` alongside your VMs.

### 6.2 The lab you're building

```
rg-networking-lab (existing vnet-a)
│
├── snet-web
│   ├── vm-lb-1  (Nginx, no public IP — "Server 1")
│   └── vm-lb-2  (Nginx, no public IP — "Server 2")
│
├── lb-web  (Standard Load Balancer, public frontend) → backend pool: vm-lb-1, vm-lb-2
│
├── snet-appgw  (new, dedicated /27 subnet — App Gateway only)
│   └── appgw-web  (Application Gateway v2, public frontend) → same two backends, by private IP
```

Both the Load Balancer and Application Gateway target the *same two backend VMs*, so you can directly compare how each one handles the same request.

### 6.3 Hands-on Lab — build the backend VMs

```bash
# Two backend VMs in snet-web, no public IPs — traffic will arrive via the LB/Gateway, not directly
az network nic create --resource-group rg-networking-lab --name nic-vm-lb-1 \
  --vnet-name vnet-a --subnet snet-web
az vm create --resource-group rg-networking-lab --name vm-lb-1 \
  --nics nic-vm-lb-1 --image Ubuntu2204 --size Standard_B1s \
  --admin-username azureuser --generate-ssh-keys --os-disk-name disk-vm-lb-1

az network nic create --resource-group rg-networking-lab --name nic-vm-lb-2 \
  --vnet-name vnet-a --subnet snet-web
az vm create --resource-group rg-networking-lab --name vm-lb-2 \
  --nics nic-vm-lb-2 --image Ubuntu2204 --size Standard_B1s \
  --admin-username azureuser --generate-ssh-keys --os-disk-name disk-vm-lb-2
```

**Portal equivalent:** **Virtual machines → + Create → Azure virtual machine**; on the **Networking** tab pick `vnet-a` and subnet `snet-web`, and set **Public IP** to **None**. Repeat for the second VM.

**Don't SSH in for this one — and don't attach a temporary public IP either.** `snet-web`'s NSG (from Module 3) only allows inbound 80/443, not port 22, so a public IP alone wouldn't actually get you SSH access here — you'd need a separate rule just to open it and then remember to close it again. Skip that entirely and use `az vm run-command invoke`, which executes a script on the VM through the Azure control plane (not a network path into the VM at all), and is worth knowing as its own admin technique:

```bash
az vm run-command invoke --resource-group rg-networking-lab --name vm-lb-1 \
  --command-id RunShellScript \
  --scripts "sudo apt-get update && sudo apt-get install -y nginx && echo 'Server 1' | sudo tee /var/www/html/index.nginx-debian.html"

az vm run-command invoke --resource-group rg-networking-lab --name vm-lb-2 \
  --command-id RunShellScript \
  --scripts "sudo apt-get update && sudo apt-get install -y nginx && echo 'Server 2' | sudo tee /var/www/html/index.nginx-debian.html"
```

### 6.4 Standard Load Balancer

```bash
PIP_LB=pip-lb-web

az network public-ip create --resource-group rg-networking-lab --name $PIP_LB \
  --sku Standard --allocation-method Static

az network lb create --resource-group rg-networking-lab --name lb-web \
  --sku Standard --public-ip-address $PIP_LB \
  --frontend-ip-name feip-web --backend-pool-name beap-web

# Add both VMs' NICs into the backend pool
az network nic ip-config address-pool add \
  --resource-group rg-networking-lab --nic-name nic-vm-lb-1 --ip-config-name ipconfig1 \
  --lb-name lb-web --address-pool beap-web
az network nic ip-config address-pool add \
  --resource-group rg-networking-lab --nic-name nic-vm-lb-2 --ip-config-name ipconfig1 \
  --lb-name lb-web --address-pool beap-web

# Health probe + load balancing rule
az network lb probe create --resource-group rg-networking-lab --lb-name lb-web \
  --name probe-http --protocol Http --port 80 --path /

az network lb rule create --resource-group rg-networking-lab --lb-name lb-web \
  --name rule-http --protocol Tcp --frontend-port 80 --backend-port 80 \
  --frontend-ip-name feip-web --backend-pool-name beap-web --probe-name probe-http
```

**Portal equivalent:** **Load balancers → + Create**, SKU **Standard**, pick/create the public IP. On the created LB, **Backend pools**, **Health probes**, and **Load balancing rules** are each their own left-nav blade — add both VMs' NICs to the backend pool, then create the probe and rule with the same settings used above.

**Test the distribution** (get the LB's public IP first with `az network public-ip show --resource-group rg-networking-lab --name pip-lb-web --query ipAddress -o tsv`):

```bash
for i in {1..10}; do curl -s http://<lb-public-ip>; done
```

You should see both "Server 1" and "Server 2" show up across those 10 requests — real traffic distribution, not a diagram. Don't expect a strict back-and-forth pattern, though: Standard Load Balancer distributes by a 5-tuple hash (source IP, source port, destination IP, destination port, protocol) rather than round robin, so the exact sequence can be uneven — Microsoft's own docs note that limited flow variation can skew distribution. Both backends getting used is the thing to verify; a perfect alternation is not guaranteed and isn't what you're testing for.

**Break/fix, live:** same `run-command` approach, no SSH needed:

```bash
az vm run-command invoke --resource-group rg-networking-lab --name vm-lb-1 \
  --command-id RunShellScript --scripts "sudo systemctl stop nginx"
```

Repeat the curl loop — after a short delay (the probe needs a couple of failed checks, not instantly), every response becomes "Server 2" (this part **is** deterministic — with only one healthy backend left, there's nothing left to hash across). Restart it the same way and it rejoins the rotation once it passes probes again:

```bash
az vm run-command invoke --resource-group rg-networking-lab --name vm-lb-1 \
  --command-id RunShellScript --scripts "sudo systemctl start nginx"
```

### 6.5 Application Gateway v2

```bash
# Dedicated subnet — App Gateway only, nothing else can live here
az network vnet subnet create --resource-group rg-networking-lab --vnet-name vnet-a \
  --name snet-appgw --address-prefix 10.0.4.0/27

# Get the two backend VMs' private IPs
az network nic show --resource-group rg-networking-lab --name nic-vm-lb-1 \
  --query "ipConfigurations[0].privateIPAddress" -o tsv
az network nic show --resource-group rg-networking-lab --name nic-vm-lb-2 \
  --query "ipConfigurations[0].privateIPAddress" -o tsv

az network public-ip create --resource-group rg-networking-lab --name pip-appgw-web \
  --sku Standard --allocation-method Static

az network application-gateway create --resource-group rg-networking-lab --name appgw-web \
  --location centralindia --sku Standard_v2 --capacity 1 \
  --vnet-name vnet-a --subnet snet-appgw \
  --public-ip-address pip-appgw-web \
  --servers <vm-lb-1-private-ip> <vm-lb-2-private-ip> \
  --priority 100
```

**Portal equivalent:** **Application Gateways → + Create** — the wizard walks **Frontends → Backends → Configuration** (routing rules) → **Tags → Review + create**, mapping directly onto the `--servers`/listener/rule flags used above; it will also prompt you to create the dedicated subnet if it doesn't already exist.

Test it the same way, against the Gateway's public IP (`az network public-ip show --resource-group rg-networking-lab --name pip-appgw-web --query ipAddress -o tsv`):

```bash
for i in {1..10}; do curl -s http://<appgw-public-ip>; done
```

Same result — both backends get used, without a guaranteed alternating pattern, same caveat as the Load Balancer test above — but the *mechanism* is different: the Gateway terminates the connection and makes its own request to the backend, so from the backend's point of view the request came from the Gateway, not the original client.

### 6.6 Self-check questions

<details>
<summary>Question 1 — What's the fundamental difference in what Load Balancer "sees" vs. what Application Gateway "sees"?</summary>

Load Balancer operates at L4 — it sees IP addresses and ports, and forwards packets without any awareness of HTTP. Application Gateway operates at L7 — it fully terminates and parses the HTTP request, so it can act on URLs, headers, and cookies, and make routing decisions Load Balancer structurally cannot (path-based routing, SSL termination, WAF inspection).
</details>

<details>
<summary>Question 2 — Why didn't the failed backend disappear from rotation instantly?</summary>

Health probes run on an interval and require a configurable number of consecutive failures before marking an instance unhealthy — this avoids removing a backend over one transient blip. The delay you observed is that detection window, not a bug or slowness in the Load Balancer itself.
</details>

<details>
<summary>Question 3 — Why does the backend see the Gateway's IP instead of the real client IP, and how does it recover that information?</summary>

Application Gateway is a true reverse proxy — it terminates the client's connection and opens its own separate connection to the backend, so at the TCP level the backend's peer genuinely is the Gateway. The original client IP is preserved at the HTTP layer instead, via the `X-Forwarded-For` header, which backend applications read explicitly if they need the real client address (for logging, geo-blocking, rate limiting, etc.).
</details>

<details>
<summary>Question 4 — Why is Application Gateway flagged as "don't leave running" while Load Balancer isn't?</summary>

Application Gateway v2 bills for its minimum instance capacity every hour regardless of traffic — an idle Gateway costs the same as a busy one. Standard Load Balancer's charges are much smaller and scale more with actual usage (rules and data processed), so leaving one up briefly during a study session doesn't carry the same fixed hourly cost.
</details>

<details>
<summary>Question 5 — Why must Application Gateway live in its own dedicated subnet with nothing else in it?</summary>

Application Gateway deploys its own scaled-out instances directly into that subnet's address space as part of how it operates — Azure enforces that the subnet contains only Application Gateway resources so those internal instances can't collide or interfere with other resources' networking. This is a hard platform requirement, not a best-practice suggestion.
</details>

### 6.7 Common mistakes / gotchas

- **Trying to put Application Gateway in `snet-web` or `snet-app`** — it will fail; it needs its own empty, dedicated subnet.
- **Leaving Application Gateway up "to poke at more later"** — this is the exact expensive-idle-time scenario Part 2 exists to prevent. Delete it the same day.
- **Expecting Load Balancer to do path-based routing or SSL termination** — those are Application Gateway-only capabilities; Load Balancer has no concept of HTTP at all.
- **Expecting instant failover** — the probe interval/threshold delay is normal and by design, not a fault.
- **Deploying anything on the Basic SKU** — it no longer exists for either service; if a script or old tutorial references `Basic`, it's out of date.

### 6.8 Cost check + cleanup

Delete the Application Gateway first — it's the highest fixed cost in this module — then the Load Balancer and its backend VMs:

```bash
az network application-gateway delete --resource-group rg-networking-lab --name appgw-web
az network public-ip delete --resource-group rg-networking-lab --name pip-appgw-web

az network lb delete --resource-group rg-networking-lab --name lb-web
az network public-ip delete --resource-group rg-networking-lab --name pip-lb-web

az vm delete --resource-group rg-networking-lab --name vm-lb-1 --yes
az vm delete --resource-group rg-networking-lab --name vm-lb-2 --yes
az disk delete --resource-group rg-networking-lab --name disk-vm-lb-1 --yes
az disk delete --resource-group rg-networking-lab --name disk-vm-lb-2 --yes
az network nic delete --resource-group rg-networking-lab --name nic-vm-lb-1
az network nic delete --resource-group rg-networking-lab --name nic-vm-lb-2

# The empty subnet itself is free to leave, but tidy it up if you'd rather not:
az network vnet subnet delete --resource-group rg-networking-lab --vnet-name vnet-a --name snet-appgw
```

### 6.9 What's next

**Module 7 — App Hosting & Containers** moves up a level of abstraction: instead of managing VMs yourself, App Service and Azure Functions run your code on Azure-managed infrastructure. This module also deliberately bridges into **AI-200** territory — Container Apps and Azure Container Registry are core AI-200 skills, so you'll get first exposure here rather than starting that exam completely cold.

Run through Module 6, and say the word for Module 7 — or ask now if the health-probe failover or the Load Balancer/Gateway comparison didn't match what you expected.

---

<a id="module-7"></a>
## Module 7 — App Hosting & Containers: App Service, Functions, ACR + Container Apps (your first AI-200 bridge lab)

**A housekeeping note first.** Back in Module 2 you created a managed identity in `rg-identity-lab`, and in Module 5 a storage account in `rg-storage-lab` — and cleanup instructed you to delete both resource groups, which was the right call at the time. This module creates its **own** identity and storage account from scratch in a fresh `rg-apps-lab`, rather than assuming those earlier ones still exist. That's not a mistake in the earlier modules — each module is deliberately self-contained after cleanup. It's also a real preview of *why* Infrastructure-as-Code (Bicep/Terraform) exists: rebuilding "the same thing" by hand every time is exactly the tedium that tooling beyond this notebook is built to remove — worth keeping in mind as a next step after AZ-104.

### 7.0 Learning objectives
- Deploy a web app to **App Service** (PaaS — no VM, no OS patching, no SSH)
- Actually complete the **Managed Identity → RBAC → Storage** pattern promised in Module 2: write code that authenticates with zero secrets and prove it works
- Stand up a minimal **Azure Function** on a Consumption plan and understand its distinct billing model
- Build a container image with **ACR Tasks** (no local Docker needed), push it to **Azure Container Registry**, and run it on **Azure Container Apps** — scale-to-zero serverless containers, your on-ramp into AI-200's core surface area

### 7.1 Concepts before you touch the portal

- **App Service** — PaaS web hosting. You deploy code; Azure manages the OS, runtime, and patching. Has an **F1 (Free) tier** — genuinely ₹0, doesn't touch your credit — with quotas (60 CPU-minutes/day, no custom domains or scaling) that are perfectly fine for a lab.
- **Managed identity, completed** — In Module 2 you created a user-assigned identity but had nothing to attach it to yet. Here, you'll attach it to an App Service, grant it a data-plane role on a storage account, and write actual code that uses `DefaultAzureCredential` — no connection string, no key, anywhere.
- **`AZURE_CLIENT_ID`** — when a resource has a *user-assigned* identity (as opposed to system-assigned), `DefaultAzureCredential` needs to know *which* identity to use if more than one is possible. Setting the `AZURE_CLIENT_ID` app setting to the identity's client ID is how you tell it.
- **Azure Functions (Consumption plan)** — serverless: you pay per execution + GB-seconds of memory, with a generous always-free monthly grant. A Function App also needs its **own** storage account for internal bookkeeping (triggers, locks, logs) — separate from any storage account your function's *logic* might read or write.
- **ACR (Azure Container Registry)** — your own private registry for container images. **ACR Tasks** (`az acr build`) build the image *in the cloud*, so you don't need Docker installed locally.
- **Container Apps** — serverless containers: you point it at an image, and it runs, scales (including **scale-to-zero** — literally no compute cost while idle), and handles ingress for you. This is the philosophical opposite of Module 6's Application Gateway, which bills every hour whether or not it's used.
- **AI-200 relevance** — Container Registry, Container Apps, and (at a deeper level than this notebook covers) AKS are core AI-200 skills. This module is your first real exposure, not a full AI-200 lab.

### 7.2 Hands-on Lab A — App Service, with the identity chain actually working

```bash
az group create --name rg-apps-lab --location centralindia \
  --tags project=azure-learning module=07-app-hosting-containers

# Fresh identity (Module 2's original is gone — see the note above)
az identity create --name id-app-lab --resource-group rg-apps-lab
CLIENT_ID=$(az identity show --name id-app-lab --resource-group rg-apps-lab --query clientId -o tsv)
PRINCIPAL_ID=$(az identity show --name id-app-lab --resource-group rg-apps-lab --query principalId -o tsv)
IDENTITY_ID=$(az identity show --name id-app-lab --resource-group rg-apps-lab --query id -o tsv)

# Fresh storage account for this module (swap stappslabbvj for your own unique name)
az storage account create --name stappslabbvj --resource-group rg-apps-lab \
  --location centralindia --sku Standard_LRS --kind StorageV2
STG_ID=$(az storage account show --name stappslabbvj --resource-group rg-apps-lab --query id -o tsv)

# Grant the identity read access to blob data
az role assignment create --assignee-object-id "$PRINCIPAL_ID" --assignee-principal-type ServicePrincipal \
  --role "Storage Blob Data Reader" --scope "$STG_ID"

# Grant yourself write access so you can create a container to list
az role assignment create --assignee "<your-upn-or-object-id>" \
  --role "Storage Blob Data Contributor" --scope "$STG_ID"
az storage container create --account-name stappslabbvj --name sample-container --auth-mode login
```

Create the app locally:

```bash
mkdir module7-app && cd module7-app
```

`app.py`:

```python
import os
from flask import Flask
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

app = Flask(__name__)
STORAGE_ACCOUNT = os.environ.get("STORAGE_ACCOUNT_NAME", "")

@app.route("/")
def hello():
    return "Hello from App Service — Module 7 lab"

@app.route("/blobs")
def list_blobs():
    credential = DefaultAzureCredential()
    account_url = f"https://{STORAGE_ACCOUNT}.blob.core.windows.net"
    client = BlobServiceClient(account_url=account_url, credential=credential)
    containers = [c.name for c in client.list_containers()]
    return {"storage_account": STORAGE_ACCOUNT, "containers": containers}

if __name__ == "__main__":
    app.run()
```

`requirements.txt`:

```
flask
gunicorn
azure-identity
azure-storage-blob
```

Deploy it:

```bash
az appservice plan create --name plan-web-lab --resource-group rg-apps-lab --sku F1 --is-linux

az webapp create --resource-group rg-apps-lab --plan plan-web-lab \
  --name app-web-lab-bvj --runtime "PYTHON:3.12"

az webapp identity assign --resource-group rg-apps-lab --name app-web-lab-bvj --identities "$IDENTITY_ID"

az webapp config appsettings set --resource-group rg-apps-lab --name app-web-lab-bvj \
  --settings STORAGE_ACCOUNT_NAME=stappslabbvj AZURE_CLIENT_ID="$CLIENT_ID"

zip -r app.zip app.py requirements.txt
az webapp deploy --resource-group rg-apps-lab --name app-web-lab-bvj --src-path app.zip --type zip
```

Test it:

```bash
HOST=$(az webapp show --resource-group rg-apps-lab --name app-web-lab-bvj --query defaultHostName -o tsv)
curl "https://$HOST/"
curl "https://$HOST/blobs"
```

The second call should return `{"storage_account": "stappslabbvj", "containers": ["sample-container"]}` — running code, in the cloud, that authenticated to storage with **zero secrets anywhere** in your code, config, or deployment. This is the payoff for Module 2's identity and Module 5's RBAC lesson, actually working end to end.

**Portal equivalent:** create the identity and storage account as in Modules 2 and 5. Then **App Services → + Create → Web App** (Runtime stack `Python 3.12`, plan `F1 Free`). On the created app: **Identity → User assigned → + Add** to attach `id-app-lab`; **Configuration → + New application setting** to add `STORAGE_ACCOUNT_NAME` and `AZURE_CLIENT_ID`; **Deployment Center** (or **Advanced Tools/Kudu → Zip Push Deploy**) to upload `app.zip`. Zip deployment specifically is genuinely easier via CLI than Portal — this is one place where reaching for the CLI isn't just habit, it's the faster tool for the job.

### 7.3 Hands-on Lab B — Azure Functions (brief — the point is the billing model, not depth)

```bash
az storage account create --name stfuncslabbvj --resource-group rg-apps-lab \
  --location centralindia --sku Standard_LRS

az functionapp create --resource-group rg-apps-lab --consumption-plan-location centralindia \
  --runtime python --runtime-version 3.11 --functions-version 4 \
  --name func-lab-bvj --storage-account stfuncslabbvj --os-type Linux
```

Notice `stfuncslabbvj` is a *second* storage account, purely for the Function App's internal bookkeeping — not the same as `stappslabbvj` above, and not something your function's own logic would necessarily touch.

**Portal equivalent:** **Function App → + Create**, Hosting **Consumption**, Runtime `Python`, and the wizard prompts you to create (or pick) the storage account as part of the same flow — you'll notice the Portal treats it as a required field, not an optional extra, reinforcing that it's platform infrastructure rather than your data.

### 7.4 Hands-on Lab C — ACR + Container Apps (the real AI-200 bridge)

```bash
# Private registry, admin login explicitly disabled — RBAC pull only, no shared keys
az acr create --resource-group rg-apps-lab --name acrlabbvj --sku Basic --admin-enabled false
```

`Dockerfile` (same app as Lab A, containerized):

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py requirements.txt ./
RUN pip install -r requirements.txt
EXPOSE 8080
CMD ["gunicorn", "-b", "0.0.0.0:8080", "app:app"]
```

Build it **in Azure** — no local Docker required:

```bash
az acr build --registry acrlabbvj --image module7-app:v1 .
```

**Portal equivalent:** **Container registries → + Create**, SKU `Basic`, and on the **Access keys** tab confirm **Admin user: Disabled**. Building the image itself, though, is one place the CLI is genuinely the better tool, not just the faster one — `az acr build` streams your local folder straight into a cloud build with one command, while the Portal's equivalent (**Tasks → Quick task**, or a full ACR Task wired to a git repo) needs your Dockerfile already sitting in a reachable source location. Use the CLI here.

Let the same identity pull from this registry:

```bash
ACR_ID=$(az acr show --name acrlabbvj --resource-group rg-apps-lab --query id -o tsv)
az role assignment create --assignee-object-id "$PRINCIPAL_ID" --assignee-principal-type ServicePrincipal \
  --role "AcrPull" --scope "$ACR_ID"
```

Set up Container Apps (one-time extension + provider registration if this is your first use):

```bash
az extension add --name containerapp --upgrade
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights

az containerapp env create --name env-apps-lab --resource-group rg-apps-lab --location centralindia

az containerapp create \
  --name ca-web-lab --resource-group rg-apps-lab --environment env-apps-lab \
  --image acrlabbvj.azurecr.io/module7-app:v1 \
  --target-port 8080 --ingress external \
  --registry-server acrlabbvj.azurecr.io --registry-identity "$IDENTITY_ID" \
  --user-assigned "$IDENTITY_ID" \
  --min-replicas 0 --max-replicas 2
```

**Portal equivalent:** **Container Apps → + Create** — the wizard's **Container** tab takes the image/registry directly and a **Scale** tab has explicit min/max replica fields matching `--min-replicas`/`--max-replicas` above; it also offers to create the Container Apps Environment for you inline if one doesn't exist yet, same as `az containerapp env create` above.

Test:

```bash
CA_URL=$(az containerapp show --name ca-web-lab --resource-group rg-apps-lab \
  --query properties.configuration.ingress.fqdn -o tsv)
curl "https://$CA_URL/"
```

`--min-replicas 0` means this container costs **nothing while idle** — the opposite billing philosophy from Module 6's Application Gateway. The trade-off is a "cold start": the first request after idle time takes a little longer while an instance spins up.

### 7.5 Self-check questions

<details>
<summary>Question 1 — What's the core operational difference between App Service (F1)/Container Apps (scale-to-zero) here, versus the VMs and Load Balancer/App Gateway from Modules 4 and 6?</summary>

Modules 4 and 6 are infrastructure you provision and pay for by the hour regardless of load (VMs, a Standard Load Balancer, and especially Application Gateway's fixed minimum capacity). App Service F1 and Container Apps at `min-replicas 0` shift cost toward actual usage — free or near-free while idle. It's IaaS-style "pay for the box" vs. PaaS/serverless "pay for what runs."
</details>

<details>
<summary>Question 2 — No connection string, key, or password appears anywhere in app.py or the app settings you set — how does `/blobs` actually authenticate?</summary>

`DefaultAzureCredential` detects it's running on an Azure resource with a managed identity attached, and (guided by the `AZURE_CLIENT_ID` app setting, since this is a user-assigned identity) requests a token for that identity from Azure's instance metadata service. That token proves the app's identity to Azure Storage, which checks it against the `Storage Blob Data Reader` role assignment you created — no secret was ever generated, stored, or transmitted by you at any point.
</details>

<details>
<summary>Question 3 — Why set `admin-enabled false` on the container registry, and how does the Container App pull images without it?</summary>

Admin credentials on ACR are a shared username/password for the whole registry — the same key-management problems as storage account keys. With admin disabled, the Container App instead authenticates as the managed identity you attached (`--registry-identity`), which was separately granted the `AcrPull` role — the same RBAC-over-shared-secret pattern from the rest of this notebook, applied to container images.
</details>

<details>
<summary>Question 4 — Why does a Function App need its own separate storage account, distinct from any storage your function's code might read or write?</summary>

The Functions runtime itself uses that storage account internally — for trigger state, execution logs, and coordination between instances. It's infrastructure for the *platform*, not data for your *application*; conflating the two is a common beginner mix-up, and reusing an application storage account for this purpose is technically possible but not the clean separation Azure expects.
</details>

<details>
<summary>Question 5 — What does `--min-replicas 0` do to cost when idle, and what's the trade-off?</summary>

At zero replicas, there is no running compute instance and therefore no compute cost while no requests are arriving. The trade-off is a **cold start**: the first request after an idle period has to wait for a new instance to spin up before it's served, which is slower than hitting an already-warm instance — a latency-for-cost trade you'd weigh deliberately in a real system, not something to accept blindly everywhere.
</details>

### 7.6 Common mistakes / gotchas

- **Skipping the Container Apps extension/provider registration** on first use — a fresh subscription needs `az extension add --name containerapp` and the `Microsoft.App` / `Microsoft.OperationalInsights` providers registered once before anything else works.
- **Enabling ACR admin credentials "just to test faster"** — undermines the entire RBAC-pull pattern this lab is built to teach.
- **Forgetting `AZURE_CLIENT_ID`** when a resource has a user-assigned (not system-assigned) identity — without it, `DefaultAzureCredential` can't reliably pick the right identity if the environment could plausibly offer more than one credential source.
- **Assuming F1 App Service can do everything Standard can** — no custom domains, no autoscaling, a hard daily CPU-minute cap. Fine for a lab, wrong for anything real.
- **Treating Container Apps and AKS as interchangeable** — Container Apps is the simpler, more managed, naturally scale-to-zero option; AKS is full Kubernetes with far more control and far more of your own responsibility. AI-200 covers both, and picking between them is itself a tested design decision, not a coin flip.

### 7.7 Cost check + cleanup

App Service F1 is free. Container Apps at `min-replicas 0` with light testing is near-zero (a small Log Analytics workspace underlies the Container Apps environment). ACR Basic and Functions Consumption are both a few rupees at most for a session like this.

```bash
az group delete --name rg-apps-lab --yes
```

### 7.8 What's next

**Module 8 — Security** goes deeper on the identity pattern you just got working: Azure Key Vault for secrets/certificates, and the same `Managed Identity → RBAC → Key Vault` chain you just proved out against storage — this time protecting something more sensitive than a blob listing.

Run through Module 7, and say the word for Module 8 — or ask now if the identity chain, ACR build, or Container App didn't come together the way you expected.

---

<a id="ext-e"></a>
## Extension Lab E — App Service Slots, Scaling & Azure Container Instances

*Sits between Modules 7 and 8 — a compact follow-on to Module 7's App Service lab, before Module 8 moves on to Key Vault.*

**Objectives**
- Deploy to a staging slot and swap into production with zero downtime — and know exactly which App Service tier makes this possible
- Distinguish scale up (bigger plan) from scale out (more instances)
- Place Azure Container Instances correctly on the App Service → Container Apps → AKS complexity ladder

**Concepts**

Deployment slots require **Standard tier or higher** — not Basic, and not the F1 Free tier Module 7 used. A slot is a fully separate, live App Service instance sharing the same plan, with its own hostname, that you can deploy to and warm up *before* it takes production traffic. **Swap** exchanges the two slots' content and most configuration at the routing level — it's not a redeploy, so it's near-instant and reversible.

Not every app setting swaps. A setting marked **"Deployment slot setting"** ("sticky") stays attached to the slot itself rather than following the code during a swap — this is exactly how you'd keep a staging slot pointed at a staging database connection string even after swapping the application code into production. Missing this distinction is a real-world outage cause: someone swaps expecting a setting to follow the code, and it doesn't.

Scale **up** = change the App Service Plan's SKU/tier (S1 → S2) — a bigger single machine, vertical scaling. Scale **out** = increase the instance count on the same plan — more machines behind the plan's built-in load balancing, horizontal scaling. Autoscale rules for App Service (like Extension Lab C's, but for web app instances instead of VMs) require Standard tier or higher.

**Azure Container Instances (ACI)** sit at the simple end of the container ladder: a single container group, no orchestration, no scale-to-zero, billed per-second while running — good for a one-off job or burst task. Container Apps (Module 7) adds scale-to-zero, revisions, and Dapr/KEDA-based scaling on top of that. AKS gives full Kubernetes control when needed, at the cost of managing far more yourself. Picking ACI for a long-running web service is usually the wrong call — it keeps billing every second it's up, with none of Container Apps' scale-to-zero economics.

**Hands-on — Portal**

1. If Module 7's `rg-apps-lab`/`app-web-lab-bvj` still exists, reuse it — otherwise this lab stands alone with fresh resources below, since (as Module 7 noted) each module's cleanup usually means earlier resources aren't still around.
2. **App Service plan → Scale up (App Service plan)** → select **Standard S1** → Apply. (F1/Basic tiers won't even show the option to add a slot — this step is required first.)
3. Web App → **Deployment slots → + Add Slot** → name `staging`.
4. Deploy a changed version of the app to the staging slot specifically (Deployment Center **on the slot**, not the main app).
5. Browse the staging slot's own URL, shown on the Deployment slots blade (slot hostname formats have changed across Azure's history — read it off the blade rather than guessing it) → confirm it shows the new version while production still shows the old one.
6. **Deployment slots → Swap** → source `staging`, target `production` → Swap. Refresh both URLs: production now shows the new version, staging now shows the old one — a true swap, not a one-way copy.
7. **App Service plan → Scale out (App Service plan)** → switch to Custom autoscale, or drag the instance count slider for a manual scale-out test.
8. **Container Instances → + Create** → `aci-ext-e`, image `mcr.microsoft.com/azuredocs/aci-helloworld`, public IP, port 80. Once running, hit its public IP in a browser.

**Hands-on — CLI**

```bash
az group create --name rg-ext-e --location centralindia \
  --tags project=azure-learning module=ext-e-slots-scaling

az appservice plan create --name plan-ext-e --resource-group rg-ext-e \
  --sku S1 --is-linux

az webapp create --resource-group rg-ext-e --plan plan-ext-e \
  --name app-ext-e-bvj --runtime "PYTHON:3.12"

echo 'from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "v1 — production"' > app.py
echo -e "flask\ngunicorn" > requirements.txt
zip -r app-v1.zip app.py requirements.txt
az webapp deploy --resource-group rg-ext-e --name app-ext-e-bvj \
  --src-path app-v1.zip --type zip

# Create the staging slot, then deploy v2 to the SLOT specifically
az webapp deployment slot create --resource-group rg-ext-e \
  --name app-ext-e-bvj --slot staging

sed -i 's/v1 — production/v2 — staged/' app.py
zip -r app-v2.zip app.py requirements.txt
az webapp deploy --resource-group rg-ext-e --name app-ext-e-bvj \
  --slot staging --src-path app-v2.zip --type zip

# Confirm they differ before swapping
az webapp deployment slot list --resource-group rg-ext-e --name app-ext-e-bvj \
  --query "[].{name:name, hostname:defaultHostName}" -o table

az webapp deployment slot swap --resource-group rg-ext-e \
  --name app-ext-e-bvj --slot staging --target-slot production

# Scale up (vertical) and scale out (horizontal)
az appservice plan update --resource-group rg-ext-e --name plan-ext-e --sku S2
az appservice plan update --resource-group rg-ext-e --name plan-ext-e --number-of-workers 2

# ACI — a single container, no orchestration, billed per second
az container create --resource-group rg-ext-e --name aci-ext-e \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 1 --memory 1 --ports 80 --ip-address Public

az container show --resource-group rg-ext-e --name aci-ext-e \
  --query ipAddress.ip -o tsv
```

**Self-check questions**

<details><summary>1. Why wouldn't Module 7's F1 Free plan have let you create a deployment slot?</summary>

Slots require Standard tier or higher; F1 (and Basic) don't support them at all — exactly why this lab starts with an explicit scale-up step.
</details>

<details><summary>2. You swap staging into production expecting the staging database connection string to follow the code. It doesn't — production is now pointed at the old connection string. What did you miss?</summary>

That connection string was marked as a "deployment slot setting," which means it's sticky to the slot, not the code — it stays put during a swap by design.
</details>

<details><summary>3. Is scaling out (more instances) the same lever as scaling up (bigger plan)?</summary>

No — scale up changes the SKU/hardware of a single instance (vertical); scale out adds more instances of the same size behind the plan (horizontal). They solve different bottlenecks and can be combined.
</details>

<details><summary>4. You need a container that runs for 30 seconds once a day to process a file, then stops. Container Apps, ACI, or AKS?</summary>

ACI — billed per-second, no ongoing orchestration needed. Container Apps' scale-to-zero is built for HTTP-triggered workloads with more variable traffic, and AKS would be significant overhead for a single scheduled task.
</details>

**Common mistakes / gotchas**

- Trying to add a slot on a Basic or Free plan and not understanding why the option is missing or grayed out — check the tier first.
- Assuming *all* app settings swap with the code — always check which ones are marked slot-specific before relying on swap behavior.
- Using ACI for an always-on service and being surprised it costs more over a month than Container Apps for the same traffic, because ACI never scales to zero.
- Forgetting Standard tier itself is not free — leaving `plan-ext-e` running is one of the more expensive line items in this whole notebook if left up for days.

**Cost check + cleanup**

Standard S1 is a genuinely non-trivial cost if left running — nowhere near the F1/free-tier costs elsewhere in this notebook. Don't leave this resource group up longer than the lab takes.

```bash
az container delete --resource-group rg-ext-e --name aci-ext-e --yes
az group delete --name rg-ext-e --yes --no-wait
```

**What's next:** Back to Module 8 — Security.

---

<a id="module-8"></a>
## Module 8 — Security: Key Vault, Deepening the Managed Identity Pattern

Same rebuild note as Module 7: `rg-apps-lab` is gone per its own cleanup, so this module stands up its own fresh resource group. By now the pattern should feel familiar rather than repetitive.

### 8.0 Learning objectives
- Create a Key Vault using the **RBAC permission model** (not the legacy Access Policies model)
- Correctly distinguish Key Vault's three object types: Secrets, Keys, Certificates
- Grant a managed identity read-only access to a secret, and grant yourself management access — different roles, deliberately
- Consume a secret **two ways**: App Service's zero-code Key Vault reference, and an explicit SDK call
- Understand soft-delete and purge protection, and why they matter beyond this lab

### 8.1 Concepts before you touch the portal

- **Three distinct object types, don't conflate them:**
  - **Secrets** — arbitrary string data: API keys, passwords, connection strings.
  - **Keys** — cryptographic key material for encrypt/decrypt/sign/verify operations, often HSM-backed. Critically, the key material itself typically never leaves the vault — you ask the vault to *perform* the operation, you don't fetch the raw key.
  - **Certificates** — a managed pairing of a key + X.509 certificate metadata, with built-in renewal support.
- **Access model: RBAC vs. Access Policies.** Key Vault supports two permission systems. The **Azure RBAC model** (roles like `Key Vault Secrets User`, `Key Vault Secrets Officer`, `Key Vault Administrator`, scoped the same way as everything else in this notebook) is what Microsoft now steers new vaults toward. The legacy **Access Policies** model is a vault-specific, coarser system you'll still see in older environments and tutorials — recognize it, but default to RBAC for anything new.
- **Soft-delete and purge protection** — soft-delete (mandatory now, always on) keeps a deleted vault or secret recoverable for a retention window (default 90 days) instead of destroying it instantly, protecting against accidental or malicious deletion. **Purge protection** (opt-in at creation) goes further: during that window, *nobody* — not even an Owner — can force a permanent purge early. This closes the gap where someone deletes something and immediately purges it to make recovery impossible.
- **The same pattern, a third time**: `App (Managed Identity) → RBAC (Key Vault Secrets User) → Key Vault Secret`. Key Vault isn't conceptually special — it's another RBAC-protected data store, purpose-built for secrets, following the exact shape you already built against Storage in Module 7.
- **Key Vault references in App Service** — a built-in mechanism (`@Microsoft.KeyVault(SecretUri=...)` app setting syntax) where App Service itself resolves a secret into an app setting at startup, using the app's identity. Your code just reads what looks like a normal environment variable — it never calls the Key Vault SDK at all.
- **`AZURE_CLIENT_ID` vs. `keyVaultReferenceIdentity` — these are two different settings for two different consumers, don't conflate them.** `AZURE_CLIENT_ID` is read by your *application code*, when the `azure-identity` SDK's `DefaultAzureCredential` needs to disambiguate which user-assigned identity to use (that's what Module 7's app used). It has **no effect** on App Service's own platform-level resolution of `@Microsoft.KeyVault(...)` references. For that, the App Service **platform itself** needs to be told which identity to use via a separate site property, `keyVaultReferenceIdentity`, set to the identity's full **resource ID** (not its client ID). By default App Service uses its system-assigned identity for reference resolution — with only a user-assigned identity attached, resolution fails silently until `keyVaultReferenceIdentity` is set explicitly.

### 8.2 Hands-on Lab

**Portal walkthrough:**

1. **Key vaults → + Create.** Resource group `rg-security-lab`, name `kv-learning-bvj`, region Central India, Pricing tier `Standard`. On the **Access configuration** tab, set **Permission model** to **Azure role-based access control** — this is the exact equivalent of `--enable-rbac-authorization true`, and it's easy to leave on the older default here if you don't look for it. Review + Create.
2. Open it → **Access control (IAM) → Add role assignment → Key Vault Secrets Officer** → assign to yourself.
3. **Objects → Secrets → + Generate/Import** → name `DemoApiKey`, value = anything.
4. Create the identity as in earlier modules (**Managed Identities → + Create → User assigned**), then on the vault's **Access control (IAM)** again → **Add role assignment → Key Vault Secrets User** → assign to that identity.
5. Create the App Service (**App Services → + Create**, plan `F1 Free`) → **Identity → User assigned → + Add** to attach the identity → **Configuration → + New application setting** → name `DemoApiKey`, value = the `@Microsoft.KeyVault(SecretUri=...)` reference string, plus `AZURE_CLIENT_ID`. **This alone will not resolve** — Microsoft's own docs for this scenario only document setting `keyVaultReferenceIdentity` via CLI/PowerShell (`az webapp update --set keyVaultReferenceIdentity=<identity-resource-id>`), not a simple Portal toggle, so run that command even if you did everything else through the Portal. The Configuration grid's **Source** column reads "Key Vault Reference" with a green check once it actually resolves.

**CLI walkthrough:**

```bash
az group create --name rg-security-lab --location centralindia \
  --tags project=azure-learning module=08-security

# Key Vault names are globally unique too — swap kv-learning-bvj for your own
az keyvault create --name kv-learning-bvj --resource-group rg-security-lab \
  --location centralindia --enable-rbac-authorization true

KV_ID=$(az keyvault show --name kv-learning-bvj --query id -o tsv)
VAULT_URI=$(az keyvault show --name kv-learning-bvj --query properties.vaultUri -o tsv)

# Grant yourself management access (create/update/delete secrets)
az role assignment create --assignee "<your-upn-or-object-id>" \
  --role "Key Vault Secrets Officer" --scope "$KV_ID"

# Wait a minute for propagation, then create a secret
az keyvault secret set --vault-name kv-learning-bvj --name DemoApiKey \
  --value "super-secret-value-never-checked-into-code"

# Fresh identity for this module
az identity create --name id-security-lab --resource-group rg-security-lab
CLIENT_ID=$(az identity show --name id-security-lab --resource-group rg-security-lab --query clientId -o tsv)
PRINCIPAL_ID=$(az identity show --name id-security-lab --resource-group rg-security-lab --query principalId -o tsv)
IDENTITY_ID=$(az identity show --name id-security-lab --resource-group rg-security-lab --query id -o tsv)

# Grant the identity READ-ONLY access — a different, narrower role than yours
az role assignment create --assignee-object-id "$PRINCIPAL_ID" --assignee-principal-type ServicePrincipal \
  --role "Key Vault Secrets User" --scope "$KV_ID"
```

**Way 1 — zero-code Key Vault reference in App Service:**

```bash
az appservice plan create --name plan-security-lab --resource-group rg-security-lab --sku F1 --is-linux

az webapp create --resource-group rg-security-lab --plan plan-security-lab \
  --name app-security-lab-bvj --runtime "PYTHON:3.12"

az webapp identity assign --resource-group rg-security-lab --name app-security-lab-bvj --identities "$IDENTITY_ID"

az webapp config appsettings set --resource-group rg-security-lab --name app-security-lab-bvj \
  --settings DemoApiKey="@Microsoft.KeyVault(SecretUri=${VAULT_URI}secrets/DemoApiKey/)" AZURE_CLIENT_ID="$CLIENT_ID"

# THE STEP THAT'S EASY TO MISS: tell the App Service PLATFORM which identity to use
# for resolving Key Vault references — AZURE_CLIENT_ID above only helps YOUR CODE's
# SDK calls, it does nothing for the platform's own @Microsoft.KeyVault(...) resolution.
az webapp update --resource-group rg-security-lab --name app-security-lab-bvj \
  --set keyVaultReferenceIdentity="$IDENTITY_ID"
```

Check **Portal → your App Service → Configuration → Application settings** — the `DemoApiKey` row should show a green "Resolved" status. Any code in this app can now read `os.environ["DemoApiKey"]` like an ordinary environment variable, with no Key Vault SDK call anywhere in the codebase. Skip the `keyVaultReferenceIdentity` step and you'll follow everything else correctly and still watch this sit unresolved — App Service defaults reference resolution to the **system-assigned** identity, which this app doesn't have.

**Way 2 — explicit SDK call** (for cases without App Service's built-in resolution, e.g. a script or a container):

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://kv-learning-bvj.vault.azure.net/", credential=credential)
secret = client.get_secret("DemoApiKey")
print(secret.value)
```

Running this from Cloud Shell tests it as **you** (via your `Key Vault Secrets Officer` role), not as the app's identity — worth noticing that "it works from Cloud Shell" and "it works from the app" are testing two different identities.

**Optional — prove RBAC enforcement is real, not decorative:** remove your own role assignment temporarily (`az role assignment delete --assignee "<your-upn>" --role "Key Vault Secrets Officer" --scope "$KV_ID"`), then try `az keyvault secret list --vault-name kv-learning-bvj` — it should fail with a `Forbidden`/`AccessDenied` error. Re-add the role assignment afterward.

### 8.3 Self-check questions

<details>
<summary>Question 1 — Secret vs. Key vs. Certificate — what's the actual distinction?</summary>

A Secret is opaque string data you fetch and use directly (API keys, passwords). A Key is cryptographic material used to *perform* encrypt/decrypt/sign/verify operations — for HSM-backed keys, the raw material typically never leaves the vault; you send the vault a request to operate on your data, not a request for the key itself. A Certificate is a managed pairing of a key with X.509 metadata, plus renewal handling. Same vault, same permission model, three different purposes.
</details>

<details>
<summary>Question 2 — Why prefer the RBAC permission model over legacy Access Policies for a new vault?</summary>

RBAC uses the exact same role/scope system as every other Azure resource — consistent auditing, consistent tooling, and it composes properly with management groups and the rest of your access model. Access Policies are a vault-specific, coarser all-or-nothing-per-permission-category system that doesn't integrate as cleanly, and it's the legacy path Microsoft's own guidance now steers away from for new vaults.
</details>

<details>
<summary>Question 3 — What does soft-delete protect against, and what does purge protection add?</summary>

Soft-delete keeps a deleted vault or secret recoverable for a retention window (default 90 days) instead of destroying it the instant delete is called — protection against an accidental or malicious delete. Purge protection adds that *nobody*, including an Owner, can force a permanent purge during that window — closing the gap where a delete is immediately followed by a purge specifically to prevent recovery.
</details>

<details>
<summary>Question 4 — In the App Service Key Vault reference pattern, does your application code ever talk to Key Vault directly?</summary>

No. The App Service platform itself resolves the `@Microsoft.KeyVault(...)` reference at startup, using whichever identity `keyVaultReferenceIdentity` names (system-assigned by default, or the user-assigned identity you explicitly set it to), and injects the plain resolved value as what looks like a normal app setting. Your application code has no Key Vault SDK calls, no vault URI, and no awareness that Key Vault is involved at all — it just reads an environment variable.
</details>

<details>
<summary>Question 5 — Why does "Key Vault Secrets Officer" make sense for you but "Key Vault Secrets User" for the app's identity?</summary>

`Secrets Officer` can create, update, and delete secrets — appropriate for the person actually managing the vault's contents. `Secrets User` can only read secret values — appropriate for an application that should consume a secret but never be able to modify or delete it. Same least-privilege principle from Module 2's RBAC lesson, applied specifically to secrets.
</details>

### 8.4 Common mistakes / gotchas

- **Using Access Policies on a new vault "because a tutorial showed it"** — creates a permission model inconsistent with the RBAC pattern used everywhere else in this notebook.
- **Forgetting `--enable-rbac-authorization true` at creation** — some paths still default to Access Policy mode depending on subscription age/CLI version, so specify it explicitly rather than assuming.
- **Granting `Key Vault Secrets Officer` (or `Key Vault Administrator`) to an application identity** that only ever needs to read — a least-privilege violation with no upside.
- **Setting `AZURE_CLIENT_ID` and assuming that's what makes the Key Vault reference resolve** — it isn't. `AZURE_CLIENT_ID` only affects your own code's SDK calls; App Service's platform-level reference resolution needs the separate `keyVaultReferenceIdentity` site property set explicitly, or it silently falls back to a system-assigned identity this app doesn't have.
- **Assuming purge protection is on by default** — soft-delete is mandatory now, but purge protection is opt-in at creation, and once enabled it generally can't be turned back off. It's a deliberate choice either way, not something to leave to chance.

### 8.5 Cost check + cleanup

Key Vault's per-operation cost at this scale is negligible; App Service F1 is free.

```bash
az group delete --name rg-security-lab --yes
```

**One thing to know:** because of soft-delete, the vault name `kv-learning-bvj` will linger in a "soft-deleted" state for up to 90 days after the resource group is gone — if you try to create another vault with that *exact* name before then, it'll conflict. If you want the name back immediately (and didn't enable purge protection), you can force it:

```bash
az keyvault purge --name kv-learning-bvj --location centralindia
```

### 8.6 What's next

**Module 9 — Monitoring** turns Azure Monitor and Log Analytics onto the resources you've already built — metrics, alert rules, and just enough KQL to run a real query, rather than treating monitoring as an afterthought bolted on at the end.

Run through Module 8, and say the word for Module 9 — or ask now if the RBAC-denied test, the Key Vault reference, or the SDK call didn't behave as expected.

---

<a id="module-9"></a>
## Module 9 — Monitoring: Azure Monitor, Log Analytics, KQL

Same rebuild note as the last two modules — fresh resource group here too. This module points monitoring at a resource you spin up specifically for it, so it's self-contained either way.

### 9.0 Learning objectives
- Distinguish **Metrics** (numeric time-series) from **Logs** (structured records queried with KQL), and why that split affects cost and retention
- Understand that nothing reaches Log Analytics until a **diagnostic setting** explicitly routes it there
- Write and run real KQL queries against real log data
- Create a metric alert rule with an action group, and actually receive the notification
- Know the CLI's workspace-identifier quirk before it costs you twenty minutes of confusion

### 9.1 Concepts before you touch the portal

- **Metrics** — lightweight numeric time-series (CPU %, request count, response time), collected automatically for most resources, stored in Azure Monitor's metrics database, near-real-time, and cheap-to-free within the default retention window. Good for dashboards and simple thresholds.
- **Logs** — structured records (HTTP logs, activity logs, custom telemetry) that must be explicitly routed into a **Log Analytics workspace** via a **diagnostic setting**, then queried with **KQL**. Richer and more flexible than metrics, but billed by ingested volume beyond a **5 GB/month free allowance per billing account** (shared across every workspace under that billing account, not a separate 5 GB for each one) — genuinely free at lab scale, but worth getting the unit right since your whole notebook is built around protecting a fixed credit pool.
- **Diagnostic settings** — the plumbing. A resource does not send anything to Log Analytics by default; you create a diagnostic setting naming which log/metric categories go where (a workspace, storage for archive, or an Event Hub for streaming out). An empty workspace right after setup almost always means this step, or the ingestion delay after it, not a real problem.
- **KQL (Kusto Query Language)** — pipe-based: `TableName | where ... | summarize ... | order by ...`. Each stage takes the previous stage's output and transforms it further — read it left to right like a Unix pipeline, not like a single SQL statement.
- **Alert rules** — three parts: a **signal/condition** (what to watch, and the threshold that counts as "alerting"), a **scope** (which resource it watches), and an **action group** (what actually happens when it fires — email, SMS, webhook, function). The condition alone only decides "has this crossed the line"; the action group is what makes that decision actionable.

### 9.2 Hands-on Lab

**Portal walkthrough:**

1. **Log Analytics workspaces → + Create**, resource group `rg-monitoring-lab`, name `law-learning-bvj`, region Central India.
2. Create the App Service as in earlier modules (`app-monitor-lab-bvj`, F1 plan), don't deploy any custom code — the default placeholder page is enough to generate HTTP logs.
3. On the App Service → **Diagnostic settings → + Add diagnostic setting** → check `AppServiceHTTPLogs` and `AppServiceConsoleLogs`, check `AllMetrics`, destination **Send to Log Analytics workspace** → pick `law-learning-bvj` → Save. This is arguably the *more* natural place to do this step — it's exactly what the CLI's `diagnostic-settings create` call does underneath.
4. Generate traffic by just refreshing the app's URL a dozen or so times in your browser.
5. **Logs** (left nav, on either the workspace or the App Service itself) is where you paste the KQL queries below directly — this part is Portal-native either way, there's no meaningfully different CLI equivalent for exploratory querying.
6. **Alerts → + Create → Alert rule** on the App Service → pick a signal (e.g. `Requests`) → set the condition → **Actions → + Create action group** → add an email receiver → Create.

**CLI walkthrough:**

```bash
az group create --name rg-monitoring-lab --location centralindia \
  --tags project=azure-learning module=09-monitoring

az appservice plan create --name plan-monitor-lab --resource-group rg-monitoring-lab --sku F1 --is-linux
az webapp create --resource-group rg-monitoring-lab --plan plan-monitor-lab --name app-monitor-lab-bvj

az monitor log-analytics workspace create \
  --resource-group rg-monitoring-lab --workspace-name law-learning-bvj --location centralindia

WEBAPP_ID=$(az webapp show --resource-group rg-monitoring-lab --name app-monitor-lab-bvj --query id -o tsv)
LAW_ID=$(az monitor log-analytics workspace show --resource-group rg-monitoring-lab \
  --workspace-name law-learning-bvj --query id -o tsv)

az monitor diagnostic-settings create --name diag-app-monitor --resource "$WEBAPP_ID" --workspace "$LAW_ID" \
  --logs '[{"category":"AppServiceHTTPLogs","enabled":true},{"category":"AppServiceConsoleLogs","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'

# Generate some actual traffic to have something worth logging
for i in {1..20}; do curl -s -o /dev/null "https://app-monitor-lab-bvj.azurewebsites.net"; done
```

**Wait a few minutes** — log ingestion is not instant. Then open **Portal → the Log Analytics workspace → Logs**, and run:

```kql
AppServiceHTTPLogs
| take 20
```

```kql
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| summarize RequestCount = count() by ScStatus
| order by RequestCount desc
```

```kql
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| summarize AvgTimeTaken = avg(TimeTaken) by bin(TimeGenerated, 5m)
| order by TimeGenerated asc
```

You can also run a query from the CLI — note the quirk below before you try it:

```bash
WORKSPACE_GUID=$(az monitor log-analytics workspace show --resource-group rg-monitoring-lab \
  --workspace-name law-learning-bvj --query customerId -o tsv)

az monitor log-analytics query \
  --workspace "$WORKSPACE_GUID" \
  --analytics-query "AppServiceHTTPLogs | take 20" \
  --output table
```

**The quirk**: almost every other command in this notebook takes a full ARM resource ID. This one wants the workspace's **`customerId`** — a separate GUID, not the resource ID — a holdover from Log Analytics predating full ARM integration. Passing the resource ID here will fail in a way that looks like a permissions problem but isn't.

**Now an alert rule, with a real action group:**

```bash
az monitor action-group create --resource-group rg-monitoring-lab \
  --name ag-learning-email --short-name agEmail \
  --action email notifyme <your-email>

az monitor metrics alert create --resource-group rg-monitoring-lab --name alert-high-requests \
  --scopes "$WEBAPP_ID" \
  --condition "total Requests > 10" \
  --window-size 5m --evaluation-frequency 1m \
  --action ag-learning-email \
  --description "Demo threshold — in a real environment, alert on Http5xx > 0 instead"
```

You already generated 20 requests above, so this should fire within a few minutes of creation. **Check your inbox for a subscription-confirmation email first** — a brand-new action group email receiver has to be confirmed once before any alert actually reaches it; skip that and every future alert silently goes nowhere.

### 9.3 Self-check questions

<details>
<summary>Question 1 — Fundamental difference between a Metric and a Log, and why it affects cost/retention?</summary>

Metrics are lightweight numeric time-series, collected automatically, near-real-time, and cheap within their default retention window. Logs are structured records that must be explicitly routed (via a diagnostic setting) into a Log Analytics workspace and queried with KQL — more flexible and detailed, but billed by ingested volume and subject to whatever retention you configure. The richer data model comes with an explicit cost/retention decision Metrics don't require.
</details>

<details>
<summary>Question 2 — Why was the workspace empty right after creating the diagnostic setting?</summary>

Two reasons stack here: nothing flows to Log Analytics until a diagnostic setting explicitly routes it there (the setting itself doesn't backfill anything), and even after it's active, ingestion has a delay of a few minutes — an empty workspace immediately after setup is expected, not broken.
</details>

<details>
<summary>Question 3 — What does the pipe do in KQL, and how does that differ from a single SQL query?</summary>

Each pipe passes the table/result on its left as input to the operator on its right, so the query reads as a left-to-right pipeline of transformations — filter, then aggregate, then sort — rather than one declarative statement describing the whole result at once. It's closer to chaining Unix commands than writing a single SQL `SELECT`.
</details>

<details>
<summary>Question 4 — What three components does every alert rule need, and what does the action group specifically add?</summary>

A signal/condition (what to watch and the threshold that counts as alerting), a scope (which resource it watches), and an action group (what happens when it fires). The condition alone only determines "has this crossed the line" — the action group is what turns that determination into an email, SMS, webhook, or automated response. Without one, an alert can fire and no one would ever find out.
</details>

<details>
<summary>Question 5 — Why did the CLI query command need the workspace's `customerId` instead of its resource ID, when almost everything else in this notebook uses a resource ID?</summary>

Log Analytics workspaces are addressed two different ways depending on the API: the ARM resource ID for management operations (like creating the diagnostic setting), and a separate `customerId` GUID for the query API itself — a legacy holdover from before Log Analytics was fully integrated into ARM. It's a genuine platform inconsistency, not a mistake on your part when the resource ID doesn't work where `customerId` is expected.
</details>

### 9.4 Common mistakes / gotchas

- **Expecting logs the instant a diagnostic setting is created** — no data flows until the resource does something *after* the setting exists, and then only after an ingestion delay.
- **Checking an idle resource's logs** — nothing to log if nothing happened; the `curl` loop above exists specifically to give you something to query.
- **Never confirming the action group's email subscription** — a new email receiver needs a one-time confirmation, or every future alert silently never arrives.
- **Treating Metrics and Logs as interchangeable** — a KQL query won't run against Metrics Explorer and vice versa; know which one a given troubleshooting task actually calls for.
- **Passing the workspace's resource ID where `customerId` is expected in CLI query commands** — see self-check Q5.

### 9.5 Cost check + cleanup

Well within the 5 GB/month free Log Analytics allowance (shared per billing account, not per workspace); App Service F1 is free; action groups don't add cost for email.

```bash
az group delete --name rg-monitoring-lab --yes
```

### 9.6 What's next

**Module 10 — Backup & Recovery** covers VM backup and Recovery Services Vaults — you'll spin up a small VM specifically for this module (the earlier ones are long since cleaned up), back it up, and understand the actual recovery point mechanics rather than just clicking "enable backup" and trusting it.

Run through Module 9, and say the word for Module 10 — or ask now if the KQL queries, the diagnostic setting, or the alert notification didn't come through as expected.

---

<a id="module-10"></a>
## Module 10 — Backup & Recovery: VM Backup, Recovery Services Vault

This module contains the single costliest misconception in the whole notebook if you skip it: **deleting a VM does not stop its backup from billing you.** That's the actual point of the lab, not a side note.

### 10.0 Learning objectives
- Create a Recovery Services Vault and understand its storage redundancy setting is independent of the source VM's disk replication
- Enable backup on a VM, understand what a backup policy actually schedules and retains
- Trigger an on-demand backup and watch a real backup job run to completion
- Know the three restore options and when each fits
- Prove, deliberately, that deleting the VM leaves the backup (and its cost) behind — and know the correct cleanup order

### 10.1 Concepts before you touch the portal

- **Recovery Services Vault** — the management/storage container for backup data. It has its **own** redundancy setting (LRS/ZRS/GRS/RA-GRS) for how the *backup data itself* is stored — completely independent of whatever replication the source VM's live disks use. A VM on LRS disks can still have GRS-replicated backups, or vice versa.
- **Backup policy** — defines the schedule (how often a backup is taken) and retention (how long each recovery point survives before automatic cleanup, e.g. daily points for 30 days). The vault comes with a `DefaultPolicy` you can inspect and use as-is for this lab.
- **Recovery points** — each one represents a complete, restorable point in time. The first backup after enabling protection is always a full snapshot (nothing to compare against yet); later ones are incremental at the block level. This distinction is invisible when you actually restore — Azure reassembles whatever chain is needed behind a single recovery point, so restoring is always "pick a point in time," never "figure out which incrementals to apply."
- **Instant Restore vs. vault tier** — the most recent recovery points are kept as fast-access snapshots ("Instant Restore") alongside the VM, then age into the vault's own storage — cheaper to keep long-term, slower to restore from since the data has to be retrieved first.
- **Three restore options**, conceptually: **Create a new VM** (fastest, test a recovery point in isolation without touching the original), **Replace existing disks** (in-place restore onto the same VM), **Restore disks only** (get the disk back without recreating a VM around it yet — useful to inspect data first).
- **The critical billing fact**: the protected-item relationship and the backup data both live in the vault, independent of the VM. Deleting the VM removes the compute resource; it does **not** tell the vault to stop protecting or storing anything. Billing (a per-instance protection fee + storage for retained recovery points) continues until you explicitly disable protection for that item.

### 10.2 Hands-on Lab

**Portal walkthrough:**

1. Create `vm-backup-demo` as in Module 4.
2. **Recovery Services vaults → + Create**, resource group `rg-backup-lab`, name `rsv-learning-bvj`, region Central India.
3. Open the vault → **Backup** (left nav) → Backup goal `Azure` / `Virtual machine` → select `vm-backup-demo` → pick or leave `DefaultPolicy` → **Enable backup**.
4. **Backup items** (left nav) → click through to the VM → **Backup Now** (top toolbar) → set a retain-until date → OK. The same pane shows job status, so you can watch the first (full) backup progress without switching screens.
5. Once it completes, recovery points are listed right there on the item's pane, with a **Restore VM** button that walks the three restore options (new VM / replace disks / disks only) as wizard tabs.
6. To reproduce the "delete VM, backup survives" lesson: delete the VM from **Virtual machines** as normal, then return to the vault's **Backup items** — it's still listed, now orphaned. **Stop backup** (top toolbar on the item) → choose **Delete Backup Data** → confirm.

**CLI walkthrough:**

```bash
az group create --name rg-backup-lab --location centralindia \
  --tags project=azure-learning module=10-backup-recovery

az vm create --resource-group rg-backup-lab --name vm-backup-demo \
  --image Ubuntu2204 --size Standard_B1s --admin-username azureuser --generate-ssh-keys

az backup vault create --resource-group rg-backup-lab --name rsv-learning-bvj --location centralindia

# The vault's OWN storage redundancy — independent of the VM's disk replication
az backup vault backup-properties set --resource-group rg-backup-lab \
  --name rsv-learning-bvj --backup-storage-redundancy LocallyRedundant

az backup protection enable-for-vm --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --vm vm-backup-demo --policy-name DefaultPolicy

# See what DefaultPolicy actually schedules and retains
az backup policy show --resource-group rg-backup-lab --vault-name rsv-learning-bvj --name DefaultPolicy
```

**Trigger a backup now, rather than waiting for the schedule:**

```bash
CONTAINER=$(az backup container list --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --backup-management-type AzureIaasVM --query "[0].name" -o tsv)
ITEM=$(az backup item list --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --container-name "$CONTAINER" --backup-management-type AzureIaasVM --query "[0].name" -o tsv)

az backup protection backup-now --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --container-name "$CONTAINER" --item-name "$ITEM" --backup-management-type AzureIaasVM \
  --retain-until "$(date -d '+30 days' '+%d-%m-%Y')"

# Watch the job — a first full backup can genuinely take 20-30+ minutes even for a small VM
az backup job list --resource-group rg-backup-lab --vault-name rsv-learning-bvj --output table
```

Don't assume it's stuck if it's still running after ten minutes — that's normal for the first, full backup.

Once it completes, list what you now have:

```bash
az backup recoverypoint list --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --container-name "$CONTAINER" --item-name "$ITEM" --backup-management-type AzureIaasVM --output table
```

### 10.3 The deliberate mistake: delete the VM, watch the backup survive

```bash
az vm delete --resource-group rg-backup-lab --name vm-backup-demo --yes
```

Now check the vault:

```bash
az backup item list --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --backup-management-type AzureIaasVM --output table
```

The protected item is still listed — the VM is gone, but the vault doesn't know or care. This is still costing you. The correct cleanup:

```bash
az backup protection disable --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --container-name "$CONTAINER" --item-name "$ITEM" --backup-management-type AzureIaasVM \
  --delete-backup-data true --yes
```

### 10.4 Self-check questions

<details>
<summary>Question 1 — Why doesn't deleting the VM stop backup billing, and what's the correct sequence to actually stop it?</summary>

The protected-item relationship and the backup data both live in the Recovery Services Vault, independently of the VM resource itself. Deleting the VM only removes the compute resource — it doesn't touch the vault's ongoing protection or stored recovery points. The correct sequence is to explicitly disable protection for that item, choosing whether to delete the retained backup data or keep it — billing for that protected instance only stops once that's done.
</details>

<details>
<summary>Question 2 — Instant Restore recovery points vs. ones that have moved to the vault tier?</summary>

Instant Restore points are kept as native disk snapshots alongside the VM for a short window — fast to restore from since no data has to move. Older points transition into the vault's own storage, which is cheaper to retain long-term but slower to restore from, since the data must first be retrieved from vault storage.
</details>

<details>
<summary>Question 3 — Why is the first backup always full while later ones are incremental, and does it matter when restoring?</summary>

The first backup has nothing prior to compare against, so it must capture everything; subsequent backups only need the blocks that changed since the last one. It doesn't matter when restoring — Azure reassembles whatever chain of incrementals a given recovery point needs behind the scenes, so restoring is always "pick a point in time," never "figure out which incremental backups to apply in order."
</details>

<details>
<summary>Question 4 — The three restore options, one line each on when you'd pick it?</summary>

**Create a new VM** — fastest, test or inspect a recovery point in isolation without touching the original. **Replace existing disks** — in-place restore onto the same VM, when you're confident you want to roll the actual machine back. **Restore disks only** — get the disk(s) back without recreating a VM yet, useful to inspect data or attach manually before deciding what to do next.
</details>

<details>
<summary>Question 5 — What does the vault's own storage redundancy setting actually protect, versus the VM's disk replication?</summary>

It governs how many copies of the *backup data itself* Azure keeps and where (same-datacenter LRS vs. cross-region GRS), completely independent of whatever replication the source VM's live disks use. They're two separate durability decisions that happen to sound similar.
</details>

### 10.5 Common mistakes / gotchas

- **Assuming VM deletion stops backup charges** — the single costliest misconception this module exists to correct.
- **Panicking that the first backup job is stuck** — 20–30+ minutes for a full initial snapshot on even a small VM is normal; check job status rather than assuming failure.
- **Trying to delete the vault while it still has protected (or even soft-deleted) backup items** — it will refuse until those are fully cleared, which can take up to the backup soft-delete retention window unless you explicitly undo the soft-delete state first.
- **Confusing the vault's redundancy setting with the VM's disk replication** — see self-check Q5; they're independent.
- **Leaving `--retain-until` too generous on an ad hoc backup** — an on-demand backup can outlive the policy's normal retention if left unbounded, quietly adding storage cost.

### 10.6 Cost check + cleanup

The correct, complete sequence (regardless of what order you did things in during the lab):

```bash
az backup protection disable --resource-group rg-backup-lab --vault-name rsv-learning-bvj \
  --container-name "$CONTAINER" --item-name "$ITEM" --backup-management-type AzureIaasVM \
  --delete-backup-data true --yes

# If the VM still exists at this point, delete it too
az vm delete --resource-group rg-backup-lab --name vm-backup-demo --yes

az group delete --name rg-backup-lab --yes
```

If the resource group delete complains about the vault, it's almost always because backup protection wasn't fully disabled first — go back to the `az backup protection disable` step before retrying.

### 10.7 What's next

**Module 11 — Infrastructure as Code** steps back from typing commands by hand entirely: ARM templates and Bicep let you describe this whole environment as a file and deploy it repeatably — directly answering the "rebuilding things from scratch" tedium flagged back in Module 7. Then **Module 12 — Capstone** pulls everything from Modules 1–11 into one deliberately-built multi-tier environment, documented and torn down completely — your dress rehearsal for both the practical judgment AZ-104 tests and the trade-off thinking AZ-305 builds on.

Run through Module 10, and say the word for Module 11 — or ask now if the backup job, the deliberate-delete demonstration, or vault cleanup didn't go as expected.

---

<a id="ext-f"></a>
## Extension Lab F — Site Recovery: Region-Level Disaster Recovery

*Sits between Modules 10 and 11 — pairs directly with Module 10's Backup lab by covering the failure scope Backup doesn't.*

**Objectives**
- Explain what Site Recovery protects against that Backup (Module 10) doesn't
- Enable replication on a VM to a secondary Azure region
- Run a Test Failover, confirm it doesn't disrupt anything live, then clean up

**Concepts**

Module 10's Backup answers "I lost or corrupted data — get me a point-in-time copy back," within one region, on a schedule (RPO = however often you back up). **Azure Site Recovery (ASR)** answers a different question: "the whole region my VM lives in just went down — get my workload running somewhere else." ASR continuously replicates a VM's disks to a secondary region, so its RPO is measured in minutes, not "since last night's backup."

**Test Failover** spins up an isolated copy of the replicated VM in the target region, in an isolated test network, without touching ongoing replication or your live production VM — it exists specifically so you can rehearse a failover with zero risk. **Failover** (the real one) is disruptive by design: it stops replication and directs a genuine cutover to the secondary region — this is what you'd do in an actual regional outage, not something to trigger for practice. **Failback** reverses the direction once the primary region is healthy again, replicating changes made during failover back and cutting back over.

Why this lab leans on the Portal rather than CLI: ASR does have a CLI extension (`az site-recovery ...`), but it operates at the level of fabrics, protection containers, and container mappings — the low-level pieces the Portal's "Enable replication" wizard wires together for you in one flow. For a concept-focused lab, walking the wizard teaches the actual DR workflow; hand-assembling fabrics and mappings via raw CLI calls is real automation-pipeline work, not a good way to learn the concept the first time.

**Hands-on — Portal**

1. You need a VM to protect — reuse one from an earlier module if it's still up, or create a small throwaway one: **Resource groups → + Create** `rg-ext-f` → **Virtual machines → + Create** → `vm-ext-f`, Ubuntu, `Standard_B1s`, `centralindia`.
2. **Recovery Services vaults → + Create** → `rsv-ext-f` in `rg-ext-f`, region: a **different** region from `centralindia` — use `southindia`, Central India's documented paired region, since ASR replicates *to* a different region by definition.
3. Inside `rsv-ext-f` → **Site Recovery → Configure disaster recovery for Azure VMs → Enable replication**. Source: `centralindia`, `vm-ext-f`. Target: `southindia` (Site Recovery offers to create matching target resources — target resource group, VNet, storage — accept the defaults for a lab). Start replication and wait for initial replication to reach **"Protected"** status (this can take a while for a real VM; for a lab-sized OS disk it's usually well under an hour).
4. Once Protected: **Site Recovery → Test Failover** → target network: the isolated test network Azure created → **Run**. Watch the job progress in Site Recovery jobs.
5. Once the test VM is up in `southindia`, note it's a separate, isolated copy — your original `vm-ext-f` in `centralindia` is completely untouched and still serving traffic.
6. **Site Recovery → Test Failover → Cleanup test failover** → confirm — this tears down the temporary test VM and test network.
7. **Site Recovery → Disable replication** for `vm-ext-f` → confirm.

CLI note — the ASR CLI surface exists, but standing up replication end-to-end this way requires creating and mapping fabrics and protection containers as separate steps before you can even enable replication on a specific VM:

```bash
az extension add --name site-recovery
az site-recovery vault list --resource-group rg-ext-f -o table
az site-recovery job list --resource-group rg-ext-f --vault-name rsv-ext-f -o table
```

— useful once you're automating an *existing* DR setup, not the tool for standing one up for the first time.

**Self-check questions**

<details><summary>1. Your backup policy runs nightly. Your VM's region has a total outage at 2pm. Does Backup alone get your workload running again quickly, somewhere else?</summary>

No — Backup gives you a restore point from last night, in the same regional concept, and doesn't stand up compute in another region on its own. That's specifically what Site Recovery does.
</details>

<details><summary>2. You ran a Test Failover for <code>vm-ext-f</code>. Did this affect production traffic to the real <code>vm-ext-f</code> at all?</summary>

No — Test Failover deliberately creates an isolated copy in an isolated test network; it doesn't touch ongoing replication or the live VM.
</details>

<details><summary>3. What's the difference between Failover and Failback?</summary>

Failover cuts real traffic over to the secondary region (disruptive, stops replication); Failback reverses that once the primary region recovers, replicating changes made during failover back and cutting over to the original region.
</details>

<details><summary>4. Why did this lab use the Portal instead of building the whole thing with <code>az site-recovery</code> commands?</summary>

ASR's CLI operates at a lower level (fabrics, protection containers, mappings) that the Portal wizard assembles automatically — appropriate to automate once you understand the moving parts, not the easiest way to learn them the first time.
</details>

**Common mistakes / gotchas**

- Confusing Backup and Site Recovery as "two ways to do the same thing" — they solve different failure scopes (data loss vs. regional loss), and a real production setup usually has both, not a choice between them.
- Running a real Failover to "test" it — that's exactly what Test Failover exists to prevent; a real failover is disruptive and stops replication.
- Forgetting to run **Cleanup** after a Test Failover — the temporary test VM and network keep costing money until torn down explicitly.
- Leaving replication enabled long after the lab is done — protected-instance charges and cross-region storage keep accruing as long as replication stays on.

**Cost check + cleanup**

Site Recovery is one of the pricier, more complex services to run for real: a per-protected-instance monthly charge, storage in *two* regions instead of one, and — if you ran the test failover — a temporary VM and network in the target region while it existed. Don't leave replication enabled beyond this lab.

```bash
# Disable replication in the Portal first (Site Recovery → Disable replication),
# then remove the source-region resources:
az group delete --name rg-ext-f --yes --no-wait
# Site Recovery also creates target-region resources (VNet, storage, cache) in
# southindia during "Enable replication" — check Resource groups for anything
# tied to rsv-ext-f's vault there and delete it too if disabling replication
# didn't already clean it up.
```

**What's next:** Back to Module 11 — Infrastructure as Code.

---

<a id="module-11"></a>
## Module 11 — Infrastructure as Code: ARM Templates & Bicep

Every module so far has been imperative — you (or a Portal wizard) typed a sequence of individual commands. This module is about describing the *end state* you want instead, and letting Azure figure out how to get there — directly answering the "rebuilding things from scratch every module" tedium flagged back in Module 7's housekeeping note.

### 11.0 Learning objectives
- Explain the difference between imperative deployment (everything so far) and declarative deployment (this module)
- Read the anatomy of a Bicep file, and recognize the ARM JSON it compiles to (AZ-104 still expects you to recognize raw ARM JSON, even though almost nobody hand-writes it anymore)
- Deploy a real resource from a template instead of a one-off command, using parameters so the file is reusable rather than a recorded script
- Run `what-if` before a real deployment, and understand why re-running an unchanged template is always safe
- Know when reaching for IaC is worth it, and when the CLI/Portal genuinely is fine (quick exploration, a true one-off lab)

### 11.1 Concepts before you write a template

- **Imperative vs. declarative** — `az vm create`, clicking through the Portal: these are imperative, you specify the steps. ARM templates and Bicep are declarative: you specify the desired end state, and Azure Resource Manager works out what needs to change to reach it. Re-running the same unchanged template is safe and does nothing on the second run — this is called **idempotency**.
- **ARM templates (JSON)** — Azure's native declarative format. Verbose, and these days generated by tooling more than hand-written — but it's still what everything ultimately compiles down to, and still something AZ-104 expects you to be able to recognize.
- **Bicep** — a concise language that compiles down to ARM JSON. This is what you should actually write by hand today. `az bicep build` shows you the compiled JSON if you want to see what it becomes.
- **Parameters** — inputs to a template (an environment name, a SKU) so the same file can deploy differently without editing the file itself.
- **Resources** — the declarations inside the template; this is the direct equivalent of every `az ... create` command you've typed across this notebook, just declared rather than issued as a step.
- **Outputs** — values a deployment hands back afterward (a generated hostname, an endpoint URL), so you don't need a separate `az ... show` call to get them.
- **`what-if`** — shows exactly what a deployment *would* change before it actually runs. The IaC equivalent of a dry run, and a genuinely important habit once templates touch more than one resource.

### 11.2 Hands-on Lab — redeploy Module 5's storage account as Bicep

Deliberately reusing something you've already built by hand (Module 5's storage account) makes the comparison direct: same result, a completely different process.

`main.bicep`:

```bicep
@description('Project tag applied to everything this template creates')
param projectTag string = 'azure-learning'

@description('Azure region for all resources')
param location string = resourceGroup().location

@description('Globally unique storage account name (lowercase, no hyphens, 3-24 chars)')
param storageAccountName string

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  tags: {
    project: projectTag
    module: '11-iac'
  }
}

output storageAccountEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

A resource-group-scoped deployment needs the resource group to already exist — creating the group itself is a separate, subscription-scoped kind of deployment, out of scope for this lab.

```bash
az group create --name rg-iac-lab --location centralindia \
  --tags project=azure-learning module=11-iac

# Dry run first — see exactly what this WOULD do before it does anything
az deployment group what-if \
  --resource-group rg-iac-lab --template-file main.bicep \
  --parameters storageAccountName=staziaclabbvj

# The real deployment
az deployment group create \
  --resource-group rg-iac-lab --template-file main.bicep \
  --parameters storageAccountName=staziaclabbvj \
  --name deploy-storage-v1
```

**Prove idempotency** — run the exact same create command again, completely unchanged:

```bash
az deployment group create \
  --resource-group rg-iac-lab --template-file main.bicep \
  --parameters storageAccountName=staziaclabbvj \
  --name deploy-storage-v2
```

No error, nothing actually changes — the account already matches the desired state described in the file. Compare that to running `az storage account create` a second time by hand, which behaves entirely differently around an already-existing resource.

**Now make a real change** — edit `sku.name` in `main.bicep` from `Standard_LRS` to `Standard_ZRS`, then redeploy with the same command (a new `--name` for the deployment). Only the SKU updates — no delete, no recreate.

Check the output value your template declared:

```bash
az deployment group show --resource-group rg-iac-lab --name deploy-storage-v2 \
  --query properties.outputs.storageAccountEndpoint.value -o tsv
```

**See the raw ARM JSON this compiles to** (the format AZ-104 questions will show you, even though you'll rarely write it by hand):

```bash
az bicep build --file main.bicep
cat main.json
```

Notice how much longer and more repetitive it is than the Bicep source — that's exactly why Bicep exists.

### 11.3 Self-check questions

<details>
<summary>Question 1 — Fundamental difference between everything in Modules 1–10 and what you just did here?</summary>

Modules 1–10 were imperative: you (or a Portal wizard) issued a sequence of individual commands describing steps to take. Bicep/ARM is declarative: you describe the resource's desired end state, and Azure Resource Manager works out what needs to happen to reach it — including doing nothing at all if it's already there.
</details>

<details>
<summary>Question 2 — What happened when you re-ran the exact same deployment unchanged, and why?</summary>

Nothing changed, and nothing errored. A declarative deployment compares the desired state in the template against the resource's actual current state and only applies the difference — since nothing in the template changed and the resource already matched it, there was no difference to apply. This is meaningfully different from re-running an imperative "create" command against something that already exists.
</details>

<details>
<summary>Question 3 — What does `what-if` actually show, and why run it before every real deployment?</summary>

It shows the exact set of creates, updates, and deletes a deployment would perform, without performing any of them — a dry run. Running it first catches unintended changes (a typo that would delete something, a parameter you forgot to override) before they happen instead of after, which matters far more once a template touches multiple resources than it does for this single-storage-account lab.
</details>

<details>
<summary>Question 4 — Why write Bicep instead of hand-authoring the ARM JSON it compiles to, if they deploy identically?</summary>

There's no deployment-time capability difference — the entire benefit is human terms. Bicep is dramatically shorter and more readable, and it catches many errors (typos in property names, wrong types) at authoring time rather than only surfacing them when a deployment fails.
</details>

<details>
<summary>Question 5 — Why did the resource group need to already exist before this deployment could run?</summary>

A resource-group-scoped deployment (`az deployment group create`) targets an existing resource group by design — it can create and manage resources inside that group, but not the group itself. Creating the resource group as part of the same deployment would require a subscription-scoped deployment instead, a related but different mechanism not covered in this lab.
</details>

### 11.4 Common mistakes / gotchas

- **Hand-writing raw ARM JSON "because that's what the exam shows"** — write Bicep, and use `az bicep build` if you ever need to see or submit the compiled JSON.
- **Skipping `what-if` and finding out what a deployment did only after it happened** — increasingly risky as templates grow to touch more than one resource.
- **Hardcoding values directly in resource blocks** instead of exposing them as parameters — defeats the entire point of writing a reusable template instead of a recorded script.
- **Assuming a resource-group-scoped deployment can create the resource group itself** — it can't; that needs subscription scope, a different deployment target entirely.
- **Treating a "no changes" idempotent redeploy as a wasted run** — it's actually useful confirmation that reality still matches what the template says it should.

### 11.5 Cost check + cleanup

```bash
az group delete --name rg-iac-lab --yes
```

### 11.6 What's next

**Module 12 — Capstone** is where this pays off directly: you're welcome — and encouraged — to build the Capstone's reference architecture as a Bicep file instead of a sequence of CLI commands. It's exactly the kind of multi-resource, "I'll want to rebuild this" environment IaC exists for.

Run through Module 11, and say the word for Module 12 — or ask now if `what-if`, the idempotent redeploy, or the ARM JSON compile step didn't behave as expected.

---

<a id="module-12"></a>
## Module 12 — Capstone: Build It Yourself, Document the Decisions, Delete Everything

This module deliberately does **not** hand you a copy-paste script. Every command you need already exists somewhere in Modules 1–11 — the point of a capstone is finding out whether you can assemble them yourself. This is also the closest thing in this notebook to how AZ-104's scenario questions and AZ-305's design questions actually read.

### 12.0 Learning objectives
- Independently design and build a small multi-tier environment from requirements, not a step list
- Write a short architecture decision record — what you chose, what you rejected, why — the actual AZ-305 skill
- Execute a complete, verified teardown across the *entire* subscription, not just one resource group
- Come out the other side with something concrete you could describe in an interview
- **Optional stretch:** build it as a Bicep file (Module 11) instead of one-off commands

### 12.1 The brief

> Design and build a small internet-facing web application on Azure that:
> 1. Serves HTTP traffic to the public internet
> 2. Keeps its data tier completely unreachable from the internet
> 3. Uses no passwords or connection strings anywhere in code or configuration
> 4. Stores at least one secret in Key Vault, consumed without the secret value ever appearing in your code
> 5. Sends logs somewhere you can query them
> 6. Can be identified and completely deleted in one sitting, with nothing left behind

### 12.2 A reference architecture (use it, or design your own that still satisfies the brief)

```
rg-capstone-lab
│
├── vnet-capstone (10.2.0.0/16)
│   ├── snet-web  (10.2.1.0/24) → nsg-web (allow 80 from Internet)
│   └── snet-db   (10.2.2.0/24) → nsg-db  (allow 5432 from snet-web only)
│
├── lb-capstone (Standard LB, public IP) → backend pool: vm-capstone-web-1, vm-capstone-web-2 (in snet-web)
├── vm-capstone-db (in snet-db, no public IP)
│
├── kv-capstone (RBAC model) — holds one real secret
├── id-capstone (user-assigned identity, attached to both web VMs)
│   └── granted: Key Vault Secrets User on kv-capstone, Storage Blob Data Reader on st-capstone
│
├── st-capstone (storage account — app assets)
├── law-capstone (Log Analytics workspace) + diagnostic settings on lb-capstone
└── an alert rule + action group watching something meaningful
```

This combines Module 3 (segmentation), Module 4 (VMs, SSH), Module 6 (Standard LB), Module 2 (identity/RBAC), Module 8 (Key Vault), Module 5 (storage), and Module 9 (monitoring) into one coherent build — every command you need has a direct precedent earlier in this notebook.

**A lighter alternative**, if you're short on time today: swap the VM/LB tier for a Container App (Module 7) fronted by its own built-in ingress, and skip the VNet/subnets/LB entirely. It satisfies requirements 1, 3, 4, 5, and 6 above, but not the network-segmentation half of requirement 2 in the same way — worth noting *why* it doesn't in your write-up (see below).

### 12.3 Verification checklist — prove each requirement, don't just assume it

| # | Requirement | How to actually verify it |
|---|---|---|
| 1 | HTTP reachable from the internet | `curl` the load balancer's (or Container App's) public endpoint from your own machine |
| 2 | Data tier unreachable from the internet | Confirm the db VM has no public IP; try (and fail) to reach it directly |
| 3 | No passwords/connection strings in code or config | Grep your own code and app settings for anything that looks like a literal credential — there should be none |
| 4 | Key Vault secret, consumed without appearing in code | Re-run Module 8's pattern; confirm via the Key Vault reference "Resolved" status or a working SDK call |
| 5 | Logs queryable | Run one real KQL query in Log Analytics against this environment's own logs |
| 6 | Fully deletable | Complete section 12.5 below and verify with the subscription-wide check |

### 12.4 Write it up — the actual AZ-305 skill

Save this as its own short document (a great first entry in whatever "career roadmap / reference material" collection you already keep):

```markdown
# Capstone Architecture Decision Record

## Requirements
(copy the brief from 12.1)

## Options Considered
- Option A: [e.g., VM + Standard LB multi-tier]
- Option B: [e.g., Container Apps, no VNet]

## Decision
[Which you chose, and the one or two sentences that actually decided it —
cost, complexity, time available, or a specific requirement one option
satisfied better than the other.]

## Trade-offs Accepted
[What you deliberately gave up by choosing as you did — e.g., "chose VM-based
LB for full network segmentation practice, at the cost of more manual patching
responsibility than a PaaS option would require."]

## Cost Estimate
[Rough ₹ for running this for a day, using Part 2's spend bands as a guide]
```

This one-page habit — options, decision, trade-offs, cost — is exactly the shape AZ-305 exam scenarios expect you to reason in, and it's a genuinely useful habit to carry into real design work afterward.

### 12.5 Full teardown — subscription-wide, not just one resource group

Delete the capstone resource group as usual:

```bash
az group delete --name rg-capstone-lab --yes
```

Then do something you haven't done yet in this notebook: check the **whole subscription** for anything left behind across all eleven modules — an identity, a soft-deleted Key Vault, an orphaned disk from a module where cleanup was interrupted.

```bash
az group list --output table
az resource list --output table
```

Anything with `project=azure-learning` still listed that you don't recognize is worth investigating before you consider this month closed out.

### 12.6 What's next

You've now built, broken, and torn down real infrastructure across the **core hands-on path spanning all five AZ-104 skill areas** (identity/governance, compute, storage, networking, monitoring) — not full coverage of every AZ-104 exam topic — see the coverage note in Part 2 for what this notebook doesn't touch and where to fill those gaps before sitting the exam. You've also taken your first real steps into AI-200 territory (Container Apps, ACR), practiced describing infrastructure declaratively instead of typing it by hand, and practiced the write-it-up habit AZ-305 rewards. From here: keep the Appendices below as a living reference, and when you're ready, this is a good point to actually schedule the AZ-104 exam while the hands-on memory is fresh — theory review closes the rest of the gap faster once you've done the labs than the other way around.

Ask me anything on the capstone, or just tell me when you want to talk through scheduling the exam or planning what comes after these 29 days.

---

<a id="appendix-a"></a>
## Appendix A — Azure CLI Cheat Sheet (grows as you go)

**Account & subscription**
```bash
az login
az account show --output table
az account list --output table
az account set --subscription "<name-or-id>"
az configure --defaults group=<rg-name> location=<region>
```

**Resource groups & tags**
```bash
az group create --name <rg> --location <region> --tags key=value
az group list --output table
az group list --tag key=value --output table
az group delete --name <rg> --yes
az resource list --resource-group <rg> --output table
az resource list --output table   # whole subscription
```

**RBAC & identity**
```bash
az ad signed-in-user show --query "{id:id, upn:userPrincipalName}" --output table
az role assignment create --assignee <upn-or-object-id> --role "<RoleName>" --scope <resource-id>
az role assignment list --scope <resource-id> --output table
az ad group create --display-name <name> --mail-nickname <name>
az identity create --name <name> --resource-group <rg>
az identity show --name <name> --resource-group <rg> --query "{clientId:clientId, principalId:principalId}"
```

**Networking**
```bash
az network vnet create --resource-group <rg> --name <vnet> --address-prefix <cidr> --subnet-name <subnet> --subnet-prefix <cidr>
az network vnet subnet create --resource-group <rg> --vnet-name <vnet> --name <subnet> --address-prefix <cidr>
az network nsg create --resource-group <rg> --name <nsg>
az network nsg rule create --resource-group <rg> --nsg-name <nsg> --name <rule> --priority <n> --direction Inbound --access Allow|Deny --protocol Tcp --source-address-prefixes <src> --destination-port-ranges <port>
az network vnet subnet update --resource-group <rg> --vnet-name <vnet> --name <subnet> --network-security-group <nsg>
az network vnet peering create --resource-group <rg> --name <peer-name> --vnet-name <vnet> --remote-vnet <remote-vnet> --allow-vnet-access true
az network vnet peering list --resource-group <rg> --vnet-name <vnet> --output table
```

**VMs**
```bash
az network public-ip create --resource-group <rg> --name <pip> --sku Standard --allocation-method Static
az network nic create --resource-group <rg> --name <nic> --vnet-name <vnet> --subnet <subnet> [--public-ip-address <pip>]
az vm create --resource-group <rg> --name <vm> --nics <nic> --image Ubuntu2204 --size Standard_B1s --admin-username azureuser --generate-ssh-keys --os-disk-name <disk>
az vm show --resource-group <rg> --name <vm> -d --query publicIps -o tsv
az vm delete --resource-group <rg> --name <vm> --yes
az disk delete --resource-group <rg> --name <disk> --yes
az network nic delete --resource-group <rg> --name <nic>
az network public-ip delete --resource-group <rg> --name <pip>
```

**Storage**
```bash
az storage account create --name <acct> --resource-group <rg> --location <region> --sku Standard_LRS --kind StorageV2
az storage container create --account-name <acct> --name <container> --auth-mode login
az storage blob upload --account-name <acct> --container-name <container> --name <blob> --file <path> --auth-mode login
az storage blob generate-sas --account-name <acct> --container-name <container> --name <blob> --permissions r --expiry <UTC datetime> --auth-mode login --as-user --full-uri
```

**Key Vault**
```bash
az keyvault create --name <vault> --resource-group <rg> --location <region> --enable-rbac-authorization true
az keyvault secret set --vault-name <vault> --name <secret-name> --value "<value>"
az keyvault secret show --vault-name <vault> --name <secret-name>
# App Service reference resolution with a USER-ASSIGNED identity needs this too —
# AZURE_CLIENT_ID alone (an app-setting, for your own code's SDK calls) is not enough:
az webapp update --resource-group <rg> --name <app> --set keyVaultReferenceIdentity=<identity-resource-id>
```

**Monitoring**
```bash
az monitor log-analytics workspace create --resource-group <rg> --workspace-name <workspace> --location <region>
az monitor diagnostic-settings create --name <name> --resource <resource-id> --workspace <workspace-id> --logs '[...]' --metrics '[...]'
az monitor log-analytics query --workspace <workspace-customerId-GUID> --analytics-query "<KQL>" --output table
az monitor action-group create --resource-group <rg> --name <name> --short-name <short> --action email <friendly-name> <email>
az monitor metrics alert create --resource-group <rg> --name <name> --scopes <resource-id> --condition "<condition>" --window-size 5m --evaluation-frequency 1m --action <action-group-name>
```

**Backup**
```bash
az backup vault create --resource-group <rg> --name <vault> --location <region>
az backup protection enable-for-vm --resource-group <rg> --vault-name <vault> --vm <vm> --policy-name DefaultPolicy
az backup protection disable --resource-group <rg> --vault-name <vault> --container-name <c> --item-name <i> --backup-management-type AzureIaasVM --delete-backup-data true --yes
```

---

<a id="appendix-b"></a>
## Appendix B — Naming & Tagging Convention

**A note on this notebook's own naming.** Throughout Modules 1–11, resources were named simply — `rg-networking-lab`, `vm-web`, `nsg-app` — deliberately, since everything here is a disposable learning environment. Real environments use a fuller pattern. Below is both: the abbreviations you actually used, and the complete Cloud Adoption Framework (CAF) style pattern for when you're naming things that matter.

### Resource type abbreviations (Microsoft CAF recommended, and what this notebook used)

| Resource type | CAF abbreviation | Used in this notebook |
|---|---|---|
| Resource group | `rg` | `rg-networking-lab` |
| Virtual network | `vnet` | `vnet-a` |
| Subnet | `snet` | `snet-web` |
| Network security group | `nsg` | `nsg-web` |
| Public IP address | `pip` | `pip-vm-web` |
| Network interface | `nic` | `nic-vm-web` |
| Route table | `rt` | `rt-mgmt-lab` |
| VNet peering | `peer` | `peer-a-to-b` |
| Load balancer (external) / (internal) | `lbe` / `lbi` | `lb-web` *(notebook simplified — didn't distinguish external/internal)* |
| Application Gateway | `agw` | `appgw-web` *(notebook used `appgw-`, close enough to CAF's `agw-`)* |
| Virtual machine | `vm` | `vm-web` |
| Managed disk | `disk` / `osdisk` | `disk-vm-web` |
| Storage account | `st` | `stazlearnbvj` *(storage account names can't contain hyphens — CAF's own exception)* |
| Key Vault | `kv` | `kv-learning-bvj` |
| Log Analytics workspace | `log` | `law-learning-bvj` *(notebook used `law-` for clarity)* |
| Recovery Services Vault | `rsv` | `rsv-learning-bvj` |
| App Service Plan | `asp` | `plan-web-lab` *(notebook used `plan-` instead of `asp-`)* |
| App Service (Web App) | `app` | `app-web-lab-bvj` |
| Function App | `func` | `func-lab-bvj` |
| Container Registry | `cr` | `acrlabbvj` *(no hyphens allowed, same exception as storage accounts)* |
| Container Apps Environment | `cae` | `env-apps-lab` *(notebook used `env-`)* |
| Container App | `ca` | `ca-web-lab` |
| User-assigned managed identity | `id` | `id-app-lab` |
| Action group | `ag` | `ag-learning-email` |

### The full real-world pattern

```
<resource-type>-<workload/app>-<environment>-<region>-<instance>
```

Example: `rg-shopsphere-prod-cindia-001` — a resource group, for the "shopsphere" workload, production environment, Central India region, instance 001 (in case you ever need a second one).

This notebook dropped `<environment>`, `<region>`, and `<instance>` because every lab here is inherently non-production and single-instance — reintroduce them the moment you're naming anything that will exist for more than a day or that a teammate will also need to read.

### Tags used in this notebook, and their real-world equivalents

| Tag key used here | Purpose | Real-world equivalent(s) you'd add |
|---|---|---|
| `project` | Groups everything under this learning effort | `application` or `workload` |
| `module` | Tracks which module/lab a resource belongs to | `environment` (`dev`/`test`/`prod`), `costCenter` |
| *(not used here)* | — | `owner` (who to contact), `dataClassification` (if handling real data) |

The habit that matters most, regardless of exact tag names: **every resource should be traceable to a reason it exists and a person responsible for it.** That's what tags are actually for — cost filtering is just the most visible payoff.

---

<a id="appendix-c"></a>
## Appendix C — Glossary

**ARM (Azure Resource Manager)** — the API layer underneath both the Portal and CLI; every action either way ends up as an ARM call.

**Availability Zone** — a physically separate datacenter within a region, with independent power/cooling/networking; ZRS and zone-redundant SKUs spread copies across these.

**Backup policy** — defines a backup schedule and retention period for a protected resource.

**CIDR** — address-range notation (`10.0.0.0/16`); the number after `/` is how many fixed bits define the network portion, so smaller = bigger range.

**Cloud Shell** — a browser-based shell (bash or PowerShell) with the Azure CLI pre-authenticated, backed by a small auto-created storage account.

**Container App** — a serverless container-hosting service with built-in scale-to-zero and ingress.

**Container Registry (ACR)** — a private registry for container images; ACR Tasks (`az acr build`) build images in the cloud without local Docker.

**Control plane vs. data plane** — control plane = managing a resource itself (create/delete/configure); data plane = accessing what's inside it (read/write actual data). Separate RBAC permission layers — being Owner on the control plane does not grant data-plane access.

**Diagnostic setting** — the explicit configuration that routes a resource's logs/metrics to a destination (Log Analytics, storage, or Event Hub); nothing flows anywhere without one.

**Entra ID** (formerly Azure AD) — the tenant-wide identity store: users, groups, service principals, managed identities.

**Health probe** — a periodic check (HTTP/HTTPS/TCP) a Load Balancer or Application Gateway runs against each backend instance, removing unhealthy ones from rotation automatically.

**Key Vault** — a managed store for Secrets (strings), Keys (cryptographic material), and Certificates, with an RBAC or (legacy) Access Policy permission model.

**KQL (Kusto Query Language)** — the pipe-based query language for Log Analytics: `Table | where ... | summarize ...`.

**Load Balancer (Standard)** — Layer 4 (TCP/UDP) traffic distribution across backend instances, without any awareness of HTTP content; Basic SKU was retired 30 September 2025.

**Log Analytics workspace** — the data store for Logs, queried with KQL; ingestion has a 5 GB/**month** free allowance shared per **billing account** (not per workspace, and not per day).

**Managed disk** — a standalone Azure resource backing a VM's OS or data disk; has its own lifecycle and its own storage cost, independent of the VM.

**Managed identity** — an identity Azure manages with no password/secret you ever see. System-assigned: tied one-to-one to a resource's lifecycle. User-assigned: a standalone object, reusable across multiple resources.

**Metric** — lightweight numeric time-series data (CPU %, request count), collected automatically, distinct from Logs.

**NIC (Network Interface)** — the resource holding a private IP; what actually places a VM into a subnet.

**NSG (Network Security Group)** — a stateful allow/deny traffic filter, evaluated lowest-priority-number-first, attachable to a subnet and/or a NIC.

**Peering (VNet)** — connects two VNets for private-IP reachability; non-transitive, and requires explicit peering objects in both directions.

**Public IP** — a separate, explicit Azure resource; a VM/NIC is not internet-reachable unless one is attached. Standard SKU is secure-by-default (denies all inbound unless an NSG allows it); Basic SKU was retired 30 September 2025.

**RBAC (Role-Based Access Control)** — Azure's access-control system: a role definition (a bundle of permissions) bound to a security principal at a scope.

**Recovery point** — a specific, complete, restorable point-in-time backup.

**Recovery Services Vault** — the management/storage container for backup data, with its own independent storage-redundancy setting.

**Region** — a physical Azure datacenter location; affects latency, service/SKU availability, and sometimes compliance.

**Resource group (RG)** — a free, logical container for resources sharing a lifecycle; the standard unit of "create together, delete together."

**Role assignment** — the binding of (principal + role definition + scope) that actually grants access.

**Role definition** — a named bundle of permissions (e.g., "Contributor").

**Route table / UDR (User-Defined Route)** — overrides a subnet's default system routes, commonly to force outbound traffic through a firewall/appliance.

**SAS (Shared Access Signature)** — a signed URL granting time-limited, scoped storage access without sharing an account key; a User Delegation SAS is signed with Entra ID credentials rather than a master key.

**Scope** — the level at which an RBAC role assignment applies: Management Group → Subscription → Resource Group → Resource, inherited downward.

**Soft delete / purge protection** — soft delete keeps a deleted object (Key Vault, backup data) recoverable for a retention window instead of destroying it instantly; purge protection prevents even an Owner from forcing an early permanent purge during that window.

**Storage account** — a globally unique namespace hosting Blob, Files, Queue, and/or Table services.

**Subnet** — a subdivision of a VNet's address space; Azure reserves 5 addresses per subnet.

**Subscription** — the billing and top-level access-management boundary within a tenant.

**Tags** — key/value metadata on a resource or resource group, used for cost filtering, ownership, and cleanup tracking.

**Tenant** — the organization-wide Entra ID identity boundary; one tenant can contain many subscriptions.

**VNet (Virtual Network)** — an isolated private address space within a region.

---

<a id="appendix-d"></a>
## Appendix D — Cost Tracking Log

Fill this in daily alongside Portal → Cost Management + Billing → Cost analysis. It's the single easiest habit for making sure this notebook stays inside your credit.

The **Date** and **Module** columns below are pre-filled against your actual 29-day window (19 Sep – 18 Oct 2026), paced at roughly one section every 1–2 days with a few buffer days built in — the same pacing discussed earlier. It's a scaffold, not a contract: if you move faster or fall behind, just relabel the dates rather than rebuilding rows. **Est. Spend** is a rough planning number from Part 2's spend bands and each module's own cost-check section; only **Actual Spend**, **Cleaned Up?**, and **Notes** are yours to fill in as you go.

| Date | Module | Resources Created | Est. Spend (₹) | Actual Spend (₹) | Cleaned Up? | Notes |
|---|---|---|---|---|---|---|
| 19 Sep | 1 | RG + tags only | 0 | | | Empty RGs cost nothing |
| 20 Sep | 2 | RG, Entra group, UAMI | 0 | | | RBAC/identity objects are free |
| 21 Sep | Ext. A | Policy assignment, lock, budget | 0 | | | Governance controls cost nothing by themselves |
| 22 Sep | 3 (part 1) | 2 VNets, 3 NSGs, peering, route table | 0 | | | Networking has no hourly cost |
| 23 Sep | 3 (cont.) | — | 0 | | | Finish CLI lab + self-check |
| 24 Sep | Ext. B | Private endpoint, private DNS zone | 5–20 | | | Small hourly + query charges |
| 25 Sep | 4 | 2× B1s VM, 1 Standard Public IP, 2 disks | 150–300 | | | |
| 26 Sep | Ext. C | VMSS (2–3× B1s), autoscale | 100–250 | | | Multiple instances draw down free-tier hours faster |
| 27 Sep | Ext. G | 1 VM in `rg-networking-lab` (ASG demo), 1 storage account in `rg-ext-g` | 10–30 | | | Reuses Module 3/4's existing network |
| 28 Sep | 5 | 1 storage account, 4 services | 5–20 | | | |
| 29 Sep | Ext. D | 1 storage account, versioning/soft delete | 5–15 | | | |
| 30 Sep | 6 (part 1) | 2 VMs, Standard Load Balancer | 200–500 | | | |
| 1 Oct | 6 (cont.) | + Application Gateway v2 | 500–1,500 | | | Expensive-service day — same-day teardown |
| 2 Oct | 7 (part 1) | App Service (F1), Functions, ACR | 0–30 | | | Mostly free-tier |
| 3 Oct | 7 (cont.) | + Container Apps | 0–30 | | | Scale-to-zero |
| 4 Oct | Ext. E | App Service Plan (S1), ACI | 150–300 | | | Standard tier isn't free — delete same day |
| 5 Oct | — | Buffer / catch-up | 0 | | | No new resources |
| 6 Oct | 8 | Key Vault, secret | 0–10 | | | |
| 7 Oct | 9 | Log Analytics, alert rule | 0–20 | | | Within 5 GB/month free tier |
| 8 Oct | 10 | Recovery Services Vault, VM backup | 10–30 | | | |
| 9 Oct | Ext. F | Site Recovery — enable replication | 100–300 | | | Start early; initial replication takes real wall-clock time |
| 10 Oct | Ext. F (cont.) | Test failover, cleanup, disable replication | 100–300 | | | Disable replication same day |
| 11 Oct | — | Buffer / catch-up | 0 | | | No new resources |
| 12 Oct | 11 (part 1) | Bicep redeploy of storage account | 5–20 | | | |
| 13 Oct | 11 (cont.) | — | 0 | | | Self-check + cleanup |
| 14 Oct | 12 (part 1) | Capstone build | 300–800 | | | Depends on your architecture — see Part 2 spend bands |
| 15 Oct | 12 (cont.) | Capstone build | 300–800 | | | |
| 16 Oct | 12 (cont.) | Write-up + full teardown | 0 | | | Subscription-wide delete |
| 17 Oct | — | Buffer / final verification | 0 | | | Confirm `az group list` is empty |
| 18 Oct | — | Credit expires | 0 | | | Final check — nothing should still be running |

If any "Actual Spend" number surprises you, that's worth a question, not a shrug.

---

<a id="appendix-e"></a>
## Appendix E — Industry Best Practices: What's Real, What's Simplified

This notebook isn't a "just make it work" shortcut version of Azure — the patterns below are the actual conventions used in production environments, deliberately scaled down for a single-subscription, single-person, throwaway lab. This appendix is the honest map of which is which.

### Present and consistent across the whole notebook

- **Tagging** — every resource group gets `project=azure-learning` + `module=<lab>` tags. Appendix B maps these to their real-world equivalents (`application`/`workload`, `environment`, `costCenter`, `owner`) so you know what to add once you're tagging something that matters.
- **CAF naming convention** — Appendix B uses Microsoft's actual Cloud Adoption Framework resource-type abbreviations (`rg-`, `vnet-`, `nsg-`, `kv-`, etc.) and shows the full real-world pattern (`<type>-<workload>-<environment>-<region>-<instance>`), even though the notebook itself uses a simplified version since everything here is disposable.
- **Resource group as blast-radius/lifecycle unit** — one RG per module, deleted as a whole, is exactly how teams scope environments in practice, not just a convenience for this notebook.
- **Least-privilege, data-plane vs. control-plane RBAC** — `Storage Blob Data Contributor`/`Reader` instead of blanket Contributor, called out explicitly as a deliberate security boundary in Module 5 and reinforced in Extension D.
- **Managed identity over secrets** — the identity chain running from Module 2 through Modules 7 and 8 exists specifically to avoid connection strings or keys in code/config, which is the actual recommended pattern, not a teaching shortcut.
- **Governance controls** — Policy, resource locks, and budgets (Extension A) are real guardrail mechanisms, not lab-only constructs.
- **Layered data protection** — versioning + soft delete + backup + Site Recovery each solve a genuinely different failure mode (Extension D, Module 10, Extension F); this mirrors how production data protection is actually layered, not redundant.
- **Infrastructure as Code** — Module 11's Bicep/`what-if` workflow is the real deployment discipline, not just a CLI alternative.

### Where it's deliberately simplified — and why

- **Tag taxonomy is thin.** Just `project`/`module` vs. a real environment's `owner`, `environment`, `dataClassification`, `costCenter`. Appendix B flags this directly rather than pretending two tags is enough for production use.
- **Naming drops `<environment>`, `<region>`, and `<instance>`** from the CAF pattern, since everything here is single-instance and non-production. Reintroduce them the moment you're naming anything that will exist for more than a day or that a teammate will also need to read.
- **Management groups, Advisor, and a few other governance layers** are discussed conceptually (Part 1, Part 2) but not hands-on labbed, since they need a multi-subscription setup this notebook doesn't have.
- **Single region, single subscription throughout** (aside from Extension F's second region for Site Recovery) — real environments typically span multiple subscriptions per environment/workload with management-group-level policy inheritance, which is out of scope for a personal learning subscription.

The habit that matters most, independent of the exact tags or naming pattern you land on: **every resource should be traceable to a reason it exists and a person responsible for it.** That's what tags, naming, and RBAC scoping are all actually for — cost filtering and exam questions are just the most visible payoff.
