# Research Analyst Agent: Chat SDK × Claude Managed Agents

**Author:** Pranav Waghmare (Northeastern University, MS Information Systems)
**Based on:** [anthropics/claude-quickstarts · managed-agents/chat-sdk](https://github.com/anthropics/claude-quickstarts/tree/main/managed-agents/chat-sdk)

A browser chat app where a Claude Managed Agent researches any topic on the web and replies with a short, sourced brief. Vercel's Chat SDK handles the chat window; Anthropic hosts the agent loop, the sandbox, and the conversation history.

## What I changed

| Change | File | Why |
| --- | --- | --- |
| Switched the model to `claude-sonnet-5-5` | `.env` (`QUICKSTART_MODEL`) | Lower cost per brief than the Opus default |
| Tightened the formatting rules in the system prompt, published as agent version 2 | `setup/agent-config.ts` | Keep briefs as chat paragraphs, never headings |
| Fixed a rendering bug: `.join("\n\n")` → `.join("")` in `rawTextOf` | `src/managed-agents.ts` | See below |
| Moved the server to port 3001 | `.env` (`PORT=3001`) | So a second quickstart can run on 3000 at the same time |

### The bug I found and fixed

Briefs rendered badly: "Key facts:" appeared as a large heading, bullets showed up empty, and every sentence sat on its own line.

- **First test:** I added stricter formatting rules to the system prompt (agent version 2). Nothing changed, which showed the model was not the cause.
- **Real cause:** when the agent cites web sources, its reply arrives as many small text blocks. `rawTextOf` joined them with a blank line (`"\n\n"`), so every cited sentence became its own markdown paragraph, and a lone `-` under `**Key facts:**` turned that line into a heading.
- **Fix:** join the blocks with `""`, since each block already carries its own spacing. The same function is used for live streaming and for replaying old chats, so one line fixes both.

## How to run it (Windows)

Needs Node 22.9+ and an Anthropic API key from [platform.claude.com](https://platform.claude.com).

```powershell
npm install
Copy-Item .env.example .env   # add ANTHROPIC_API_KEY, QUICKSTART_MODEL=claude-sonnet-5-5, PORT=3001
npm run setup                 # one time: creates the agent + environment; paste the two printed IDs into .env
npm run dev                   # open http://127.0.0.1:3001 and ask for a brief on any topic
```

After editing `setup/agent-config.ts`, run `npm run update-agent` to publish a new agent version (never re-run `npm run setup`, which would create a duplicate agent).

## What I learned

- An agent is configuration (model, system prompt, tools), created once and updated by version.
- The tool loop, sandbox, and conversation storage run on Anthropic's side; the local server is a thin bridge with no database.
- Everything in a session is an event (`agent.message`, `agent.tool_use`, `session.status_idle`), and the app just reacts to event types.
- When output looks wrong, change one thing at a time: the prompt test pointed the search at the code.

---

*The original quickstart README follows below.*

