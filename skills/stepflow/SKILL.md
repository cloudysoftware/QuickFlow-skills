---
name: stepflow
description: This skill should be used when asked to create, generate, or design a business process stepflow, process map, or workflow as a JSON file. Triggers on requests like "create a stepflow for X", "generate a process flow", "map out the X process", "build a flow for importing into StepFlow/QuickFlow", "document the onboarding process", or "map the procurement workflow". Also applies when the user mentions process mapping, workflow design, role-to-step mapping, or wants to document how a business process works, even if they do not say "stepflow" explicitly.
---

# Generate StepFlow

Generate complete V2 StepFlow JSON files ready for import into the QuickFlow app by Cloudy Software Limited. Works for any business domain: HR, IT, finance, operations, procurement, legal, health and safety, compliance, customer service, and more. All generated text uses UK English spelling (e.g. organisation, colour, behaviour, authorise, licence, catalogue, centre, programme) unless the user specifies another language. Never use emdash characters or emojis in generated text.

## Workflow

### 1. UNDERSTAND

Parse the user's request for: process domain, any specific roles/stages/steps mentioned, desired level of detail, and whether the user's organisation has specific terminology or systems they want referenced.

### 2. RESEARCH

Use web search to build domain knowledge before generating. This serves two purposes: producing an accurate, industry-standard process, and discovering real URLs to include as ActionLinks on relevant actions.

**What to search for:**
- Industry-standard process steps and best practices for the domain (e.g. "UK employee onboarding process steps", "ITIL change management workflow")
- Regulatory or compliance bodies relevant to the process (e.g. ACAS, ICO, HSE, HMRC, FCA)
- Government portals, official forms, and guidance pages that process participants would actually use
- Well-known software platforms commonly used in the domain (e.g. ServiceNow for ITSM, Workday for HR, Xero for finance)

**Collect URLs as you research.** When you find a page that a person performing a specific action would need to visit, note it for inclusion as an ActionLink. Prefer official .gov.uk, professional body, and established vendor URLs. Never guess at domain-specific compliance or regulatory steps; ask the user.

### 3. PLAN THE FLOW

Before writing JSON, outline the flow structure in your working context. This planning step prevents reference errors and produces better-connected flows:

1. **List roles** with their types - decide every role, its `roleType`, and short `key` upfront
2. **List lanes** in process order - each lane represents a stage
3. **For each lane, list steps** - note which roles are involved and what actions each step contains
4. **Mark which actions get links** - assign URLs collected during research to the actions where they add genuine value

This plan does not need to be shown to the user; it is working context to ensure the JSON is internally consistent.

### 4. GENERATE JSON

Produce a complete V2 JSON file following the schema below. Save to `docs/flows/<slug>.json` by default (user can specify a different path). Derive the slug from the flow title: lowercase, replace non-alphanumeric characters with hyphens, trim leading/trailing hyphens, truncate to 40 characters.

### 5. SELF-CHECK

Before writing the file, verify these constraints against the planned structure. This catches the most common generation errors:

- [ ] Every `roleId` used in `roleAssignments` and `actions.roleIds` exists in the top-level `roles` array
- [ ] Every step has at least one role assignment
- [ ] All IDs are unique (no two roles, lanes, steps, or actions share an ID)
- [ ] `miniMap.stages` has exactly one entry per lane, in the same order, with matching `label` (lane title) and `bg` (lane colour)
- [ ] Every `link` has both `label` and `url`; no partial links; `link` property omitted entirely where not needed
- [ ] `content.title` matches the top-level `title`
- [ ] `emails` is `{}`
- [ ] `meta.schemaVersion` is the literal number `2`
- [ ] Role `sortOrder` values are sequential starting from 0
- [ ] Every role in `roles` is referenced by at least one step's `roleAssignments` or action's `roleIds` (no orphaned roles)
- [ ] Every `roleId` in an action's `roleIds` also appears in that step's `roleAssignments` (actions are performed by roles assigned to the step)
- [ ] UK English spelling in all text content

### 6. PRESENT SUMMARY

Show the user: role count and names (with types), lane/stage overview, step and action counts, number of action links included, and file path. If any roles have `roleType: "process"`, note that the user can link them to other flows in the app after import.

