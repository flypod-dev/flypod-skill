# Security

## Reporting a vulnerability

Email **security@flypod.dev** with what you found and how to reproduce it. You
should hear back within a few working days. Please do not open a public issue
for a security problem.

The same contact is published at https://flypod.dev/.well-known/security.txt.

## Scope

This repository holds the flypod agent skill and its plugin manifests. It runs
no code of its own: the skill tells an agent to run the `flypod` CLI from npm,
and the plugin manifests point at flypod's remote MCP server
(`https://flypod.dev/mcp`). Reports about the CLI, the MCP server or the hosted
service at flypod.dev are welcome at the same address.

To report a site hosted on flypod that is phishing, malware or otherwise
abusive, use **abuse@flypod.dev** or https://flypod.dev/abuse.
