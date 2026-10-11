---
name: shopify-app-builder-guided
description: Guide a user from idea to a running Shopify app on the React Router template, with Polaris web components, Admin GraphQL and Shopify App Pricing, asking discovery questions first and using only Shopify tools.
---

# Shopify app builder (guided)

Build an embedded Shopify app with the official **React Router app template**, **Polaris web components** (`s-*`), **Admin GraphQL**, and **Shopify App Pricing**. Always run the three phases in order. Never skip a gate.

## Hard rule: only Shopify tools

- Scaffold, run, configure and deploy with **Shopify CLI** (`shopify app ...`). Never use another scaffolder or framework.
- Look things up with the **Shopify Dev MCP** (`learn_shopify_api`, `search_docs_chunks`). Do not answer Shopify questions from memory.
- Validate every GraphQL operation and every Polaris component with the Dev MCP `validate` tool before writing it to disk. Fix and re-validate until clean.
- UI is **Polaris web components only** (`s-page`, `s-section`, `s-button`, ...). Never import `@shopify/polaris`, `@shopify/polaris-react`, Tailwind, MUI, or any other UI kit.
- Use only the packages the template ships with (`@shopify/shopify-app-react-router`, `@shopify/app-bridge-react`, Prisma session storage). Ask before adding any other dependency.

---

## Phase 1: Discovery (one question at a time)

Ask the questions below **one at a time**. Send exactly one question per message, then stop and wait for the answer before asking the next. Never list several questions in one message. Do not write code or run commands until all five are answered.

Start with a one-line intro ("I'll ask you 5 quick questions, one at a time."), then:

1. **What's the name of your app?**
2. **Describe your app in one or two sentences.** Who is it for, and what problem does it solve?
3. **What features should it have?** List them, and mark which ones are must-have for the first version.
4. **Will the app be free or paid?**
   - If **free**, move on to question 5.
   - If **paid**, ask these follow-ups one at a time:
     a. What plans do you want? (name of each plan)
     b. For each plan: price, and monthly or yearly?
     c. Do you want a free trial? How many days?
     d. Which features does each plan unlock?
5. **What's the basic flow?** What does a merchant do from installing the app to getting value from it? (for example: install, pick a plan, set up settings, see results on the dashboard)

Rules while asking:
- Show the question number and total, like **"Question 2 of 5"**.
- Briefly acknowledge each answer (one short line) before the next question.
- If an answer is vague, ask one short follow-up about that answer only, then continue.
- If the user answers several questions at once, accept them and skip to the next unanswered one.

After the last answer, show a short summary of all five answers and ask: **"Is this correct? Shall I design the architecture?"** Wait for a "yes" before Phase 2.

---

## Phase 2: Architecture, then connect the Partner account

### 2a. Write the architecture

Before writing it, call `learn_shopify_api` for `polaris-app-home`, `admin`, and (if paid) `app-pricing`, and search docs for anything feature-specific. Then produce `ARCHITECTURE.md` with these sections:

1. **Overview**: name, description, target merchant.
2. **Access scopes**: the minimum list. Derive it from the `Required scopes:` line that `validate` prints for each planned GraphQL operation. Never request broad scopes "just in case".
3. **Pages (routes)**: one row per page: route file, Polaris template (Homepage, Index, Details, Settings), main components, loader data, actions.
   - `app/routes/app._index.tsx` home / dashboard
   - `app/routes/app.<feature>.tsx` per feature
   - `app/routes/app.settings.tsx` if there are settings
4. **Navigation**: the `<NavMenu>` links in `app/routes/app.tsx`.
5. **Data**:
   - Shopify data read and written through Admin GraphQL (list each query and mutation by name).
   - App-owned data: prefer **metafields / metaobjects** on Shopify; use Prisma tables only for data that must not live on the shop. Every app table carries a `shop` column, and every query filters by it.
6. **Webhooks**: declared in `shopify.app.toml` (app-specific subscriptions), including `app/uninstalled` and the mandatory compliance topics (`customers/data_request`, `customers/redact`, `shop/redact`).
7. **Billing** (if paid): use **Shopify App Pricing**. Plans, prices and trials are configured in the Partner Dashboard, not in code. The app:
   - checks the merchant's subscription with the Partner API `activeSubscription` query,
   - redirects merchants without a plan to the Shopify-hosted plan selection page (`https://admin.shopify.com/store/{store_handle}/charges/{app_handle}/pricing_plans`) using the framework's `redirect` from `authenticate.admin`, so it leaves the iframe,
   - gates features per plan handle,
   - reports usage with the App Events API for usage-based plans.
   Do **not** use `appSubscriptionCreate`, `billing.request`, or a `billing` config block unless the user explicitly needs Manual Pricing (for example a one-time purchase).
8. **Extensions** (only if a feature needs one): admin action/block, theme app extension, Function, etc. Generate with `shopify app generate extension`.
9. **Build order**: a numbered list of small milestones, each one testable on a dev store.

Show the architecture and ask for edits. Do not continue until the user approves it.

### 2b. Connect the Shopify account

Ask these **one at a time**, waiting for each answer:

1. **"Do you have a Shopify Partner account?"** If not, point them to https://www.shopify.com/partners to create one (free), and wait until they have it.
2. **"Do you have a development store to test the app on?"** If not, offer to create one with `shopify store create dev` (show the command and wait for a yes).
3. **"Are you ready to log in to Shopify? The CLI will open your browser."**

- Check the toolchain first: `shopify version` (if missing: `npm install -g @shopify/cli@latest`; if old: `shopify upgrade`) and `node -v`.
- Login happens through Shopify CLI's browser flow the first time a command needs it. Never ask for passwords, API secrets or tokens in chat.
- Optionally ask for the **organization ID** (from the Dev Dashboard URL) so `app init` can run without prompts.

