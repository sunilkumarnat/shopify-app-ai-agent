# Shopify App AI Agent

A Claude Code skill that guides an AI assistant through building a Shopify app with the official React Router template, Polaris web components, Admin GraphQL and Shopify App Pricing, using only Shopify tools (Shopify CLI and the Shopify Dev MCP).

The skill lives at [`.claude/skills/shopify-app-builder-guided/SKILL.md`](.claude/skills/shopify-app-builder-guided/SKILL.md). Claude Code loads it automatically when you work in this repository.

## Workflow

1. **Discovery**: asks for the app name, description, features, billing plans and basic flow.
2. **Architecture**: drafts scopes, pages, data, webhooks, billing and build order for review, then asks you to connect your Shopify Partner account.
3. **Build**: asks permission, creates the app in the Dev Dashboard with `shopify app init --template=reactRouter`, and builds it milestone by milestone.
