# PHISHSCOPE: MASTER PRESENTATION & VIVA SURVIVAL GUIDE

> **CONFIDENTIAL & ACADEMIC PREPARATION MATERIAL**  
> *Project Name:* **PhishScope — Hybrid AI Phishing Detection with XAI & Browser Interception**  
> *Target:* Final Semester MCA Project Presentation & Viva-Voce Examination  
> *Author / Developer:* Sian Julian (Department of Computer Applications)

---

## TABLE OF CONTENTS
1. [The 60-Second "Elevator Pitch" (How to Introduce Your Project)](#1-the-60-second-elevator-pitch)
2. [High-Level Project Summary (In Simple Plain English)](#2-high-level-project-summary)
3. [The Complete Step-by-Step Workflow (From URL Click to Verdict)](#3-the-complete-step-by-step-workflow)
4. [Architecture & "Where is the Database?" (Crucial Examiner Trap)](#4-architecture--where-is-the-database)
5. [The 10 Phases of Development (What Was Built in Each Phase)](#5-the-10-phases-of-development)
6. [Real Bugs, Errors, and Challenges Occurred (And How You Solved Them)](#6-real-bugs-errors-and-challenges-occurred)
7. [The Machine Learning & XAI Deep Dive (Simple Explanations of Complex Math)](#7-the-machine-learning--xai-deep-dive)
8. [The Browser Extension & Real-Time Interception (Manifest V3)](#8-the-browser-extension--real-time-interception)
9. [Top 30 Expected Viva-Voce Questions & Perfect Model Answers](#9-top-30-expected-viva-voce-questions--perfect-model-answers)
10. [Presentation Slide-by-Slide Outline & Demo Script](#10-presentation-slide-by-slide-outline--demo-script)

---

# 1. THE 60-SECOND "ELEVATOR PITCH"
*(Memorize this paragraph to start your presentation with extreme confidence!)*

> "Respected examiners, traditional anti-phishing tools rely either on **static blacklists** like Google Safe Browsing, which fail against new zero-hour attacks that last less than 4 hours, or **black-box machine learning models**, which misclassify complex modern web apps and cannot explain *why* a link is dangerous.
> 
> My project, **PhishScope**, solves both problems. It is an intelligent, multi-layered phishing defense platform that combines a **21-feature extraction engine**, a **rule-based heuristic scorer**, a **Random Forest classifier with 98.55% accuracy**, and **SHAP Explainable AI** that explains risks in plain human language. 
> 
> Everything is connected to an always-on **Google Chrome Manifest V3 extension** that intercepts dangerous links *before* the browser even connects to them, keeping users safe with zero clicks and sub-40-millisecond latency."

---

# 2. HIGH-LEVEL PROJECT SUMMARY
### What is PhishScope?
PhishScope is a cybersecurity tool that stops you from falling for phishing links. It looks at the website's address (the URL) *before* the page opens, analyzes whether it's genuine or a fake scam, explains why, and blocks it if it's dangerous.

### Why not just use existing tools?
| Problem in Existing Systems | How PhishScope Solves It |
| :--- | :--- |
| **Blacklists are slow:** Zero-day phishing links last 2 to 4 hours. Blacklists take 12 to 24 hours to update. | PhishScope inspects the structural patterns of the URL itself using Machine Learning in real time (<40ms). |
| **Black-box AI:** Machine learning models give a score (e.g. 0.89) but cannot explain *why*. | PhishScope uses **SHAP (Explainable AI)** to output plain-English reasons (e.g., *"This website uses a lookalike name to imitate PayPal"*). |
| **False Positives on Complex Sites:** Naive AI flags ChatGPT, GitHub, and YouTube because their URLs are long and complex. | PhishScope has an **in-memory Trusted Domain verification system** and heuristic consensus that eliminates false alarms. |
| **User had to manually click a button:** Passive extensions only check a page after it already starts loading. | PhishScope uses **pre-navigation interception**: it catches the click in Chrome, pauses the page, checks with the backend, and blocks it if malicious. |

---

# 3. THE COMPLETE STEP-BY-STEP WORKFLOW

Here is the exact lifecycle of what happens when a user clicks a link:

```
[User Clicks Link in Chrome]
         │
         ▼
[Chrome Extension: background.js]
  • Intercepts via chrome.webNavigation.onBeforeNavigate
  • Is it chrome:// or internal? ───► YES ───► Let it pass
  • Is it in Memory Session Whitelist? ──► YES ───► Let it pass
  • Otherwise: Instantly redirects tab to 'analyzing.html?url=...'
         │ (Aborts dangerous HTTP request before connecting!)
         ▼
[analyzing.js triggers API Call]
  • Sends JSON POST request to http://127.0.0.1:5000/analyze
         │
         ▼
[Flask Backend: app.py]
  • Step 1: Trusted Domain Check (trusted_loader.py)
    Is it google.com, github.com, chatgpt.com, youtube.com?
    ├── YES ──► Returns SAFE instantly (Score: 0, Confidence: VERY_HIGH) [Latency: 0.1ms]
    └── NO  ──► Proceeds to Step 2
         │
  • Step 2: Feature Extraction (feature_extractor.py)
    Extracts 21 numerical & structural features (Entropy, Length, Hyphens, TLD risk, Brand Mismatch, Punycode, etc.)
         │
  • Step 3: Dual Analysis Engine
    ├── Heuristic Scorer: Checks 10 weighted rules ──► Score (0-100) & Reasons
    └── Machine Learning: Random Forest (trained on 100k URLs) ──► Phishing Probability
         │
  • Step 4: Hybrid Consensus Engine (decision_tree.py)
    Arbitrates the two signals. (Prevents ML from overriding clear heuristics & vice versa)
    Produces Final Verdict: SAFE / SUSPICIOUS / DANGEROUS
         │
  • Step 5: Explainable AI (xai/explainer.py)
    SHAP TreeExplainer calculates which top 3 features caused this verdict.
    human_explanations.py converts them into Basic, Advanced, and Technical English sentences.
         │
         ▼
[JSON Returned to Extension]
  • Logged in chrome.storage.local (History of last 50 scans)
  • Verdict Action:
    ├── SAFE: Injects green toast notification ("TRUSTED WEBSITE") -> Redirects to original URL.
    ├── SUSPICIOUS: Redirects to 'suspicious.html' with warning details and override buttons.
    └── DANGEROUS: Fires OS notification ("Threat Blocked!") -> Redirects to 'warning.html'.
         │
[User Choice on Warning Page]
  ├── Clicks "Go Back to Safety" ──► Goes back in browser history.
  └── Clicks "Continue Anyway"   ──► Adds to session whitelist & allows navigation.
```

---

# 4. ARCHITECTURE & "WHERE IS THE DATABASE?"
*(CRITICAL EXAMINER TRAP: Read this carefully!)*

### The Examiner asks: *"Show me your database! Where are your MySQL/MongoDB tables?"*
### Your Flawless Answer:
> *"Respected examiner, PhishScope intentionally uses a **Stateless In-Memory & File-Based Storage Architecture** rather than a traditional relational database (like MySQL) or document database (like MongoDB). 
> 
> Here is the software engineering rationale for that architectural decision:
> 1. **Ultra-Low Latency Requirement:** PhishScope intercepts web browsing in real time. Querying an external database over TCP adds 10 to 50 milliseconds of overhead per click. In our design, trusted domains and brand lists are loaded into **in-memory hash sets ($O(1)$ lookup time)** taking less than **0.12 milliseconds**.
> 2. **Client-Side Privacy:** User scan history is stored directly inside the user's browser using `chrome.storage.local` (holding the last 50 scans). This ensures that user browsing history is never stored on a central server, preserving user privacy.
> 3. **Model & Dataset Persistence:** The machine learning model is serialized as a binary artifact (`random_forest.pkl` via Joblib), and dataset pipelines operate directly on optimized CSV matrices (`cleaned.csv`, `balanced.csv`, `train.csv`).
> 
> Therefore, our data layer consists of **RAM Hash Sets**, **Serialized Model Binaries**, **CSV Data Matrices**, and **Local Browser Storage**."*

### Breakdown of PhishScope's Data Storage:
| Storage Layer | Technology Used | What It Stores | Size / Capacity |
| :--- | :--- | :--- | :--- |
| **In-Memory Cache (RAM)** | Python `set()` in `trusted_loader.py` | Top trusted enterprise domains for false-positive bypass | $O(1)$ lookup, instantaneous |
| **Reference Lists** | Text files (`brands.txt`, `suspicious_tlds.txt`) | 100+ monitored global brands and high-risk TLDs | Stored on disk, loaded at boot |
| **Trained ML Models** | Joblib Serialized Binaries (`.pkl`) | 100-tree Random Forest classifier weights & nodes | ~20 MB on disk, loaded in RAM |
| **Datasets** | CSV Files (`phishing.csv`, `legitimate_urls.csv`) | 115,213 raw URLs down to 99,996 balanced training records | Flat CSV files |
| **Client Scan History** | `chrome.storage.local` | The user's last 50 scanned URLs, verdicts, and timestamps | Managed via circular array in browser |
| **Warning State** | `chrome.storage.local` (`phishscope_last_warning`) | Active warning details (reasons, score, summary) | Transient JSON object |

---

# 5. THE 10 PHASES OF DEVELOPMENT
*(This proves you built the project methodically from scratch!)*

* **Phase 1: Production-Ready Feature Extraction**
  * Built URL validation, normalization, and extracted 21 lexical/structural features.
  * Implemented Shannon entropy calculation, brand matcher, Punycode decoder (`xn--`), ASCII/Unicode homoglyphs, and IPv4/IPv6 detection.
  * **Tests:** 50/50 tests passed.
* **Phase 2: Heuristic Scoring Engine**
  * Created rule-based engine with 10 weighted rules (IP address, entropy, brand mismatch, risky TLD, etc.).
  * Generates 0–100 risk score, confidence levels (LOW, MEDIUM, HIGH), and readable trigger reasons.
  * **Tests:** 32/32 tests passed (82 cumulative).
* **Phase 3: Flask REST API**
  * Built lightweight microservice with `GET /`, `GET /health`, and `POST /analyze`.
  * Added CORS headers, JSON validation, and error handlers (400, 404, 405, 415, 500).
  * **Tests:** 28/28 tests passed (110 cumulative).
* **Phase 4: Dataset Pipeline**
  * Ingested 65,213 PhishTank URLs and 50,000 Legitimate URLs (Total: 115,213).
  * Cleaned, deduplicated, and balanced the dataset to 99,996 URLs (49,998 each).
  * Automated 80/20 train/test split: 79,996 train URLs, 20,000 test URLs.
  * **Tests:** 24/24 tests passed (134 cumulative).
* **Phase 5: Machine Learning Classifier**
  * Benchmarked 4 algorithms: Random Forest (99.36% train), Decision Tree (99.36%), Logistic Regression (97.11%), and Linear SVM (96.88%).
  * Trained Random Forest (`n_estimators=100`) on 80,196 rows in 0.90s.
  * Achieved **98.55% Accuracy, 98.99% Precision, 98.09% Recall, 98.54% F1-Score, 99.59% ROC-AUC**.
  * **Tests:** 21/21 tests passed (155 cumulative).
* **Phase 6: Hybrid Consensus Engine**
  * Combined Heuristics (50.41% accuracy alone) with Random Forest (98.55%) into an arbitration decision tree.
  * Blended score: `30% Heuristics + 70% ML Probability`.
  * Result: Maintained 98.54% accuracy while providing complete human explainability.
  * **Tests:** 22/22 tests passed.
* **Phase 7: Explainable AI (SHAP)**
  * Integrated SHAP `TreeExplainer` on the Random Forest model.
  * Optimized per-URL attribution from >300ms down to ~31ms.
  * Formatted top 3 features into Basic, Advanced, and Technical English sentences.
  * **Tests:** 9/9 tests passed.
* **Phase 8: React Single Page Application**
  * Built modern web dashboard with React 19, Vite, and Tailwind CSS.
  * Created interactive URL Analyzer with risk meter, VerdictBadge, and Recharts analytics.
  * **Tests:** 15 frontend tests.
* **Phase 9: Chrome Extension (Manifest V3)**
  * Developed browser extension with background service worker (`background.js`).
  * Implemented `activeTab`, `storage`, `notifications`, and dark-mode popup UI.
  * **Tests:** 4 manual verification flows.
* **Phase 10: False-Positive Mitigation & Real-Time Interception**
  * Solved false alarms on ChatGPT, GitHub, and YouTube using `trusted_domains.txt` and `trusted_loader.py`.
  * Implemented pre-navigation interception using `chrome.webNavigation.onBeforeNavigate`.
  * Created `analyzing.html`, `warning.html` (with Go Back / Continue Anyway), and safe green toasts.
  * **Total Test Suite:** 212 automated tests passed with 100% pass rate.

---

# 6. REAL BUGS, ERRORS, AND CHALLENGES OCCURRED
*(Examiners LOVE this! It proves you actually programmed the system yourself.)*

### Bug 1: The False Positive Crisis on Complex Modern Websites
* **The Error:** When testing real URLs like `https://chatgpt.com/c/uuid-1234`, `https://github.com/microsoft/vscode`, or `https://youtube.com/watch?v=dQw4w9WgXcQ`, the system incorrectly classified them as **SUSPICIOUS**.
* **Root Cause:** Modern web apps use long URLs, complex query parameters, GUIDs, and high-entropy hashes. Naive lexical analysis saw high Shannon entropy ($>3.5$) and URL length ($>75$), triggering heuristic risk points.
* **How We Solved It:** 
  1. We engineered a Subdomain-Aware **Trusted Domain Loader** (`trusted_loader.py`) backed by `trusted_domains.txt`. It inspects domains in $O(1)$ time and grants immediate SAFE bypass for verified platforms.
  2. We wrote `augment_false_positives.py` to inject real-world complex SaaS URLs into the training split so the Random Forest model learned that deep path tokens on benign root domains are safe.
  3. Logged and verified in `false_positive_report.csv`: ChatGPT, GitHub, and YouTube all moved from SUSPICIOUS to **SAFE (VERY_HIGH confidence)**.

### Bug 2: SHAP Calculation Was Too Slow for Real-Time Browsing
* **The Error:** In initial Phase 7 integration, browser navigation felt sluggish; opening any link had an unbearable delay of 300 to 500 milliseconds.
* **Root Cause:** The `shap.TreeExplainer` library by default runs full additivity checks (`check_additivity=True`) to verify mathematical convergence for every single tree.
* **How We Solved It:** In `xai/explainer.py`, we set `check_additivity=False` and implemented singleton explainer caching (`_EXPLAINER_CACHE`). This slashed explanation time from 350ms down to **31.2 milliseconds**, meeting our <50ms real-time SLA.

### Bug 3: Browser History "Trapping" & Redirect Loops in Chrome
* **The Error:** When a user was blocked by `warning.html` and clicked "Go Back to Safety", the browser would navigate back to the original phishing link, which immediately got intercepted again, trapping the user in an infinite redirect loop!
* **Root Cause:** When `background.js` used `chrome.tabs.update()`, it pushed a new entry onto the browser session history stack. Clicking "Back" simply popped back to the intercepted URL.
* **How We Solved It:** We modified `background.js` and `analyzing.js` to use `window.location.replace()` via `chrome.scripting.executeScript`. This **replaces** the history entry instead of pushing a new one, allowing `chrome.tabs.goBack()` to return smoothly to the safe preceding website.

### Bug 4: Python 3.9 Type Union Syntax Crash
* **The Error:** Running tests on Python 3.9 threw: `TypeError: unsupported operand type(s) for |: 'type' and 'NoneType'` in `hybrid/decision_tree.py`.
* **Root Cause:** Line 3 used modern PEP 604 pipe syntax (`tuple[str | None, ...]`), which is native to Python 3.10+ but unsupported in Python 3.9 unless `from __future__ import annotations` is imported.
* **How We Solved It:** Documented Python 3.10+ as the recommended execution runtime and added future annotations support.

### Bug 5: Heuristic Engine Alone Was Failing (Low Recall)
* **The Error:** When we ran `hybrid/evaluation/accuracy_comparison.py`, the standalone heuristic engine scored only **50.41% accuracy**!
* **Root Cause:** Attackers frequently create short, clean-looking phishing URLs on generic domains without hyphens or obvious keywords. Heuristics alone missed them (false negatives), while flagging long safe URLs (false positives).
* **How We Solved It:** We created the **Hybrid Consensus Engine**. We gave Random Forest 70% weight for statistical pattern recognition and Heuristics 30% weight for structural sanity checks, accompanied by the 6 override rules in `decision_tree.py`. This yielded **98.54% hybrid accuracy** with zero black-box obscurity.

---

# 7. THE MACHINE LEARNING & XAI DEEP DIVE
*(How to explain the AI concepts simply)*

### 1. What features does PhishScope extract?
PhishScope extracts **21 features** grouped into 4 categories:
1. **Structural Features:** URL length, dot count, hyphen count, subdomain depth, token count.
2. **Mathematical Features:** Shannon Entropy of the domain, digit-to-letter ratio.
3. **Security / Identity Features:** Raw IP address usage, HTTPS presence, embedded credentials (`@`).
4. **Threat Intelligence Features:** High-risk TLD points (e.g., `.tk`, `.xyz`), Brand mismatch (e.g. `paypal.com.evil.com`), ASCII lookalike / homoglyph presence (`g00gle.com`), and Punycode encoding (`xn--...`).

### 2. What were the top 3 most decisive features?
From our exact Gini feature importance analysis (`feature_importance.csv`):
1. **`url_length` (35.86% importance):** Phishing URLs are overwhelmingly longer due to token stuffing, redirection parameters, and deep paths.
2. **`subdomains` (20.83% importance):** Phishing sites nest multiple subdomains to disguise the root registrar.
3. **`entropy` (17.65% importance):** Random alphanumeric strings generated by phishing kits exhibit significantly higher character unpredictability.

### 3. What is Shannon Entropy?
> *"Entropy measures the randomness or unpredictability of characters in a string. In English, normal company names like `apple.com` or `google.com` have repeated vowels and predictable consonants (Entropy < 3.0). Phishing domains like `xk92mz-portal.xyz` use pseudo-random strings to evade domain filters, resulting in high entropy ($> 3.5$)."*

### 4. What is SHAP (Explainable AI)?
> *"SHAP comes from cooperative game theory (Shapley values). Think of the ML model's prediction as a team winning a game. SHAP calculates the exact individual contribution of each 'player' (feature) toward the final score. 
> For example, SHAP tells us: 'URL length added +0.32 toward the Phishing verdict, Suspicious TLD added +0.21, while having HTTPS subtracted -0.05'. We take the top 3 positive contributors and translate them into human sentences."*

---

# 8. THE BROWSER EXTENSION & REAL-TIME INTERCEPTION

### Why is Chrome Manifest V3 significant?
Google deprecated Manifest V2 to improve security and performance. Under Manifest V3:
* Persistent background pages are **banned**; only ephemeral **Service Workers** are allowed.
* Remotely hosted code is **prohibited**; all logic must be bundled locally.
* PhishScope is 100% Manifest V3 compliant: `background.js` runs as an event-driven service worker and communicates with our local Flask API via asynchronous `fetch()`.

### How does Zero-Click Interception work?
1. The service worker listens to `chrome.webNavigation.onBeforeNavigate`.
2. When the user clicks a link, *before Chrome opens a socket or downloads HTML*, the extension intercepts the event.
3. It immediately updates the tab to internal `analyzing.html`.
4. If the Flask API returns `DANGEROUS`, the tab is redirected to `warning.html` and a Chrome system notification is displayed.
5. The malicious server never receives an HTTP connection, protecting the user from **drive-by downloads** and **zero-day browser exploits**.

---

# 9. TOP 30 EXPECTED VIVA-VOCE QUESTIONS & PERFECT MODEL ANSWERS

### Category A: General & Conceptual Questions

#### Q1: What is the main objective of PhishScope?
**Answer:** PhishScope is an automated, real-time phishing detection system that protects web browser users from credential theft and malicious links. It combines lexical feature extraction, rule heuristics, supervised machine learning (Random Forest), and SHAP Explainable AI, all packaged into a Chrome Manifest V3 extension that blocks threats before web pages load.

#### Q2: What is the difference between URL-based detection and Content-based detection?
**Answer:** Content-based detection inspects the downloaded HTML, images, and JavaScript of a web page. This requires downloading the page first (exposing the user to drive-by malware) and adds heavy latency. URL-based detection (used in PhishScope) inspects only the uniform resource locator string *before* connecting, providing zero-exposure, privacy-preserving, and ultra-fast (<40ms) protection.

#### Q3: Why did you not just use Google Safe Browsing?
**Answer:** Google Safe Browsing is a centralized blacklist. Studies show that modern phishing campaigns launch ephemeral domains that stay active for less than 4 hours. Because blacklists rely on human reporting and crawling, they have an update delay of 12 to 24 hours. PhishScope uses machine learning to catch **zero-hour phishing** the moment it appears.

---

### Category B: Machine Learning & Mathematics

#### Q4: What machine learning algorithm did you use, and why?
**Answer:** We benchmarked four models: Random Forest, Decision Tree, Logistic Regression, and Linear SVM. We selected **Random Forest** (`n_estimators=100`) because ensemble bagging reduces variance, prevents overfitting, handles non-linear feature interactions seamlessly, and achieved a peak testing accuracy of **98.55%** and ROC-AUC of **99.59%**.

#### Q5: What dataset did you use for training?
**Answer:** We used a combined corpus of **115,213 raw URLs**: 65,213 verified phishing URLs from **PhishTank** and 50,000 verified legitimate URLs from global top registries. After cleaning, deduplication, and class balancing, we produced an exact **99,996 URL dataset** split into **79,996 training URLs** (80%) and **20,000 testing URLs** (20%).

#### Q6: What are your key evaluation metrics?
**Answer:** On the 20,000 unseen test URLs:
* **Accuracy:** 98.55%
* **Precision:** 98.99%
* **Recall:** 98.09%
* **F1-Score:** 98.54%
* **ROC-AUC:** 99.59%

#### Q7: In phishing detection, is Precision more important or Recall?
**Answer:** Both are critical, but in consumer cybersecurity, **Recall is paramount** because a False Negative means a user visits a phishing site and gets their bank credentials stolen. However, excessive False Positives cause "warning fatigue" where users disable the tool. PhishScope maintains 98.09% recall while using our trusted domain filter to push precision to 98.99%.

#### Q8: What is a Confusion Matrix, and what were your results?
**Answer:** A Confusion Matrix cross-tabulates actual vs. predicted labels. On our 20,000 test set:
* **True Negatives (Safe correctly identified):** 9,901
* **False Positives (Safe misclassified as phishing):** 99 (less than 1%)
* **False Negatives (Phishing missed):** 191
* **True Positives (Phishing caught):** 9,809

#### Q9: What is the mathematical formula for Shannon Entropy?
**Answer:** 
$$H(X) = -\sum_{i=1}^{n} P(x_i) \log_2 P(x_i)$$
Where $P(x_i)$ is the probability of occurrence of character $x_i$ in the domain name. If $H(X) > 3.5$, it indicates high character randomness.

#### Q10: How does Punycode detection work?
**Answer:** Attackers use Internationalized Domain Names (IDN) with Cyrillic or Greek letters that look identical to Latin letters (e.g. Cyrillic 'а' in `pаypal.com`). The browser encodes this using Punycode starting with the `xn--` prefix. PhishScope inspects the domain for `xn--`, decodes it, and flags it as a potential homoglyph spoofing attack.

---

### Category C: Explainable AI & Hybrid Engine

#### Q11: What is Explainable AI (XAI) and why is it needed here?
**Answer:** Standard machine learning classifiers are "black boxes"—they output a classification without explaining why. If a security analyst or user sees a site blocked, they need to know the rationale. PhishScope uses **SHAP (SHapley Additive exPlanations)** to calculate exact mathematical feature attributions and translates them into plain human sentences.

#### Q12: How does the Hybrid Consensus Engine work?
**Answer:** It combines the heuristic score (0–100) and ML probability via a weighted blend:
$$\text{Hybrid Score} = \text{round}(0.3 \times \text{Heuristic} + 0.7 \times (\text{ML Probability} \times 100))$$
It then executes a 6-rule decision tree (`hybrid/decision_tree.py`). For example, if ML flags phishing but the heuristic score is $\le 15$ with no corroborating evidence, it prevents a false positive by keeping the verdict SAFE.

#### Q13: What happens when the Heuristic engine and ML model disagree?
**Answer:** 
* If Heuristic says SAFE but ML says PHISHING with low corroboration, we classify it as **SAFE** (preventing false alarms on benign apps).
* If ML has very high confidence ($>95\%$) and corroborating heuristics ($>15$), we escalate to **SUSPICIOUS** or **DANGEROUS**.
* If Heuristic detects extreme danger ($>35$ points) but ML says SAFE, the system flags it as a novel zero-day threat and marks it **DANGEROUS** (Medium confidence).

---

### Category D: System Architecture & Web Technologies

#### Q14: Where is the database in your project?
**Answer:** *(Recite the answer from Section 4)*: PhishScope uses an in-memory and file-based persistence architecture for ultra-low latency (<0.12ms lookup) and client privacy (`chrome.storage.local`), avoiding the 30–50ms overhead of relational databases.

#### Q15: What technologies are used in the Frontend?
**Answer:** React 19, Vite (build tool), Tailwind CSS (styling), Axios (API communication), React Router (page routing), and Recharts (SVG data charts).

#### Q16: What technologies are used in the Backend?
**Answer:** Python 3.10, Flask (WSGI REST API), Flask-CORS, Scikit-Learn, SHAP, Pandas, NumPy, and Joblib.

#### Q17: What API endpoints exist in your Flask service?
**Answer:** 
1. `GET /`: Returns service metadata, version (0.3), author (Sian Julian), and status.
2. `GET /health`: Health-check endpoint returning `{"status": "healthy"}`.
3. `POST /analyze`: Main endpoint receiving `{"url": "...", "benchmark": bool}` and returning heuristic, ML, hybrid, and XAI explanation JSON objects.

#### Q18: What HTTP status codes does your API handle?
**Answer:** 
* `200 OK`: Successful analysis.
* `400 Bad Request`: Missing or malformed URL.
* `404 Not Found`: Unknown endpoint.
* `405 Method Not Allowed`: Invalid HTTP method.
* `415 Unsupported Media Type`: Payload is not `application/json`.
* `500 Internal Server Error`: Unhandled server exception.

---

### Category E: Chrome Extension & Interception

#### Q19: How does the Chrome extension intercept web traffic?
**Answer:** It uses the `chrome.webNavigation.onBeforeNavigate` API in `background.js`. When a navigation starts, it checks if the URL is in our temporary whitelist. If not, it redirects the tab to `analyzing.html`, aborting the outbound HTTP connection while it queries `POST /analyze`.

#### Q20: What are the permissions requested in `manifest.json`?
**Answer:** `activeTab`, `tabs`, `storage` (for history), `notifications` (for OS alerts), `webNavigation` (for interception), and `scripting` (for toast injection).

#### Q21: What happens if the user visits a safe site like Google or Wikipedia?
**Answer:** The domain is recognized by `trusted_loader.py` in 0.1ms. The API returns SAFE with a score of 0. The extension injects a sleek green toast at the top-right corner saying *"TRUSTED WEBSITE — Verified by PhishScope"* and allows the page to load without delay.

#### Q22: What happens if the user visits a dangerous phishing link?
**Answer:** The API returns DANGEROUS. The extension fires a native Chrome desktop notification (*"Threat Blocked!"*) and redirects the tab to `warning.html`. The page clearly displays the threat score, detected reasons, and provides two buttons: "Go Back to Safety" and "Continue Anyway".

#### Q23: What does the "Continue Anyway" button do?
**Answer:** If an advanced user insists on visiting the link, clicking "Continue Anyway" sends a message to `background.js`, which adds that exact URL to an in-memory `TEMP_WHITELIST` set and allows navigation. It is cleared once the browser restarts.

---

### Category F: Advanced & Defense Questions

#### Q24: What are the limitations of PhishScope?
**Answer:** 
1. It is purely lexical: it analyzes the URL string, not the rendered DOM or page images.
2. It currently requires a local Python Flask server running on `localhost:5000`.
3. The trusted domain list is currently static and updated via file rather than dynamic cloud feeds.

#### Q25: How would an attacker try to bypass PhishScope?
**Answer:** An attacker could compromise an existing legitimate domain and host phishing deep in a sub-path, or use shortened URLs (e.g. `bit.ly`). In future work, we plan to implement unshortening of redirect URLs and visual screenshot analysis using CNNs.

#### Q26: Why didn't you use Deep Learning (like CNN or LSTM)?
**Answer:** We evaluated this: deep learning on character sequences requires significant GPU resources, has high inference latency (>150ms), and is notoriously difficult to interpret with SHAP. Random Forest trained on 21 engineered lexical features runs in **1.8 milliseconds** on standard CPUs and achieves **98.55% accuracy**, making it far more practical for consumer endpoints.

#### Q27: How did you test your project?
**Answer:** We executed **212 automated tests** across all 10 phases using `pytest` and `vitest`. This included unit tests for feature extraction, heuristic scoring, API routes, dataset pipelines, machine learning predictions, hybrid rules, and SHAP formatting, alongside manual end-to-end tests for the Chrome extension.

#### Q28: How fast is the end-to-end analysis?
**Answer:** 
* Trusted domain bypass: **0.12 ms**
* Feature extraction: **3.82 ms**
* Heuristic scoring: **0.45 ms**
* Random Forest prediction: **1.84 ms**
* SHAP explanation: **31.20 ms**
* **Total pipeline latency: ~36.97 milliseconds**, well below human perception thresholds (100ms).

#### Q29: Can PhishScope run offline?
**Answer:** Yes! Because the Random Forest model (`.pkl`), brand lists, and heuristic rules reside locally on the host machine, PhishScope can analyze URLs and protect the user completely offline without internet connectivity.

#### Q30: What is your plan for future enhancement?
**Answer:** 
1. Convert the Random Forest model to **ONNX Runtime Web**, allowing the Chrome extension to execute inference directly inside JavaScript without requiring a Python backend.
2. Add visual layout matching using computer vision to compare cloned web pages against legitimate brand layouts.
3. Introduce an enterprise dashboard with centralized telemetry.

---

# 10. PRESENTATION SLIDE-BY-SLIDE OUTLINE & DEMO SCRIPT

### Suggested 10-Slide Structure:
* **Slide 1: Title Slide**
  * *Title:* PhishScope — Intelligent Hybrid Phishing Detection Platform
  * *Subtitle:* Powered by Machine Learning, Explainable AI (SHAP), and Manifest V3 Navigation Interception
  * *Student Name, Guide, College & Dept.*
* **Slide 2: Problem Statement & Motivation**
  * The rise of zero-hour phishing (lifespan < 4 hours).
  * Limitations of blacklists (slow) and black-box ML (unexplainable, high false positives).
* **Slide 3: Proposed Architecture**
  * Display the 4-tier architecture diagram: Client Extension $\rightarrow$ Flask API $\rightarrow$ Hybrid Consensus Engine $\rightarrow$ XAI & Reference Data.
* **Slide 4: Feature Engineering (21 Features)**
  * Breakdown: Length, Subdomains, Shannon Entropy, TLD Risk, Brand Mismatch, Punycode, Homoglyphs.
  * Highlight top 3 features (Length 35.8%, Subdomains 20.8%, Entropy 17.6%).
* **Slide 5: Machine Learning & Evaluation**
  * 100k balanced URL dataset (PhishTank + Legitimate).
  * Benchmark table: Random Forest vs. Decision Tree vs. Logistic Regression vs. SVM.
  * Key Metrics: **98.55% Accuracy, 98.99% Precision, 98.09% Recall, 99.59% ROC-AUC**.
* **Slide 6: Explainable AI (SHAP) & Human Explanations**
  * Why explainability matters in cybersecurity.
  * How SHAP values are mapped into Basic, Advanced, and Technical English sentences.
* **Slide 7: Hybrid Decision Consensus & False Positive Reduction**
  * Why Heuristics alone scored 50.41% vs. Hybrid 98.54%.
  * How ChatGPT, GitHub, and YouTube false positives were resolved via `trusted_loader.py`.
* **Slide 8: Chrome Extension Manifest V3 Implementation**
  * Ephemeral Service Worker architecture.
  * Synchronous pre-navigation interception (`onBeforeNavigate`), `analyzing.html`, and `warning.html`.
* **Slide 9: Experimental Results & Benchmarks**
  * Confusion Matrix and ROC curve.
  * Latency table: Total backend execution time = **36.97 ms**.
* **Slide 10: Conclusion & Future Scope**
  * Summary of contributions.
  * Future directions: In-browser ONNX inference and visual layout matching.

---

### The 3-Minute Live Demo Script:
1. **Show Backend Running:**
   * Open terminal: `python backend/app.py` $\rightarrow$ Show Flask starting cleanly on port 5000.
2. **Show Frontend Web Analyzer:**
   * Open `http://localhost:5173` in browser.
   * Enter a safe link: `https://google.com` $\rightarrow$ Instant SAFE badge, 0 score.
   * Enter a malicious spoof link: `http://secure-paypal-login-verify.xyz` $\rightarrow$ Immediate DANGEROUS badge, score ~86, and explain the SHAP cards showing why.
3. **Show Real-Time Browser Extension:**
   * Open Chrome with the loaded unpacked extension.
   * Navigate to a safe site (`github.com`) $\rightarrow$ Point out the green verification toast at top right.
   * Navigate to a simulated phishing link $\rightarrow$ Show how Chrome stops immediately, displays `analyzing.html`, triggers a desktop notification, and locks on `warning.html`.
   * Click "Go Back to Safety" to demonstrate seamless return without history trapping!

---
*(End of Presentation & Viva Guide — You are 100% prepared to excel in your presentation tomorrow!)*
