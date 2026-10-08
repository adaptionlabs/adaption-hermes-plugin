# Adaption Plugin for Hermes Agent

Connect [Hermes Agent](https://hermes-agent.nousresearch.com) to
[Adaption](https://adaptionlabs.ai) for dataset management, fine-tuning, and
AutoScientist workflows.

The setup has two parts:

- **The Adaption MCP server** gives Hermes the tools. You add it once and sign
  in with your Adaption account in the browser.
- **This plugin** adds three skills that teach Hermes how to use those tools.
  It is an [Agent Plugins](https://agent-plugins.org) package, which Hermes
  installs natively.

## Installation

### Hermes Desktop

1. **Add the MCP server.** Open **Capabilities → MCP** and add a server:
   - Name: `adaption`
   - Type: **Streamable HTTP**
   - URL: `https://api.prod.adaptionlabs.ai/api/v1/mcp`
   - Auth: **OAuth**

   Or open this link, which pre-fills the same server for you to confirm:

   ```
   hermes://mcp/install?name=adaption&config=eyJ1cmwiOiJodHRwczovL2FwaS5wcm9kLmFkYXB0aW9ubGFicy5haS9hcGkvdjEvbWNwIiwiYXV0aCI6Im9hdXRoIn0
   ```

2. **Sign in.** Click **Authenticate** on the `adaption` server, then
   **Open in browser**. Choose the organization and click **Authorize**.
3. **Install the skills.** Open **Capabilities → Plugins → Install from Git**,
   enter `adaptionlabs/adaption-hermes-plugin`, keep the agent plugin checked,
   and confirm. Then enable **adaption** in the plugins list.

   Or open:

   ```
   hermes://plugin/install?repo=adaptionlabs/adaption-hermes-plugin&enable=1
   ```

Both links only open a confirmation dialog in Hermes Desktop; nothing is
installed until you confirm.

### Hermes CLI

#### 1. Add the Adaption MCP server

```bash
hermes mcp add adaption --url https://api.prod.adaptionlabs.ai/api/v1/mcp --auth oauth
```

Hermes opens the Adaption sign-in page. Choose the organization and click
**Authorize**. Tokens are cached under `~/.hermes/mcp-tokens/` and refreshed
automatically.

To sign in again later, for example after revoking access:

```bash
hermes mcp login adaption
```

On a remote host, follow Hermes' guide for OAuth over SSH, or use an API key
as described below.

#### 2. Install the skills

```bash
hermes plugins install adaptionlabs/adaption-hermes-plugin --no-enable
hermes plugins enable adaption
```

Start a new session, or run `/reload-mcp` in a running one, to pick up the
tools. Run `skills_list` to see the installed Adaption skills.

### Headless use with an API key

For CI or a gateway with no one to complete a browser sign-in, use an Adaption
API key instead of OAuth. Create one at
[adaptionlabs.ai/app/settings](https://adaptionlabs.ai/app/settings?tab=api_keys),
then:

```bash
hermes mcp add adaption --url https://api.prod.adaptionlabs.ai/api/v1/mcp --auth header
```

Paste the API key when Hermes asks for a bearer token. Hermes stores it in your
profile's `.env`, not in `config.yaml`.

### Why the MCP server is not bundled

Agent Plugins packages can declare MCP servers in `mcp.json`, but that format
has no way to request OAuth, and Hermes only signs in to servers configured
with `auth: oauth`. A bundled server would connect without credentials and
fail. Adding the server in Desktop or with `hermes mcp add` gives Hermes the
OAuth setting it needs.

### Local development

Hermes discovers packages in `~/.hermes/plugins/<name>/`. Copy a checkout
there to test changes before they are pushed:

```bash
git clone https://github.com/adaptionlabs/adaption-hermes-plugin.git
rsync -a --delete --exclude .git adaption-hermes-plugin/ ~/.hermes/plugins/adaption/
hermes plugins enable adaption
```

Re-run `rsync` and start a new session after every change.

## Features

### 📊 Dataset Management (11 tools)

- **Import** datasets from HuggingFace, Kaggle, or Google Sheets
- **Adapt** datasets with Adaption's processing pipeline
- **Augment** datasets with synthetic domain/general rows
- **Translate** and **localize** dataset content
- **Combine** multiple datasets
- **Export** processed results

### 🎯 Fine-tuning (5 tools)

- Browse available **base models**
- Get **hyperparameter recommendations**
- Launch **AutoScientist training runs**
- Monitor **training progress** and results

### 🔬 Invent (2 tools)

- Explore available **domains and subdomains**
- **Generate synthetic datasets** from natural language descriptions

## Available Skills

| Skill | Description |
|-------|-------------|
| `adaption-dataset` | Dataset import, processing, and transformation workflows |
| `adaption-training` | AutoScientist training run management |
| `adaption-invent` | Synthetic data generation with Invent |

## Example Usage

> "List my Adaption datasets"

> "Import this HuggingFace dataset and adapt it for fine-tuning"

> "Start fine-tuning on my adapted dataset and pick a suitable base model"

> "Generate 1000 customer service examples using Invent"

> "Check the status of my training job"

## Requirements

- Hermes Agent with Agent Plugins support (`hermes update` to get the latest)
- An Adaption account

## Support

- **Documentation**: [docs.adaptionlabs.ai](https://docs.adaptionlabs.ai)
- **Issues**: [GitHub Issues](https://github.com/adaptionlabs/adaption-hermes-plugin/issues)
- **Email**: support@adaptionlabs.ai

## License

MIT License - see [LICENSE](LICENSE) for details.
