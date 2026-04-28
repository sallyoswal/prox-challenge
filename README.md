# Vulcan OmniPro 220 — AI Welding Assistant

A multimodal reasoning agent for the Vulcan OmniPro 220 multiprocess welder (Harbor Freight #57812), built with the Anthropic Claude SDK.

![Agent Screenshot](https://github.com/prox-technologies/prox-challenge/raw/main/product.webp)

## ✨ What It Does

This agent answers deep technical questions about the OmniPro 220 — and critically, **generates visual responses** when text alone isn't enough:

| Question Type | Response Format |
|---|---|
| Polarity setup | SVG wiring diagram with color-coded cables |
| Duty cycle | Interactive calculator with sliders |
| Troubleshooting | HTML decision flowchart |
| Material settings | Interactive settings configurator |
| Weld diagnosis (photo) | Visual defect analysis |

## 🚀 Setup (< 2 minutes)

```bash
git clone <your-fork>
cd <your-fork>
cp .env.example .env
# Add your Anthropic API key to .env
npm install
npm start
```

Open **http://localhost:3000**

> The OmniPro 220 owner's manual, quick-start guide, and selection chart are already included in `files/`. The agent loads the manual automatically on startup — no extra steps needed.

## 🏗 Architecture

```
User Question + Optional Image
        ↓
Express Server (multer for file upload)
        ↓
Claude Sonnet (vision-capable)
    ├── System prompt: full manual knowledge + visual response rules
    ├── Conversation history: multi-turn context
    └── PDF document: raw manual pages (if available)
        ↓
Structured JSON response:
    ├── text: conversational explanation
    └── artifact: { type, title, content } — HTML/SVG visual
        ↓
Frontend renders artifact in inline card + expandable side panel
```

### Key Design Decisions

**1. Knowledge-in-prompt vs RAG**  
For a 48-page manual, I chose to encode the critical knowledge (duty cycle tables, polarity setups, troubleshooting matrices, material settings) directly in the system prompt as structured data. This ensures sub-second responses without a retrieval step, and the agent can cross-reference multiple sections in a single inference.

RAG would shine for a 500+ page manual. At 48 pages, the system prompt approach gives more reliable cross-referencing (the model can reason holistically, not just retrieve).

**2. Structured JSON output**  
The agent always responds in `{ text, artifact }` JSON. This separates the conversational explanation from the visual, letting the UI render them independently. The artifact panel expands on demand for a larger view.

**3. Visual-first philosophy**  
The system prompt has explicit rules: if the question involves polarity → generate SVG diagram; if duty cycle → generate interactive calculator; if troubleshooting → generate flowchart. The agent is instructed to *draw* what's cognitively hard to explain in words.

**4. Multimodal input**  
Users can attach photos of their weld bead or machine setup. Claude's vision analyzes weld defects (porosity, spatter, undercut) against the manual's diagnosis chart.

**5. Conversation memory**  
Full conversation history is passed on each request, enabling follow-up questions ("now show me the same for 120V").

## 🧪 Sample Questions to Test

**Technical accuracy:**
- "What's the duty cycle for MIG welding at 200A on 240V?"
- "What polarity do I need for flux-cored welding with gasless wire?"
- "Can I run 0.035" wire on this machine for 1/4" steel?"

**Multimodal response:**
- "Show me a polarity diagram for TIG welding"
- "Give me an interactive duty cycle calculator for all processes"
- "Build me a settings configurator for MIG welding different steel thicknesses"

**Image-based:**
- Attach a photo of a weld and ask "What's wrong with this weld?"
- Attach a photo of the front panel and ask "What do these settings mean?"

**Troubleshooting:**
- "I'm getting porosity in my MIG welds. Walk me through diagnosis."
- "My wire keeps bird-nesting. What should I check?"

## 📁 Project Structure

```
prox-challenge/
├── server.js              # Express API + Claude agent (streaming SSE)
├── public/
│   └── index.html         # Frontend UI (single-file, streaming client)
├── files/
│   ├── owner-manual.pdf       # Full 48-page OmniPro 220 manual
│   ├── quick-start-guide.pdf
│   └── selection-chart.pdf
├── .env.example
├── package.json
└── README.md
```

## 🔧 Environment Variables

| Variable | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | ✅ | Your Anthropic API key |
| `PORT` | No | Server port (default: 3000) |

## 💡 Design Philosophy

The target user just bought a $1,100 welder and is standing in their garage. They don't have time to read 48 pages. They need:

1. **Fast, accurate answers** — not hedge-everything LLM hedging
2. **Visual clarity** — a wiring diagram is worth 500 words
3. **Interactive tools** — a duty cycle slider is more useful than a table
4. **Appropriate confidence** — they're not idiots, they just need the right info

The agent treats them like a knowledgeable friend who happens to have memorized the manual.
