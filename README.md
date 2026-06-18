# 🤖 MATLAB Academy Auto-Solver

An AI-powered automation tool that completes **MATLAB Onramp** exercises on [MATLAB Academy](https://matlabacademy.mathworks.com/) automatically. It uses browser automation (Playwright) and AI vision/code generation (Groq LLM) to read tasks, generate MATLAB code, type it into the interactive environment, and advance through the course.

---

## ✨ Features

- **Automatic exercise solving** — reads task descriptions, generates MATLAB code via AI, and types it into the browser
- **Dual mode support** — handles both **Command Window** and **Live Editor** exercise types
- **AI-powered code generation** — uses Groq's Llama models to generate correct MATLAB commands
- **Vision-based error recovery** — takes screenshots and uses multimodal AI to detect errors and retry with corrected code
- **Stealth browsing** — uses `playwright-stealth` to avoid bot detection
- **Session persistence** — saves login cookies so you only need to authenticate once
- **Auto-retry logic** — retries failed answers up to 3 times with AI-guided corrections

---

## 📁 Project Structure

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

## 🚀 How to Run

### Prerequisites

- **Python 3.9+**
- **A MathWorks account** with access to [MATLAB Onramp](https://matlabacademy.mathworks.com/details/matlab-onramp/gettingstarted)
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

### Step 5: Run the Auto-Solver

```bash
python auto_solver.py
```

The solver will:
1. Open a browser with your saved session
2. Navigate to the MATLAB Onramp course and click **Resume**
3. Detect each exercise type (Command Window or Live Editor)
4. Use AI to generate the correct MATLAB code
5. Type the code, verify correctness, and advance to the next task
6. Repeat until the course is complete or 5 consecutive failures occur

---

## 🛠️ Utility Scripts

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

## ⚙️ Configuration

| Variable | Location | Description |
|----------|----------|-------------|
| `GROQ_API_KEY` | `.env` file | Your Groq API key for AI code generation |
| `COURSE_LANDING_URL` | `auto_solver.py` | The MATLAB Academy course URL (default: MATLAB Onramp) |
| `MAX_RETRIES` | `auto_solver.py` | Max retry attempts per task (default: 3) |

---

## 🧩 How It Works

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Read Task   │────▶│  AI Generates │────▶│  Type Code   │
│  Description │     │  MATLAB Code  │     │  in Browser  │
└──────────────┘     └───────────────┘     └──────┬───────┘
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

1. **Frame Detection** — locates the MATLAB interactive iframe within the page
2. **Mode Detection** — determines if the exercise uses the Command Window or Live Editor
3. **Task Extraction** — reads the task description from the DOM
4. **Code Generation** — sends the task to Groq's Llama model to generate MATLAB code
5. **Code Entry** — types the generated code into the appropriate input area
6. **Verification** — checks for "Correct!" banner or uses vision AI to detect errors
7. **Retry/Advance** — retries with corrected code on failure, or advances on success

---

## ⚠️ Disclaimer

This tool is intended for **educational and personal use only**. Use it responsibly and in accordance with MathWorks' terms of service. The authors are not responsible for any misuse or consequences resulting from the use of this tool.

---

## 📄 License

This project is open source. Feel free to fork, modify, and improve it.
