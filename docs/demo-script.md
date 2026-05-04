# GoalFlow Demo Script

This document provides a suggested demo script for presenting GoalFlow in an academic setting.

The purpose of the demo is to show how GoalFlow works as a Telegram-based football probability and signal system built with n8n, external APIs and custom rule-based logic.

---

## 1. Demo Goal

The goal of this demo is to show that GoalFlow can:

- receive football fixture commands through Telegram
- validate user input
- retrieve historical football data
- calculate expected goals
- apply a Poisson probability model
- estimate Over goal probabilities
- simulate live monitoring alerts
- explain its decision-making logic

---

## 2. Tools to Open Before the Demo

Before starting the presentation, open:

- Telegram with the GoalFlow bot
- n8n workflow canvas
- API-Football dashboard or API documentation
- GitHub repository
- README.md
- docs/architecture.md
- docs/model-logic.md

Optional:

- screenshots of the workflow
- screenshots of previous bot responses

---

## 3. Short Project Introduction

Suggested explanation:

```txt
GoalFlow is a Telegram-based football analysis system that estimates goal probabilities using historical match data, expected goals, Poisson distribution and live monitoring logic.

The system was built with n8n, integrates external football APIs, and uses custom JavaScript code nodes to process data and generate explainable alerts.
```

---

## 4. Explain the Problem

Suggested explanation:

```txt
Football matches generate a large amount of data, such as goals, shots, shots on target, xG, corners and attacking pressure.

The problem is that raw statistics alone are difficult to interpret quickly.

GoalFlow solves this by transforming football data into structured probabilities and guided alerts.
```

---

## 5. Explain the Target Audience

Suggested explanation:

```txt
The target audience includes football analysts, sports data enthusiasts, content creators and users who want a structured way to interpret football match data.

For the academic prototype, the main focus is not betting accuracy, but the automation and data-processing logic behind the system.
```

---

## 6. Explain the Main Features

GoalFlow currently has three main commands:

```txt
/help
/analyse
/monitor
```

### `/help`

Shows the usage guide and supported leagues.

### `/analyse`

Runs the pre-match probability model.

### `/monitor`

Runs the live monitoring simulation model.

---

## 7. Demo Step 1 — Show `/help`

Send this command in Telegram:

```txt
/help
```

Explain:

```txt
The help command explains how to use the bot, which commands are available, which leagues are supported in the MVP and how the home/away logic works.
```

Mention:

```txt
The first team is treated as the home team.
The second team is treated as the away team.
```

---

## 8. Demo Step 2 — Show Error Handling

Send an invalid command:

```txt
hello
```

Explain:

```txt
The bot does not fail silently. It returns a guided error message explaining which commands are available.
```

Then send an invalid format:

```txt
/analyse Chelsea Manchester City Premier League
```

Explain:

```txt
The bot validates the command format and explains that the vertical bar and team separator are required.
```

Then send an unsupported league:

```txt
/analyse Chelsea vs Manchester City | MLS
```

Explain:

```txt
The bot validates supported leagues in the MVP and guides the user towards the accepted format.
```

---

## 9. Demo Step 3 — Run Pre-Match Analysis

Send:

```txt
/analyse Chelsea vs Manchester City | Premier League
```

Alternative:

```txt
/analyse Arsenal vs Liverpool | Premier League
```

Explain the output:

```txt
The system retrieves historical fixture data, filters the home team's home matches and the away team's away matches, then calculates average goals scored and conceded.
```

Mention:

```txt
This is more contextual than using generic team averages because teams often perform differently at home and away.
```

---

## 10. Explain Expected Goals Calculation

Suggested explanation:

```txt
GoalFlow estimates expected goals using a simplified xG-like formula.

Home xG is calculated by combining the home team's average goals scored at home with the away team's average goals conceded away.

Away xG is calculated by combining the away team's average goals scored away with the home team's average goals conceded at home.
```

Formula:

```txt
Home xG =
(Home average goals scored + Away average goals conceded) / 2

Away xG =
(Away average goals scored + Home average goals conceded) / 2
```

---

## 11. Explain Poisson Distribution

Suggested explanation:

```txt
After estimating expected goals, GoalFlow applies a Poisson distribution to calculate the probability of each team scoring 0, 1, 2 or 3 goals.

It also combines possible scorelines to estimate Over 1.5, Over 2.5 and Over 3.5 probabilities.
```

Formula:

```txt
P(X = k) = (λ^k × e^-λ) / k!
```

---