---

## Phase 3: Permission, create the app, build

### 3a. Ask permission

Show exactly what will happen and wait for an explicit "yes":

```bash
shopify app init --template=reactRouter --name="<App name>" --path=<parent folder>
# adds --organization-id=<id> when known
```

Side effects: creates a new app in the user's Dev Dashboard organization and a local project folder.

### 3b. Create and configure

1. Run `shopify app init` as shown. Pick the TypeScript flavor if asked.
2. `cd` into the project. Put the approved scopes in `shopify.app.toml` under `[access_scopes]`, and add the webhook subscriptions from the architecture.
3. Run `shopify app config validate --json` and fix any errors.
4. Run `shopify app dev` (ask which dev store if prompted) and have the user open the preview URL to confirm the template loads.

### 3c. Build milestone by milestone

For each milestone in the build order:

1. Look up the API with `search_docs_chunks`.
2. Write the GraphQL operation, run `validate` (`api: "admin"`), and note its required scopes. If a new scope appears, add it to `shopify.app.toml` and tell the user they'll need to re-approve.
3. Write the route: `loader` and `action` call `authenticate.admin(request)` and use `admin.graphql(...)`. Always read `userErrors` from mutations and return them to the UI.
4. Write the UI with Polaris web components and run `validate` (`api: "polaris-app-home"`).
5. Test on the dev store via `shopify app dev`. Report what the user should click to verify.

Only run `shopify app deploy` when the user asks, after showing what will be released.

---

## Reference patterns (validated)

### Route with loader, action and Admin GraphQL

```tsx
// app/routes/app.products.tsx
import type { ActionFunctionArgs, LoaderFunctionArgs } from "react-router";
import { useFetcher, useLoaderData } from "react-router";
import { authenticate } from "../shopify.server";

export const loader = async ({ request }: LoaderFunctionArgs) => {
  const { admin } = await authenticate.admin(request);
  const res = await admin.graphql(
    `#graphql
    query AppProducts($first: Int!, $after: String) {
      products(first: $first, after: $after, sortKey: UPDATED_AT, reverse: true) {
        nodes { id title status }
        pageInfo { hasNextPage endCursor }
      }
    }`,
    { variables: { first: 25 } },
  );
  const { data } = await res.json();
  return { products: data.products.nodes, pageInfo: data.products.pageInfo };
};

export const action = async ({ request }: ActionFunctionArgs) => {
  const { admin } = await authenticate.admin(request);
  const res = await admin.graphql(
    `#graphql
    mutation AppCreateProduct($product: ProductCreateInput!) {
      productCreate(product: $product) {
        product { id title status }
        userErrors { field message }
      }
    }`,
    { variables: { product: { title: "New product" } } },
  );
  const { data } = await res.json();
  return { errors: data.productCreate.userErrors, product: data.productCreate.product };
};

export default function Products() {
  const { products } = useLoaderData<typeof loader>();
  const fetcher = useFetcher<typeof action>();
  const busy = fetcher.state !== "idle";

  return (
    <s-page heading="Products">
      <s-button
        slot="primary-action"
        variant="primary"
        loading={busy}
        onClick={() => fetcher.submit({ intent: "create" }, { method: "post" })}
      >
        Create product
      </s-button>
      <s-section padding="none">
        <s-table>
          <s-table-header-row>
            <s-table-header listSlot="primary">Title</s-table-header>
            <s-table-header listSlot="inline">Status</s-table-header>
          </s-table-header-row>
          <s-table-body>
            {products.map((p) => (
              <s-table-row key={p.id}>
                <s-table-cell>{p.title}</s-table-cell>
                <s-table-cell>
                  <s-badge tone={p.status === "ACTIVE" ? "success" : "neutral"}>{p.status}</s-badge>
                </s-table-cell>
              </s-table-row>
            ))}
          </s-table-body>
        </s-table>
      </s-section>
    </s-page>
  );
}
```

Required scopes for this example: `read_products`, `write_products`.

### Navigation (`app/routes/app.tsx`)

Add each page to `<NavMenu>` as `<Link to="/app/<page>">Label</Link>`. Keep `rel="home"` on the home link. Do not remove the template's `ErrorBoundary` and `headers` exports.

### Webhook route

Webhook routes live outside the `app.` layout, for example `app/routes/webhooks.app.uninstalled.tsx`, and call `authenticate.webhook(request)`. Keep handlers idempotent and respond fast.

## Polaris web component rules

- No imports: `s-*` elements are global. App Bridge comes from `@shopify/app-bridge-react` (`useAppBridge`, `NavMenu`, `TitleBar`).
- Props are camelCase (`alignItems`, `gridTemplateColumns`).
- Boolean props can be shorthand (`disabled`, `loading`); string props never can (`padding="base"`, not `padding`).
- Put a form field next to a button with `s-grid gridTemplateColumns="1fr auto"`, not an inline `s-stack`.
- Start pages from the Polaris App Home templates (Homepage, Index, Details, Settings) and compositions (Setup guide, Empty state, Index table, Metrics card).
- Use `shopify.toast.show(...)` for success messages and `s-banner tone="critical"` for errors.

## Pitfalls

- Never call the Admin API from the browser. Go through a route `loader`/`action`, or send `shopify.idToken()` as a Bearer token to your own backend route.
- Admin GraphQL is paginated: use `pageInfo.endCursor` and `after`, never a huge `first`.
- Handle `userErrors` on every mutation; a 200 response can still be a failure.
- Scope changes need the merchant to re-approve; batch them.
- Use the API version the template pins in `shopify.server.ts`, and check the Dev MCP for changes when bumping it.
- Don't put secrets in code or chat; they live in the CLI-managed `.env`.
