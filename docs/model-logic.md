# GoalFlow Model Logic

This document explains the logic behind the GoalFlow probability models, including the pre-match analysis model and the live monitoring model.

GoalFlow is not designed as a machine learning system in the MVP stage. Instead, it uses a transparent rule-based and probabilistic approach that can be explained, tested and improved over time.

---

## 1. Model Overview

GoalFlow has two main analytical modules:

1. **Pre-match Probability Model**
   - Uses historical team performance
   - Compares home and away context
   - Estimates expected goals
   - Applies Poisson probability
   - Predicts Over goal probabilities and likely scorelines

2. **Live Monitoring Model**
   - Uses match context and live indicators
   - Applies time-window logic
   - Calculates pressure and chance quality components
   - Estimates goal probability
   - Generates an alert decision

---

## 2. Pre-Match Probability Model

The pre-match model estimates the expected number of goals before a fixture.

It uses:

- home team home performance
- away team away performance
- average goals scored
- average goals conceded
- historical Over 1.5 and Over 2.5 rates

The first team entered by the user is treated as the home team.

Example:

```txt
/analyse Chelsea vs Manchester City | Premier League
```

In this example:

```txt
Chelsea = home team
Manchester City = away team
```

---

## 3. Home and Away Context

GoalFlow does not simply analyse general team averages.

Instead, it applies contextual filtering:

```txt
Home team → only home matches
Away team → only away matches
```

This improves the relevance of the analysis because teams often perform differently at home and away.

Example:

```txt
Chelsea home performance
Manchester City away performance
```

---

## 4. Expected Goals Estimation

GoalFlow estimates expected goals using a simplified xG-like formula.

### Home Team Expected Goals

```txt
Home xG =
(Home average goals scored + Away average goals conceded) / 2
```

### Away Team Expected Goals

```txt
Away xG =
(Away average goals scored + Home average goals conceded) / 2
```

### Total Expected Goals

```txt
Total xG =
Home xG + Away xG
```

This gives the system a baseline estimate of how many goals the match may produce.

---

## 5. Poisson Distribution

After estimating expected goals, GoalFlow applies the Poisson distribution.

The Poisson distribution is commonly used to model the probability of a number of events happening within a fixed interval.

In football modelling, it can be used to estimate the probability of a team scoring a specific number of goals.

### Formula

```txt
P(X = k) = (λ^k × e^-λ) / k!
```

Where:

- `λ` is the expected goals value
- `k` is the number of goals
- `e` is Euler's number

---

## 6. Goal Distribution

GoalFlow calculates the probability of each team scoring:

```txt
0 goals
1 goal
2 goals
3 goals
```

Example output:

```txt
Chelsea:
0 goals: 18.4%
1 goal: 31.2%
2 goals: 26.5%
3 goals: 15.0%
```

This makes the model explainable because the user can see how likely different scoring outcomes are.

---

## 7. Over Goal Probabilities

GoalFlow calculates probabilities for:

```txt
Over 1.5 goals
Over 2.5 goals
Over 3.5 goals
```

This is calculated by combining possible scorelines and summing the probabilities where total goals exceed the target line.

Example:

```txt
Over 2.5 goals =
Probability of total goals being 3 or more
```

---

## 8. Most Likely Scorelines

GoalFlow generates the most likely scorelines by calculating the probability of combinations such as:

```txt
0-0
1-0
1-1
2-1
2-2
3-1
```

The system currently evaluates scorelines from 0 to 5 goals for each team.

The top five most likely scorelines are shown to the user.

---

## 9. Live Monitoring Model

The live monitoring model is used by the `/monitor` command.

In the MVP, this module uses simulated live data to demonstrate how the system would behave with real-time match statistics.

The live model estimates the probability of a goal occurring based on:

- match minute
- current score
- live xG
- shots
- shots on target
- shots inside the box
- corners
- dangerous attacks
- goalkeeper saves
- recent pressure
- league intensity
- scoreline context

---

## 10. Monitoring Windows

GoalFlow uses time-window based logic.

Different match periods are linked to different target markets.

```txt
20'–55' → Over 1.5 goals
56'–75' → Over 0.5 remaining goal
```

This design is based on the idea that the same statistics should not be interpreted equally at different moments of the match.

For example:

- At 25 minutes, the model is interested in whether the match can reach at least 2 total goals.
- At 65 minutes, the model is interested in whether at least 1 more goal can occur.

---

## 11. Internal Live Model Components

The live model calculates five internal components.

### 11.1 Pressure Score

Measures attacking pressure.

It considers:

- total shots
- second-half shots
- dangerous attacks
- corners
- blocked shots
- recent pressure increase

High pressure suggests that the match is producing enough attacking activity to justify monitoring.

---

### 11.2 Chance Quality Score

Measures whether attacking pressure is producing real chances.

It considers:

- xG
- shots on target
- shots inside the box
- goalkeeper saves

This prevents the system from overrating empty possession or weak territorial dominance.

---

### 11.3 League Intensity Score

Each supported league has an intensity value.

