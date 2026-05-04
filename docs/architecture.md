# GoalFlow Architecture

This document describes the technical architecture of GoalFlow, including its main components, data flow, integrations and workflow logic.

GoalFlow is built as a workflow automation system using n8n. It receives user commands through Telegram, processes football data using external APIs and custom JavaScript logic, and returns formatted analysis or monitoring alerts to the user.

---

## 1. High-Level Architecture

```txt
Telegram User
↓
Telegram Bot
↓
n8n Workflow
↓
Command Parser
↓
Validation Layer
↓
Football Data API / Simulated Live Data
↓
GoalFlow Model Logic
↓
Telegram Response
```

GoalFlow uses Telegram as the main user interface and n8n as the automation and processing layer.

---

## 2. Main Components

### Telegram Bot

The Telegram bot is the user-facing interface of GoalFlow.

Users interact with the system by sending commands such as:

```txt
/analyse Chelsea vs Manchester City | Premier League
/monitor Chelsea vs Manchester City | Premier League
/help
```

The bot receives the message and passes it to the n8n workflow.

---

### n8n Workflow

n8n is responsible for:

- receiving Telegram messages
- extracting commands and parameters
- validating user input
- calling football data APIs
- running JavaScript code nodes
- calculating probabilities
- formatting responses
- sending messages back to Telegram

---

### Football Data API

The MVP uses API-Football / API-Sports for historical football data.

The API is used to retrieve:

- team information
- league data
- historical fixtures
- match results
- home/away performance data

Due to free API limitations, the MVP uses the 2024 dataset for the pre-match model.

---

### JavaScript Code Nodes

Custom JavaScript code nodes are used inside n8n to implement GoalFlow's logic.

These nodes handle:

- command parsing
- input validation
- team validation
- league-team validation
- expected goals calculation
- Poisson probability calculation
- live monitoring simulation
- alert decision logic

---

## 3. Telegram Commands

GoalFlow currently supports three main commands.

### `/analyse`

Runs the pre-match analysis model.

```txt
/analyse Home Team vs Away Team | League
```

Example:

```txt
/analyse Arsenal vs Liverpool | Premier League
```

The first team is treated as the home team.  
The second team is treated as the away team.

---

### `/monitor`

Runs the live monitoring model.

```txt
/monitor Home Team vs Away Team | League
```

Example:

```txt
/monitor Chelsea vs Manchester City | Premier League
```

In the MVP version, this command uses simulated live match data.

---

### `/help`

Displays usage instructions, supported commands and supported leagues.

---

## 4. Pre-Match Analysis Flow

The `/analyse` command follows this workflow:

```txt
Telegram Trigger
↓
Extract Command
↓
Is Valid Input?
↓
Search Home Team
↓
Search Away Team
↓
Validate Teams
↓
Are Teams Valid?
↓
Home Fixtures
↓
Away Fixtures
↓
Validate League Membership
↓
Are Fixtures Valid?
↓
Calculate Analysis
↓
Send Message
```

### Step-by-step explanation

1. **Telegram Trigger**  
   Receives the user's Telegram message.

2. **Extract Command**  
   Extracts:
   - command
   - home team
   - away team
   - league
   - league ID
   - league intensity
   - season

3. **Is Valid Input?**  
   Checks whether the input format is valid.

4. **Search Home Team**  
   Searches the API for the home team.

5. **Search Away Team**  
   Searches the API for the away team.

6. **Validate Teams**  
   Confirms that both teams were found and are not the same team.

7. **Are Teams Valid?**  
   Routes valid teams to the next stage or sends an error message.

8. **Home Fixtures**  
   Retrieves historical fixtures for the home team in the selected league and season.

9. **Away Fixtures**  
   Retrieves historical fixtures for the away team in the selected league and season.

10. **Validate League Membership**  
    Ensures that both teams have fixture data in the selected league and that enough matches are available.

11. **Calculate Analysis**  
    Calculates:
    - home team home performance
    - away team away performance
    - expected goals
    - Poisson probabilities
    - most likely scorelines

12. **Send Message**  
    Sends the final formatted analysis to the user via Telegram.

