---
title: "Using Claude Code to Build Production-Ready Apps"
description: "A practical Claude Code workflow for moving from a quick prototype to a production-ready Node.js app with GitHub, Docker, tests, security scans, and CI."
date: 2026-05-13
categories: [AI]
tags: [claude-code, ai-coding, devops, github-actions, docker, nodejs]
---

Claude Code is fast for prototypes, but production work needs structure. The trick is to guide it like a senior engineer would guide a junior developer: small prompts, clear boundaries, tests, security checks, and repeatable deployment commands.

This post captures a compact workflow for using Claude Code to build and harden a Node.js app.

## Install Claude Code

On WSL Ubuntu or Linux:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

If `~/.local/bin` is missing from `PATH`, add it:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

On Windows PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

![Claude Code home](/assets/img/posts/2026/2026-05-13-claude-code-production/claude-code-home.webp)

> Keep authentication tokens out of screenshots, notes, terminal recordings, and Git history.
{: .prompt-warning }

## Make the Prototype Maintainable

Claude may generate everything inside `public/index.html`. Before the codebase grows, split the frontend into separate files.

```text
Refactor the public/index.html file such that css styling is in its own separate file.
```

```text
Separate out the Javascript code in index.html into its own file as well.
```

Then ask Claude to capture the project conventions:

```text
Update the CLAUDE.md file.
```

Use `CLAUDE.md` as the project memory for style rules, folder layout, testing expectations, and deployment commands.

## Add GitHub Integration

Install GitHub CLI:

```bash
apt install gh
```

Initialize Git and set identity:

```bash
git init
git config user.email "you@example.com" && git config user.name "your-github-user"
```
Generate GitHub token

![github token](/assets/img/posts/2026/2026-05-13-claude-code-production/github-token.png)

GitHub Token Permission

![github token permission](/assets/img/posts/2026/2026-05-13-claude-code-production/github-token-permission.png)

Set a GitHub token in your shell profile:

```bash
export GH_TOKEN=<your-github-token>
```

Example Claude prompts:

```text
Create a new private repo named "radio-calico" for this project.
```

```text
Generate a README.md suitable for this GitHub project.
```

Install the Claude GitHub integration from Claude Code:

```text
/install github-app
```

![Claude Code GitHub install](/assets/img/posts/2026/2026-05-13-claude-code-production/cc-github-install.webp)

> Use the minimum required GitHub token permissions. Rotate the token immediately if it was pasted into a public file or screenshot.
{: .prompt-danger }

## Add Tests and Security Checks

Ask Claude to design tests before it writes broad changes:

```text
Think about how to build a unit testing framework for both the front end and the backend ratings system. Submit a PR.
```

For security scanning:

```text
Add security scanning using npm audit and add a make target to run security tests.
```

Keep common commands in `Makefile` so humans and CI run the same workflow.

## Dockerize the App

Test Docker first:

```bash
docker run hello-world
```

If Docker runs under a WSL user named `unni`, add that user to the Docker group:

```bash
usermod -aG docker unni
```

Restart WSL from PowerShell:

```powershell
wsl --shutdown
```

Prompt Claude to package the app:

```text
Package this project up into a Docker container with dev and prod releases, such that it can be deployed in a self contained manner.
```

Expected commands after Docker Compose is added:

```bash
# Development
docker compose up dev

# Production
docker compose up prod -d

# Rebuild after dependency changes
docker compose build
```

Then add Make targets:

```text
Create make targets for this project such that I can easily start prod, dev, test cases.
```

## Move Toward Production

Once the basic container works, harden the architecture:

```text
For the production deployment, modify the system such that it uses Postgresql database and nginx for the frontend web server.
```

Add CI:

```text
Integrate our unit tests and security scans into our Github Actions workflow.
```

Verify Claude reused existing Make targets:

```text
Does this new workflow incorporate the "make test" and "make security" capabilities we already had?
```

## Useful Production Prompts

| Goal               | Prompt                                                                                |
| ------------------ | ------------------------------------------------------------------------------------- |
| Performance        | `Analyze the website we have created and think about how to optimize its page speed.` |
| Architecture       | `Generate a system architecture diagram for this system in Mermaid format.`           |
| Duplication review | `Invoke the <agent-name> to fix any duplicated logic.`                                |

> Phrases like `Think deeply` can improve reasoning, but they also burn more tokens. Use them only for architecture, security, or debugging work that deserves the extra cost.
{: .prompt-tip }


## Additional Notes

Install Claude in Github App

![claude github app](/assets/img/posts/2026/2026-05-13-claude-code-production/install-github-app-claude.png)

Refer claude in github issue to resolve it. 

![claude github bot](/assets/img/posts/2026/2026-05-13-claude-code-production/claude-github-issue.png)


## Practical Takeaway

Claude Code works best when it is driven through production checkpoints: split files early, document project memory in `CLAUDE.md`, keep commands in `Makefile`, add tests, scan dependencies, containerize, and wire everything into GitHub Actions.
