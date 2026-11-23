# Weekly Brief, 2026-11-23

_Test run. Saved manually because that run had no GitHub access. Uses the older single-solution format._

**Winner: ToolGate, a local proxy that auto-approves read-only MCP tool calls and sends risky ones to your phone.**

**Evidence limits:** Search returned titles and links only, not post text. Pages were read through a fetch helper. Reddit and X were not reached directly. Several quotes come from authors promoting their own tools. Evidence is moderate, not strong.

## 1. Problem
Developers using AI agents with MCP tools get asked to approve nearly every call, even read-only ones. The built-in allowlists often don't work. Overnight or unattended agents just stall. The only alternatives are "approve everything" or "run everything".

Quotes:
- "Every MCP tool call prompts for approval on every invocation, forever." and "There is no working configuration short of Run Everything." (Mike_Hitchcock, Cursor forum, Aug 26, 2026) [link](https://forum.cursor.com/t/mcp-allowlist-entries-are-written-but-never-matched-every-mcp-call-re-prompts/169566)
- "Every time the agent calls an MCP tool, it still asks me to approve, even though the tool is explicitly allowlisted." (Lai_Jinyi, Cursor forum, Jun 29, 2026) [link](https://forum.cursor.com/t/mcp-tools-in-permissions-allowlist-are-still-prompting-for-approval-every-time/164296)
- "A single research task can trigger dozens of permission prompts for tools that literally cannot modify anything." (Claude Code issue thread, undated) [link](https://github.com/anthropics/claude-code/issues/25966)

Also: "Your agent is either off or full admin. There is nothing in between." (Marco Arras, dev.to, a pitch for his own gateway) [link](https://dev.to/marras0914/mcp-has-no-security-model-heres-how-to-fix-it-in-2-minutes-5f7o)

**Who hurts:** anyone running Claude Code, Cursor or similar agents with several MCP servers, especially on long or unattended runs. It happens every session, and the severity is moderate to high.

## 2. Why current tools fail
- Each client's own allowlist is per-app, buggy, and can be bypassed with a local config change.
- MCP gateways exist, for example Cordon, but only one self-promotional post was seen. Features and adoption are unverified.
- Most of what was found has nothing to do with phone-based approval. Unattended runs just block.

## 3. Proposed solution (one core feature)
A policy proxy that sits between any MCP client and its servers. A tool is auto-allowed when its name or annotations mark it read-only. Anything else goes to a queue, and you approve it from your phone, for example through Telegram.

## 4. Architecture
- **Stack:** Python or Node, using the MCP SDK, with a YAML policy file and SQLite for the audit log. A Telegram bot handles approvals.
- **Components:** an MCP server facing the agent, an MCP client facing the real servers, a policy engine, an approval queue, and an audit log.
- **Data flow:** agent calls tool, proxy checks policy. If read-only, forward it. If not, send a Telegram prompt, wait for the answer, then forward or deny. Log the result.

## 5. MVP scope
- **Week 1:** pass-through proxy for stdio MCP servers, a read-only classifier from tool annotations plus name rules, a YAML allow/deny file, and an audit log.
- **Week 2:** Telegram approvals with a timeout and a default deny, a "remember this choice" option, and setup docs for Claude Code and Cursor.

## 6. Biggest risk
Clients may fix their own allowlists soon. Read-only classification can also be wrong, because tool annotations are optional and many servers don't set them.

## 7. Runner-ups
- **Pre-call agent budget enforcer:** a proxy that blocks a call before it fires. The pain is real, but many tools already pitch this, so the gap is thin. [example thread](https://forum.langchain.com/t/how-are-you-handling-runaway-agent-costs/3913)
- **Project decision memory for coding agents:** a store for the "why" behind architecture choices, shared across sessions. The complaint is common, but memory tools are crowded. [example](https://www.producthunt.com/p/ai-dimag/what-s-the-most-annoying-thing-your-ai-coding-assistant-forgets-between-sessions)

## Wildcard and India niche
No solid evidence this week for the Indian small business niche or a wildcard.

## Scores (1-5)
| Idea | Pain | Gap | Buildable in 4 weeks | Skills fit |
|---|---|---|---|---|
| ToolGate | 4 | 3 | 4 | 4 |
| Budget enforcer | 4 | 2 | 4 | 4 |
| Decision memory | 4 | 2 | 3 | 4 |

The scores are judgment, not measured data.