### 7. OFFER REFINEMENT

Ask if they want to adjust roles, lanes, steps, actions, links, notes, or descriptions.

## V2 Export Schema

```
V2ExportFile {
  meta: {
    schemaVersion: 2              // CONSTANT: always the number 2
    appName: "QuickFlow"          // CONSTANT: always "QuickFlow"
    exportedAt: string            // ISO 8601 timestamp; use current date/time at generation
  }
  flowType: "stepflow"            // CONSTANT: always "stepflow"
  title: string                   // flow title
  description?: string            // one to two sentences summarising the end-to-end process
  roles: V2ExportRole[]           // all roles referenced in the flow
  content: FlowContent            // the full flow structure
}

V2ExportRole {
  id: string                      // "role_<8hex>" e.g. "role_a1b2c3d4"
  key: string                     // lowercase alphanumeric + underscores, unique within file
  label: string                   // display name e.g. "Hiring Manager"
  description?: string            // what this role does in this process
  roleType: RoleType              // see Role Types below
  sortOrder: number               // 0-based, sequential
  style: { bg: string, fg: string }  // hex colours from palette
}

FlowContent {
  id: string                      // full UUID e.g. "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
  title: string                   // must match top-level title
  emails: {}                      // ALWAYS empty object
  miniMap: {
    title: string                 // "Process Overview" or similar
    stages: [                     // one entry per lane, same order as lanes array
      { label: string, bg: string }  // label = lane title, bg = lane colour
    ]
  }
  lanes: FlowLane[]
}

FlowLane {
  id: string                      // "lane_<8hex>"
  title: string                   // stage name
  laneNote: string                // "" if no note needed
  color: string                   // hex colour from palette
  steps: FlowStep[]
}

FlowStep {
  id: string                      // "step_<8hex>"
  title: string                   // step name
  description: string             // what happens (never repeat the title)
  stepNote: string                // "" unless genuinely worth flagging
  roleAssignments: [              // at least one required
    {
      roleId: string              // must exist in roles array
      note: string                // why/how this role is involved in this step
    }
  ]
  actions: [                      // most steps should have at least one
    {
      id: string                  // "action_<8hex>"
      label: string               // action name
      notes: string               // actionable detail, or "" if label is self-explanatory
      roleIds: string[]           // roles performing this action
      link?: ActionLink           // optional; omit property entirely if no link
    }
  ]
}

ActionLink {
  label: string                   // display text for the link
  url: string                     // valid URL; both fields required together
}

RoleType = "person" | "team" | "organisation" | "process" | "alert" | "system" | "none"
```

## Role Types

Classify every role using the most specific type that applies. Getting this right matters because QuickFlow renders a different icon for each type, helping users visually scan who does what.

| roleType | When to use | Examples |
|----------|------------|----------|
| `person` | An individual role or job title | Manager, Director, Employee, Applicant, Candidate |
| `team` | A group, department, or committee | HR Team, Finance Department, IT Support, Recruitment Panel |
| `organisation` | An external organisation, body, or agency | HMRC, Companies House, DBS, a client company, a supplier |
| `process` | References another process or sub-flow | Onboarding Process, Payroll Run, Background Check |
| `alert` | A notification, automated email, or trigger | Email Notification, SMS Alert, Reminder Trigger |
| `system` | Software system, platform, or tool | ATS, HRIS, CRM, ERP, SharePoint, ServiceNow |
| `none` | Does not clearly fit any of the above | Use sparingly; prefer a specific type |

**Classification tips:**
- "HR" alone is ambiguous. If it means the HR department acting as a group, use `team`. If it means a specific HR role like "HR Business Partner", use `person`.
- Government bodies (HMRC, ICO, HSE) are `organisation` even though they are also "systems" in some sense. Reserve `system` for software.
- When a step says "the system sends an email", create separate roles: one `system` role for the platform and one `alert` role for the notification itself. This gives the user clear visibility of automated touchpoints.

## Action Links

Links turn actions into launchpads. When someone reads the flow in QuickFlow, a linked action shows a clickable URL they can follow to complete the work. This is one of the most valuable features of a well-crafted stepflow.

