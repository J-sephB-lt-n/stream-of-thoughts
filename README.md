# stream-of-thoughts
An unordered record of things which I want to be documented

[Association, Bias & Causation](#association-bias--causation), [Database Normalisation](#database-normalisation), [Kelly Criterion](#the-kelly-criterion)

## Association, Bias & Causation

```math
$$\begin{array}{lcl}
\underbrace{E\Big[Y\Bigl|T=1\Big] - E\Big[Y\Bigl|T=0\Big]}_{
    \substack{ \text{Difference between} \\ \text{treatment group means} } 
    }
    &=& 
    \underbrace{E\Big[Y(1)-Y(0)\Bigl|T=1\Big]}_{
    \substack{\text{Average Treatment effect} \\ \text{on the Treated (ATT)} }} + 
    \underbrace{\Bigg(E\Big[Y(0)\Bigl|T=1\Big]-E\Big[Y(0)\Bigl|T=0\Big]\Bigg)}_{
        \text{Selection Bias}
        } \\
\space &\space& \space \\
Y_i &=& \text{outcome of interest (on individual } i)\\
T_i &=& \begin{cases}1 \quad \text{if individual } i \text{ received treatment} \\
0 \quad \text{if individual } i \text{ did not receive treatment}\end{cases} \\
Y_i(1) &=& \text{outcome which would have been observed for individual } i \text{ if they had received the treatment} \\
Y_i(0) &=& \text{outcome which would have been observed for individual } i \text{ if they had NOT received the treatment} \\
\end{array}$$
```

Here is a simulation in python showing this to be true:
```python
import random
import statistics

N_INDIVIDUALS: int = 100_000
random.seed(69)

untreated_prob_of_dying: list[float] = [
    random.uniform(0, 1) for _ in range(N_INDIVIDUALS)
]
treated_prob_of_dying: list[float] = [
    # treatment halves probability of death #
    0.5 * p
    for p in untreated_prob_of_dying
]
assigned_treatment_group: list[str] = [
    # biased by probability of dying #
    random.choices(["treated", "untreated"], weights=(p, 1 - p))[0]
    for p in untreated_prob_of_dying
]
prob_of_dying: list[float] = [
    (
        untreated_prob_of_dying[idx]
        if treat_grp == "untreated"
        else treated_prob_of_dying[idx]
    )
    for idx, treat_grp in enumerate(assigned_treatment_group)
]
mean_prob_of_dying_treated_group: float = statistics.mean(
    [
        prob_of_dying[idx]
        for idx, treat_grp in enumerate(assigned_treatment_group)
        if treat_grp == "treated"
    ]
)
mean_prob_of_dying_untreated_group: float = statistics.mean(
    [
        prob_of_dying[idx]
        for idx, treat_grp in enumerate(assigned_treatment_group)
        if treat_grp == "untreated"
    ]
)
att: float = statistics.mean(
    [
        (treated_prob_of_dying[idx] - untreated_prob_of_dying[idx])
        for idx, treat_grp in enumerate(assigned_treatment_group)
        if treat_grp == "treated"
    ]
)
selection_bias: float = statistics.mean(
    [
        untreated_prob_of_dying[idx]
        for idx, treat_grp in enumerate(assigned_treatment_group)
        if treat_grp == "treated"
    ]
) - statistics.mean(
    [
        untreated_prob_of_dying[idx]
        for idx, treat_grp in enumerate(assigned_treatment_group)
        if treat_grp == "untreated"
    ]
)

print(
    f"""
                      E[Y|T=1] - E[Y|T=0] = {(mean_prob_of_dying_treated_group - mean_prob_of_dying_untreated_group):.5f}
                     ATT + selection_bias = {(att + selection_bias):.5f} 

                    ATT: E[Y(1)-Y(0)|T=1] = {att:.5f}
Selection Bias: E[Y(0)|T=1] - E[Y(0)|T=0] = {selection_bias:.5f}
"""
)
```
```
                      E[Y|T=1] - E[Y|T=0] = 0.00176
                     ATT + selection_bias = 0.00176 

                    ATT: E[Y(1)-Y(0)|T=1] = -0.33341
Selection Bias: E[Y(0)|T=1] - E[Y(0)|T=0] = 0.33516
```

## Database Normalisation

```<this section is still under construction>```

### 1st Normal Form

To be in first normal form, each table cell must contain a single value (e.g. not anything like an array, json or nested table within the cell).

Example: Not in first normal form:

| Name     | Skills
|----------|-----------
| Joe      | python,unicyling,piano 
| Napoleon | nunchuck,bow hunting,computer hacking

Example: In first normal form:

| Name | Skill
|------|--------------
| Joe  | python
| Joe  | unicyling
| Joe  | piano 
| Napoleon | nunchuck
| Napoleon | bow hunting
| Napoleon | computer hacking

### Database Normalisation: 2nd Normal Form



### Database Normalisation: 3rd Normal Form


## The Kelly Criterion 

The [Kelly Criterion](https://en.wikipedia.org/wiki/Kelly_criterion) (or [Kelly Strategy](https://en.wikipedia.org/wiki/Kelly_criterion)) is a result from probability theory. In a specific repeated game (which resembles some gambling games and investment scenarios), it is the strategy achieving maximum gain/reward in the long run. 

The formulation is as follows:

In a single game:

* The player invests (risks) $100r\%$ of their total assets/portfolio/wealth $A$. 

* With probability $w$, they earn a return of $100g\%$ on their investment/risk. i.e. $A_{t+1}=A_t(1+rg)$ 

* With probability $1-w$, they lose $100b\%$ of their investment/risk. i.e. $A_{t+1}=A_t(1-rb)$

After playing $n$ consecutive games, the expected value of their total assets/portfolio/wealth is:

$$P_n \quad=\quad A_0(1+rg)^{wn}(1-rb)^{(1-w)n}$$

The value of $r$ maximizing $P_n$ can be found by solving the equation 

$$\frac{d}{d r}\frac{log(P_n)}{n} \quad=\quad 0$$

The solution is:

$$arg\space max_r\space P_n \quad=\quad \displaystyle\frac{w}{b} - \frac{1-w}{g}$$