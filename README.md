# OwAI agent skills

Reusable instructions for agents connected to [Organize with AI](https://organizewith.ai).
This package contains only instructions and synthetic call examples: no media,
credentials, application binaries, analytics, install hooks, or automatic tool calls.

## owai-media-library · 1.0.0

Search photos and videos, show previews, curate collections and groups, review
duplicates, and use approved deletion/recovery workflows through OwAI MCP.

1. Connect OwAI using the [setup guide](https://organizewith.ai/en/guides/connect-ai-assistants).
   Remote endpoint: `https://index.organizewith.ai/mcp`. Both local and remote MCP
   require Cloud Access; free desktop use does not include MCP.
2. Install with the [Skills CLI](https://github.com/vercel-labs/skills):

   ```bash
   npx skills add visheratin/owai-agent-skills --skill owai-media-library
   ```

   Choose the intended agent and installation scope in the prompts. Alternatively,
   install `skills/owai-media-library` using your client's supported mechanism;
   keep `SKILL.md` and `references/` together. A client needs Agent Skills support
   as well as MCP support; adding an MCP connection alone does not install skills.
3. Ask, for example, “Use OwAI to find five snow photos from 2024 in Holiday
   Photos,” substituting a folder you actually use. Review requested changes
   before approving them.

The skill does not grant OAuth permissions, switch processing modes, enable
remote media access, or bypass confirmations. Remote previews send the selected
preview to the assistant; they are optional and require an online owner.

## Verification

The maintainers validate every JSON example against the MCP server's advertised
wire schema and exercise representative workflows with isolated synthetic
fixtures. This is not a guarantee of an agent's behavior or a directory approval.
Tool discovery remains authoritative when server/client versions differ.

This package is maintained in [visheratin/owai-agent-skills](https://github.com/visheratin/owai-agent-skills),
separately from the private application source. GitHub publication does not imply
endorsement or approval by an MCP or skills directory.
