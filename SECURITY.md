# Security policy

## Reporting a vulnerability

Please report security problems privately, not in a public issue or pull request.

Use GitHub's private reporting for this repository: open the **Security** tab and choose **Report a vulnerability**, or go straight to
https://github.com/tomevault-io/lint-action/security/advisories/new

Include what you found, how to reproduce it, and what you think the impact is. We will acknowledge the report, keep you updated while we investigate, and credit you in the fix unless you ask us not to.

## Scope

This repository is the TomeVault lint GitHub Action. In scope: anything in `action.yml` or its documented usage that could run unintended code on a runner, leak a secret, or let an unsafe instruction file pass the gate.

If the problem is in the TomeVault service at tomevault.io rather than in this repository, report it the same way here and say so.

## Supported versions

Only the latest release is supported. The moving `v1` tag always points at it. For the strictest supply-chain posture, pin the action to a full commit SHA instead.
