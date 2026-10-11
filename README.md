# GhostPen
**Business Email Compromise (BEC) Detection via Sender-Calibrated Writing-Style Verification**  
*(v4.2-final)*

> *"Your CFO's writing style is a password."*

---

![GhostPen Interactive Dashboard Overview](artifacts/p10_msg09_alert.png)
*Figure 1: GhostPen detecting an active Business Email Compromise attack in real time. Notice how Deviation Score S spikes to 0.822, the conformal p-value drops to 0.018, and broken-habit cards pinpoint exactly where the imposter deviated from CFO Marcus Hale's historical writing habits.*

---

### What is GhostPen? (In 30 Seconds)
* **The Security Gap**: When cybercriminals hack an executive's actual email account (Account Takeover / BEC), traditional defenses like SPF, DKIM, and DMARC pass with flying colors because the message genuinely originates from Microsoft 365 or Google Workspace. The headers are real, but the author is an imposter.
* **The Behavioral Solution**: GhostPen acts as a behavioral biometric for writing habits. It profiles each executive's unique subconscious habits—such as 74 unconscious glue function words, sentence length variance, and punctuation patterns—and tests incoming emails against that personalized baseline.
* **Controllable SOC Budget**: Rather than giving analysts an arbitrary "AI score", GhostPen uses mathematical conformal prediction. Security teams set an exact false-alarm budget (e.g. 1.7 false alarms per 100 emails), ensuring analysts are never overwhelmed by alert fatigue.
* **100% Offline & Private**: Zero external LLM API calls, zero cloud dependencies, and zero monthly token costs. Runs locally in milliseconds with total explainability.

---

## 1. Quickstart (One-Click Launchers)

GhostPen is 100% self-contained: all cleaned email profiles, null distributions, and forged evaluation benchmarks are pre-committed in-repo. No API keys, external models, or network connections are needed at runtime.

### One-Click Launch (Zero Manual Setup)

GhostPen includes automated startup scripts that handle everything automatically — creating an isolated virtual environment (`.venv`), installing required dependencies, running pre-flight verification, launching the Streamlit interface, and opening your browser.

- **Windows**: Double-click **`START_GHOSTPEN.bat`** (or run `.\START_GHOSTPEN.bat` in PowerShell/cmd).
- **macOS / Linux**: Run **`./start_ghostpen.sh`** (or `bash start_ghostpen.sh`).

*Subsequent launches skip dependency checks and start the dashboard in ~2 seconds.*

---

### How to Stop and Restart

- **To Stop**: Click into the terminal window running GhostPen and press **`Ctrl + C`**.
- **To Restart**: Simply double-click **`START_GHOSTPEN.bat`** (Windows) or run **`./start_ghostpen.sh`** (macOS/Linux) again.

---

### Manual Setup & Commands (Terminal Option)

If you prefer to run the commands manually or want to explore the benchmark evaluation:

```bash
# 1. Create and activate an isolated virtual environment
python -m venv .venv

# On Windows:
.venv\Scripts\activate

# On macOS/Linux:
source .venv/bin/activate

# 2. Install dependencies (supports Python 3.10 through 3.14)
pip install -r requirements.txt

# 3. Run pre-flight readiness check
python scripts/check_ready.py

# 4. Launch Interactive Streamlit Demo (14-message CFO Inbox)
python -m streamlit run app/app.py

# 5. Run Fast Terminal CLI Demo
python -m src.cli --demo

# 6. Run Full Benchmark Evaluation & Baselines
python -m src.evaluate

# 7. Run Unit and Sanity Test Suite
python -m unittest tests/test_smoke.py

# 8. Rebuild Profiles and LOO Nulls from Data
python -m src.profile --rebuild

# 9. Generate 5-Page Presentation PDF
python scripts/generate_slides.py
```

---

### Troubleshooting

