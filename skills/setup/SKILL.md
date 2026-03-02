---
name: handlebar-setup
description: Connect an AI agent to Handlebar governance platform
---

# Handlebar Connection Skill

Connect an AI agent to the Handlebar governance platform. This skill analyzes the agent codebase and prepares the information needed to configure governance in Handlebar.

## When to Use

Use this skill when the user wants to:
- Connect an agent to Handlebar
- Set up Handlebar governance
- Onboard an agent to Handlebar

## Workflow

### Step 1: Handlebar Setup Information

**INFORM THE USER**:

> "To connect your agent to Handlebar, you'll need an account and API key:
>
> **Sign up**: https://app.gethandlebar.com  
> (Handlebar is currently operating a waitlist - if you don't have access, email contact@gethandlebar.com to request it)
>
> **Create an API key**: Org Settings > API Keys > Create API key
>
> **Set the environment variable**:
> ```bash
> export HANDLEBAR_API_KEY=hb_your_api_key_here
> # Or add to .env file
> HANDLEBAR_API_KEY=hb_your_api_key_here
> ```
>
> Don't worry if you don't have this yet - the API key can be added after we've onboarded your agent. Let's continue with the setup."

Proceed to Step 2.

### Step 2: Detect Agent Framework

Search the codebase for framework indicators:

| Framework | Detection Pattern | Package |
|-----------|-------------------|---------|
| **Vercel AI SDK v5+** | `"ai": "^5.x"` or `import { Agent } from "ai"` | `@handlebar/ai-sdk-v5` |
| **LangChain JS** | `"@langchain/core"` or `import { AgentExecutor }` | `@handlebar/langchain` |
| **LangChain Python** | `from langchain.X import Y` | `handlebar-langchain` |
| **Google ADK Python** | `from google.adk.X import Y` | `handlebar-google-adk` |
| **LlamaIndex TS** | `"llamaindex"` or `import { FunctionTool }` | Custom with `@handlebar/core` |
| **OpenAI SDK** | `"openai"` with tool calls | Custom with `@handlebar/core` |
| **Anthropic SDK** | `"@anthropic-ai/sdk"` with tool_use | Custom with `@handlebar/core` |
| **Google Gemini** | `"@google/generative-ai"` | Custom with `@handlebar/core` |
| **Custom** | Manual agent loop | `@handlebar/core` |

**Output**: Report the detected framework to the user.

### Step 3: Configure Framework on Handlebar

Based on detected framework, provide integration instructions.
Some frameworks are supported directly with Handlebar packages.
Those linked to a "core" package (`@handlebar/core` for JS and `handlebar-core` for Python)
do not yet have direct support,
however they can still be integrated by connecting to the agent's lifecycle methods.

---

First, let's learn how to connect an agent. For a Javascript, Typescript, or Python agent:

- For Vercel AI, read https://handlebar.mintlify.app/integrations/vercel-agents
- For Google ADK, read https://handlebar.mintlify.app/integrations/google-adk-agents
- For langchain (either language), read https://handlebar.mintlify.app/integrations/langchain-agents
- For custom Python, read https://handlebar.mintlify.app/integrations/custom-python-integration
- For custom JS/TS, read https://handlebar.mintlify.app/integrations/custom-javascript-integration

---

#### For Non-JavaScript/Python Agents

If the agent is built in a language other than JavaScript, TypeScript, or Python (e.g. Go, Rust, Java):

**INFORM THE USER**:

> "[Language] is not yet supported by Handlebar SDKs.
>
> Please contact the Handlebar team at contact@gethandlebar.com to let them know the agent framework you want to use. We will endeavour to support it as soon as possible.
>
> In the meantime, let's continue with the agent and rule analysis so you're ready when support is available."

Then proceed to Step 4 to complete the codebase assessment.

---

### Step 4: Assess Codebase for Agent Purpose

Analyze the agent to gather information for Handlebar configuration.

#### 4.i: Tool Analysis

For each tool in the agent, extract:

1. **Tool name**
2. **Description** - What does it do?
3. **Summary** - One-line purpose
4. **Suggested categories** from:
   - Data: `read`, `write`, `delete`
   - Sensitivity: `pii`, `phi`, `financial`, `sensitive`
   - Scope: `internal`, `external`
   - Risk: `irreversible`, `high-risk`
   - Auth: `auth`, `admin-only`, `manager-only`

**Output format**:

```
## Tool Analysis

| Tool | Summary | Categories |
|------|---------|------------|
| getUserProfile | Fetches user profile data | read, pii, internal |
| issueRefund | Processes customer refunds | write, financial, irreversible |
| sendEmail | Sends email to customer | write, external |
```

#### 4.ii: Agent Intent & Workflow

Based on the tool analysis above, determine what the agent is trying to accomplish:

**Analyze:**

1. **Primary workflow** - What business process does this agent support?
   - Look at the combination of tools and how they would be used together
   - Consider the system prompt if available
   - Example: "Patient appointment booking and management"

2. **Agent goal** - What is the agent ultimately trying to achieve for the user?
   - Example: "Help patients book, reschedule, or cancel appointments"

3. **Workflow stages** - What steps does the agent typically take?
   - Example: "1. Verify patient identity → 2. Check availability → 3. Book appointment → 4. Send confirmation"

4. **Domain** - What industry/sector does this agent operate in?
   - Healthcare, Finance, E-commerce, HR, Legal, Customer Support, etc.

**Output format**:

```
## Agent Intent & Workflow

**Domain**: Healthcare

**Primary workflow**: Patient appointment management

**Agent goal**: Help patients book, modify, and cancel appointments with their healthcare provider

**Typical workflow**:
1. Verify patient identity (lookup_patient, verify_dob)
2. Understand patient need (conversation)
3. Check availability (check_slots)
4. Book/modify/cancel appointment (book_appointment, cancel_appointment)
5. Send confirmation (send_confirmation_sms, send_confirmation_email)

**Key interactions**:
- Patient ↔ Agent: Conversational booking
- Agent ↔ Clinical system: Appointment CRUD
- Agent ↔ Patient: Notifications
```

#### 4.iii: Jurisdiction & User Impact

Look for indicators in the codebase:

**Jurisdiction signals:**
- Regulatory references: `NHS`, `HIPAA`, `GDPR`, `FCA`, `PCI-DSS`
- Domain suffixes: `.nhs.uk`, `.gov`, `.eu`
- Currency: `£` (UK), `$` (US), `€` (EU)
- Phone formats: `+44` (UK), `+1` (US)
- ID formats: NHS number, SSN, national ID patterns

**User impact signals:**
- User types: patients, customers, employees, public
- Data sensitivity: health records, financial data, personal info
- Action severity: payments, deletions, account changes

**Output format**:

```
## Jurisdiction & User Impact

**Detected jurisdiction**: UK (NHS references, £ currency, +44 phone format)

**Users impacted**: Patients

**Data sensitivity**: 
- PHI (health records)
- PII (contact details)

**Regulatory considerations**:
- UK GDPR
- NHS Data Security and Protection Toolkit
- Caldicott Principles

**High-risk actions**:
- Book/cancel appointments (affects patient care)
- Access medical records (PHI exposure)
```

If jurisdiction cannot be inferred, **ASK THE USER**:

> "I couldn't determine the jurisdiction from the codebase. Where will this agent operate?
> - UK
> - US  
> - EU
> - Other (please specify)"

### Step 5: Connect the agent code to Handlebar

According to the integration documentation you reviewed earlier, we will now connect the agent to Handlebar.

1. Connect the minimal lifecycle hooks (either using the provided wrapper in a client library, or the lifecycle hooks defined in the core packages/custom integration). Let the Handlebar client use default values where possible: do NOT write out every config argument explicitly.
2. Provide appropriate agent metadata: provide a slug to Handlebar based on the agent's purpose, and provide agent tags if possible
3. If there is a clear user id passed into the agent flow already, then configure Handlebar with that enduser/actor ID. Otherwise, let your user know that Handlebar can be configured to track endusers, and inform the user of the code change they would need to make to enable that.
4. Provide Handlebar the tool metadata according to the data you collected in previous steps.

### Final Output

Provide a summary report for Handlebar configuration and **save it to a file** for use by the rule generation skill.

**Create `.handlebar` folder within `./claude` and save to `.claude/.handlebar/agent-config.json`**:

```json
{
  "agent": {
    "slug": "[agent-slug]",
    "name": "[agent-name]",
    "framework": "[detected framework]",
    "package": "[package to install]"
  },
  "tools": [
    { "name": "toolName", "summary": "...", "categories": ["read", "pii"] }
  ],
  "intent": {
    "domain": "[healthcare/finance/etc.]",
    "workflow": "[primary workflow]",
    "goal": "[agent goal]"
  },
  "context": {
    "jurisdiction": "[UK/US/EU]",
    "users": "[who is impacted]",
    "regulations": ["regulation1", "regulation2"],
    "highRiskActions": ["action1", "action2"]
  }
}
```

**Output to user**:

```
# Handlebar Configuration Summary

## Agent
- **Framework**: [detected framework]
- **Package**: [package to install]

## Tools
| Tool | Summary | Categories |
|------|---------|------------|
| ... | ... | ... |

## Intent
- **Domain**: [domain]
- **Workflow**: [primary workflow]
- **Goal**: [agent goal]

## Context
- **Jurisdiction**: [detected/specified]
- **Users**: [who is impacted]
- **Regulations**: [applicable regulations]
- **High-risk actions**: [list]

## Next Steps
1. Install the package: `npm install [package]`
2. Add the integration code (above)
3. Run `/handlebar_rule_generation` to generate governance rules

---

Configuration saved to `.claude/.handlebar/agent-config.json`
```

**ASK THE USER**:

> "Please review the configuration above. Is this information correct?
>
> - If yes, you can proceed with `/handlebar-rule-generation` to generate governance rules
> - If anything needs to be changed, let me know and I'll update the configuration
>
> **Important**: Only proceed with rule generation once you are satisfied that the information mentioned above is accurate and complete. This configuration will be saved to `.claude/.handlebar/agent-config.json` and the generated rules will be based on it, so any inaccuracies here will affect the quality of your governance rules."
