# Chapter 2 — The Minimal Agentic Controller

Chapter 1 argued that mechanical controllers, software state machines, utility systems, planners, and AI agents all share a common structure.

This chapter defines that structure more precisely.

The goal is not to build the most sophisticated possible model.

It is the opposite.

We want the **smallest useful model** that can describe:

* a thermostat,
* a PID controller,
* a finite-state machine,
* a pushdown controller,
* a utility-based controller,
* a planner,
* and an LLM-based agent.

If one abstract model can describe all of them without becoming meaningless, then we have a foundation for Agentic Control Theory.

The minimal controller we will use has five essential elements:

1. a **plant**,
2. an **observation** of that plant,
3. an **agent** with internal state,
4. an **objective**,
5. a **policy** that selects actions which produce effects on the plant.

The entire system forms a feedback loop.

In plain language:

```text
Observe
→ Remember
→ Decide
→ Act
→ The plant changes
→ Observe again
```

Everything in this book grows from this loop.

---

## 2.1 The Plant

The **plant** is the system being influenced by the controller.

In classical control, the plant is usually physical.

Examples include:

* a motor,
* a vehicle,
* a furnace,
* a chemical process,
* a robotic arm.

In software systems, the plant may instead be:

* a game world,
* a process scheduler,
* a database,
* a network service,
* a simulated economy,
* another controller.

The important property is not that the plant is physical.

The important property is that it has **state** which can change over time.

We will represent the plant state as:

$$
x_t
$$

where \(t\) indicates time or control step.

The exact meaning of \(x_t\) depends on the system.

For a motor:

```text
position
velocity
temperature
```

For a game character:

```text
position
health
current target
inventory
```

For a software service:

```text
queue length
resource usage
request state
error state
```

The symbol is the same.

The meaning is domain-specific.

---

## 2.2 Observation

A controller usually does not have direct access to the entire plant state.

Instead, it receives an **observation**.

We write an observation as:

$$
o_t
$$

An observation is any information available to the agent about the current condition of the plant.

This distinction is important:

> **Observation is not necessarily the same thing as state.**

A physical sensor may measure only part of a system.

A temperature sensor cannot directly reveal airflow.

A camera cannot directly reveal every hidden object in a room.

A game AI may know only what is visible.

A software controller may know only what telemetry exposes.

An LLM agent may know only what has been placed into its current context.

So we can think of observation as a view of the plant:

$$
o_t = H(x_t)
$$

where \(H\) describes how plant state becomes observable information.

A more realistic form may include noise:

$$
o_t = H(x_t,n_t)
$$

where \(n_t\) represents measurement noise or uncertainty.

In normal language:

> The plant has some true condition, and the controller sees only what its sensors, telemetry, APIs, or context reveal.

---

## 2.3 Plant State, Controller State, and Estimated State

The word **state** is used in several different ways, so we need to distinguish them clearly.

### Plant state

The **plant state** describes the system being controlled.

$$
x_t
$$

Examples:

* motor velocity,
* robot position,
* database contents,
* game-world conditions.

### Controller state

The controller may also maintain its own internal state.

We will write this as:

$$
m_t
$$

The letter \(m\) is useful because this state often functions as **memory**.

Examples include:

* accumulated PID integral error,
* current finite-state-machine state,
* a timer,
* a cooldown,
* a control stack,
* a previous decision,
* a conversation history,
* a commitment to a selected behavior.

The plant state and controller state are not the same thing.

A motor may be rotating at 1000 RPM while the controller remembers that it has been above its target for 250 milliseconds.

That remembered fact belongs to the controller, not the motor.

### Estimated state

Sometimes the controller needs to infer information it cannot directly observe.

We may call this:

$$
\hat{x}_t
$$

The symbol means:

> an estimate of the actual plant state.

A robot may estimate its position from noisy sensors.

A software controller may infer service health from incomplete telemetry.

An AI agent may construct a model of a situation from partial observations.

This gives us three related but distinct concepts:

