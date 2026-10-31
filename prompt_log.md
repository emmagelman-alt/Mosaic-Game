# Prompt log: Mosaic Master

This log explains how the code got the way it is, so you don't have to work it out from the code. My prompts are quoted exactly as I typed them, typos included. The text under each prompt is a short note on what it changed. Those notes are not part of the prompts.

## AI model and tools

| What | Used for |
|---|---|
| **Claude Code** (desktop app, Code tab) | All of the coding, testing and publishing below, in one session on Oct 28, 2026 |
| **Claude Opus 5.5** (`claude-opus-5-5`) | The model behind Claude Code. For a while it was also the in-game judge (Claude's vision API), but that was removed (see prompt 4) |
| **pygame 2.6.1**, Python 3.9 | The game itself: `mosaic_master.py`, one file |
| **Pyodide 0.27.7 + pygame-ce 2.4.1** | Runs the same `mosaic_master.py` in a web browser (`index.html`) |
| **GitHub Pages** | Hosts the public link: https://emmagelman-alt.github.io/Mosaic-Game/ |

How Claude tested its work: it ran the game with no window (pygame's "dummy" video driver), had a script click through every screen and save screenshots, and looked at them. It tested the web version by clicking through it in Claude Code's built-in browser.

## Where the design came from

Claude didn't invent the design. It built from two files I wrote before starting:
- **`README.md`**, the game concept at the top: title screen, shop with customers, workspace, scoring, and multiplayer with codes, timer and voting
- **`Desktop - 1.png`**, my wireframe of the workspace: tile pile on the left, canvas, request text, finish button, and six shape buttons

---

## 1. Kickoff

**Me:**

> connect to the Mosaic game repo, read the two attached files, ask me 1 question at a time until you have 95% certainty you understand what I want to build, and then begin building

Claude cloned the repo, read the README and the wireframe, and then asked five questions, one at a time. Each time I picked one of the options it offered. My answers are below exactly as sent. The questions are paraphrased.

| Claude's question | My answer | What it means in the code |
|---|---|---|
| How should "an API call runs a similarity check" work? | > Claude vision API | Built at first, then removed in prompt 4. Scoring is now `placeholder_score()` |
| Same Wi-Fi only, or players anywhere over the internet? | > Anywhere (internet) | Multiplayer plan: a small relay server online (e.g. on Render), with both games connecting to it using a room code. **Not built yet** |
| What does "tiles only overlap if double clicked" mean? | > A: toggle a tile | Double-clicking a canvas tile switches it into overlap mode (it glows). Normal tiles that would overlap slide to the nearest open spot instead: `settle()` searches outward in a spiral |
| Should the tiles be pixelated too? | > A: pixel everywhere | The whole game is drawn at 640×360 and scaled up, so the tiles look pixelated too |
| Single player endless, or days of 5 customers? | > A: endless | Customers keep coming. The top-left corner shows pieces served and your average score |

Smaller choices Claude listed and I didn't change: requests are text from a built-in list of about 60 (`PROMPTS`), **R** rotates a tile, dragging a tile off the canvas removes it, and in multiplayer the host picks the time limit.

## 2. Build it (single player only)

**Me:**

> build the basic working version. you have max 30 minutes

I stopped that reply partway and sent:

> focus on just implementing single player first

This produced `mosaic_master.py`. The main parts, in file order:
- **Settings and layout:** screen size, the tile color palette (`PALETTE`), shapes, request prompts, and the positions of everything on screen
- **Tiles:** `Tile` (position, rotation, overlap flag, pixel-perfect collision mask) and `make_pile()`, which fills the pile with every color in 4 sizes of the selected shape. Dragging from the pile makes a copy, so the pile never runs out
- **Customers:** `Customer` draws a random pixel person and walks them along the line
- **Backgrounds:** drawn in code once at startup; there are no image files
- **`Game`:** three screens (`"title"`, `"shop"`, `"work"`). Each has its own click handling, update and draw methods. In the shop, `shop_state` runs arriving → request → judging → verdict

## 3. Commit and push

**Me:**

> Commit and push

Claude committed the code, but it couldn't push because it has no GitHub login on my computer. That's why the commits on GitHub say "Add files via upload": I uploaded the files through the website.

## 4. Remove the AI scoring for the handoff

**Me:**

> remove the api stuff for now since i have to hand this off,

This took out the Claude API call, the API-key setup, and the `anthropic` package. Scoring is now **`placeholder_score(tiles)`**. It doesn't look at what the picture shows; it only rewards covering the canvas, using several colors and using enough tiles. To add real scoring, replace that function with anything that returns `(score, comment)`. The customer still "thinks" for `JUDGE_SECONDS` (2.5 s) before showing the score.

## 5. Getting the files onto GitHub

These prompts were about uploading, not code:

> give me aother way beside terminala nd esktop

> i did it on my safari

> i put them in the game rpo

> Leave it

My first upload went to my **portfolio** repo by mistake, so copies of the game files also sit at the top level of `15-113-portoflio`. I chose to leave them there. The real copy is this repo.

## 6. A link to play in the browser

**Me:**

> give me a link to play the game

> give me a sharable link

> it says it cant make a sharable link

Claude first published a private Claude page, but my account couldn't make it public. So it wrote **`index.html`** for GitHub Pages instead, and I turned Pages on in the repo settings. How it works:
- The page loads Pyodide (Python for the browser) and pygame-ce from a public CDN, then fetches **this repo's `mosaic_master.py`**. The link always plays whatever is on `main`.
- It **doesn't call `Game.run()`**. It runs its own loop that calls `game.handle_event()`, `game.update()` and `game.draw()` about 60 times a second, and pauses between frames so the browser stays responsive.

**What this means if you change `mosaic_master.py`:** keep `handle_event`, `update` and `draw` quick. No `time.sleep`, threads, or loops that wait for something; they would freeze the browser version. This matters most for multiplayer: Python's `websockets` library won't work in the browser, so the web version would need the browser's own WebSocket (Pyodide can reach it through `js.WebSocket`).

## 7. This log

**Me:**

> make a prompt_log.md
> Which AI model(s) and tool(s) you used, and your important, non-trivial prompts verbatim rather than AI-written summaries.
> Here it does double duty: your partner can read how the code got this way, which is often faster than reverse-engineering it. Write it with them in mind. and upload it to the repo

---

## Still to do

- **Multiplayer:** create/join with a code, a timer chosen by the host (30 s to 10 min), both players' mosaics side by side, voting, and next round / return home. The title screen's Multiplayer button only says "coming soon".
- **Real scoring:** replace `placeholder_score()` with a check of how close the mosaic is to the request. README tweak 3 asks for a low- or no-cost way to do it.
- **Tweaks from the README:** a way to scale tiles, and a way to duplicate tiles.

## Quirks worth knowing

- Clicking a tile and then grabbing it again within 0.35 s counts as a double-click (`DOUBLE_CLICK_SECONDS`) and toggles overlap instead of starting a drag.
- If a dropped tile has no open spot within about 100 px, it isn't placed, and a message says "No room".
- To run it locally, follow the README. The web version needs no setup.
