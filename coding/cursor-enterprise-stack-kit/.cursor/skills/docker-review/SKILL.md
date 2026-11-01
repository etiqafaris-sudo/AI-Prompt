---
name: docker-review
description: Review Dockerfiles, Compose files, and .dockerignore files for security, image size, .NET runtime compatibility, and operational gaps. Use when the user asks for a container or Docker best-practices review.
---
# Docker review

Inspect all relevant Dockerfiles, Docker Compose files, and `.dockerignore` files.

Identify:

1. Security vulnerabilities and misconfigurations.
2. Incorrect .NET runtime or operating-system compatibility.
3. Build-layer caching and image-size problems.
4. Secrets or sensitive files that could enter image layers.
5. Root execution, excessive privileges, and unnecessary network exposure.
6. Missing health checks, resource constraints, and graceful shutdown.
7. Missing image vulnerability scanning and CI/CD validation.

Classify findings by severity and confidence. Explain the evidence and recommended fix for each finding.

Do not modify files during the initial review. After presenting the findings, propose a prioritized remediation plan.
