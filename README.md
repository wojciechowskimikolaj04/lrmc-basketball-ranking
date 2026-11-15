# Markov Chains as a Ranking Tool — LRMC Ranking of the Polish Basketball League

Bachelor's thesis project · **Actuarial Science & Financial Analysis, Lodz University of Technology (2026)**
Author: **Mikołaj Wojciechowski** · Supervisor: Dr. Grzegorz Skalski
Thesis (in Polish): see the `thesis/` folder

## Why this project

Basketball has been part of my life since childhood, so when choosing a thesis topic I wanted to connect the mathematics
I was studying with the sport I know best. The spark came from the article by **Paul Kvam and Joel Sokol**,
*A Logistic Regression/Markov Chain Model for NCAA Basketball* (2006), which ranks college teams using Markov chains.
I decided to check how the same idea works for the Polish top league — **ORLEN Basket Liga, regular season 2025/2026**.

## The question

A league table treats every win the same — a 2-point win counts exactly like a 30-point blowout.
**Does a ranking that accounts for margin of victory and home-court advantage tell a different story about which teams are really the strongest?**

## Method (LRMC — Logistic Regression / Markov Chain)

1. **Data** — I collected the results of all **240 regular-season games** (16 teams, home and away vs. every opponent)
   by hand from the official league website, organised as 120 game/rematch pairs ([`games_plk.csv`](games_plk.csv)).
2. **Logistic regression** — using the game/rematch pairs, I estimated how the home margin of victory `x` translates into
   the probability of also winning the rematch away: `P(win away | x) = σ(β₀ + β₁x)`.
   Estimates: **β₀ = −0.411** (p = 0.040), **β₁ = 0.0346** (p = 0.014). A team needs to win at home by about **12 points**
   to be a 50/50 favourite in the away rematch — that is the size of home-court advantage in the data.
3. **Markov chain** — each team is a state. A "hypothetical fan" picks one of their current team's games and moves to the
   home team with probability `p = σ(β₀ + β₁x)`, or to the away team with probability `1 − p`. This gives a 16×16
   stochastic transition matrix.
4. **Stationary distribution** — the chain is irreducible and aperiodic (checked in the code), so it has a unique stationary
   distribution π (π = πT), computed by power iteration. **Teams are ranked by π.**

The thesis also covers the theory behind it: Markov property, transition matrices, Chapman–Kolmogorov equations,
irreducibility, aperiodicity, existence and convergence to the stationary distribution, and maximum-likelihood estimation
for logistic regression.

## Results

| Team | Official table | LRMC | Change |
|---|:-:|:-:|:-:|
| Legia Warszawa | 1 | 1 | – |
| **King Szczecin** | 2 | 8 | **▼6** |
| WKS Śląsk Wrocław | 3 | 3 | – |
| Energa Trefl Sopot | 4 | 5 | ▼1 |
| **Dziki Warszawa** | 5 | 2 | **▲3** |
| AMW Arka Gdynia | 6 | 6 | – |
| **Anwil Włocławek** | 7 | 4 | **▲3** |
| ORLEN Zastal Zielona Góra | 8 | 7 | ▲1 |
| MKS Dąbrowa Górnicza | 9 | 9 | – |
| Tasomix Rosiek Stal Ostrów Wlkp. | 10 | 10 | – |
| Górnik Zamek Książ Wałbrzych | 11 | 12 | ▼1 |
| Arriva LOTTO Twarde Pierniki Toruń | 12 | 11 | ▲1 |
| PGE Start Lublin | 13 | 14 | ▼1 |
| Energa Czarni Słupsk | 14 | 13 | ▲1 |
| Tauron GTK Gliwice | 15 | 15 | – |
| Miasto Szkła Krosno | 16 | 16 | – |

**Key findings**

- The two rankings mostly agree: the same leader and an almost identical bottom half.
- **King Szczecin** finished 2nd in the table but only 8th in LRMC: they won many close games, with a point differential
  of just +59 (vs. +202 for Legia). In the playoffs they lost 1–3 in the quarter-final to **Zastal Zielona Góra**, which
  LRMC ranked higher (7th) despite its 8th place in the table. Zastal then reached the final.
- **Dziki Warszawa** (5th → 2nd) and **Anwil Włocławek** (7th → 4th) won convincingly, and LRMC rewards that.
- Limitation: Anwil, ranked 4th by LRMC, was knocked out in the play-in. The model sees only scores, not injuries,
  roster changes or off-court problems.

## What I built on, and what I did myself

The R code started from an existing open-source implementation for NCAA basketball:
**[elliottmerriam/LRMC](https://github.com/elliottmerriam/LRMC)**. That script scrapes US college results and uses
fixed parameters from the literature to turn point spreads into probabilities.

My own work:
- **collected the full dataset** for the 2025/2026 ORLEN Basket Liga season by hand;
- **restructured the model around game/rematch pairs**, a format the Polish league allows and NCAA does not;
- **estimated the logistic regression on this league's own data** (`glm`), instead of using fixed constants from the NCAA paper;
- **rebuilt the transition matrix** for a 16-team double round-robin and **added checks** that the matrix is stochastic,
  irreducible and aperiodic;
- **compared** the ranking with the official table and with the actual playoff results (in the thesis);
- wrote the full theoretical part (chapters 1–3 of the thesis).

## Repository structure

```
├── LRMC_final3.Rmd   # full analysis (R Markdown)
├── games_plk.csv     # 240 games, 120 game/rematch pairs
└── thesis/           # bachelor's thesis (PDF, Polish)
```

## How to run

Keep `LRMC_final3.Rmd` and `games_plk.csv` in the same folder, open the `.Rmd` in RStudio and click **Knit**
(or run the chunk). Only base R is needed. The script prints the regression results, checks of the transition matrix
properties and the final LRMC ranking, and draws the fitted logistic curve.

## References

- Kvam, P., Sokol, J. S. (2006). *A Logistic Regression/Markov Chain Model for NCAA Basketball*. Naval Research Logistics, 53(8).
- elliottmerriam (2015). *LRMC model for predicting NCAA basketball games in R*. https://github.com/elliottmerriam/LRMC
- Privault, N. (2024). *Discrete Stochastic Processes: Tools for Machine Learning and Data Science*. Springer.
- Grimmett, G. R., Stirzaker, D. R. (2020). *Probability and Random Processes*, 4th ed. Oxford University Press.
- Murphy, K. P. (2012). *Machine Learning: A Probabilistic Perspective*. MIT Press.
- Data: ORLEN Basket Liga, https://plk.pl (schedule and table, accessed 17.06.2026).

## Tech

R (base: `glm`, matrix algebra) · R Markdown · LaTeX