```text
actual plant state
        x

observed information
        o

estimated plant state
       x̂
```

Keeping them separate becomes increasingly important as systems become more complicated.

---

## 2.4 Memory

Memory is simply controller state that persists across decisions.

In the simplest controller, there may be almost none.

For example:

```text
if temperature < target:
    heater = on
```

The current observation is enough.

But many controllers need history.

A PID controller needs accumulated error.

A state machine needs to know its active state.

A pushdown controller needs a stack.

A utility controller may need commitment history to avoid constantly switching decisions.

An LLM-based controller may maintain task memory, conversation memory, or an external working set.

So:

> **Memory is the portion of controller state retained from previous control steps.**

In Agentic Control Theory, memory is not treated as something special reserved for intelligent agents.

It exists whenever past information affects future action.

That can be as small as one scalar.

---

## 2.5 Objective

An agent acts for some reason.

That reason is represented by an **objective**.

The objective describes which outcomes are preferred.

We will write it abstractly as:

$$
r_t
$$

The symbol \(r\) is traditional in control theory, where it often means reference or setpoint.

In Agentic Control Theory, the meaning is broader.

An objective can be:

* a target temperature,
* a desired velocity,
* a goal state,
* a utility function,
* a cost function,
* a priority,
* a task,
* a natural-language instruction.

Examples:

```text
Maintain 22 °C.
```

```text
Keep altitude at 1000 m.
```

```text
Survive.
```

```text
Choose the behavior with highest utility.
```

```text
Repair the failed service.
```

The form varies.

The role is the same.

> **An objective describes what the agent should prefer.**

---

## 2.6 Policy

The **policy** is the rule used by the agent to decide what to do.

We represent the policy as:

$$
\pi
$$

A policy may be tiny.

```text
if too cold:
    turn heater on
```

It may be mathematical.

```text
motor command = gain × error
```

It may be discrete.

```text
if EnemyVisible:
    transition to Chase
```

It may be comparative.

```text
choose the behavior with highest utility
```

It may involve planning.

```text
search possible action sequences and choose the best one
```

It may involve an LLM.

```text
reason about current context and propose the next action
```

The policy is the mechanism that maps what the agent knows and wants into what it does.

A useful general form is:

$$
(m_{t+1},a_t)
=
\pi(o_t,m_t,r_t)
$$

This looks intimidating if encountered cold, but its meaning is straightforward.

Given:

* what I currently observe,
* what I currently remember,
* and what I am trying to achieve,

determine:

* what action I should take,
* and what I should remember next.

In plain language:

```text
new memory, action
    =
policy(
    current observation,
    current memory,
    current objective
)
```

That is the heart of the controller.

---

## 2.7 Action

An **action** is what the agent selects.

We write it as:

$$
a_t
$$

Examples:

* apply 3 volts,
* open valve,
* transition to Patrol,
* push Attack state,
* send HTTP request,
* retry operation,
* invoke planner,
* ask an LLM to analyze a problem.

The action belongs to the controller's decision process.

But an action is not necessarily identical to what actually happens.

That distinction gives us the concept of **effect**.

---

## 2.8 Effect

An **effect** is the consequence an action produces in the plant.

Suppose a controller commands:

```text
motor torque = 10 N·m
```

That is the action.

The actual motor response may depend on:

* friction,
* load,
* saturation,
* failure,
* delay.

Likewise, a software agent might issue:

```text
restart service
```

but the service may fail to restart.

The selected action and the resulting effect are therefore distinct concepts.

We can describe this as:

```text
Agent chooses action
        ↓
Action is applied
        ↓
Plant responds
        ↓
Effect appears in plant state
```

This distinction becomes extremely useful when dealing with:

* unreliable actuators,
* network failures,
* delayed commands,
* permission failures,
* partial execution,
* uncertain environments.

An agent controls through actions.

Reality responds through effects.

---

## 2.9 Transition

A **transition** describes a change from one state to another.

There are two important transition types in an agentic control system.

