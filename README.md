# First Responder AI Portal

**Grounded AI + immersive training for electric-vehicle emergencies.**

A web and VR (Meta Quest) training portal that pairs **360° walk-around videos** of real EV response training with an **AI assistant grounded in official manufacturer documents**, plus a **Gaussian-splat VR viewer** of scanned vehicles with hazard hotspots.

**Live demo:** https://nec4-jumpstart.streamlit.app

<p align="center">
  <img src="docs/media/vr-demo.webp" alt="In-headset recording: a trainee asks a question from inside the 360° training video and the answer cites the ERG page, Rescue Sheet and video timestamp" width="720">
  <br><em>Recorded on a Meta Quest 3: asking a question from inside the 360° training video. The answer cites the ERG page, the Rescue Sheet and the video timestamp it came from.</em>
</p>

| Desktop portal: 360° lecture + training assistant | VR splat viewer: hazard hotspots on a scanned EV |
|---|---|
| ![Desktop portal with 360° training video and chat panel](docs/media/portal-desktop.png) | ![Gaussian-splat scan of a Chevrolet Equinox EV; selecting a red hotspot opens the First Responder Loop cutting procedure](docs/media/splat-vr-demo.webp) |
| | <sub>Gaussian-splat scan of a Chevrolet Equinox EV (`apps/splat-vr/`). Pointing at a hotspot opens the cutting procedure quoted from the ERG, with its page citation. Markers carry an **unverified-placement** banner until their position is checked against the ERG diagrams.</sub> |

---

## Why this exists

EVs change the rules for firefighters and rescuers: orange high-voltage cables that must never be cut, 12 V systems that keep airbags live, battery fires that can take thousands of gallons of water. The authoritative answers are in **Emergency Response Guides (ERGs) and Rescue Sheets**, dozens of pages per model and different for every vehicle. Nobody reads a 60-page PDF at a crash scene.

In this setting a **wrong answer is worse than no answer**, so the system is designed to:

1. **Retrieve per vehicle, not globally.** Each of the 13 vehicles has its own vector namespace, so a Tesla procedure can never leak into a Chevy answer.
2. **Defer instead of guess.** With no vehicle identified, the assistant gives a generic safety baseline and asks which vehicle, instead of inventing a model-specific procedure.
3. **Cite the source**, and say so when the document does not contain what was asked.
4. **Train in context.** Trainees ask questions from inside a 360° video or a VR scan of the vehicle, not from a separate chat window.

---

## Results so far

**Grounding and abstention (24-question study, [`docs/EV_responder_QA_comparison.md`](docs/EV_responder_QA_comparison.md))**

| Condition | Outcome |
|---|---|
| Question names **no vehicle** (16 questions) | 16/16 deferred with a generic safety baseline + "which vehicle?" (no fabricated procedures) |
| Same 16 questions **with a vehicle named** (spread across all 13 vehicles) | **14/16 grounded, source-cited answers**; **2/16 correct refusals**: the asked-about feature is not in that vehicle's documents, and the assistant said so instead of inventing it |
| 5 new vehicle-specific questions | 5/5 grounded, source-cited |

**Retrieval depth.** With the default top-k = 4, vehicle-specific chunks fell outside the retrieved set and answers mixed facts across vehicles. Raising top-k to 10 on every per-vehicle retriever fixed the observed cross-vehicle hallucinations. The setting is now enforced by an automated drift check against the live workflow (`ops/n8n_sync.py --check`).

**Transcript QA: work in progress.** A 90-question ground-truth set (`ops/eval/eval_questions.json`) is run automatically against the live assistant (`ops/eval/run_eval.py`). The first 30-question transcript run scored 12 pass / 3 fail / 15 needing hand review under a keyword-overlap heuristic. At least one failure traces to a lecture transcript not yet indexed. Next: index it, replace the heuristic with a rubric-based grader, and report retrieval recall@k.

---

## How it works

```
                       ┌─────────────────────────────── Browser / Meta Quest ───────────────────────────────┐
                       │  360° video portal (IWSDK / WebXR)   ·   in-VR chat HUD + push-to-talk voice        │
                       │  Gaussian-splat VR viewer (three.js + Spark) with ERG hazard hotspots              │
                       └───────────────────────────────┬────────────────────────────────────────────────────┘
                                                       │ question + session id
                                                       ▼
                                   n8n agent (router LLM)  ──  picks a tool per vehicle / per corpus
                                                       │
                     ┌─────────────────────────────────┼──────────────────────────────────┐
                     ▼                                 ▼                                  ▼
           13 per-vehicle namespaces          360° video transcripts            conversation memory
           (ERG + Rescue Sheet, top-k=10)     (Whisper, timestamped)            (Postgres, per session)
                     └──────────── Pinecone (text-embedding-3-small) ─────────────┘
```

