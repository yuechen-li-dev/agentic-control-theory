# Chapter 1 — What Is Control?

Control is one of those ideas that appears everywhere while somehow acquiring a different vocabulary in every field that touches it.

A mechanical engineer may think of a motor, a sensor, a setpoint, and a PID controller.

A computer scientist may think of a state machine, a scheduler, a game AI, or a program choosing what branch to execute next.

A roboticist may think of perception, planning, and actuation.

An AI engineer may think of an agent observing an environment, selecting actions, calling tools, and pursuing a goal.

These systems are usually taught as if they belong to different intellectual families.

They do not.

At their core, they are all concerned with the same problem:

> **Given some understanding of the current situation, what should happen next?**

That is the control problem.

This book develops **Agentic Control Theory**: a framework for treating mechanical controllers, software controllers, state machines, utility systems, planners, and AI agents as different expressions of the same underlying structure.

The starting point is intentionally simple.

Before transfer functions, state-space equations, pushdown automata, utility functions, or language models, we begin with something more basic:

**agency**.

---

## 1.1 Agency

In ordinary language, something has **agency** if it can act.

More precisely, an agent has some ability to choose among possible actions and thereby influence what happens next.

That does not require intelligence in the human sense.

It does not require consciousness.

It does not require language.

It does not even require complicated decision-making.

A thermostat has a tiny amount of agency.

A PID controller has agency.

A finite-state machine has agency.

A game AI has agency.

A robot has agency.

A human operator has agency.

An LLM-based autonomous system has agency.

These systems vary enormously in complexity, but they share one important property:

> They can select effects that influence future state.

That is the broad sense of agency used throughout this book.

An **agent** is therefore:

> **A system with agency: a system capable of selecting actions that influence the future state of some environment or system.**

This definition is deliberately broad.

The word *agent* has recently become strongly associated with large language models and autonomous AI systems. That is useful in one context, but too narrow for our purposes.

An agent does not become an agent merely because someone connected it to an LLM.

A one-line thermostat can be an agent.

A PID loop can be an agent.

An operating-system scheduler can be an agent.

What changes between them is not whether agency exists, but how much machinery is used to express it.

---

## 1.2 The Simplest Control Loop

The simplest useful model of agency is:

```text
Observe
→ Decide
→ Act
→ The world changes
→ Observe again
```

That loop is the conceptual center of this book.

Everything else is elaboration.

A more detailed agent may:

```text
Observe
→ Update internal state
→ Evaluate objectives
→ Select an action
→ Produce an effect
→ Environment changes
→ Repeat
```

Different fields describe this loop differently.

A mechanical engineer may write:

```text
Measure plant
→ Compute control input
→ Actuate plant
→ Plant evolves
→ Measure again
```

A computer scientist may write:

```text
Read state
→ Evaluate condition
→ Select transition
→ Execute action
→ State changes
→ Repeat
```

An AI engineer may write:

```text
Observe context
→ Reason
→ Select tool or action
→ Execute
→ Environment changes
→ Observe again
```

These descriptions emphasize different implementation details, but structurally they are extremely similar.

That similarity is not accidental.

They are all control loops.

---

## 1.3 What Is a Controller?

A **controller** is an agent whose actions are chosen in order to influence the behavior of another system.

That other system is traditionally called the **plant**.

This gives us the first important distinction:

* the **controller** decides what to do;
* the **plant** is the thing whose behavior is being influenced.

In classical mechanical control, the distinction is easy to visualize.

A controller commands a motor.

The motor and attached mechanism are the plant.

A sensor measures the result.

The controller uses that measurement to decide what to command next.

But nothing about the idea of a plant requires gears, shafts, motors, or even physical matter.

A plant can also be:

* a game world,
* a robot,
* a database,
* a network service,
* an operating system,
* a simulated economy,
* a manufacturing process,
* a vehicle,
* another software agent,
* or another controller.

Agentic Control Theory uses the word **plant** in this generalized sense:

> **A plant is any system whose state may be influenced by the actions of a controller.**

