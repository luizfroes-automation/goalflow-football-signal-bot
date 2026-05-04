# GoalFlow - Football Probability & Signal System

GoalFlow is a Telegram-based football analysis system designed to estimate goal probabilities using historical match data, contextual home/away performance and live monitoring logic.

The project was built using workflow automation with n8n and integrates external football APIs, Telegram messaging and rule-based decision models.

This is an academic computer science prototype focused on automation, API integration, probabilistic modelling and explainable decision-making.

---

## 📌 Overview

GoalFlow provides two main features:

1. **Pre-match Analysis**
   - Uses historical team data
   - Compares home team home performance against away team away performance
   - Estimates expected goals
   - Applies a Poisson probability model
   - Calculates Over 1.5, Over 2.5 and Over 3.5 probabilities
   - Predicts the most likely scorelines

2. **Live Monitoring**
   - Uses a simulated live monitoring model for the MVP
   - Applies time-window based logic
   - Estimates the probability of a goal occurring during specific match periods
   - Generates alerts based on pressure, chance quality, xG, match context and league intensity

Signals are sent through Telegram and can be stored for later analysis and performance evaluation.

---

## 🎯 Objectives

- Learn workflow automation using n8n
- Integrate external football data APIs
- Build a Telegram-based football analysis bot
- Create a rule-based probability model
- Apply expected goals and Poisson distribution
- Generate automated football goal alerts
- Validate user input and supported leagues
- Prepare the system for future data storage and performance analysis

---

## ⚙️ Tech Stack

- **n8n** - workflow automation
- **API-Football / API-Sports** - football data provider
- **Telegram Bot API** - user interaction and alerts
- **JavaScript Code Nodes** - custom logic and probability model
- **Google Sheets** - planned signal storage and analysis
- **GitHub** - documentation and version control

---

## 🔄 System Workflow

### Pre-match Analysis Flow

```txt
Telegram Message
↓
Extract Command
↓
Validate Input
↓
Search Teams
↓
Validate Teams
↓
Fetch Historical Fixtures
↓
Validate League-Team Combination
↓
Calculate Expected Goals
↓
Apply Poisson Model
↓
Send Analysis via Telegram
```

### Live Monitoring Flow

```txt
Telegram Message
↓
Extract Command
↓
Validate Input
↓
Live Monitoring Simulation
↓
Calculate Pressure and Quality Scores
↓
Estimate Goal Probability
↓
Generate Alert Decision
↓
Send Alert via Telegram
```

---

## 🤖 Telegram Commands

### `/analyse`

Runs a pre-match probability analysis.

Format:

```txt
/analyse Home Team vs Away Team | League
```

Example:

```txt
/analyse Chelsea vs Manchester City | Premier League
```

The first team is treated as the home team.  
The second team is treated as the away team.

The command returns:

- Home team home performance
- Away team away performance
- Expected goals estimate
- Poisson goal distribution
- Over 1.5, Over 2.5 and Over 3.5 probabilities
- Most likely scorelines

---

### `/monitor`

Runs the live monitoring model.

Format:

```txt
/monitor Home Team vs Away Team | League
```

Example:

```txt
/monitor Chelsea vs Manchester City | Premier League
```

In the MVP version, the live monitor uses simulated match data to demonstrate the model without consuming large volumes of live API requests.

The monitoring model uses two active time windows:

```txt
20'–55' → Over 1.5 goals
56'–75' → Over 0.5 remaining goal
```

---

### `/help`

Displays the user guide, supported commands and supported leagues.

---

## 🏆 Currently Supported by GoalFlow MVP

The football API supports many leagues, but this MVP currently supports a controlled set of competitions for validation and testing:

- Premier League
- La Liga
- Serie A
- Bundesliga
- Ligue 1
- UEFA Champions League

---

## 📊 Pre-match Probability Model

GoalFlow estimates expected goals using historical home/away performance.

```txt
Home expected goals =
(Home average goals scored + Away average goals conceded) / 2

Away expected goals =
(Away average goals scored + Home average goals conceded) / 2

Total expected goals =
Home xG + Away xG
```

The model then applies a Poisson distribution to estimate goal probabilities.

```txt
P(X = k) = (λ^k × e^-λ) / k!
```

Where:

- `λ` is the expected goals value
- `k` is the number of goals
- `e` is Euler's number

The model calculates:

- probability of each team scoring 0, 1, 2 or 3 goals
- Over 1.5 goals probability
- Over 2.5 goals probability
- Over 3.5 goals probability
- most likely scorelines

---

## 📡 Live Monitoring Model