### Controller transition

The controller's own state may change.

$$
m_t \rightarrow m_{t+1}
$$

Example:

```text
Patrol → Chase
```

or:

```text
stack = [Explore]
→
stack = [Explore, AvoidObstacle]
```

### Plant transition

The plant state also changes.

$$
x_t \rightarrow x_{t+1}
$$

We can represent this more explicitly as:

$$
x_{t+1}
=
F(x_t,a_t,d_t)
$$

where:

* \(x_t\) is current plant state,
* \(a_t\) is the action applied,
* \(d_t\) represents disturbances or external influences,
* \(F\) describes how the plant evolves.

In plain language:

> The plant's next state depends on its current state, what the controller does, and whatever else the world does to it.

This is true for both software and physical systems.

---

## 2.10 Disturbance

A controller does not own the universe.

Other things happen.

We call external influences on the plant **disturbances**.

We write them as:

$$
d_t
$$

Examples include:

* wind acting on an aircraft,
* a changing mechanical load,
* another process modifying shared state,
* a network outage,
* another player in a game,
* a human operator,
* an unexpected request spike.

Disturbances matter because control exists precisely because the future is not determined solely by the controller.

A perfectly predictable plant with no uncertainty and no external influences may need far less feedback.

Real systems are rarely that polite.

---

## 2.11 Feedback

A control loop becomes closed when the consequences of previous actions influence future decisions.

That is **feedback**.

A generic feedback loop looks like:

```text
Plant state
    ↓
Observation
    ↓
Agent
    ↓
Action
    ↓
Plant changes
    ↓
New observation
    ↺
```

The important feature is not merely repetition.

The important feature is that the next action depends on the observed result of previous actions.

Without this, the system is open-loop.

With it, the system can compensate for:

* disturbances,
* error,
* uncertainty,
* changing conditions,
* failed actions.

Feedback is what allows the controller to correct itself.

---

## 2.12 Timescale

Every controller operates at some **timescale**.

This may be represented by a sample period:

$$
\Delta t
$$

or by an update frequency.

Examples:

```text
motor current loop      → 10 kHz
motion controller       → 1 kHz
game behavior logic     → 30 Hz
planner                 → 1 Hz
LLM reasoning           → seconds
human supervision       → minutes or hours
```

The exact numbers vary enormously by application.

The important idea is:

> **Different controllers can participate in the same system while operating at different timescales.**

A slow controller may set goals for a faster controller.

A faster controller may stabilize behavior between slow decisions.

For example:

```text
LLM:
"Go inspect machine 4."

        ↓

Navigation controller:
"Move along this path."

        ↓

Velocity controller:
"Maintain 1.2 m/s."

        ↓

Motor controller:
"Apply this current."
```

None of these controllers replaces the others.

They occupy different layers.

---

## 2.13 The Minimal Agentic Control Model

We now have enough concepts to write the minimal model.

### Observation

$$
o_t
=
H(x_t,n_t)
$$

The controller receives an observation derived from plant state, possibly corrupted or limited by noise.

Plain language:

> The controller sees some imperfect view of reality.

### Policy

$$
(m_{t+1},a_t)
=
\pi(o_t,m_t,r_t)
$$

The controller uses observation, memory, and objective to determine its next action and memory state.

Plain language:

> Given what I see, what I remember, and what I want, decide what to do next.

### Plant transition

$$
x_{t+1}
=
F(x_t,a_t,d_t)
$$

The plant evolves according to its current state, the controller's action, and external disturbances.

Plain language:

> The world changes because of what the controller did and because of everything else happening to it.

Together:

$$
\boxed{
o_t = H(x_t,n_t)
}
$$

$$
\boxed{
(m_{t+1},a_t)=\pi(o_t,m_t,r_t)
}
$$

$$
\boxed{
x_{t+1}=F(x_t,a_t,d_t)
}
$$

This is our first formal model of an agentic control system.

It is intentionally generic.

Nothing in these equations requires:

