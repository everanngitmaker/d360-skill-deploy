# d360-deploy

A Claude Code skill for managing Salesforce Data 360 metadata deployments across multiple environments using a branch-per-org pipeline.

## What it does

Guides you through the full deployment lifecycle:

- **Retrieve** metadata from a dev org into a feature branch
- **Promote** changes through environments in order (e.g. dev → stage → prod)
- **Deploy** to each org and handle required manual UI steps (Data Kit deploys, KQ field re-assignment)
- **Set up** a new Data 360 devops repo from scratch
- **Troubleshoot** failed deploys, `ExtDataTranFieldTemplate` errors, and manifest issues

## Usage

Invoke in Claude Code with phrases like:

- "deploy this change to stage"
- "promote feature/my-change to prod"
- "set up a new Data Cloud pipeline repo"
- "retrieve metadata from my dev org"
- "my deploy failed, help me troubleshoot"

Claude will read your `config/pipeline.config` to find your actual promotion order and org aliases before doing anything.

## Pipeline structure

```
feature/change → dev → stage → [uat →] prod
```

One git branch per environment. Scripts handle retrieve, PR creation, and deployment. Manual Data Kit deploys in the Salesforce UI are required after each metadata deploy.

## Scripts

The pipeline scripts (`1-retrieve.sh`, `2-pr.sh`, `3-deploy.sh`) live in [d360-deploy-cli-pipeline](https://github.com/everanngitmaker/d360-deploy-cli-pipeline) — use that repo to set up or copy the pipeline into your project.

| Script | Purpose |
|--------|---------|
| `1-retrieve.sh` | Retrieves metadata from an org and commits it to the feature branch |
| `2-pr.sh` | Creates a PR to the next environment, enforcing promotion order |
| `3-deploy.sh` | Deploys the current branch to a target org with pre-deploy conflict checks |

## Known platform workarounds (automated)

| Issue | Workaround |
|-------|------------|
| `ExtDataTranFieldTemplate` unsupported by Metadata API | Stripped before retrieve/deploy, restored after |
| `KQ_` fields cause deploy errors | Deleted before deploy, removed from manifests |
| Orphaned manifest members | Detected and removed automatically |

## Dependencies

This skill requires a pipeline repo based on [d360-deploy-cli-pipeline](https://github.com/everanngitmaker/d360-deploy-cli-pipeline). Your project repo must have:

- `config/pipeline.config` — defines `PROMOTION_ORDER` and `ORG_BRANCH_MAP`
- `scripts/` — contains `1-retrieve.sh`, `2-pr.sh`, `3-deploy.sh`
- `manifests/` — contains your `package.xml` manifest(s)

Clone or copy from `d360-deploy-cli-pipeline` to get started.

## Installation

This skill is part of the [my-skills](https://github.com/everanngitmaker) plugin collection. Copy the skill directory into your Claude Code skills folder:

```bash
cp -r d360-deploy ~/.claude/plugins/marketplaces/my-skills/plugins/my-skills/skills/
```
