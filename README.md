# Shopify App Builder (Guided) — a Claude skill

A Claude skill that walks you from an app idea to a running, embedded Shopify app. It asks the right questions first, designs the architecture with you, and only then builds, using **only Shopify's own tools**:

- **Shopify CLI** to create, run, configure and deploy the app
- The official **React Router app template** (`@shopify/shopify-app-react-router`)
- **Polaris web components** (`s-page`, `s-section`, `s-table`, ...) for every screen
- **Admin GraphQL API** for reading and writing store data
- **Shopify App Pricing** for paid plans
- The **Shopify Dev MCP** to look up current docs and validate every query and component

The skill file is at [`.claude/skills/shopify-app-builder-guided/SKILL.md`](.claude/skills/shopify-app-builder-guided/SKILL.md).

---

## Contents

- [How it works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Install the skill](#install-the-skill)
- [Use the skill](#use-the-skill)
- [Example session](#example-session)
- [What the generated app looks like](#what-the-generated-app-looks-like)
- [Rules the skill follows](#rules-the-skill-follows)
- [Troubleshooting](#troubleshooting)
- [Customizing the skill](#customizing-the-skill)

---

## How it works

The skill runs three phases in order and stops for your approval between each one.

### Phase 1: Discovery

Claude asks five questions **one at a time**, waiting for your answer before the next. Paid apps get short follow-ups about plans, prices, trials and features.

| # | Question | What to give |
|---|----------|--------------|
| 1 | **App name** | The name shown in the Shopify admin |
| 2 | **App description** | One or two sentences: who it's for and what problem it solves |
| 3 | **App features** | A list, with the must-haves for v1 marked |
| 4 | **Billing** | Free, or paid. If paid: each plan's name, price, monthly or yearly, trial days, and which features it unlocks |
| 5 | **Basic flow** | What a merchant does from install to getting value |

Claude then sums up your answers and waits for your "yes".

### Phase 2: Architecture and account connection

Claude reads the current Shopify docs through the Dev MCP and writes an `ARCHITECTURE.md` covering:

- **Access scopes**: the minimum set, taken from what Shopify's validator reports for each planned GraphQL operation
- **Pages**: each route, which Polaris template it uses, its data and actions
- **Navigation**: the app's menu in the Shopify admin
- **Data**: Admin GraphQL queries and mutations, plus metafields, metaobjects or app tables
- **Webhooks**: including `app/uninstalled` and the mandatory privacy (GDPR) topics
- **Billing**: plans through Shopify App Pricing, plan checks and feature gating
- **Extensions**: only if a feature needs one
- **Build order**: small milestones you can test one at a time

You review and edit it. Then Claude checks that Shopify CLI is installed and asks, one at a time, whether you have a **Shopify Partner account**, whether you have a **development store**, and whether you're ready to log in.

### Phase 3: Create and build

Claude shows the exact command it will run and waits for your permission:

```bash
shopify app init --template=reactRouter --name="<Your App Name>"
```

This creates the app in your **Dev Dashboard** and a local project folder. Claude then:

1. Sets the approved scopes and webhooks in `shopify.app.toml`
2. Validates the config with `shopify app config validate --json`
3. Starts `shopify app dev` so you can open the app on your dev store
4. Builds each milestone: look up docs, write and validate the GraphQL, write the route, write and validate the Polaris UI, then tell you what to click to test it

It only runs `shopify app deploy` when you ask.

---

## Prerequisites

You need these before Phase 3. Claude checks for them and helps you fix anything missing.

| Requirement | How to get it |
|-------------|---------------|
| **Node.js** (current LTS) | [nodejs.org](https://nodejs.org) |
| **Shopify CLI** | `npm install -g @shopify/cli@latest` (update with `shopify upgrade`) |
| **Shopify Partner account** | Free at [shopify.com/partners](https://www.shopify.com/partners) |
| **Development store** | Create one in the Dev Dashboard, or run `shopify store create dev` |
| **Shopify Dev MCP** (strongly recommended) | Lets Claude search the docs and validate code. See [Shopify Dev MCP](https://shopify.dev/docs/apps/build/devmcp) |
| **Git** (optional) | For version control of the generated app |

You'll log in to Shopify through the browser when the CLI asks. **Never paste passwords, API secrets or tokens into the chat.**

---

## Install the skill

Pick the option that matches where you use Claude.

### Option A: Claude Code, in this project only

Clone this repo and open Claude Code inside it. The skill loads automatically from `.claude/skills/`.

```bash
git clone https://github.com/sunilkumarnat/shopify-app-ai-agent.git
cd shopify-app-ai-agent
claude
```

### Option B: Claude Code, in every project

Copy the skill folder into your personal skills folder:

**macOS / Linux**

```bash
git clone https://github.com/sunilkumarnat/shopify-app-ai-agent.git
mkdir -p ~/.claude/skills
cp -r shopify-app-ai-agent/.claude/skills/shopify-app-builder-guided ~/.claude/skills/
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/sunilkumarnat/shopify-app-ai-agent.git
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse shopify-app-ai-agent\.claude\skills\shopify-app-builder-guided "$HOME\.claude\skills\"
```

Restart Claude Code. Type `/skills` to confirm `shopify-app-builder-guided` is listed.

### Option C: Claude.ai or the Claude desktop app

1. Download or clone this repo.
2. Zip the `shopify-app-builder-guided` folder so the zip contains the folder with `SKILL.md` inside it:
   ```bash
   cd shopify-app-ai-agent/.claude/skills
   zip -r shopify-app-builder-guided.zip shopify-app-builder-guided
   ```
3. In Claude, open **Settings**, find the **Skills** section (under Capabilities), and upload the zip.
4. Make sure the skill is switched on.

> Skills on Claude.ai need code execution enabled on your plan. If you don't see a Skills section, check that it's turned on in your settings.

### Install the Shopify Dev MCP (recommended)

The skill expects Claude to have the Shopify Dev MCP so it can read current docs and validate code. In Claude Code:

```bash
claude mcp add shopify-dev-mcp -- npx -y @shopify/dev-mcp@latest
```

For other clients, follow [Shopify's Dev MCP setup guide](https://shopify.dev/docs/apps/build/devmcp).

---

## Use the skill

Start a new conversation and describe what you want. Any of these will trigger it:

- "Help me build a Shopify app"
- "I want to create a Shopify app for &lt;idea&gt;"
- "Use the shopify-app-builder-guided skill"

In Claude Code you can also call it directly:

```
/shopify-app-builder-guided
```

Then:

1. **Answer the questions as they come.** Claude asks one at a time. The more specific your features and flow, the better the architecture.
2. **Review `ARCHITECTURE.md`.** Change pages, scopes or plans now; it's much cheaper than later.
3. **Confirm your Partner account and dev store**, and log in when Shopify CLI opens the browser.
4. **Say "yes"** when Claude shows the `shopify app init` command.
5. **Test each milestone** on your dev store as Claude finishes it, and tell Claude what to change.
6. **Ask for a deploy** when you're ready: "deploy the app".

### Tips

- Run Claude Code in the folder where you want the app created.
- Keep `shopify app dev` running in its own terminal while you build.
- If you change scopes, reopen the app on your dev store to approve them.
- For paid apps, set up plans in the **Partner Dashboard** (Distribution → Manage listing → Pricing content). Claude will remind you.

---

## Example session

```text
You:    Help me build a Shopify app.

Claude: I'll ask you 5 quick questions, one at a time.
        Question 1 of 5: What's the name of your app?

You:    Stock Alert Pro

Claude: Nice name.
        Question 2 of 5: Describe your app in one or two sentences.

You:    Emails merchants when products run low on stock.

Claude: Question 3 of 5: What features should it have?

You:    Low-stock threshold per product, daily email digest, dashboard of low items

Claude: Question 4 of 5: Will the app be free or paid?

You:    Paid.

Claude: What plans do you want?

You:    Free (10 products) and Pro $9.99/month (unlimited, 7-day trial)

Claude: Question 5 of 5: What's the basic flow?

You:    Install → pick plan → set thresholds → see dashboard → get emails

Claude: Here's the summary... Shall I design the architecture?

You:    Yes.

Claude: [writes ARCHITECTURE.md: scopes read_products, read_inventory; pages
        Dashboard, Products, Settings; inventory_levels/update webhook; App
        Pricing plans "free" and "pro"; 5 milestones]
        Any edits? Then: do you have a Partner account and dev store?

You:    Looks good. Yes, I have both.

Claude: I'll run:
          shopify app init --template=reactRouter --name="Stock Alert Pro"
        This creates the app in your Dev Dashboard. OK to proceed?

You:    Yes.

Claude: [creates the app, configures scopes, starts dev server, builds milestone 1]
```

---

## What the generated app looks like

```text
your-app/
├── app/
│   ├── routes/
│   │   ├── app.tsx                 # layout: auth + NavMenu
│   │   ├── app._index.tsx          # home / dashboard
│   │   ├── app.<feature>.tsx       # one route per feature
│   │   ├── app.settings.tsx        # settings (if needed)
│   │   └── webhooks.*.tsx          # webhook handlers
│   ├── shopify.server.ts           # shopifyApp() config, API version
│   └── db.server.ts                # Prisma client
├── prisma/schema.prisma            # sessions + app tables
├── extensions/                     # only if the app needs extensions
├── shopify.app.toml                # scopes, webhooks, app URLs
├── ARCHITECTURE.md                 # written by the skill in Phase 2
└── package.json
```

Each feature route follows the same pattern:

```tsx
export const loader = async ({ request }) => {
  const { admin } = await authenticate.admin(request);
  const res = await admin.graphql(`#graphql query { ... }`);
  return (await res.json()).data;
};

export default function Feature() {
  const data = useLoaderData();
  return (
    <s-page heading="Feature">
      <s-section>...</s-section>
    </s-page>
  );
}
```

---

## Rules the skill follows

- **Only Shopify tools.** Shopify CLI, the React Router template, Polaris web components, Admin GraphQL and the Dev MCP. No other scaffolders, UI kits or frameworks.
- **Validate everything.** Every GraphQL operation and Polaris component is checked with the Dev MCP validator before it's written.
- **Least privilege.** Scopes come from what the validator says each operation needs.
- **Ask before side effects.** Creating the app, creating a dev store, adding dependencies and deploying all wait for your explicit "yes".
- **No secrets in chat.** Login goes through Shopify CLI's browser flow; secrets stay in the CLI-managed `.env`.
- **Modern billing.** Paid apps use Shopify App Pricing (plans in the Partner Dashboard), not hand-written Billing API charges, unless you need a one-time purchase.
- **Privacy built in.** Mandatory compliance webhooks are always included.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `shopify: command not found` | `npm install -g @shopify/cli@latest` |
| A CLI command is missing or behaves oddly | `shopify upgrade`, then `shopify version` |
| Claude doesn't use the skill | Mention it by name, or in Claude Code run `/shopify-app-builder-guided`. Check `/skills` lists it |
| Claude answers Shopify questions from memory | Install the Shopify Dev MCP (see above) |
| App doesn't load in the admin | Make sure `shopify app dev` is running and open the preview URL it prints |
| "Access denied" on a GraphQL call | The scope is missing from `shopify.app.toml`; add it and reopen the app to re-approve |
| Config errors before dev or deploy | `shopify app config validate --json` and fix what it reports |
| Plan selection page is empty | Plans need a display name and features for each listing language in the Partner Dashboard |

---

## Customizing the skill

Edit [`SKILL.md`](.claude/skills/shopify-app-builder-guided/SKILL.md) to change the behaviour, for example:

- Add or change discovery questions in **Phase 1**
- Add sections to the architecture template in **Phase 2a**
- Add your own code conventions under **Reference patterns**

Keep the frontmatter `name` and `description` lines; Claude uses the description to decide when to use the skill.

---

## Useful links

- [Shopify CLI for apps](https://shopify.dev/docs/apps/build/cli-for-apps)
- [`shopify app init` reference](https://shopify.dev/docs/api/shopify-cli/app/app-init)
- [Shopify App package for React Router](https://shopify.dev/docs/api/shopify-app-react-router)
- [Polaris web components (App Home)](https://shopify.dev/docs/api/app-home)
- [Admin GraphQL API](https://shopify.dev/docs/api/admin-graphql)
- [Shopify App Pricing](https://shopify.dev/docs/apps/launch/billing/shopify-app-pricing)
- [Shopify Dev MCP](https://shopify.dev/docs/apps/build/devmcp)
