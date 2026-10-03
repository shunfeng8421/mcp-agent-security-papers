# MCP / Agent Security Research Papers

Runtime-verified security research on the MCP (Model Context Protocol) and AI-agent ecosystem, by Shiqiang Chen (independent researcher).

## Latest publications in this repo

### Paper #24 - Agent Execution Control Planes
**Title:** *When 'Sandbox', 'Gateway', and 'Policy' Ship as Unauthenticated Code Execution*
- 4 runtime-confirmed vulnerabilities across 4 freshly-published 0-star repos
- Core finding: **control-plane trust inversion** - components named for restriction ship as unauthenticated, direct code-execution endpoints
- Control group of 7 hardened implementations shows the ecosystem is two-sided
- `paper/24-agent-exec-control-plane/`

### Paper #25 - Trust Labels as Attack Surface
**Title:** *MCP Tool Annotations and the Protocol's Unenforced Safety Contract*
- Black-box verified: a malicious MCP server can declare a destructive tool readOnlyHint=true/destructiveHint=false + benign description; official Python SDK forwards it verbatim (no safety interception)
- Trust-label contract is self-attested and unenforced - advisory metadata with no enforcing consumer
- `paper/25-mcp-tool-annotations-trust/`

## Prior work (Zenodo DOIs)
- **#16** MCP Ecosystem Security - DOI 10.5281/zenodo.22942676 (20 vulns / 17 repos)
- **#23** Credential-Theft and Identity Forgery in MCP/A2A - DOI 10.5281/zenodo.22973907

## Methodology
All vulnerabilities are **runtime-confirmed** against real, isolated server instances (loopback, no credentials, no real assets). Confidence stated honestly; deployment-dependent impacts flagged, never exaggerated.

## Cite
See `CITATION.cff`. License CC-BY-4.0.