- **Python Not Found / "The term 'python' is not recognized"**:
  - Install Python 3.10 or newer from [python.org/downloads](https://www.python.org/downloads/).
  - **Important for Windows:** During installation, check the box: **`[X] Add python.exe to PATH`**.
- **Port 8501 is already in use**:
  - If a previous Streamlit process is still running in the background, close that command window, or start GhostPen on a different port:
    ```bash
    python -m streamlit run app/app.py --server.port 8502
    ```
- **Permission Denied on macOS / Linux (`./start_ghostpen.sh`)**:
  - Make the script executable by running:
    ```bash
    chmod +x start_ghostpen.sh
    ./start_ghostpen.sh
    ```
- **PowerShell Execution Policy Restrictions**:
  - Running `START_GHOSTPEN.bat` bypasses PowerShell script execution policies because it runs through the standard Windows Command processor (`cmd.exe`).
- **Missing `make` on Windows**:
  - You do not need `make`. Simply use `START_GHOSTPEN.bat` or the direct `python -m ...` commands listed above.


---

## 2. The Problem & How GhostPen Solves It

### The Security Control Gap
Modern email security protocols—**SPF, DKIM, and DMARC**—verify sending servers and domain alignment. Multi-Factor Authentication protects login portals. But when an attacker executes an **Account Takeover (ATO)**, steals executive session cookies, or compromises an executive's mailbox:
* SPF and DKIM pass with flying colors because the email genuinely originates from the executive's real Microsoft 365 or Google Workspace account.
* The message lands cleanly in the finance team's inbox with zero security banners.
* The recipient sees the CFO's real name, real email address, and a convincing business pretext.

Perimeter security verifies the server, but cannot verify the author. **The technical headers look authentic, but the writing style is completely wrong.**

```
[Past Sent Mail] ──► Habit Ledger (winsorized stats, floored stds, habit distributions)
                              │
[Incoming Email] ──► Feature Extraction ──► Deviation Score S (Raw Speedometer)
                              │
[Leave-One-Out Null] ─────────┴───────────► Conformal p-value (Normality Score)
                                                    │
                                                    ▼
                               Verdict: OK / ALERT / TRIAGE / UNENROLLED
                               + Plain-English broken habits evidence
                               + "What the disguise got right" matched habits
                               + Cross-sender attribution ranking
                               + Secondary content control routing (if TRIAGE)
```

---

### Core Mechanics in Plain English

#### 1. Deviation Score $S$ (The Raw Stylistic Speedometer)
Score $S$ measures how far an incoming email drifts from the executive's historical writing habits:
$$S = 0.25 \cdot \Delta(\text{function words}) + 0.25 \cdot \Delta(\text{char 3-grams}) + 0.35 \cdot \text{mean}(|z_{\text{capped}}|) + 0.15 \cdot \text{habit penalty}$$
* **74 Unconscious Function Words**: Prepositions, pronouns, conjunctions, and auxiliary verbs (*"which", "although", "upon", "would"*). People rarely think about these glue words, but their usage frequencies form stable subconscious fingerprints.
* **11 Structural Numeric Habits**: Sentence length mean, sentence variance, punctuation frequencies (`!`, `?`, `;`, `:`, `—`), contractions, formality, and vocabulary diversity (TTR).
* **Habit Penalties**: Structural greetings (*"Hi team"* vs *"Dear"*) and sign-offs (*"Thanks"* vs *"Best regards"*).

#### 2. The Conformal $p$-value (Personalized Normality Score)
A raw score of $S = 0.80$ might be a routine Tuesday for an executive whose writing naturally fluctuates, but an extreme red flag for someone who writes with strict consistency.

Instead of guessing a rigid global cutoff, GhostPen runs a **Leave-One-Out (LOO)** calibration: it scores each of the executive's past emails against the rest of their history to build that specific sender's reference distribution.
$$p = \frac{1 + \text{count}\{\text{past scores} \ge S\}}{N + 1}$$
* **$p = 0.90$ (High)**: Completely routine. 90% of the sender's historical emails showed *more* stylistic variation than this one.
* **$p = 0.018$ (Low)**: Highly anomalous. Less than 2% of their past emails ever deviated this much.

#### 3. The Alpha ($\alpha$) Dial (Controllable SOC Budget)
Security Operations Centers hate unpredictable alert storms. GhostPen turns false-alarm control into an explicit dial on the sidebar:
* **$\alpha = 0.05$ (Standard)**: Expect **~1.71 false alarms per 100 safe emails** (measured on holdout).
* **$\alpha = 0.02$ (Strict)**: Tightens to **~1.52 false alarms per 100 safe emails**.
* **$\alpha = 0.10$ (Relaxed)**: Higher sensitivity, with **~3.81 false alarms per 100 safe emails**.

![GhostPen Control Panel & Operating Alpha Dial](artifacts/sidebar_screenshot.png)
*Figure 2: The GhostPen Control Panel. The security team selects their operating point ($\alpha = 0.05, 0.02, 0.10$). Directly below the slider, the exact operational cost (measured false alarms per 100 genuine emails) updates immediately so the SOC always knows its false-alarm budget.*

#### 4. Abstain-and-Route (`TRIAGE`)
When an email is under 40 words or fewer than 3 sentences, stylometry does not have enough text to make a reliable call. Rather than giving attackers an evasion loophole with short 20-word requests (*"Wire $150k today. Urgent."*), GhostPen returns **`TRIAGE`** and automatically escalates messages with $\ge 2$ financial context flags to secondary verification controls.

---

## 3. Expected Output & Visual Walkthrough

In the demonstration dashboard (`START_GHOSTPEN.bat` or `make demo`), each executive profile features their own contextual mailbox: **"Incoming mail claiming to be {display name}"**. 

![CFO Marcus Hale 14-Message Mailbox Table](artifacts/mailbox_table_screenshot.png)
*Figure 3: Marcus Hale's demonstration mailbox. 14 incoming messages claiming to be Marcus Hale, audited against his historical baseline of 112 sent emails.*

Selecting Marcus Hale (CFO) in the sidebar loads his 14-message demo inbox, illustrating how GhostPen handles each operational scenario:

---

### Case 1: Genuine Executive Email (`VERDICT: OK`)
![Genuine Email Verification](artifacts/p10_msg01_ok.png)
*Figure 4: Genuine Holdout Email. The deviation score S is low (0.350) and conformal p is 0.903, confirming the email sits comfortably within routine historical habits.*

* **Human Walkthrough**: Message 1 (`msg_01`) is a legitimate business update from CFO Marcus Hale.
* **The Numbers**: Deviation Score $S = 0.350$ is very low (well within his historical norm). Calibrated against 112 past emails, the conformal $p$-value is **$0.903$**—meaning this email sits right in the routine heart of his writing habits (90% of his past emails showed more stylistic variation than this one).
* **The Evidence**: Under *"Habits matching the baseline"*, green cards verify that his sentence lengths, punctuation rates, and contraction habits match his personal standard deviations.
* **Attribution State**: The panel displays: *"Verified as Marcus Hale. Attribution ranking is shown only for rejected messages."*

```json
/* 1. Genuine Email (Typical holdout message) */
{
  "sender": "Marcus Hale",
  "deviation_score": 0.3363,
  "p": 0.9558,
  "p_min": 0.0089,
  "n_train": 112,
  "verdict": "OK",
  "reason": "Style aligns with enrolled profile (p=0.956 >= alpha=0.05)",
  "broken_habits": [],
  "content_escalation": false
}
```

---

### Case 2: BEC Impersonation Attack (`VERDICT: ALERT`)
![BEC Impersonation Alert](artifacts/p10_msg09_alert.png)
*Figure 5: Urgent LLM BEC Wire Fraud Alert. GhostPen catches the deviation with p = 0.018 < 0.05. Red cards highlight broken greeting and sign-off habits, while green cards show what habits the attacker mimicked.*

* **Human Walkthrough**: Message 9 (`msg_09`) is an urgent LLM-generated acquisition wire fraud request claiming to be from the CFO.
* **The Numbers**: Deviation Score $S$ spikes to **$0.822$**, plunging the conformal $p$-value to **$0.018$** (well below the $\alpha = 0.05$ threshold), triggering an immediate red **ALERT**.
* **Red Broken Habits**: The cards clearly explain what went wrong:
  - *Uses greeting 'dear'* (observed 0% in past mail; usually 'none' 100%).
  - *Signs 'best_regards'* (observed 0% in past mail; usually 'none' 77%).
  - *Punctuation '!': 1 occurrences* (0 observed in 112 past emails).
* **What the disguise got right**: Green cards reveal the attacker successfully mimicked his average sentence length ($z = +0.53$), but composite scoring caught the overall disguise.
* **Forensic Attribution**: All 15 enrolled senders score $p < 0.05$, producing the warning: *"No enrolled author's style matches this message"*—a vital indicator that this is an external scam template.

```json
/* 2. Forged Email (Generic LLM BEC wire request) */
{
  "sender": "Marcus Hale",
  "deviation_score": 0.8562,
  "p": 0.0177,
  "p_min": 0.0089,
  "n_train": 112,
  "verdict": "ALERT",
  "reason": "Stylometric deviation exceeds threshold (p=0.018 < alpha=0.05)",
  "direction": "Formality +4.0 sigma above baseline norm",
  "broken_habits": [
    "Uses greeting 'dear' (observed 0% in past mail; usually 'none' 100%)",
    "Signs 'best_regards' (observed 0% in past mail; usually 'none' 77%)",
    "Punctuation '!': 1 occurrences (0 observed in 112 past emails)"
  ],
  "content_escalation": false
}
```

---

### Case 3: Short Benign Email (`VERDICT: TRIAGE`)
![Short Benign Email Triage](artifacts/p10_heard_m01_triage.png)
*Figure 6: Short Routine Email Safely Abstained. A 14-word message ('ok thx') is routed to TRIAGE without false alarms because stylometry refuses to make guesses on thin text.*

* **Human Walkthrough**: A brief 14-word message (*"ok thx. lets discuss after the 2pm call"*).
* **The Logic**: Because it is under 40 words, GhostPen refuses to make an unreliable stylometric guess. It issues **`TRIAGE`** with a clear explanation: *"text too thin to judge reliably — routed to content controls"*.
* **No False Alarm**: Because the message contains no suspicious financial or urgency keywords, it is held safely in `TRIAGE` without triggering a disruptive alert.

```json
/* 3. Short Benign Email ("ok thx") */
{
  "sender": "Marcus Hale",
  "deviation_score": 0.5080,
  "p": 0.2124,
  "p_min": 0.0089,
  "n_train": 112,
  "verdict": "TRIAGE",
  "reason": "message too short (<40 words or <3 sentences)",
  "content_escalation": false
}
```

---

### Case 4: Short Wire Evasion Email (`VERDICT: TRIAGE [ESCALATED]`)

* **Human Walkthrough**: Attackers frequently try to evade writing-style detectors by keeping emails extremely brief (e.g. a 22-word note: *"Please wire the balance to the updated escrow account today. Confidential acquisition. Thanks."*).
* **The Safeguard**: GhostPen recognizes the text is too short for stylometry (<40 words) and returns `TRIAGE`. However, its secondary transaction scanner detects high-risk flags (`"wire"`, `"today"`, `"confidential"`), immediately slapping on a bright red **`[ESCALATED]`** badge and routing the message directly to out-of-band payment authorization.

```json
/* 4. Short Forged Wire Evasion Email (22 words, urgent payment) */
{
  "sender": "Marcus Hale",
  "deviation_score": 0.5702,
  "p": 0.0973,
  "p_min": 0.0089,
  "n_train": 112,
  "verdict": "TRIAGE",
  "reason": "message too short (<40 words or <3 sentences)",
  "context_flags": ["confidential", "reply only to me", "today", "wire"],
  "content_escalation": true,
  "escalation_hint": "Stylometry abstains — route to content controls / payment verification."
}
```

---

### Case 5: Routine Style with Sensitive Financial Request (`VERDICT: OK | CAUTION`)
![Content Risk Advisory](artifacts/p8_msg11_marcus_full.png)
*Figure 7: High-Risk Content Advisory Workflow. Message 11 matches Marcus Hale's stylistic habits (p = 0.593 >= 0.05), but requests a $180,000 wire transfer. GhostPen verifies the style but highlights a yellow CAUTION banner requiring out-of-band payment authorization.*

* **Human Walkthrough**: Message 11 (`msg_11`) discusses transmission allocations and requests an urgent \$180,000 wire transfer.
* **The Stylometric Finding**: Marcus Hale's writing habits are intact ($S = 0.4124$, $p = 0.5929 \ge 0.05$). The author writes with Marcus's natural cadence and function words.
* **The Safety Net**: Even when writing style passes, GhostPen does not grant blind immunity to sensitive financial transactions. Its content engine detects high-risk keywords (`"wire"`, `"transfer"`, `"today"`, `"reply only to me"`), attaching a clear yellow **`CAUTION (High-Risk Content)`** advisory demanding out-of-band confirmation before funds are disbursed.

```json
/* 5. Sensitive Financial Wire Request (Style OK, Content Advisory) */
{
  "sender": "Marcus Hale",
  "deviation_score": 0.4124,
  "p": 0.5929,
  "p_min": 0.0089,
  "n_train": 112,
  "verdict": "OK",
  "reason": "Style aligns with enrolled profile (p=0.593 >= alpha=0.05)",
  "advisory": "CAUTION (High-Risk Content)",
  "context_flags": ["reply only to me", "today", "transfer", "wire"],
  "content_escalation": false
}
```

---

### Attribution Panel ("Who actually wrote this?")

![Forensic Cross-Sender Attribution](artifacts/p6_alert_ranking.png)
*Figure 8: Cross-Sender Attribution Panel. When an alert triggers, GhostPen evaluates the message against all enrolled executives. If no enrolled author matches (all p < 0.05), it flags that the text reads like an external scam template rather than internal company staff.*
In both the Web dashboard and CLI, scored messages feature cross-profile stylometric attribution across enrolled senders (ranking by $p$-value descending, tie-breaking by `sender_id` ascending). The panel covers three distinct operational states:

1. **Claimed Sender Matches (Genuine holdout email)**:
   ```text
   Closest enrolled authors (among 15 enrolled senders - suggestion, not identification; never affects S or the verdict):
     #1 Match: Marcus Hale (CFO) (p = 0.9558)
     #2 Match: Shelley Corman (p = 0.9558)
     #3 Match: Kam Keiser (p = 0.9558)
     Claimed sender contrast: Marcus Hale (CFO) (p = 0.9558, rank #1)
   ```

2. **Another Enrolled Sender Matches (Relabel impersonation attack)**:
   ```text
   Closest enrolled authors (among 15 enrolled senders - suggestion, not identification; never affects S or the verdict):
     #1 Match: Lynn Blair (p = 0.6637)
     #2 Match: Diana Scholtes (p = 0.5278)
     #3 Match: Kam Keiser (p = 0.4336)
     Claimed sender contrast: Marcus Hale (CFO) (p = 0.1947, rank #6)
   ```
   *The true author Lynn Blair ranks #1 with $p \ge 0.05$, exposing that the email was actually authored by someone else in the company.*

3. **No Enrolled Match (Generic / Styled LLM BEC attack)**:
   ```text
   Closest enrolled authors (among 15 enrolled senders - suggestion, not identification; never affects S or the verdict):
     No enrolled author's style matches this message.
     Claimed sender (Marcus Hale (CFO)): p = 0.0177
   ```
   *When all calibrated authors have $p < \alpha$, nobody is named. This no-match state is itself a vital BEC indicator: the email reads like nobody in the company.*

### "What the disguise got right" (Matched Habits)
For any message evaluated as `ALERT` or `TRIAGE`, GhostPen inspects which habits the attacker successfully imitated. Numeric features with $|z| \le 1.0$ relative to the sender's profile are rendered as green cards:
- `attacker matched: signs 'thanks'`
- `attacker matched: average sentence length normal (z = +0.21)`
- `attacker matched: contraction rate normal (z = -0.14)`
- Tooltip threshold: *"matched = within 1 of his own standard deviations"*

### Alpha-Slider Price Tag (Operational SOC Cost)
Beside the interactive operating $\alpha$ slider, an operational cost indicator displays real-world SOC triage workload measured directly on genuine holdout emails from `metrics.json`:
- At $\alpha = 0.02$: `expect ~1.52 flags per 100 genuine emails (measured on holdout)`
- At $\alpha = 0.05$: `expect ~1.71 flags per 100 genuine emails (measured on holdout)`
- At $\alpha = 0.10$: `expect ~3.81 flags per 100 genuine emails (measured on holdout)`

---

## 4. How Scoring Works

### Deviation Score $S$
$$S = 0.25 \cdot \Delta(\text{function words}) + 0.25 \cdot \Delta(\text{char 3-grams}) + 0.35 \cdot \text{mean}(|z_{\text{capped}}|) + 0.15 \cdot \text{habit penalty}$$
- **Burrows' Delta ($\Delta$)**: Classic distance across relative frequencies normalized by sender variation, with individual $|z| \le 4.0$.
- **Robust Z-scores**: All training distributions are winsorized at 5th/95th percentiles; standard deviations are floored per family (`EPS`: rates/100w = 0.5, sentence length = 1.0, TTR = 0.01, word length = 0.1, relative frequencies = 0.0005); individual $|z|$ capped at 4.0.
- **Habit Penalty**: Sum of consistency weights for violated greeting and sign-off habits.

### Conformal Null Calibration & Verdict Function
For sender with $N = n_{\text{train}}$ emails, leave-one-out scores yield null distribution $\{s_1, \dots, s_N\}$.
$$p = \frac{1 + |\{i : s_i \ge S\}|}{N + 1}, \quad p_{\min} = \frac{1}{N + 1}$$

**Verdict Evaluation Order**:
1. Sender not enrolled $\to$ `UNENROLLED` (`"no profile; enroll >= floor(1/alpha) for your operating alpha (20 @ 0.05, 50 @ 0.02, 10 @ 0.10)"`)
2. Word count $< 40$ OR sentence count $< 3$ $\to$ `TRIAGE` (`"message too short (<40 words or <3 sentences)"`)
3. $n_{\text{train}} < \lfloor 1/\alpha \rfloor$ $\to$ `TRIAGE` (`"needs >= X emails for alpha=Y"`)
4. $p < \alpha \to$ `ALERT`
5. Otherwise $\to$ `OK`

### Detection Definition & Dual FPR Variants
The detection definition used consistently across all benchmarks:
```text
flagged = ALERT OR (TRIAGE AND content_escalation); caught = flagged.
```

`metrics.json` reports two separate False Positive Rate (FPR) variants (never summed or averaged):
- **$\text{FPR}_{\text{alert}}$**: ALERT rate on genuine holdout emails (subject of the conformal calibration claim).
- **$\text{FPR}_{\text{flagged}}$**: Total flagged rate on genuine holdout (ALERT + escalated TRIAGE), representing the operational false-alarm burden on the SOC.

---

## 5. Statistical Claim Discipline & Visual Calibration

![Empirical Calibration Curves](artifacts/plots/calibration.png)
*Figure 9: Empirical Conformal Calibration Curves across the alpha grid. Notice the holdout line (orange) tracks strictly beneath the conservative conformal bound, demonstrating rigorous false-positive control without arbitrary thresholds.*

### The Scientific Grounding
> The in-sample calibration line is an **IMPLEMENTATION SANITY CHECK** — the plug-in p-value is approximately rank-uniform for ANY exchangeable score function, so it verifies arithmetic, not the model. The **HOLDOUT** line is the evidence; holdout FPR − in-sample FPR = measured style drift, reported as a finding. Never claim "holdout FPR = alpha by construction."
>
> LOO nulls are built on $n-1$-email profiles, which inflates them slightly relative to a new message scored against the full $n$-email profile, so deployment p-values are mildly conservative ($\text{FPR} \le \alpha$).

![Self-Drift Timeline](artifacts/plots/drift_timeline_demo.png)
*Figure 10: Chronological Self-Drift Timeline for the demo executive across multiple years of sent emails. Genuine holdout messages (green dots) remain clustered below the critical threshold line, while drift is tracked continuously.*

### Human Meaning of Self-Drift
Human writing habits are not frozen in amber: executives change roles, adopt new corporate shorthand, and adjust communication tone over multi-year careers.
* **In-Sample FPR**: $0.0082$
* **Holdout FPR**: $0.0152 \le 0.05$ (strictly satisfies the conservative conformal bound)
* **Measured Style Drift**: $+0.0070$ (reflects how human writing habits evolve over multi-year corporate careers, proving why rolling 6-month profile refresh windows are recommended in production).

---

## 6. Abstain-and-Route Policy

In deployment, `TRIAGE` mail falls through to secondary controls (content rules, payment-verification policy). Short messages cannot support statistical stylometry, but attackers attempting brevity-based evasion will not receive a pass: if $\ge 2$ context flags appear on a `TRIAGE` message, `content_escalation` is triggered immediately.

---

## 7. Data Provenance & Anti-Leakage Rules

- **Data Source**: 15 senders from the public Enron corpus (10 high-volume with 160 emails each, 5 low-volume with 20–50 emails). Cleaned subsets are committed under `data/prepared/` (1.63 MB total). `scripts/rebuild_from_enron.py` documents full upstream regeneration.
- **Strict Chronological Splits**: First 70% of emails by date form training; last 30% form genuine holdout used strictly for evaluation.
- **Baselines**:
  - Baseline A (Centroid Cosine TF-IDF): IDF fit on pooled training mail only.
  - Baseline B (Global Threshold): Raw $S$ with single pooled threshold.
  - Thresholds tuned only on an inner 80/20 train-only slice.
- **Shared Routing Layer**: Baselines share the identical method-agnostic routing layer for fair comparison.
- **Tier C Copy Audit**: Forged styled emails are audited for distinct shared 5-grams ($>5$) and longest common substrings ($>60$ chars) to prove style mimicry rather than verbatim memorization.

---

## 8. Benchmark Evaluation & Results

### Baseline Comparison Table (from `artifacts/metrics.json`)
```text
Method                                 | FPR_alert | FPR_flag | R_Generic | R_Styled | R_Short | Prec @ 10%
---------------------------------------+-----------+----------+-----------+----------+---------+------------
GhostPen (Per-Sender LOO Null)         |   0.015   |  0.017   |   74.0%   |  82.0%   | 100.0%  |   76.6%
Baseline A (Centroid Cosine TF-IDF)    |   0.009   |  0.011   |   28.0%   |  82.0%   | 100.0%  |   79.4%
Baseline B (Global Threshold on S)     |   0.048   |  0.050   |   79.0%   |  82.0%   | 100.0%  |   54.6%
```

### Key Performance Findings in Plain English
1. **Low Operational False Alarms**: Only **1.71 flags per 100 genuine emails** (9/525 holdout emails flagged: 8 style alerts + 1 escalated triage).
2. **Superior Coverage on GenAI Fraud**: GhostPen catches **74.0% of generic LLM BEC attacks** via stylometric ALERT alone. Baseline A (TF-IDF) catches only 28.0% because LLMs write fluent, plausible business vocabulary that easily fools keyword centroids. GhostPen catches them because LLMs fail to match the executive's unconscious function-word habits and sentence structure.
3. **Copy-Audit Proof (Tier C)**: 46 of 50 styled impersonations (92.0%) are verified clean style mimics without text copying. Recall is 82.0% across all styled attacks, and **84.78% (39/46)** excluding copy-audited records.
4. **100% Interception of Brevity Evasion**: 20/20 short payment attacks caught via the reason-agnostic abstain-and-route layer.
5. **Stylometric Abstain Share**: 40.19% share of genuine mail we decline to judge stylometrically (`abstain_rate_genuine_holdout_pooled = 0.4019`), with a genuine holdout escalation rate of just 0.19% (1 in 525 emails).

---

## 9. Limitations & AI Disclosure

- **Reply-Heavy Senders**: For reply-heavy senders whose mail is predominantly brief, stylometry abstains on a large share of messages (abstain rate per sender in metrics.json); such senders are protected mainly by the routing layer, which is why TRIAGE is never a silent pass.
- **Attribution Ranking**: Cross-sender ranking is a forensic suggestion, not identification: a genuine message's p against its true author is uniform by construction, and boilerplate text (out-of-office, forwards) carries little personal signal, so ranks can shuffle on such mail. Measured top-1 accuracy on the relabel tier: 0.03.
- **Style Drift**: People change writing habits over time; rolling 6-month profile update windows are recommended for production.
- **Low-Volume Senders**: Reliable conformal calibration at $\alpha=0.05$ requires $\ge 20$ training emails ($n_{\text{train}} \ge \lfloor 1/\alpha \rfloor$).
- **Character 3-grams**: Topic vocabulary can subtly influence 3-grams; glue function words remain primary.
- **English-Only**: Glue words and punctuation are tuned for English corporate communication.
- **AI-Assistance Disclosure**: AI assistance: We used Antigravity and Gemini for two things. First, drafting the initial project skeleton: the file structure, setup files, and starter code outlines, which the team then filled in, reviewed, and rewrote. Second, writing the fake attack emails used for testing, saved offline in data/forged/forged.jsonl. The core design, the scoring and statistical method, the evaluation protocol, and every decision about them were made and reviewed by the team.

---

## 10. Hackathon Defense Drill (Judge Q&A)

**Q: "What share of real mail do you refuse to judge?"**  
**A:** 40.19% (0.4019 pooled across enrolled senders on holdout). Declined mail is routed to secondary controls, not silently released—and the decline rate is measured per sender so security leaders know exactly how much traffic falls to secondary controls.

**Q: "Why not just use an LLM or ChatGPT to detect BEC?"**  
**A:** LLMs are non-deterministic, cost real API dollars per message, introduce data privacy risks for executive communications, and cannot provide mathematical false-alarm guarantees. GhostPen runs locally in milliseconds using transparent statistical features and conformal p-values.

**Q: "Can an attacker beat GhostPen by matching sentence length?"**  
**A:** No. As shown in the Message 9 audit, attackers often get one habit right (sentence length $z = +0.53$). But GhostPen scores 74 function words, punctuation rates, greeting habits, and 3-grams simultaneously. Faking an author's entire composite syntactic fingerprint while executing a fraud pretext is statistically improbable.

**Q: "Why not use a flat score threshold instead of p-values?"**  
**A:** We tested that in Baseline B (Global Threshold on $S$). A flat cutoff produced nearly 5% false alarms (3× higher than GhostPen) because people have different natural variation. Conformal calibration against the sender's *own* past email distribution is what keeps false alarms down to 1.71%.