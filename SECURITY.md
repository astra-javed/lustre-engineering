# Security Policy

## Scope

This repository contains educational labs, infrastructure examples, and engineering notes. Do not place real credentials, private keys, production configuration, or sensitive infrastructure information in the repository.

## Reporting a vulnerability

Please report suspected security vulnerabilities privately to the repository owner rather than opening a public issue with sensitive details.

Include enough information to reproduce the issue safely, the affected path or component, and any relevant version or commit information.

## Secret exposure

If a credential, token, private key, or other secret is accidentally committed:

1. Stop using the exposed credential.
2. Revoke or rotate it immediately.
3. Remove it from the working tree and repository history as appropriate.
4. Assess whether downstream systems were exposed.

Removing a secret from the latest commit alone does not make an exposed credential safe.