Example:

```txt
Premier League → 82
Bundesliga → 84
La Liga → 74
Serie A → 70
Ligue 1 → 72
UEFA Champions League → 80
```

This gives the model a contextual baseline based on the expected rhythm and attacking tendency of each competition.

---

### 11.4 IEG - Expected Goal Imminence

IEG estimates whether a goal appears likely in the next few minutes.

It considers:

- total xG
- shots on target
- box entries
- recent pressure acceleration
- competitive scoreline
- losing team pressure
- favourite team pushing

IEG is designed to capture goal imminence rather than general match quality.

---

### 11.5 ICO - Offensive Chaos Index

ICO measures the level of attacking chaos in the match.

It considers:

- corners
- blocked shots
- dangerous attacks
- final-third entries
- competitive scoreline
- emotional match state
- red cards

A high ICO suggests that the match is open, unstable or likely to produce dangerous situations.

---

## 12. Estimated Goal Probability

The internal scores are converted into an estimated goal probability.

The MVP formula is:

```txt
Goal Probability =
Pressure Score × 0.25
+ Chance Quality Score × 0.30
+ IEG × 0.25
+ ICO × 0.10
+ League Intensity × 0.10
+ Trigger Bonuses
- Penalties
```

The result is capped between:

```txt
0% and 95%
```

The system avoids returning 100% because football outcomes are uncertain by nature.

---

## 13. Contextual Triggers

GoalFlow uses contextual triggers to avoid relying only on raw statistics.

---

### 13.1 Latent Conversion Trigger

This trigger activates when pressure appears likely to convert into a goal.

It may be activated by factors such as:

- competitive scoreline
- good value for the match minute
- strong territorial dominance
- box entries
- corners
- blocked shots
- dangerous attacks
- favourite team pushing
- losing team pressing
- opponent defending deep
- emotionally live game
- recent pressure increasing

When active, this trigger increases the estimated goal probability.

---

### 13.2 Convertible Unilateral Pressure Trigger

This trigger activates when one team is applying strong, high-quality pressure.

It may be activated by factors such as:

- draw or one-goal difference
- 4+ shots on target from one side
- 7+ shots from one side
- 3+ shots inside the box from one side
- dominant xG above 0.70
- 2+ corners from the dominant side
- 2+ goalkeeper saves
- constant final-third entries
- opponent defending deep

This allows the model to recognise situations where one team is creating enough danger even if the match is not chaotic on both sides.

---

## 14. Penalty Logic

GoalFlow applies penalties to reduce false positives.

### False Pressure Penalty

Applied when the match has high territorial pressure but lacks real attacking quality.

Example:

```txt
High dangerous attacks
Low shots on target
Low shots inside the box
Few corners
Few blocked shots
```

This prevents the model from overvaluing sterile pressure.

---

### Dead Opponent Penalty

Applied when one team leads comfortably and the losing team shows little reaction.

Example:

```txt
Two-goal lead
Losing team not pressing
Low emotional intensity
No recent pressure increase
```

This helps avoid alerts in matches where the game state is no longer urgent.

---

### Time Window Penalty

Applied when the match is in a sensitive time period and lacks imminent pressure.

Examples:

- 20'–30': more selective because the match may still be developing
- 35'–45': requires signs of imminent pressure before half-time

---

## 15. Alert Decision Bands

The final probability is mapped to decision bands.

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

## 16. Alert Rules

GoalFlow applies the following alert rules:

```txt
Probability >= 70% → Valid alert
68–69% → Conditional alert if a strong trigger is active
64–67% → Watchlist, upgraded only in very strong late-window contexts
Below 64% → No alert
```

This prevents the model from generating alerts too easily while still allowing strong contextual situations to be recognised.

---

## 17. Why the Model is Explainable

GoalFlow is designed to be explainable.

Instead of returning only a final prediction, it shows:

- match context
- live xG
- model components
- estimated probability
- alert status
- main reasons

This makes the system more transparent and easier to evaluate.

---

## 18. MVP Limitations

The model has important limitations:

- live data is simulated in the MVP
- historical data is limited by the free API plan
- the xG model is simplified
- the live probability formula is heuristic
- the model is not trained using machine learning
- no betting recommendation is provided
- the system does not guarantee prediction accuracy

---

## 19. Future Model Improvements

Future improvements may include:

- replacing simulated live data with real live statistics
- improving xG estimation using richer data
- calibrating probabilities against historical outcomes
- comparing model probabilities with betting market odds
- adding machine learning calibration
- adding league-specific weighting
- adding team-specific attacking and defensive ratings
- storing alert results for performance tracking
- building a dashboard for long-term analysis

---

## 20. Summary

GoalFlow combines simple statistical modelling with rule-based football interpretation.

The pre-match module uses expected goals and Poisson probability.

The live monitoring module uses pressure, chance quality, match context and time-window logic to estimate goal probability.

The goal of the MVP is not to guarantee prediction accuracy, but to demonstrate how automation, APIs and custom decision logic can be combined into a football analysis system.