The live monitoring module estimates goal probability using a custom rule-based model.

It considers:

- match minute
- current score
- live xG
- total shots
- shots on target
- shots inside the box
- corners
- dangerous attacks
- goalkeeper saves
- recent pressure
- league intensity
- scoreline context

The model calculates internal components:

- **Pressure Score**
- **Chance Quality Score**
- **League Intensity**
- **IEG - Expected Goal Imminence**
- **ICO - Offensive Chaos Index**

These components are converted into an estimated goal probability.

---

## ⏱️ Monitoring Windows

GoalFlow applies different target markets depending on the match minute.

```txt
20'–55' → Over 1.5 goals
56'–75' → Over 0.5 remaining goal
```

The reason for this design is that the same match statistics should not be interpreted equally at different points of the game.

For example:

- A high-pressure match at 25' may suggest potential for Over 1.5 goals.
- A high-pressure match at 65' may be more relevant for Over 0.5 remaining goal.

---

## 🚨 Alert Decision Bands

```txt
0–38%   = Poor goal expectation
39–52%  = High risk
53–63%  = Watchlist
64–67%  = Aggressive watchlist
68–69%  = Conditional alert
70–73%  = Valid alert
74–79%  = Good alert
80–100% = Strong alert
```

A valid alert is generated when the estimated goal probability reaches at least 70%.

A conditional alert may be generated between 68% and 69% if strong contextual triggers are active.

---

## 🧠 Contextual Triggers

GoalFlow includes contextual triggers to improve decision quality.

### Latent Conversion Trigger

Activated when attacking pressure shows signs that it may soon convert into a goal.

Examples:

- competitive scoreline
- strong territorial dominance
- corners in sequence
- blocked shots
- dangerous attacks
- favourite team pushing
- recent pressure increasing

### Convertible Unilateral Pressure Trigger

Activated when one team is applying strong and high-quality pressure.

Examples:

- 4+ shots on target from one side
- 7+ shots from one side
- 3+ shots inside the box
- dominant xG above 0.70
- opponent defending deep
- multiple goalkeeper saves

---

## ✅ Input Validation

GoalFlow validates:

- command format
- supported league
- team names
- league-team combination
- minimum available match data
- home/away context

If an invalid request is sent, the bot returns a guided error message with examples and supported formats.

Examples of supported separators between teams:

```txt
Chelsea x Manchester City
Chelsea vs Manchester City
Chelsea v Manchester City
```

---

## 📂 Project Structure

```txt
goalflow-football-signal-bot/
│
├── docs/
│   ├── architecture.md
│   ├── model-logic.md
│   ├── api-notes.md
│   └── demo-script.md
│
├── workflows/
│   └── goalflow-n8n-workflow.json
│
├── data/
│   ├── sample-live-data.json
│   └── signal-records.csv
│
├── screenshots/
│   ├── analyse-command.png
│   ├── monitor-command.png
│   └── n8n-workflow.png
│
├── report-notes/
│   └── academic-report-notes.md
│
└── README.md
```

---

## 🚧 Current Status

Project in development.

Completed:

- Telegram bot setup
- n8n workflow integration
- API-Football connection
- Dynamic command parsing
- League and team validation
- Pre-match expected goals model
- Poisson probability model
- Most likely scoreline calculation
- Live monitoring simulation
- Guided error messages
- Markdown-formatted Telegram responses

Planned:

- Google Sheets signal storage
- Signal performance dashboard
- Real live data integration
- Duplicate signal prevention
- More leagues and competitions
- Improved probability calibration

---

## 📊 Future Improvements

- Connect live match statistics from a premium API
- Store generated alerts in Google Sheets
- Build a dashboard for performance evaluation
- Add cache to reduce API usage
- Add user-specific monitored fixtures
- Compare model probability against market odds
- Add machine learning calibration
- Expand supported leagues
- Improve xG estimation
- Export reports for academic analysis

---

## ⚠️ MVP Limitations

This is an academic prototype.

Current limitations:

- live monitoring uses simulated data
- historical data is limited by the free API plan
- the model is not trained using machine learning
- the probability model is heuristic and simplified
- no betting recommendation is provided
- the system is not intended for real-money betting decisions

---

## ⚠️ Disclaimer

This project is for educational and research purposes only.

GoalFlow does not provide financial advice, betting advice or guaranteed predictions. The probability model is a prototype and should not be used for real-money betting decisions.

---

## 👨‍💻 Author

**Luiz Carlos Froes**

Computer Science Student | AI Automation & Workflow Developer

Building chatbots, email automation and workflow systems using n8n, APIs and CRM integrations.