## 12. Explain Most Likely Scorelines

Suggested explanation:

```txt
GoalFlow calculates possible scoreline combinations and returns the most likely results based on the Poisson model.
```

Example:

```txt
1-1
2-1
1-2
2-2
```

---

## 13. Demo Step 4 — Run Live Monitoring

Send:

```txt
/monitor Chelsea vs Manchester City | Premier League
```

Explain:

```txt
The monitoring module currently uses simulated live data in the MVP.

This was chosen because free API plans often limit live data requests, and using simulation makes the demo reliable even when no live match is available.
```

---

## 14. Explain Monitoring Windows

Suggested explanation:

```txt
The live monitoring model does not interpret all minutes equally.

Between 20 and 55 minutes, it analyses the match for Over 1.5 goals.

Between 56 and 75 minutes, it analyses the match for Over 0.5 remaining goal.
```

Windows:

```txt
20'–55' → Over 1.5 goals
56'–75' → Over 0.5 remaining goal
```

Explain:

```txt
This is important because the same statistics can mean different things depending on the match minute.
```

---

## 15. Explain Live Model Components

Suggested explanation:

```txt
The live model calculates several internal components before producing a final goal probability.
```

Components:

```txt
Pressure Score
Chance Quality Score
League Intensity
IEG - Expected Goal Imminence
ICO - Offensive Chaos Index
```

Explain briefly:

```txt
Pressure Score measures attacking volume.
Chance Quality Score measures whether the pressure is producing real chances.
League Intensity adds competition context.
IEG estimates whether a goal appears imminent.
ICO measures offensive chaos and match instability.
```

---

## 16. Explain Alert Decision

Suggested explanation:

```txt
The final result is expressed as an estimated goal probability rather than just a raw score.

The system then maps that probability into decision bands such as Watchlist, Valid Alert, Good Alert or Strong Alert.
```

Decision bands:

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

---

## 17. Explain What Makes the Project Original

Suggested explanation:

```txt
The originality of GoalFlow is not just that it sends football alerts.

The system combines workflow automation, API integration, contextual home/away analysis, expected goals, Poisson probability and time-window based live monitoring.

It also explains why an alert is generated, instead of only returning a signal.
```

Key original elements:

- contextual home/away filtering
- expected goals model
- Poisson probability
- time-window based market selection
- explainable live alert logic
- guided error handling
- Telegram-based interaction

---

## 18. Explain Technical Architecture

Show the n8n workflow.

Suggested explanation:

```txt
The workflow is split into different stages: command extraction, validation, API requests, probability calculation and Telegram response.

This makes the project modular and easier to expand.
```

Point out:

- Telegram Trigger
- Extract Command
- validation nodes
- API request nodes
- Calculate Analysis
- Live Monitor Simulation
- Send Message

---

## 19. Explain MVP Limitations

Suggested explanation:

```txt
This is an academic MVP, so the objective is to demonstrate the system architecture and model logic rather than guarantee prediction accuracy.
```

Limitations:

- live monitoring uses simulated data
- free API plan limits available historical data
- season is fixed to the available dataset
- probability model is heuristic
- no real-money betting advice is provided
- no machine learning calibration yet

---

## 20. Explain Future Improvements

Suggested explanation:

```txt
In the future, GoalFlow could be connected to a premium live data API, store alerts in Google Sheets, and build a dashboard to evaluate signal performance.
```

Future improvements:

- real live statistics integration
- Google Sheets signal storage
- dashboard for performance tracking
- duplicate alert prevention
- API caching
- more leagues
- machine learning calibration
- odds comparison
- user-specific monitoring

---

## 21. Suggested Closing Statement

```txt
GoalFlow demonstrates how automation, APIs and custom probability logic can be combined into a functional football analysis system.

The project is designed as an explainable academic prototype, with clear potential for future expansion into a more advanced data-driven sports analytics tool.
```

---

## 22. Backup Demo Commands

Use these if one command does not return enough data:

```txt
/analyse Chelsea vs Manchester City | Premier League
/analyse Arsenal vs Liverpool | Premier League
/analyse Real Madrid vs Barcelona | La Liga
/analyse Inter vs Juventus | Serie A
/analyse Bayern Munich vs Borussia Dortmund | Bundesliga
```

Monitor demo:

```txt
/monitor Chelsea vs Manchester City | Premier League
```

Help:

```txt
/help
```

Error handling:

```txt
hello
/analyse Chelsea Manchester City Premier League
/analyse Chelsea vs Manchester City | MLS
```