* continuous control,
* discrete control,
* artificial intelligence,
* optimization,
* planning,
* or even complicated memory.

Those are specializations.

---

## 2.14 Example: Thermostat

Consider a simple thermostat.

### Plant

The room.

### Plant state

```text
room temperature
```

### Observation

```text
measured temperature
```

### Objective

```text
desired temperature = 22 °C
```

### Controller state

Possibly none.

A thermostat with hysteresis might remember whether heating is currently active.

### Policy

```text
if temperature < 21.5:
    heater on

if temperature > 22.5:
    heater off
```

### Action

```text
heater on/off command
```

### Effect

The heater changes the rate at which room temperature evolves.

### Disturbance

```text
outside temperature
open window
people entering room
sunlight
```

The entire thermostat fits the minimal agentic model.

No AI required.

---

## 2.15 Example: PID Controller

Consider a motor velocity controller.

### Plant

Motor and mechanical load.

### Plant state

```text
position
velocity
possibly temperature/current/etc.
```

### Observation

Measured velocity.

### Objective

Desired velocity.

### Controller state

The PID controller may remember:

* accumulated error,
* previous error.

### Policy

The controller computes an output from:

* current error,
* accumulated error,
* error rate.

### Action

Motor command.

### Effect

Motor torque changes motor velocity.

### Feedback

New velocity is measured on the next update.

A PID controller therefore fits the same model.

Its policy simply happens to be compact and mathematical.

---

## 2.16 Example: Finite-State Machine

Consider a game character.

States:

```text
Idle
Patrol
Chase
Attack
Flee
```

### Plant

The game world.

### Observation

```text
enemy visible
distance to enemy
health
position
```

### Controller state

Current behavior state.

```text
Chase
```

### Objective

Defeat threats while surviving.

### Policy

```text
if low health:
    Flee
else if enemy in range:
    Attack
else if enemy visible:
    Chase
else:
    Patrol
```

### Action

```text
move
attack
change state
```

### Effect

The character and game world change.

Again, the same model applies.

---

## 2.17 Example: Pushdown Controller

Finite-state machines become awkward when behavior has nested structure.

Suppose an agent is patrolling.

It encounters an obstacle.

It should temporarily avoid the obstacle and then resume the patrol exactly where it left off.

A pushdown controller can represent this naturally:

```text
[Patrol]
```

becomes:

```text
[Patrol, AvoidObstacle]
```

When avoidance completes:

```text
[Patrol]
```

The current controller state is no longer merely one enum value.

It is a stack.

That stack is still just:

$$
m_t
$$

The minimal model does not care whether controller memory is:

* one number,
* one enum,
* a stack,
* a tree,
* a world model,
* a database.

It remains controller state.

This is one of the strengths of using a general model.

---

## 2.18 Example: Utility Controller

Suppose several behaviors may be valid at once.

```text
Attack
Heal
Flee
Reload
TakeCover
```

A strict priority list may be too crude.

Instead, the controller evaluates utility:

```text
Attack    = 0.72
Heal      = 0.91
Flee      = 0.44
Reload    = 0.30
TakeCover = 0.78
```

The policy chooses the most desirable option according to some arbitration rule.

This is still:

$$
a_t=\pi(o_t,m_t,r_t)
$$

The policy has simply become more expressive.

If the controller remembers its previous choice to avoid constant switching, that memory becomes part of \(m_t\).

---

## 2.19 Example: LLM-Based Agent

Now consider an LLM-based software agent.

### Plant

A software development environment.

### Observation

```text
source files
compiler errors
test results
user instructions
tool output
```

### Controller state

```text
working memory
task history
current plan
external notes
```

### Objective

```text
Fix the failing build without breaking existing behavior.
```

### Policy

The LLM reasons over its current context and decides what tool or action to invoke.

### Actions

```text
read file
edit file
run compiler
run tests
search documentation
```

### Effects

Files change.

Commands execute.

Test results change.

### Feedback

The next reasoning step incorporates the result.