The plant is simply the thing being controlled.

---

## 1.4 For Mechanical Engineers: The Familiar Picture

The classical feedback-control diagram usually looks something like this:

```text
Reference
   │
   ▼
Controller
   │
   ▼
Actuator
   │
   ▼
 Plant
   │
   ▼
Sensor
   │
   └──────────── feedback ────────────┐
                                      │
                                      └── back to controller
```

Suppose we want a motor shaft to rotate at a desired speed.

The desired speed is the **reference** or **setpoint**.

The sensor measures the actual speed.

The controller compares desired speed with measured speed.

It decides how much voltage or current to apply.

The motor responds.

The controller measures the result again.

This is a closed-loop controller.

The important point is not the specific mathematics used to calculate the motor command.

The important point is the structure:

1. observe,
2. evaluate,
3. act,
4. observe again.

A PID controller is one possible implementation of that loop.

It is not the definition of control itself.

---

## 1.5 For Computer Scientists: You Already Build Controllers

Now consider an ordinary software state machine.

```text
Idle
 ├─ enemy visible → Attack
 ├─ low health    → Flee
 └─ otherwise     → Idle
```

Every update, the system:

1. reads some state,
2. evaluates conditions,
3. selects a transition,
4. performs an action,
5. updates its state.

That is also a control loop.

Consider an operating-system scheduler.

It observes:

* runnable processes,
* priorities,
* resource availability,
* elapsed time.

It then decides which process runs next.

Its actions influence the future behavior of the computer.

That is control.

Consider a retry policy in a distributed system.

```text
request failed
→ wait
→ retry
→ inspect result
→ retry again or give up
```

Also control.

Computer science is full of controllers.

It simply does not always call them that.

---

## 1.6 State: The First Shared Concept

One of the clearest bridges between computer science and classical control is the word **state**.

Both disciplines use it constantly.

They merely emphasize different aspects of it.

### The computer-science view

In software, state is the stored information that influences what a program will do next.

Examples include:

* the current enum value in a state machine,
* the call stack,
* variables in memory,
* the contents of a queue,
* the currently active task,
* a cooldown timer,
* the top state in a behavior stack.

Change the state, and future program behavior may change.

### The mechanical-engineering view

In control theory, state usually refers to the variables required to describe the current condition of a dynamical system well enough to determine its future evolution, given future inputs.

For a moving object, position alone may not be enough.

Two objects can occupy the same position while moving in opposite directions.

Position and velocity together provide a more complete description of state.

### The agentic view

Agentic Control Theory treats these as instances of the same idea:

> **State is information about a system that matters to its future behavior.**

Sometimes that state is physical.

Sometimes it is computational.

Sometimes it is both.

A robot may simultaneously have:

* physical state,
* estimated state,
* mission state,
* navigation state,
* task-stack state,
* conversation state.

None of these categories invalidates the others.

They simply operate at different levels of the same system.

---

## 1.7 Transitions: How State Changes

Once we have state, we need a way for it to change.

In computer science, we often call this a **transition**.

```text
Idle → Patrol
Patrol → Chase
Chase → Attack
```

In mechanical systems, state also changes, but usually continuously.

Velocity changes.

Temperature rises.

Pressure falls.

Position evolves.

The underlying idea is the same:

> A transition describes how current state becomes future state.

For discrete systems, we may eventually write:

$$
x_{t+1}=F(x_t,u_t)
$$

For continuous systems:

$$
\dot{x}=f(x,u)
$$

There is no need to worry about the notation yet.

Both expressions are saying roughly:

> “What happens next depends on where the system is now and what is being done to it.”

One describes change as steps.

The other describes change as continuous evolution.

Agentic Control Theory needs both.

---

## 1.8 Effects

An agent needs some way to influence the plant.

We will call the result of such an action an **effect**.

An effect is:

> **A change an agent attempts to produce in the state or behavior of a plant.**

Examples include:

