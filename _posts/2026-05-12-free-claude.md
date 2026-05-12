---
title: "Run Claude Code with OpenRouter Free Models"
description: "Configure Claude Code to use OpenRouter as the Anthropic-compatible API endpoint, including setup steps, model configuration, common pitfalls, and debugging tips."
date: 2026-05-12
categories: [AI]
tags: [claude-code, openrouter, ai-coding, devops, api-configuration]
---

Claude Code is useful for terminal-based code editing, but the default API path is not always the cheapest option for experiments. OpenRouter provides an Anthropic-compatible endpoint, which makes it possible to point Claude Code at OpenRouter and use the `openrouter/free` model.

This is a practical setup for engineers who want low-cost AI-assisted coding without running a self-hosted LLM.


> Free model availability, limits, and quality can change on the provider side. Treat this as a development setup, not a production dependency.
{: .prompt-info }

## Problem

Self-hosted coding models often struggle with real code editing workflows. In practice, Claude Code with OpenRouter gives a simpler path:

| Option                   | Operational Notes                                                        |
| ------------------------ | ------------------------------------------------------------------------ |
| Self-hosted LLM          | More control, but weaker coding quality and more infrastructure overhead |
| Claude Code + OpenRouter | Easier setup, API-based, better fit for local coding workflows           |

## Step 1: Sign Out of Claude Code

Start by logging out from the current Claude Code session.

```bash
/logout
```

## Step 2: Create an OpenRouter API Key

Create an OpenRouter account and generate an API key from the dashboard.

![OpenRouter API key page](/assets/img/posts/2026-05-12-free-claude/openrouter-apikey.png)

OpenRouter reference:

```text
https://openrouter.ai/docs/guides/coding-agents/claude-code-integration
```

## Step 3: Update Claude Code Settings

Open the Claude Code settings file from your home directory.

```bash
cd ~
vim .claude/settings.json
```

Add the OpenRouter endpoint and API token.

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api",
    "ANTHROPIC_AUTH_TOKEN": "<your-openrouter-api-key>",
    "ANTHROPIC_API_KEY": ""
  }
}
```

## Step 4: Configure the Free OpenRouter Model

Set `ANTHROPIC_MODEL` to `openrouter/free`.

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api",
    "ANTHROPIC_AUTH_TOKEN": "<your-openrouter-api-key>",
    "ANTHROPIC_API_KEY": "",
    "ANTHROPIC_MODEL": "openrouter/free"
  }
}
```

> Do not commit `.claude/settings.json` or publish your OpenRouter API key in a blog post, screenshot, GitHub repository, or terminal recording.
{: .prompt-warning }
