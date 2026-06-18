# MATLAB Academy Auto-Solver

An AI-powered automation tool that completes **any MATLAB Academy course** on [MATLAB Academy](https://matlabacademy.mathworks.com/) automatically. It uses browser automation (Playwright) and AI vision/code generation (Groq LLM) to read tasks, generate MATLAB code, type it into the interactive environment, and advance through the course.

Works with all MATLAB Academy courses including:
- MATLAB Onramp
- MATLAB Desktop Tools and Troubleshooting Scripts
- MATLAB Project-Based Learning
- Any other MATLAB Academy interactive course

---

## Features

- **Works with any MATLAB Academy course** — just change the URL in `auto_solver.py`
- **Automatic exercise solving** — reads task descriptions, generates MATLAB code via AI, and types it into the browser
- **Dual mode support** — handles both **Command Window** and **Live Editor** exercise types
- **Auto-skips video/lecture pages** — detects non-interactive pages and clicks Next automatically
- **AI-powered code generation** — uses Groq's Llama models to generate correct MATLAB commands
- **Vision-based error recovery** — takes screenshots and uses multimodal AI to detect errors and retry with corrected code
- **Stealth browsing** — uses `playwright-stealth` to avoid bot detection
- **Session persistence** — saves login cookies so you only need to authenticate once
- **Auto-retry logic** — retries failed answers up to 3 times with AI-guided corrections
- **Section break and text support** — handles "Add a section" and "Add text" type tasks in Live Editor

---

## Project Structure

```
matlab-automation/
├── auto_solver.py       # Main solver — runs the full automation loop
├── setup_session.py     # One-time login — saves session cookies to state.json
├── inspect_dom.py       # DOM inspector — debug tool for examining page structure
├── test_typing.py       # Typing test — verifies command window input works
├── requirements.txt     # Python dependencies
└── README.md            # You are here
```

| File | Purpose |
|------|---------|
| `auto_solver.py` | Core automation engine. Detects exercise mode, generates code with AI, types into the MATLAB environment, verifies correctness, and navigates to the next task. |
| `setup_session.py` | Opens a browser for you to manually log in to MATLAB Academy. Saves authentication cookies to `state.json` for reuse. |
| `inspect_dom.py` | Developer utility to inspect the DOM structure of MATLAB Academy pages for debugging selectors. |
| `test_typing.py` | Quick test to verify that Playwright can successfully type into the MATLAB command window. |

---

## How to Run

### Prerequisites

- **Python 3.9+**
- **A MathWorks account** with access to [MATLAB Academy](https://matlabacademy.mathworks.com/)
- **A Groq API key** — get one free at [console.groq.com](https://console.groq.com/)

### Step 1: Clone the Repository

```bash
git clone https://github.com/areal-moon/matlab-automation.git
cd matlab-automation
```

### Step 2: Install Dependencies

```bash
pip install -r requirements.txt
pip install groq
```

Then install the Playwright browsers:

```bash
playwright install chromium
```

### Step 3: Set Up Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
```

### Step 4: Save Your MATLAB Academy Session

Run the session setup script to log in and save your authentication cookies:

```bash
python setup_session.py
```

This will:
1. Open a Chromium browser window
2. Navigate to MATLAB Academy
3. **You manually log in** with your MathWorks account
4. Once you're logged in and can see the course, **close the browser window**
5. Your session is saved to `state.json`

> **Note:** You only need to do this once. The saved session will be reused until it expires.

### Step 5: Set Your Target Course

Open `auto_solver.py` and update the `DIRECT_URL` in the `resume_course` function to the course you want to solve:

```python
DIRECT_URL = "https://matlabacademy.mathworks.com/v1/portal.html?course=YOUR_COURSE#chapter=1&lesson=1"
```

Example course URLs:

| Course | URL |
|--------|-----|
| MATLAB Onramp | `...?course=gettingstarted#chapter=1&lesson=1` |
| Desktop Tools & Troubleshooting | `...?course=otmldts#chapter=1&lesson=1` |
| Project-Based Learning | `...?course=otmlprc#chapter=1&lesson=1` |

### Step 6: Run the Auto-Solver

```bash
python auto_solver.py
```

The solver will:
1. Open a browser with your saved session
2. Navigate directly to your target course and chapter
3. Auto-skip video and lecture pages
4. Detect each exercise type (Command Window or Live Editor)
5. Use AI to generate the correct MATLAB code
6. Type the code, verify correctness, and advance to the next task
7. Repeat until the course is complete

---

## Utility Scripts

### Inspect DOM Structure

If you need to debug or update selectors:

```bash
python inspect_dom.py
```

### Test Typing into Command Window

To verify Playwright can interact with the MATLAB environment:

```bash
python test_typing.py
```

---

## Configuration

| Variable | Location | Description |
|----------|----------|-------------|
| `GROQ_API_KEY` | `.env` file | Your Groq API key for AI code generation |
| `DIRECT_URL` | `auto_solver.py` | The direct course URL to solve (supports any MATLAB Academy course) |
| `MAX_RETRIES` | `auto_solver.py` | Max retry attempts per task (default: 3) |

---

## How It Works

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Load Page   │────▶│  Video/Lecture │────▶│  Click Next  │
│              │     │  Page?        │ Yes │  (Auto-skip) │
└──────────────┘     └───────┬───────┘     └──────────────┘
                             │ No
                     ┌───────▼───────┐
                     │  Read Task    │
                     │  Description  │
                     └───────┬───────┘
                             │
                     ┌───────▼───────┐     ┌──────────────┐
                     │  AI Generates │────▶│  Type Code   │
                     │  MATLAB Code  │     │  in Browser  │
                     └───────────────┘     └──────┬───────┘
                                                  │
                     ┌───────────────┐     ┌──────▼───────┐
                     │  Retry with   │◀────│  Check if    │
                     │  Correction   │ No  │  Correct?    │
                     └───────────────┘     └──────┬───────┘
                                                  │ Yes
                                           ┌──────▼───────┐
                                           │  Advance to  │
                                           │  Next Task   │
                                           └──────────────┘
```

1. **Page Detection** — checks if the current page is a video/lecture or an interactive exercise
2. **Auto-Skip** — automatically clicks Next on non-interactive pages (videos, lectures, surveys)
3. **Frame Detection** — locates the MATLAB interactive iframe within the page
4. **Mode Detection** — determines if the exercise uses the Command Window or Live Editor
5. **Task Extraction** — reads the task description from the DOM
6. **Code Generation** — sends the task to Groq's Llama model to generate MATLAB code
7. **Code Entry** — types the generated code into the appropriate input area
8. **Verification** — checks for "Correct!" banner or uses vision AI to detect errors
9. **Retry/Advance** — retries with corrected code on failure, or advances on success

---

## Disclaimer

This tool is intended for **educational and personal use only**. Use it responsibly and in accordance with MathWorks' terms of service. The authors are not responsible for any misuse or consequences resulting from the use of this tool.

---

## License

This project is open source. Feel free to fork, modify, and improve it.
