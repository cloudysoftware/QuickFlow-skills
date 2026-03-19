# QuickFlow Skills

Claude Code and Cowork plugin for generating QuickFlow process flows by Cloudy Software Limited.

## Skills

### stepflow

Generates complete StepFlow JSON files ready for import into QuickFlow. Covers any business domain (HR, IT, finance, operations, procurement, etc.) with correct role type classification, optional action links with real URLs discovered via web search, and UK English text.

## Installation

### Claude Code

Add the marketplace, then install the plugin:

```
/plugin marketplace add cloudysoftware/quickflow-skills
/plugin install quickflow-skills@quickflow-skills
/reload-plugins
```

### Claude Cowork

Open Cowork and navigate to **Plugins** in the sidebar. Click **+**, then add the marketplace `cloudysoftware/quickflow-skills`. Browse to **quickflow-skills** and click **Add plugin**.

Alternatively, from any Cowork or Claude Code session:

```
/plugin marketplace add cloudysoftware/quickflow-skills
/plugin install quickflow-skills@quickflow-skills
/reload-plugins
```

### Team setup

To make this plugin available to all team members automatically, add it to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "quickflow-skills": {
      "source": {
        "source": "github",
        "repo": "cloudysoftware/quickflow-skills"
      }
    }
  },
  "enabledPlugins": {
    "quickflow-skills@quickflow-skills": true
  }
}
```

### Local development

```
/plugin marketplace add ./path/to/quickflow-skills
/plugin install quickflow-skills@quickflow-skills
/reload-plugins
```

## Usage

Ask Claude to create a stepflow:

- "Create a stepflow for employee onboarding"
- "Generate a process flow for invoice approval"
- "Map out the recruitment process"
- "Build a flow for IT change management"
- "Document the grievance handling process"
- "Map the procurement workflow"

The skill uses web search to discover real URLs for action links when building flows for specific domains.
