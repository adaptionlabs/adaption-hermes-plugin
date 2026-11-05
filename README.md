# Adaption Plugin for Hermes Agent

Connect [Hermes Agent](https://hermes-agent.nousresearch.com) to
[Adaption](https://adaptionlabs.ai) for dataset management, fine-tuning, and
AutoScientist workflows.

The plugin is an [Agent Plugins](https://agent-plugins.org) package, which
Hermes installs natively. It contains:

- **The Adaption MCP server**, which gives Hermes the tools. You sign in with
  your Adaption account in the browser the first time it starts.
- **Three skills** that teach Hermes how to use those tools.

## Installation

The plugin connects through [`mcp-remote`](https://www.npmjs.com/package/mcp-remote),
so it needs Node.js 20.18.1 or later and npm on your `PATH`.

### Hermes Desktop

Open **Capabilities → Plugins → Install from Git**, enter
`adaptionlabs/adaption-hermes-plugin`, keep the agent plugin checked, and
confirm. Then enable **adaption** in the plugins list.

Or open this link, which only opens a confirmation dialog in Hermes Desktop:

```
hermes://plugin/install?repo=adaptionlabs/adaption-hermes-plugin&enable=1
```

### Hermes CLI

```bash
hermes plugins install adaptionlabs/adaption-hermes-plugin --no-enable
hermes plugins enable adaption
```

### Sign in

Start a new session. The first time the server starts, your browser opens the
Adaption sign-in page. Choose the organization and click **Authorize**. The
token is kept and refreshed, so you sign in once.

Hermes waits about a minute for the server to start. If you take longer to
sign in, run `/reload-mcp` once you have authorized. To sign in ahead of time
instead, run:

```bash
npx -y -p mcp-remote@0.1.49 mcp-remote-client \
  'https://api.prod.adaptionlabs.ai/api/v1/mcp?source=hermes-plugin' \
  --transport http-only --header X-Adaption-Source:hermes-plugin \
  --static-oauth-client-info '{"client_id":"adaption-hermes-plugin"}'
```

Run `skills_list` to see the installed Adaption skills.

### Approvals

The plugin marks the server `untrusted`. Hermes runs the read-only tools
(listing and inspecting datasets, runs, models, and domains) without asking,
and asks before every tool that imports data or launches work: import,
adaptation, augmentation, translation, localization, combining datasets,
dataset generation, and AutoScientist runs. In cron jobs and other unattended
runs, those tools are refused.

### Headless use with an API key

For CI or a gateway with no one to complete a browser sign-in, use an Adaption
API key instead. Create one at
[adaptionlabs.ai/app/settings](https://adaptionlabs.ai/app/settings?tab=api_keys),
then:

```bash
hermes mcp add adaption --url 'https://api.prod.adaptionlabs.ai/api/v1/mcp?source=hermes-plugin' --auth header
```

Answer yes when Hermes asks whether the server requires authentication, then
paste the API key when it asks for a bearer token. Hermes stores it in your
profile's `.env` as `MCP_ADAPTION_API_KEY`, not in `config.yaml`. To skip the
prompt, set `MCP_ADAPTION_API_KEY` in that `.env` before running the command.

A server named `adaption` in `config.yaml` replaces the plugin's server, so
the skills keep working with the key. Add `trust: untrusted` to that entry to
keep the approvals described above.

### Connecting without Node.js

Hermes can also sign in to the server itself, without `mcp-remote`. Add it to
`config.yaml`, which replaces the plugin's server:

```bash
hermes config set mcp_servers.adaption.url 'https://api.prod.adaptionlabs.ai/api/v1/mcp?source=hermes-plugin'
hermes config set mcp_servers.adaption.headers.x-adaption-source hermes-plugin
hermes config set mcp_servers.adaption.auth oauth
hermes config set mcp_servers.adaption.oauth.client_id adaption-hermes-plugin
hermes mcp login adaption
```

In Hermes Desktop, open this link instead and click **Authenticate** on the
`adaption` server:

```
hermes://mcp/install?name=adaption&config=eyJ1cmwiOiJodHRwczovL2FwaS5wcm9kLmFkYXB0aW9ubGFicy5haS9hcGkvdjEvbWNwP3NvdXJjZT1oZXJtZXMtcGx1Z2luIiwiaGVhZGVycyI6eyJYLUFkYXB0aW9uLVNvdXJjZSI6Imhlcm1lcy1wbHVnaW4ifSwiYXV0aCI6Im9hdXRoIiwib2F1dGgiOnsiY2xpZW50X2lkIjoiYWRhcHRpb24taGVybWVzLXBsdWdpbiJ9fQ
```

The `oauth` block is required: Adaption does not support dynamic client
registration, which Hermes falls back to without it. If you set the server up
this way before version 0.3.0 of the plugin, it keeps working; remove it with
`hermes mcp remove adaption` to switch to the plugin's server.

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

## What this plugin does on your machine

- It runs the pinned `mcp-remote@0.1.49` through `npx`, which downloads it
  from npm on first use. On first sign-in it opens a browser and a local
  callback port, and it stores the OAuth tokens under `~/.mcp-auth/`.
- Every tool call goes to `api.prod.adaptionlabs.ai` and acts on your Adaption
  account. The tool set is served by Adaption at run time and is not covered
  by the pinned commit. Datasets you import or generate are processed by
  Adaption.
- Adaptation, augmentation, translation, localization, dataset generation, and
  AutoScientist training runs spend Adaption credits. Hermes asks before each
  of those calls (see [Approvals](#approvals)), and the skills tell Hermes to
  request a cost estimate first.
- The skills are Markdown instructions. The plugin ships no Python code, hooks,
  or background processes beyond the `mcp-remote` bridge while a session runs.

## Requirements

- Hermes Agent 0.21.6 or later (`hermes update` to get the latest)
- Node.js 20.18.1 or later and npm, unless you connect without Node.js
- An Adaption account

## Support

- **Documentation**: [docs.adaptionlabs.ai](https://docs.adaptionlabs.ai)
- **Issues**: [GitHub Issues](https://github.com/adaptionlabs/adaption-hermes-plugin/issues)
- **Email**: support@adaptionlabs.ai

## License

MIT License - see [LICENSE](LICENSE) for details.
