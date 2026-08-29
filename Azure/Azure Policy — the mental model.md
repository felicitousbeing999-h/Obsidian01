#azure



Think of Azure Policy as:

> **Rules → Group rules → Assign rules → Exempt exceptions → Evaluate → Fix/Attest**

![Image](https://images.openai.com/static-rsc-4/nL1vnjoHnYw7MZopsMATop9lnBdwEGmnRH-c809iIJw5TKVAhyRd3OSZPgagKwOYcKPQQRiclPxCcc8Ft244XGJ405h1nS1lgGOTR6x7G5Fy0jMRe_TJkJH8GnO8YlLSpnVLLLcCfJbeC1msIFTP-02EbOaPj1fqZOP5uH2n5FSMjRwqViwVXKN7gMBGf5s4?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/s0pyPGk5aZ1miCq_VeYNjTFZQWB59odnF848YFt30jkdZCHnr09Sp08AvOdpMhC6YUwAVonhfc0bMF84TEVZ5YMzVFv_vt0lYXfV03nWeMVEsQzf3uJyh2S2R3WzJXaSMK5tN5oOmqqsZVX0KxXANPgW9TAneytdcv-5KB-WW17ypfqfBrZQRKqvwnx0FZQz?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/q6x-ATVJlfBlA5C2Jkg9aRx_MpUBS9NvtpeMV0gps4WWVVqZOSH_3DI_pSbRgtpmd7Nan40N0OON5t1rnwnTyFGwn9DbocYf2NIUPQ3iBTmtub_ME7DDhIxZDY8xGFzRQezQARnPKF4PDkUoAZoWhIPDznS32zzVLKpAfVrAjzirDNzEja8lDdGsoWO5MVpd?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/qyvAdsIDV9LPt4iaN7eRtf5Z5rnyDaeyIL8xcy6LttrdPKxsUJb-ax28aNX6RDxMp9kEWnL3NjgFuAwM7Hs-P6418PAzfNXDv2hvtNtYtfvkkpw-fzOz2P590RPwENKM5ubzQ4FelO7s27sDnuUVC1VmQC_cQid0LsWQCbpEZ5ogPbSQmNNNeDqSucLqz6kL?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/GFxmfza8zz0eCmO65wZCF72xjsCzzzIGAOhAPUgVf7Kx55uI4G0wJ5dyGS_ipNpLgLBWkFZCIMGbVejF3GZz9aM-uOZ9cGHTwVkCmq90npZ0zNpdIdITeUo09R-CBO68GeDkLkN_u1AVhV8jQ6xrfBfFBVzP_zoTt0Mp7Ui_XQyDwSw4T5L4AOTA3U-UZT5W?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/DUbrAIyvrCCK3erCC6d442VQyXLGdoPouVNUGqbtSvPsYuV2c_YGNr39lWZm0UPHV4iCK5l6G50eNw_H8SkBuyJHN2LmOXnX0Gdjhql-Xgk8ZW8xTPeT1fnQumTWOXeOiUGiiAVFo5cZdBqG6os1lJCR78SI5HS9Ib7czAlbWhS4wWz2rNZ1O3yrSizT-vFU?purpose=fullsize)

### 1. The hierarchy: **Where does the policy apply?**

```mermaid
flowchart TB
    T["Microsoft Entra Tenant / Root"]
    MG["Management Group"]
    SUB["Subscription"]
    RG["Resource Group"]
    R["Resource"]

    T --> MG
    MG --> SUB
    SUB --> RG
    RG --> R

    MG -. "Governance inherited ↓" .-> SUB
    SUB -. "Governance inherited ↓" .-> RG
    RG -. "Governance inherited ↓" .-> R
```

**Key idea:** Lower scopes inherit policy from higher scopes.

So if you assign a policy at a **Management Group**, subscriptions beneath it inherit that governance.

---

# 2. The six Azure Policy resources

This is the part worth memorizing:

```mermaid
flowchart LR
    PD["Policy Definition<br/>What is the rule?"]
    INIT["Initiative / Policy Set<br/>Group of rules"]
    PA["Assignment<br/>Where does it apply?"]
    EX["Exemption<br/>Where is it intentionally excluded?"]
    AT["Attestation<br/>Manually prove compliance"]
    REM["Remediation<br/>Fix non-compliant resources"]

    PD --> PA
    PD --> INIT
    INIT --> PA
    PA --> EX
    PA --> AT
    PA --> REM
```

### The six in plain English

|Resource|Think of it as|Purpose|
|---|---|---|
|**Policy Definition**|📜 Rule|Defines the condition + effect|
|**Initiative**|📚 Rulebook|Groups multiple policies|
|**Assignment**|🎯 Deployment of the rule|Determines where/how policy applies|
|**Exemption**|🚪 Exception|Intentionally excludes something|
|**Attestation**|✍️ Manual proof|Declares compliance for manual policies|
|**Remediation**|🔧 Fix|Brings non-compliant resources into compliance|

---

# 3. Policy Definition = **WHAT?**

A policy definition contains:

```text
IF resource matches condition
THEN apply effect
```

Example:

```mermaid
flowchart LR
    R["Azure Storage Account"]
    C{"HTTPS traffic<br/>enabled?"}
    D["Deny"]
    A["Allow"]

    R --> C
    C -->|"No"| D
    C -->|"Yes"| A
```

For example:

> **All Storage Accounts must require HTTPS.**

The definition describes the rule.

It does **not** by itself mean every resource in Azure is subject to it.

That's where **assignment** comes in.

---

# 4. Initiative = **WHAT + WHAT + WHAT?**

Suppose your organization wants a security baseline:

```mermaid
flowchart TB
    I["Security Baseline Initiative"]

    I --> P1["Storage must use HTTPS"]
    I --> P2["Disallow public IPs"]
    I --> P3["Require tags"]
    I --> P4["Allowed locations"]
    I --> P5["Enable diagnostic logs"]
```

Instead of assigning five policies individually:

```text
Assignment
    ↓
Security Baseline Initiative
    ├── Policy A
    ├── Policy B
    ├── Policy C
    ├── Policy D
    └── Policy E
```

You assign **one initiative**.

### Built-in vs Custom

**Built-in policy**

Microsoft provides it.

**Custom policy**

Your organization writes it because the built-ins don't satisfy your requirement.

---

# 5. Assignment = **WHERE?**

This is probably the most important distinction:

> **Definition says WHAT. Assignment says WHERE.**

```mermaid
flowchart LR
    D["Policy Definition<br/>Allowed Locations"]

    D --> A["Assignment"]
    A --> MG["Management Group"]
    A --> SUB["Subscription"]
    A --> RG["Resource Group"]
```

An assignment can also control:

- **Parameters**
    
- **Exclusions**
    
- **Resource selectors**
    
- **Overrides**
    
- **Enforcement mode**
    
- **Non-compliance messages**
    
- **Managed identity** for remediation
    

---

# 6. Inclusion vs Exclusion vs Exemption

This is where people commonly mix things up.

### Exclusion

You configure the **assignment** to not apply to something.

```mermaid
flowchart LR
    A["Policy Assignment<br/>Subscription A"]

    A --> ALL["All resources"]
    A -. "excluded" .-> RG["Legacy Resource Group"]
```

### Exemption

The resource remains within the assignment but receives a formal exemption.

```mermaid
flowchart LR
    A["Policy Assignment"]
    R["Resource"]
    E["Policy Exemption"]

    A --> R
    R --> E
```

And exemptions have two reasons:

```text
Exemption
├── Mitigated
│   └── Requirement satisfied another way
│
└── Waiver
    └── Temporary acceptance of non-compliance
```

### Architect's distinction

**Exclusion:**

> "Don't evaluate this scope."

**Exemption:**

> "This scope is governed, but we formally recognize an exception."

That's a useful governance distinction.

---

# 7. Attestation = **MANUAL COMPLIANCE**

Some compliance requirements can't simply be determined from resource properties.

For those, Azure Policy supports **manual effects**.

```mermaid
flowchart LR
    P["Manual Policy"]
    R["Target Resource"]
    A["Attestation"]
    C["Compliance State"]

    P --> R
    R --> A
    A --> C
```

Think:

> "We can't automatically prove this control. An authorized person attests that it has been satisfied."

---

# 8. Remediation = **FIX IT**

Policy detects:

```text
❌ Resource is non-compliant
```

A remediation task can bring it into compliance for policies using effects such as:

- `modify`
    
- `deployIfNotExists`
    

```mermaid
flowchart LR
    R["Resource"]
    P["Policy Evaluation"]
    NC["❌ Non-compliant"]
    REM["Remediation Task"]
    C["✅ Compliant"]

    R --> P --> NC --> REM --> C
```

For `deployIfNotExists` / `modify`, the assignment can use a **managed identity** so Azure Policy has the permissions required to perform the remediation.

---

# The entire Azure Policy lifecycle

This is the diagram I'd keep in your Obsidian notes:

```mermaid
flowchart TB
    H["Azure Governance Requirement"]

    H --> D["1. Policy Definition<br/>WHAT must be true?"]

    D --> I["2. Initiative<br/>GROUP related policies"]

    D --> A["3. Assignment"]
    I --> A

    A --> S["Scope<br/>Management Group<br/>Subscription<br/>Resource Group"]

    A --> CFG["Assignment Configuration<br/>Parameters<br/>Selectors<br/>Overrides<br/>Enforcement Mode"]

    S --> E{"Policy Evaluation"}

    E -->|"Compliant"| C["✅ Compliant"]
    E -->|"Non-compliant"| NC["❌ Non-compliant"]

    A --> EX["Exemption<br/>Mitigated / Waiver"]

    NC --> R["4. Remediation<br/>modify / deployIfNotExists"]
    R --> C

    D --> M["Manual Policy"]
    M --> AT["5. Attestation"]
    AT --> C
```

## Staff-architect mental model

If you're designing Azure governance, ask these six questions:

```text
┌─────────────────────────────────────────────┐
│              AZURE POLICY                   │
├─────────────────────────────────────────────┤
│  WHAT?       → Policy Definition             │
│  GROUP WHAT? → Initiative                    │
│  WHERE?      → Assignment + Scope            │
│  EXCEPTION?  → Exemption                     │
│  PROVE IT?   → Attestation                   │
│  FIX IT?     → Remediation                   │
└─────────────────────────────────────────────┘
```

And the **one sentence to remember**:

> **Definitions define the rule, initiatives bundle rules, assignments apply them, exemptions formalize exceptions, attestations establish manual compliance, and remediations fix violations.**

One subtle but important governance point: **Azure Policy is not merely a "deny bad resources" mechanism.** It can **audit, enforce, deploy configuration, modify resources, track compliance, and support regulatory governance** depending on the policy effect and assignment configuration.