* applying torque to a motor,
* opening a valve,
* setting a desired velocity,
* writing to a database,
* sending a network message,
* issuing a game movement command,
* pushing a state onto a control stack,
* starting another controller,
* calling an external tool,
* invoking an LLM.

In physical systems, effects are often realized by **actuators**.

A motor is an actuator.

A hydraulic cylinder is an actuator.

A valve is an actuator.

But software control requires a broader word.

An HTTP request is not normally called an actuator.

A function call is not an actuator.

A state transition is not an actuator.

They can all still produce effects.

So throughout this book:

> **Actuation is one physical form of effect.**

Effects are the more general concept.

---

## 1.9 Feedback

A controller becomes much more useful when it can observe what happened after acting.

This is **feedback**.

Consider two approaches to heating a room.

### Open-loop control

```text
Turn heater on for 20 minutes.
```

The controller does not check the temperature.

Maybe the room reaches the desired temperature.

Maybe it does not.

Maybe somebody opened a window.

Maybe the heater is broken.

The controller does not know.

### Closed-loop control

```text
Measure temperature.
If too cold, heat.
Measure again.
Adjust.
Repeat.
```

Now the controller responds to the actual condition of the system.

That is feedback.

The same distinction appears constantly in software.

An open-loop software command might be:

```text
Send request.
Assume success.
```

A closed-loop version might be:

```text
Send request.
Observe response.
If unsuccessful, change behavior.
Repeat as necessary.
```

Feedback turns action into control.

---

## 1.10 Objectives

Why does the controller choose one action rather than another?

Because it has some concept of a desired outcome.

We will call that an **objective**.

Objectives can take many forms.

A mechanical controller may have a setpoint:

```text
desired temperature = 22 °C
```

A game AI may have a utility function:

```text
AttackScore = 0.8
FleeScore   = 0.3
HealScore   = 0.6
```

A robot may have a target position.

A scheduler may try to minimize latency.

A database controller may try to preserve consistency.

An autonomous software agent may receive:

```text
Find the cause of this failure and repair it.
```

These look very different.

But each supplies a criterion by which one future state is preferred over another.

So:

> **An objective describes what outcomes a controller should prefer.**

A setpoint is one form of objective.

A utility function is another.

A goal state is another.

A constraint may restrict which objectives or actions are acceptable.

A natural-language instruction can encode an objective too.

---

## 1.11 Policy: How an Agent Chooses

A controller still needs some rule connecting observations and objectives to actions.

That rule is its **policy**.

A policy can be extremely simple.

```text
if temperature < target:
    heater = on
else:
    heater = off
```

Or:

```text
if EnemyVisible:
    State = Chase
```

Or:

```text
choose action with highest utility
```

Or:

```text
ask a language model to reason about the situation
```

A policy is simply:

> **The rule by which an agent selects what to do.**

The complexity of that rule can vary enormously.

That variation gives us a useful way to think about different kinds of controllers.

---

## 1.12 A Spectrum of Controllers

Consider several controllers arranged loosely by expressive complexity.

### Thermostat

```text
Too cold → heat
Too hot  → stop heating
```

Tiny state.

Tiny policy.

Very fast.

Very predictable.

### PID controller

A PID controller reacts not only to present error, but also to accumulated error and the rate at which error is changing.

It therefore has more internal structure than a simple thermostat.

Still, its policy is compact and mathematically rigid.

### Finite-state machine

A finite-state machine can represent discrete modes:

```text
Idle
Patrol
Chase
Attack
Flee
```

Its behavior depends explicitly on which state is currently active.

### Pushdown controller

A pushdown controller adds stack-like memory.

Instead of merely changing from one state to another, it can suspend one control context, enter another, and later resume where it left off.

That makes nested behavior much easier to express.

### Utility controller

A utility controller evaluates several possible actions or transitions and compares their desirability.

Instead of saying only:

```text
if condition:
    transition
```

it may say:

```text
How desirable is each option right now?
```

### Planning system

A planner considers sequences of future actions.

Its policy may involve predicting consequences before acting.