**When to include links:**
- Government and regulatory portals (e.g. https://www.gov.uk/government/organisations/hm-revenue-customs)
- Official guidance and policy documents (e.g. ACAS early conciliation, ICO data protection guidance)
- External forms, applications, or submission portals
- Well-known SaaS platforms commonly used in the domain
- Internal templates, handbooks, or policy pages (use placeholder URLs like `https://intranet.example.com/hr/policies/grievance` and note in the action's `notes` field that the URL should be updated)

**When NOT to include links:**
- Generic actions where no specific resource exists ("Review and approve" does not need a link)
- When the URL would be speculative or likely wrong
- On every action indiscriminately; aim for roughly 20-40% of actions in a typical flow to have links, concentrated on the actions where a resource genuinely helps

**When web search was used during research**, prefer real URLs discovered during that search over placeholder URLs. Real links to .gov.uk pages, professional body resources, and vendor documentation make the flow immediately useful after import.

## Content Quality

The difference between a useful stepflow and a generic one is in the quality of descriptions, notes, and action detail. Follow these principles:

**Step descriptions** should explain the substance of what happens, not just restate the title. Bad: "The manager reviews the request." Good: "Line manager assesses the business justification, checks team budget availability, and verifies the request aligns with approved equipment standards."

**Role assignment notes** should explain why that role is involved in that specific step, not just that they are. Bad: "Performs this step." Good: "Validates that the candidate's qualifications meet the regulated minimum standards for the role."

**Action notes** should give enough detail that someone unfamiliar with the process could follow them. Include specific documents to prepare, criteria to check, or decisions to make. Use `""` only when the action label is truly self-explanatory (e.g. "Sign the contract").

**Step notes** are for genuine caveats, regional variations, or exceptions. "This step is required by UK employment law" or "Timescales vary depending on DBS check type" are good step notes. Do not add a note to every step.

**Lane notes** flag stage-level context. "All steps in this stage must be completed within 5 working days of the offer being accepted" is a good lane note. Most lanes need no note.

## Colour Palette

### Primary role/lane colours (use first, cycling by index)

| Index | bg      | fg      |
|-------|---------|---------|
| 0     | #ffe0b3 | #8a5a00 |
| 1     | #d6eaff | #004a80 |
| 2     | #e8ffd6 | #346600 |
| 3     | #ccf2f4 | #005f60 |
| 4     | #f9d6ff | #6b0080 |
| 5     | #fff4cc | #7a5c00 |
| 6     | #ffd6d6 | #800000 |
| 7     | #e0e0e0 | #333333 |

If more than 8 roles are needed, generate additional pairs: pastel background with dark high-contrast foreground. Never repeat a `bg` colour already assigned.

Roles and lanes draw from the same palette independently by their respective indices (role `sortOrder` for roles, lane position for lanes). They may share colours. Each lane's `color` value is the `bg` hex from the palette at the same index as the lane's position (0-based), cycling back to index 0 if more than 8 lanes.

## ID Formats

| Entity  | Format | Example |
|---------|--------|---------|
| Role    | `role_<8hex>` | `role_a1b2c3d4` |
| Lane    | `lane_<8hex>` | `lane_e5f6a7b8` |
| Step    | `step_<8hex>` | `step_c9d0e1f2` |
| Action  | `action_<8hex>` | `action_3a4b5c6d` |
| Content | Full UUID | `f47ac10b-58cc-4372-a567-0e02b2c3d479` |

Generate hex characters randomly. Every ID must be unique within the file. If you notice a collision, generate a completely new random hex; never append `_2` or other suffixes.

## Sizing Guidance

- Typical: 4-8 lanes, 2-5 steps per lane, 12-25 total steps, 4-8 roles
- Complex: 10-15+ lanes, 40+ steps, 10-15+ roles
- If a process exceeds roughly 50 steps or 15 lanes, suggest breaking into sub-process flows as separate JSON files. Use `process`-type roles to mark connection points between them.

## Example

For a complete working example demonstrating `person`, `team`, `organisation`, `system`, and `alert` role types, ActionLink usage, and all structural conventions, read `examples/it-equipment-request.json`.