---

## 5. Live Monitoring Flow

The `/monitor` command follows this workflow:

```txt
Telegram Trigger
↓
Extract Command
↓
Is Valid Input?
↓
Is Monitor Command?
↓
Live Monitor Simulation
↓
Send Message
```

The live monitoring module currently uses simulated data to demonstrate how the system would behave with real-time match statistics.

This avoids unnecessary API consumption during the MVP stage and ensures that the demo can work even when no live match is available.

---

## 6. Input Validation

GoalFlow includes several validation layers to improve reliability and user experience.

The system validates:

- recognised commands
- correct command format
- supported leagues
- valid team names
- home team and away team are not the same
- selected teams belong to the selected league
- sufficient historical fixture data exists
- valid home/away context

If validation fails, the bot sends a guided error message showing the correct format and examples.

---

## 7. Supported Leagues in the MVP

The API provider supports many football competitions, but the GoalFlow MVP currently supports a controlled list for testing and validation.

Currently supported by GoalFlow MVP:

- Premier League
- La Liga
- Serie A
- Bundesliga
- Ligue 1
- UEFA Champions League

Each league is mapped to:

- league name
- API league ID
- league intensity score

Example:

```txt
Premier League → ID 39 → Intensity 82
Bundesliga → ID 78 → Intensity 84
```

---

## 8. Data Flow

### User input example

```txt
/analyse Chelsea vs Manchester City | Premier League
```

### Parsed data

```json
{
  "command": "analyse",
  "homeTeamName": "Chelsea",
  "awayTeamName": "Manchester City",
  "leagueName": "Premier League",
  "leagueId": 39,
  "leagueIntensity": 82,
  "season": 2024
}
```

### API data retrieved

- Chelsea fixtures in Premier League 2024
- Manchester City fixtures in Premier League 2024

### Model output

- home performance statistics
- away performance statistics
- expected goals
- Poisson probabilities
- estimated goal market probabilities
- most likely scorelines

### Telegram response

The final response is formatted using Telegram Markdown.

---

## 9. Data Source Strategy

The MVP uses a hybrid data strategy:

### Historical data

Historical fixture data is retrieved from API-Football / API-Sports.

This is used for the `/analyse` command.

### Live data

The `/monitor` command currently uses simulated live data.

This design was chosen because:

- the free API plan has request limitations
- live monitoring can consume many requests
- live matches may not be available during the academic presentation
- simulation makes the demo reliable and repeatable

In a production version, simulated live data would be replaced with a real live football statistics API.

---

## 10. Response Formatting

GoalFlow uses Telegram Markdown to make responses easier to read.

Responses include:

- section titles
- bold values
- emojis
- separators
- guided explanations

Example:

```txt
📊 GoalFlow - Pre-match Analysis

⚽ Fixture
Match: Chelsea vs Manchester City
Competition: Premier League
```

---

## 11. Error Handling

GoalFlow handles common user errors, including:

- unknown command
- missing fixture details
- invalid fixture format
- unsupported league
- team not found
- same team entered twice
- invalid league-team combination
- not enough match data

Each error message provides:

- clear explanation
- correct format
- example command
- suggestion to use `/help`

---

## 12. Technical Limitations

Current MVP limitations:

- live monitoring uses simulated data
- free API plan limits historical and live coverage
- season is fixed to the 2024 dataset
- the model is rule-based and heuristic
- no database is currently required
- Google Sheets storage is planned but not yet implemented
- the system does not currently run continuous background monitoring
- the bot only supports a controlled list of leagues

---

## 13. Future Architecture Improvements

Planned improvements include:

- real live data integration
- Google Sheets storage for generated alerts
- duplicate signal prevention
- API response caching
- user-specific monitored fixtures
- scheduled monitoring intervals
- dashboard for performance evaluation
- expanded league support
- production API provider integration
- database storage
- machine learning calibration

---

## 14. Summary

GoalFlow is designed as a modular football probability system.

The architecture separates:

- user interaction
- command parsing
- validation
- data retrieval
- probability modelling
- response formatting

This makes the system easier to maintain, test and expand in future versions.
