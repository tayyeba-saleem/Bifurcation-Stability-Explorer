# Bifurcation & Stability Explorer

## What this project does, in plain terms

This project is a general-purpose tool for studying **discrete dynamical systems** — models
where a quantity at "step n+1" is computed from its value at "step n," repeated over and over.
These show up constantly in population biology, ecology, economics, and physics: think of a
population of animals whose size next year depends on this year's size, or a host-parasitoid
pair (like a caterpillar and the wasp that preys on it) whose numbers rise and fall together
year after year.

The central question this tool answers is: **for a given model, what happens in the long run —
does the system settle into a stable balance, oscillate in a repeating cycle, or descend into
unpredictable (chaotic) behavior — and how does that answer change as some parameter of the
model is tuned?**

Rather than answering this question for one specific model, the tool is built to accept *any*
model the user types in, as a mathematical formula, and automatically works out its long-run
behavior. That generality is what makes it a reusable piece of software rather than a one-off
script.

## Why this is a meaningful thing to build

- **Stability analysis is a core technique across applied math.** Any time a model is expressed
  as "next value depends on current value," understanding whether it settles down or blows up is
  usually the first real question worth asking.
- **The classic example — the logistic map — is one of the most famous demonstrations in all of
  mathematics** that very simple equations can produce extraordinarily complex, chaotic behavior.
  It's a natural benchmark because its behavior is exhaustively documented, so a tool that
  reproduces it correctly has proven itself.
- **Host-parasitoid population models (like Nicholson-Bailey)** are a standard model in
  mathematical ecology, and generalizing a stability-analysis pipeline to work on them
  demonstrates that the tool isn't limited to toy examples — it works on real research-grade
  models too.

## How the tool actually works, step by step

1. **You describe the model as a formula.** For example, the logistic map is written as
   `x_next = r * x * (1 - x)`, where `r` is a parameter that can be tuned. The tool uses a
   symbolic math library (`sympy`) to understand this formula the way a human mathematician would
   — as an actual equation, not just a black-box function.

2. **It finds the equilibrium points.** An equilibrium (or "fixed point") is a value the system
   stays at once it reaches it — plugging it in and stepping forward leaves it unchanged. The
   tool searches numerically for every such point, for any value of the parameter you choose.

3. **It checks whether each equilibrium is stable.** Being *at* an equilibrium isn't the whole
   story — if you nudge the system slightly away from it, does it return (**stable**), or does it
   drift further away (**unstable**)? The tool answers this using the **Jury stability
   conditions**, a classical mathematical test that looks at the local sensitivity of the model
   (its Jacobian) around the equilibrium and determines the answer directly from that, without
   ever needing to simulate the system forward in time.

4. **It draws the bifurcation diagram.** This is the signature visualization in this field: for
   every value of the parameter, the tool runs the model forward many steps, throws away the
   early "settling in" period, and plots wherever the system ends up. Scanning across many
   parameter values at once reveals exactly where the system's behavior changes character — where
   a single stable point splits into a two-point cycle, then a four-point cycle, and eventually
   into chaos.

5. **It comes with a live, interactive front-end.** A small Streamlit web app lets anyone type in
   their own formula and instantly see the resulting bifurcation diagram, without touching any
   code — turning the underlying engine into an actual usable tool rather than just a script.

## What the results show

**Validation against the logistic map (a system with known, textbook behavior):**

| Parameter (`r`) | Equilibrium found | Classified as |
|---|---|---|
| 0.5 | 0.0 | stable |
| 2.5 | 0.6 | stable |
| 3.2 | 0.6875 | **unstable** |
| 3.6 | 0.7222 | **unstable** |

This is exactly the textbook result: the logistic map's positive equilibrium is stable for
smaller `r`, then loses stability right around `r = 3` — the well-known **period-doubling
threshold** — after which the system starts oscillating instead of settling down. The tool
reproduced this transition automatically, without being told in advance where it should occur,
which is the whole point of validating it this way.

The bifurcation diagram generated for this system (`r` from 2.5 to 4.0) shows the classic
period-doubling cascade: a single line splitting into two, then four, then dissolving into the
dense scatter of chaos as `r` approaches 4 — visually confirming the same story the stability
table tells numerically.

**Validation against the Nicholson-Bailey host-parasitoid model:**

| Parameter (λ) | Equilibrium found | Classified as |
|---|---|---|
| 1.5 | (1.216, 0.405) | **unstable** |
| 2.0 | (1.386, 0.693) | **unstable** |
| 3.0 | (1.648, 1.099) | **unstable** |

This also matches known theory: the classic (unmodified) Nicholson-Bailey model is famous in
ecology precisely *because* its positive equilibrium is always unstable — real host-parasitoid
populations in nature tend to oscillate or crash rather than settle into a steady balance unless
some additional stabilizing mechanism is added to the model. The tool correctly identified this
instability at every parameter value tested, matching the textbook result exactly.

## How to reproduce it

1. Open the notebook in Jupyter (no GPU required — everything here runs on CPU).
2. Run all cells top to bottom. The logistic map and Nicholson-Bailey validations, plus the
   bifurcation diagram, run in well under a minute.
3. To try the interactive version, run the final cell to generate `app.py`, then run
   `streamlit run app.py` in a terminal and enter any formula of your own.

## Adapting it beyond the demo

The version here validates against the plain Nicholson-Bailey model. The natural next step is to
swap in a stabilized or modified variant of that model — the kind studied in more advanced
ecological research — and confirm the tool reproduces that model's known (and typically more
interesting) stability behavior, turning this from a validation exercise into a genuine research
tool.