The LLM agent is therefore not conceptually outside control theory.

It simply uses an unusually expressive and computationally expensive policy.

---

## 2.20 Expressiveness Is Not Correctness

At this point it is tempting to rank controllers by sophistication.

That would be a mistake.

A more expressive controller is not automatically better.

An LLM may be able to reason about a problem that a PID controller cannot even represent.

But a PID controller may provide:

* deterministic timing,
* predictable behavior,
* extremely low latency,
* straightforward stability analysis.

An LLM provides none of those automatically.

Similarly, a utility controller may express richer behavior than a finite-state machine, but may introduce:

* oscillation,
* unstable arbitration,
* poor tuning,
* unpredictable transitions.

Agentic Control Theory therefore distinguishes:

> **policy expressiveness**

from:

> **control quality**

A powerful policy can still be a terrible controller.

---

## 2.21 Intelligence Is Optional

Nothing in the minimal model requires intelligence.

The agent needs only enough agency to select an action.

This gives us a spectrum:

```text
fixed rule
→ proportional feedback
→ PID
→ finite-state controller
→ pushdown controller
→ utility controller
→ planner
→ learned policy
→ LLM reasoning
```

The structure remains recognizable throughout.

The sophistication of the policy changes.

The existence of the control loop does not.

---

## 2.22 The Model Does Not Yet Guarantee Anything

The minimal agentic model is descriptive.

It tells us what pieces exist.

It does not yet tell us whether the controller is good.

In particular, the model does not guarantee:

### Stability

The controller may cause state to diverge.

### Optimality

The controller may choose poor actions.

### Safety

The controller may produce dangerous effects.

### Controllability

The available actions may be insufficient to reach the desired state.

### Observability

The available observations may be insufficient to understand the plant.

### Robustness

The controller may fail when conditions differ slightly from its assumptions.

### Determinism

The same observation may not always produce the same action.

### Bounded latency

The controller may take too long to decide.

### Correct objectives

The objective itself may be wrong.

These properties require additional theory.

The minimal model gives us the vocabulary required to ask the questions.

---

## 2.23 A Controller as an Agent

We can now state the central abstraction more precisely.

A controller is an agent characterized by:

* observations,
* controller state,
* objectives,
* a policy,
* actions,
* effects,
* and an interaction timescale.

The controller participates in a loop with a plant whose state evolves according to both the controller's actions and external influences.

In compact form:

```text
Plant
  ↓
Observation
  ↓
Agent
├─ Memory
├─ Objective
└─ Policy
  ↓
Action
  ↓
Effect
  ↓
Plant
  ↺
```

This is the **minimal agentic controller**.

Every later control architecture in this book will be described as a specialization or composition of this model.

---

## 2.24 Why This Abstraction Matters

The value of this abstraction is not that it makes every controller identical.

It does the opposite.

It gives us a common language for describing their differences.

We can compare two controllers by asking:

* What can each observe?
* What state does each retain?
* What objectives can each represent?
* What policies can each express?
* What actions can each select?
* What effects can each produce?
* How quickly can each respond?
* What uncertainty can each tolerate?
* How are controllers composed?

This allows us to discuss a PID loop and an LLM agent in the same theoretical framework without pretending that they are operationally equivalent.

That distinction will be important throughout this book.

---

## 2.25 Where We Go Next

We now have a generic model:

$$
o_t = H(x_t,n_t)
$$

$$
(m_{t+1},a_t)=\pi(o_t,m_t,r_t)
$$

$$
x_{t+1}=F(x_t,a_t,d_t)
$$

The next question is:

> What happens when the plant and controller evolve continuously rather than as discrete computational transitions?

That brings us back to classical control theory.

In the next chapter, we will reinterpret:

* proportional control,
* PID,
* feedback,
* state-space control,
* stability,
* damping,
* and controller bandwidth

through the agentic model introduced here.

The goal is not to replace classical control theory.

It is to show that classical control is one particularly important and well-developed region inside the larger space of agentic control.