### LLM-based controller

A language model can evaluate rich semantic context, produce plans, generate tools, interpret ambiguous objectives, and reason across domains.

Its policy is extraordinarily expressive compared with a thermostat or PID loop.

But it is also slower, more expensive, and usually less predictable.

This leads to an important idea:

> **Greater policy expressiveness does not automatically mean better control.**

Sometimes a PID loop is exactly the right controller.

Sometimes an LLM is absurdly inappropriate.

A language model should not be deciding motor current every millisecond.

The useful question is not:

> “Which controller is smartest?”

It is:

> “Which controller has the right capabilities, reliability, and timescale for this layer of the problem?”

---

## 1.13 Timescale

Controllers do not all operate at the same speed.

A motor controller might update thousands of times per second.

A game AI may update tens of times per second.

A route planner may reconsider once every few seconds.

An LLM may take seconds to produce a decision.

A human manager may make strategic decisions once per day or once per month.

These can still participate in one larger control system.

For example:

```text
LLM mission planner
        ↓
Behavior controller
        ↓
Motion planner
        ↓
PID motor controller
        ↓
Physical motor
```

The slow controller does not replace the fast controller.

It controls at a different level.

A higher-level controller may decide:

```text
Move the robot to the loading station.
```

A lower-level controller determines the path.

Another lower-level controller regulates wheel velocity.

Another regulates motor current.

All are agents.

All are controllers.

They simply act at different timescales and levels of abstraction.

---

## 1.14 Controllers Can Control Controllers

Once control is understood broadly, another fact becomes obvious:

> A controller can itself be part of a plant controlled by another controller.

Suppose a high-level behavior system changes the target velocity given to a PID loop.

From the PID controller's point of view, the motor is the plant.

From the behavior controller's point of view, the PID-controlled motor subsystem may itself be treated as part of the plant.

From an autonomous mission planner's point of view, the entire robot may be the plant.

Control therefore composes.

```text
Mission Controller
        ↓
Behavior Controller
        ↓
Motion Controller
        ↓
Motor Controller
        ↓
Motor
```

Each layer can abstract the details beneath it.

This is one of the main reasons a unified theory is useful.

Real systems are rarely one controller acting directly on one simple plant.

They are networks and hierarchies of controllers acting upon systems that contain other controllers.

---

## 1.15 Intelligence Is Not the Same as Agency

It is tempting to treat increasingly sophisticated controllers as increasingly intelligent versions of the same thing.

That is only partly useful.

Agency and intelligence are different properties.

Agency concerns the capacity to act.

Intelligence concerns the sophistication with which actions can be selected, predicted, or reasoned about.

A PID controller has agency but essentially no general reasoning capability.

A state machine can express complicated behavior without possessing general intelligence.

An LLM may provide extremely broad reasoning capability while still depending on an external controller to determine when it should be invoked, what context it should receive, and what may be done with its response.

This distinction matters because modern AI terminology often conflates the two.

Agentic Control Theory does not require every agent to be intelligent.

It requires only that the agent participate in selecting effects.

---

## 1.16 The Same Problem in Different Dialects

We can now put the major concepts side by side.

| Mechanical-control language | Computer-science language | Agentic-control language |
| --------------------------- | ------------------------- | ------------------------ |
| Plant                       | Environment / system      | Plant                    |
| Sensor measurement          | Input / observation       | Observation              |
| State vector                | Program state             | State                    |
| Controller                  | Decision logic            | Agent / controller       |
| Control law                 | Algorithm / policy        | Policy                   |
| Setpoint                    | Goal / target             | Objective                |
| Actuator command            | Action / effect           | Effect                   |
| Dynamics                    | State transition          | Transition               |
| Feedback loop               | Update loop               | Feedback                 |
| Sampling period             | Tick / update rate        | Timescale                |

The words are not perfectly interchangeable in every context.

Nor should they be.

Each discipline developed terminology suited to its problems.

The important observation is that the structures beneath those terms overlap far more than their traditional vocabularies suggest.