- **Ingestion:** PDFs are parsed with `unstructured` (OCR fallback for image-heavy pages), chunked, embedded and stored per vehicle, with `doc_type` metadata separating ERGs from Rescue Sheets. Training videos are transcribed locally with Whisper and indexed with timestamps so answers can point back into the video.
- **Serving:** the n8n agent routes each question to the right vehicle's retriever (or the transcript corpus) and answers only from what it retrieves.
- **XR:** built on Meta's Immersive Web SDK. It runs in any browser, in a desktop Quest emulator, or on a real headset. On Quest, voice input falls back to server-side Whisper because the Quest Browser has no native speech recognition.
- **3D scans:** Gaussian-splat captures of real vehicles, calibrated for true scale and cropped to fit a standalone headset's rendering budget. Hazard markers stay flagged `verified: false` until confirmed against the ERG diagrams. A tool that tells a responder where to cut must never present a guess as fact.

**Corpus:** 13 EV models (BMW, Cadillac, Chevrolet, Ford, GM BrightDrop, Hyundai, Nissan, Rivian, Tesla, Volkswagen), 23 ERG/Rescue-Sheet PDFs, plus the transcripts of the 360° training videos.

---

## Try it in 3 steps

You do **not** need any API keys. The chatbot already talks to a hosted AI, so you can just run the page.

**1. Install Node.js** (version 20 or newer) from https://nodejs.org

**2. Download the project and install it**

```bash
git clone https://github.com/Irfan-Gazi0/RAG_Responder.git
cd RAG_Responder/apps/portal
npm install
```

**3. Start it**

```bash
npm run dev
```

Your browser opens at **https://localhost:8081**.
It may warn "Not Secure". That's expected for a local test page. Click *Advanced → Proceed*.

### Things to play with

- **Drag** on the video to look around in 360°.
- **Ask the chatbot** something, for example:
  - *Where are the no-cut zones on a Tesla Model S?*
  - *How do I disable the high-voltage battery on a Chevy Bolt?*
  - *How do I disable the high-voltage battery?* (no vehicle named; watch it ask which one)
- Try the **Enter VR** button if you have a VR headset (Meta Quest). No headset? A built-in emulator runs on your computer.

---

## What's in the folders

| Folder | What it is |
|---|---|
| `apps/portal/` | **The 360° portal you just ran** (IWSDK + TypeScript; start here) |
| `apps/splat-vr/` | Gaussian-splat VR viewer with hazard hotspots and hand tracking |
| `apps/v1/` | Older A-Frame version (kept as a fallback) |
| `ingestion/` | PDF + transcript ingestion notebooks, Whisper transcription |
| `ops/` | Live-workflow drift check and the automated eval harness |
| `deploy/` | Publish scripts (S3 + CloudFront) with post-deploy integrity checks |
| `docs/` | Evaluation write-ups |

---

## Want to go deeper? (optional)

These steps need your own accounts and API keys. Skip them if you just want to try the portal.

- **Use your own database and AI:** create a `.env` file in the project root with `OPENAI_API_KEY` and `PINECONE_API_KEY`, then run the notebooks in `ingestion/notebooks/` (Python 3.10).
- **Run the evaluation:** `python3.10 ops/eval/run_eval.py --sample 10`
- **Run the old version:** `cd apps/v1 && python3 -m http.server 8080`, then open http://localhost:8080/inspector_portal.html

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `npm install` fails or complains about the Node version | Install Node 20 or newer: `node --version` |
| Browser says the connection is not private | Normal for local testing. Click *Advanced → Proceed to localhost* |
| Page is blank | Wait a few seconds, then refresh. Check the terminal for errors |
| Chatbot doesn't reply | The hosted AI may be offline. Try again later, or set up your own (see above) |
| Port 8081 already in use | Close the other program using it, or stop the earlier `npm run dev` |

---

## Author

**Irfan Gazi** · [GitHub](https://github.com/Irfan-Gazi0)

## License

Apache-2.0. See `LICENSE`.