---

## 1.17 A First Unified Model

We can now describe a generic controller without committing to mechanical or computational implementation.

An agent:

1. receives observations,
2. maintains whatever state or memory it requires,
3. has some objective,
4. uses a policy to choose an action,
5. produces an effect upon a plant,
6. observes the resulting state,
7. repeats.

In compact form:

```text
Observation
    ↓
  Agent
 ┌───────────────┐
 │ state/memory  │
 │ objective     │
 │ policy        │
 └───────────────┘
    ↓
  Effect
    ↓
   Plant
    ↓
new Observation
```

Later we will formalize this.

For now, the important thing is to recognize the pattern.

A PID controller fits it.

A state machine fits it.

A utility AI fits it.

A planner fits it.

A language-model agent fits it.

The details differ enormously.

The skeleton does not.

---

## 1.18 What Agentic Control Theory Claims

Agentic Control Theory begins with three simple claims.

### 1. Controllers are agents

A controller observes some condition, selects among possible effects according to a policy, and influences future state.

Its agency may be minimal or extremely expressive.

### 2. Agents are controllers

An agent operating toward an objective controls some aspect of its environment, even if that environment is computational rather than mechanical.

### 3. Control systems differ in representation, policy, effects, and timescale

The difference between a PID loop and an LLM agent is enormous in implementation, but it is not necessary to treat them as members of entirely unrelated conceptual categories.

Their differences can instead be described in terms such as:

* what they observe,
* what state they retain,
* what objectives they represent,
* what policies they can express,
* what effects they can produce,
* what plants they influence,
* and how quickly they operate.

This gives us a common vocabulary in which systems from mechanical engineering and computer science can be described together.

---

## 1.19 Why Bother Unifying Them?

A unified model is useful only if it helps us build or understand something.

The practical motivation is that modern systems increasingly combine several kinds of control at once.

A robot may contain:

* current controllers,
* velocity controllers,
* trajectory controllers,
* state machines,
* utility selection,
* path planning,
* computer vision,
* language-model reasoning,
* and human supervision.

Software infrastructure may contain:

* health checks,
* retry controllers,
* rate controllers,
* schedulers,
* policy engines,
* planners,
* autonomous remediation,
* and human operators.

Games combine:

* animation control,
* locomotion control,
* state machines,
* utility AI,
* planning,
* scripting,
* physics,
* and increasingly language models.

Treating every layer as an unrelated special case makes composition harder to reason about.

A shared framework lets us ask common questions instead:

* What does this controller observe?
* What internal state does it retain?
* What objective is it pursuing?
* What effects can it produce?
* How are competing actions selected?
* How quickly may it react?
* What happens when observations are delayed?
* Can it oscillate between decisions?
* Can higher-level controllers override it?
* Can the controller fail safely?
* Can its behavior be inspected and reproduced?

Those are control questions whether the implementation contains differential equations, `switch` statements, utility scores, or transformer inference.

---

## 1.20 The Road Ahead

This chapter has intentionally avoided most of the mathematics and machinery of control theory.

We have not yet discussed:

* transfer functions,
* stability,
* poles,
* damping,
* Lyapunov functions,
* finite automata,
* pushdown automata,
* utility arbitration,
* hysteresis,
* planning,
* observers,
* optimal control,
* or language-model cognition.

Those will come later.

First we need a common foundation.

The central idea is simple:

> **Control is the process by which an agent observes a system, selects effects according to some objective and policy, and thereby influences what happens next.**

Mechanical control and computational control are not identical disciplines.

Their tools are different.

Their constraints are different.

Their histories are different.

But beneath those differences lies a shared structure.

Once that structure is made explicit, PID controllers, state machines, utility systems, planners, and AI agents can be discussed within one language.

That language is the subject of this book.

In the next chapter, we will define the **minimal agentic controller** and turn the intuitive model introduced here into a precise set of primitives:

**agent, plant, observation, state, objective, policy, effect, transition, feedback, and timescale.**
