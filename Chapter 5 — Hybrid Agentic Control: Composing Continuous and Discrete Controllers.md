# Chapter 5 — Hybrid Agentic Control: Composing Continuous and Discrete Controllers

The previous two chapters introduced two major control traditions.

Continuous control focuses on systems whose state evolves through time:

* position,
* velocity,
* temperature,
* pressure,
* current,
* force.

Discrete control focuses on systems whose behavior changes through explicit modes:

* idle,
* patrol,
* chase,
* attack,
* fault,
* recovery.

Neither is sufficient by itself for many real systems.

A robot may need a continuous controller to regulate motor velocity while a discrete controller decides whether it should navigate, dock, recharge, or stop.

A vehicle may use continuous steering and throttle control while discrete logic selects cruise, braking, parking, or emergency modes.

A software service may use continuous or numerical feedback to regulate request rates while discrete logic decides whether to retry, fail over, restart, or shut down.

A modern autonomous system may contain all of these at once.

The goal of Agentic Control Theory is not to force continuous and discrete control into one algorithm.

The goal is to give them one compositional model.

The central idea is:

> **Controllers can control other controllers.**

A lower-level controller may itself become part of the plant seen by a higher-level controller.

This is the bridge that allows continuous control, state machines, utility systems, planners, and LLM agents to coexist in one hierarchy.

---

## 5.1 Regulation and Mode Selection

Continuous and discrete controllers tend to answer different kinds of questions.

Continuous control often asks:

> How much?

Examples:

```text
How much torque?
How much current?
How much steering angle?
How much cooling?
How fast should we move?
```

Discrete control often asks:

> Which mode?

Examples:

```text
Should we navigate?
Should we stop?
Should we dock?
Should we recharge?
Should we investigate?
```

These are not competing questions.

They occur at different levels.

A discrete controller might choose:

```text
Mode = Navigate
```

A continuous controller underneath might compute:

```text
DesiredVelocity = 1.2 m/s
```

Another controller underneath that might compute:

```text
MotorTorque = 3.6 N·m
```

The upper controller decides what kind of behavior should happen.

The lower controller determines how to realize that behavior.

---

## 5.2 Controllers as Plants

The simplest plant-controller relationship is:

```text
Controller
    ↓
Plant
```

But real systems quickly become layered.

Suppose we have:

```text
Velocity Controller
    ↓
Motor
```

From the velocity controller's perspective, the motor is the plant.

Now add a trajectory controller:

```text
Trajectory Controller
    ↓
Velocity Controller
    ↓
Motor
```

The trajectory controller does not need to directly understand motor current.

It can treat:

```text
Velocity Controller + Motor
```

as one controlled subsystem.

From its perspective, that entire subsystem is the plant.

This leads to a foundational rule:

> **A controller together with its plant can be treated as a new plant by a higher-level controller.**

This recursive composition is one of the most important ideas in Agentic Control Theory.

---

## 5.3 Closed-Loop Subsystems

Suppose a velocity controller regulates a motor so that:

```text
DesiredVelocity
```

becomes:

```text
ActualVelocity
```

The pair:

```text
Velocity Controller
+
Motor
```

forms a closed-loop subsystem.

A higher-level controller may not care how the subsystem works internally.

It may simply assume:

```text
Command velocity
→ subsystem attempts to achieve velocity
```

The internal loop can then be abstracted.

This is exactly how software systems are often composed.

A caller does not need to know every internal state transition of a service.

It interacts through an interface.

Control composition works similarly.

---

## 5.4 Setpoints as Inter-Controller Messages

One of the simplest ways for one controller to control another is to change its objective.

Suppose a high-level controller decides:

```text
DesiredPosition = 10 m
```

A position controller may translate that into:

```text
DesiredVelocity = 1.5 m/s
```

A velocity controller may translate that into:

```text
DesiredTorque = 4.0 N·m
```

A motor-current controller may translate that into:

```text
DesiredCurrent = 6.2 A
```

So the hierarchy becomes:

```text
Position objective
      ↓
Position controller
      ↓
Velocity objective
      ↓
Velocity controller
      ↓
Torque objective
      ↓
Motor controller
      ↓
Physical plant
```

Each controller converts a higher-level objective into a lower-level objective.

This gives us another important principle:

> **Setpoints are often messages between controllers.**

The output of one control layer becomes the objective of another.

---

## 5.5 Hierarchical Control

A system organized this way is **hierarchical**.

A rough autonomous-system hierarchy might look like:

```text
Mission Layer
    ↓
Behavior Layer
    ↓
Trajectory Layer
    ↓
Stabilization Layer
    ↓
Actuation Layer
    ↓
Plant
```

These names are not mandatory.

The important pattern is:

* higher layers operate at greater abstraction,
* lower layers operate closer to the plant.

A high-level controller may decide:

```text
Inspect machine 4.
```

A behavior controller decides:

```text
Navigate to machine 4.
```

A trajectory controller decides:

```text
Follow this path.
```

A velocity controller decides:

```text
Move at this speed.
```

A motor controller decides:

```text
Apply this current.
```

The entire hierarchy is one control system.

---

## 5.6 Abstraction Increases Upward

Moving upward through a hierarchy, objectives usually become more semantic.

For example:

```text
Motor current
↑
Wheel torque
↑
Vehicle velocity
↑
Path following
↑
Destination
↑
Mission objective
```

The lower layers care about precise physical quantities.

The higher layers care about behavior and goals.

This is one reason an LLM can be extremely useful at a high level while being entirely inappropriate at a low level.

The high level may ask:

> Which machine should we inspect next?

The low level asks:

> What current should this motor receive during the next millisecond?

Those are different control problems.

---

## 5.7 Timescale Separation

Control layers usually operate at different timescales.

For example:

```text
Motor current control     10,000 Hz
Velocity control           1,000 Hz
Trajectory control           100 Hz
Behavior control              10 Hz
Mission planning             0.1 Hz
LLM deliberation        seconds
Human supervision      minutes/hours
```

These numbers are illustrative, not universal.

The principle is:

> **Faster layers regulate faster dynamics. Slower layers modify goals and modes.**

A high-level controller should not need to micromanage every fast transient.

That is what lower-level controllers are for.

---

## 5.8 Why Timescale Separation Matters

Suppose a motor requires stabilization every millisecond.

Now imagine a mission planner takes two seconds to decide what to do next.

If the motor waits for the planner:

```text
motor unstable
→ planner still thinking
→ machine falls over
```

This is obviously unacceptable.

Instead:

```text
slow controller:
    choose objective occasionally

fast controller:
    continuously maintain stability
```

So while the high-level agent is thinking:

```text
Should I go left or right?
```

the lower-level controller continues ensuring:

```text
Do not fall over.
```

This is one of the strongest practical arguments for hierarchical agentic control.

---

## 5.9 Discrete Controllers Can Select Continuous Controllers

A discrete transition may change which continuous controller is active.

For example:

```text
Cruise
Brake
EmergencyStop
```

Each mode may use different control laws.

In `Cruise`:

```text
maintain speed
```

In `Brake`:

```text
reduce velocity according to deceleration profile
```

In `EmergencyStop`:

```text
maximize safe deceleration
```

The discrete state therefore determines:

> Which continuous control policy currently has authority?

This is a classic form of hybrid control.

---

## 5.10 Continuous State Can Trigger Discrete Transitions

The direction also goes the other way.

A continuous variable may trigger a discrete mode change.

For example:

$$
T > 100^\circ C
$$

may cause:

```text
Normal
→ OverheatProtection
```

or:

$$
Battery < 10\%
$$

may cause:

```text
Mission
→ ReturnToCharge
```

or:

$$
Distance < 0.5\text{ m}
$$

may cause:

```text
Navigate
→ Dock
```

So:

```text
continuous observation
→ discrete transition
→ different continuous controller
```

The two traditions are already intertwined.

---

## 5.11 Hybrid State

A system containing both continuous and discrete control has both kinds of state.

Let:

$$
x
$$

represent continuous state.

Let:

$$
m
$$

represent discrete mode.

We can combine them conceptually:

$$
z=(x,m)
$$

For example:

```text
continuous state:
    position = 4.2 m
    velocity = 1.1 m/s
    battery = 18%

discrete mode:
    Navigate
```

The full system state includes both.

This is a **hybrid state**.

---

## 5.12 Continuous Dynamics Depend on Mode

The continuous dynamics may change depending on discrete mode.

We can write:

$$
\dot{x}=f_m(x,u)
$$

This means:

> The system's continuous behavior depends on which discrete mode \(m\) is active.

For example:

```text
Mode = Cruise
→ one control law

Mode = Docking
→ another control law

Mode = EmergencyStop
→ another control law
```

The plant has continuous dynamics.

The discrete controller changes which dynamics or controller apply.

---

## 5.13 Guards Connect the Two Worlds

A discrete transition may use continuous state as a guard.

For example:

```text
when Speed < 0.1 m/s:
    Braking → Stopped
```

or:

```text
when Temperature > 100 °C:
    Normal → Cooling
```

or:

```text
when DistanceToDock < 0.5 m:
    Navigate → Dock
```

This is one of the simplest hybrid-control structures:

```text
continuous state
    ↓
guard
    ↓
discrete transition
```

The transition may then change the continuous policy.

---

## 5.14 Control Authority

When multiple controllers exist, we must ask:

> Who is allowed to command what?

This is **control authority**.

Suppose we have:

```text
Mission Planner
    ↓
Navigation Controller
    ↓
Collision Avoidance
    ↓
Motor Control
```

The mission planner may request:

```text
Go forward.
```

But collision avoidance observes:

```text
Wall directly ahead.
```

Which controller wins?

Usually:

```text
Collision Avoidance
```

must have authority to override the higher-level command.

This suggests:

> Higher abstraction does not necessarily mean greater immediate authority.

---

## 5.15 Authority Is Often Local

A useful hierarchy might be:

```text
Mission goal
    ↓
Navigation
    ↓
Safety override
    ↓
Actuation
```

The mission controller decides what the system should try to accomplish.

The safety controller decides what actions are currently permissible.

For example:

```text
Mission:
    move forward

Safety:
    collision imminent

Result:
    stop
```

The safety controller may be lower in semantic abstraction but stronger in immediate authority.

This is common in physical systems.

Emergency stop circuitry is not expected to negotiate politely with the mission planner.

---

## 5.16 Objectives and Constraints

This gives us a useful distinction.

An **objective** says:

> What should the controller prefer?

A **constraint** says:

> What is allowed?

For example:

```text
Objective:
    reach destination quickly

Constraints:
    do not exceed safe speed
    do not collide
    do not leave operating area
```

Higher-level agents may produce objectives.

Lower-level safety controllers may enforce constraints.

Both participate in the control hierarchy.

---

## 5.17 Overrides

An **override** occurs when one controller temporarily takes authority away from another.

For example:

```text
Normal Navigation
    ↓
Obstacle detected
    ↓
Collision Avoidance override
    ↓
Obstacle cleared
    ↓
Return to Navigation
```

This looks suspiciously similar to the pushdown control structure from Chapter 4.

That is not accidental.

We may model:

```text
[Navigation]
```

then:

```text
[Navigation, AvoidObstacle]
```

then:

```text
[Navigation]
```

A discrete control stack can encode temporary authority transfer.

---

## 5.18 Interrupts and Safety

Some overrides are urgent enough to behave like interrupts.

For example:

```text
EmergencyStop
CriticalOvertemperature
LossOfStability
HumanAbort
```

These should not normally participate in ordinary utility competition.

We do not want:

```text
ContinueMission utility = 0.91
EmergencyStop utility   = 0.89

therefore continue mission
```

That would be absurd.

Instead:

```text
if EmergencyStopRequired:
    preempt ordinary control
```

This introduces an important distinction:

> **Not every control decision should be reduced to one global utility score.**

Hard constraints and authority rules remain useful.

---

## 5.19 Stability Does Not Automatically Compose

Suppose the low-level velocity controller is perfectly stable.

Now place above it a supervisor that alternates:

```text
DesiredVelocity = +10
DesiredVelocity = -10
DesiredVelocity = +10
DesiredVelocity = -10
```

every few milliseconds.

The velocity controller may faithfully track those commands.

The overall system is still terrible.

Conversely, suppose the discrete supervisor is perfectly sensible but commands a badly tuned unstable continuous controller.

The overall system is still unstable.

Therefore:

> **Stable components do not automatically produce a stable composition.**

Control must be analyzed across boundaries.

---

## 5.20 Mode Chatter in Hybrid Systems

Suppose:

```text
Mode A
when Temperature > 100
→ Mode B
```

and:

```text
Mode B
when Temperature < 100
→ Mode A
```

If temperature hovers around 100:

```text
99.9
100.1
99.8
100.2
```

the controller switches rapidly:

```text
A
B
A
B
A
B
```

This is hybrid chatter.

Chapter 4 introduced hysteresis as one solution.

Instead:

```text
A → B when Temperature > 102
B → A when Temperature < 98
```

Now the system has a stable switching region.

---

## 5.21 Dwell Time and Minimum Commitment

A second stabilization strategy is to require a mode to remain active for some minimum period.

For example:

```text
once Docking begins:
    remain in Docking for at least 500 ms
```

unless a safety interrupt occurs.

In hybrid-systems language, this resembles **dwell time**.

In computational behavior systems, we may call it **minimum commitment**.

The concepts are closely related.

Both say:

> **Do not switch control regimes faster than the system can meaningfully respond.**

This is one of the cleanest bridges between mechanical and computational control.

---

## 5.22 Transition Cost

Switching controllers may itself have cost.

A change of control mode may require:

* reinitialization,
* actuator reconfiguration,
* trajectory cancellation,
* context reconstruction,
* network calls,
* model invocation,
* lost work.

So even if another mode appears slightly better, switching may not be worthwhile.

We may model:

$$
U_{\text{effective}}
=
U_{\text{candidate}}
-
C_{\text{switch}}
$$

or enforce the cost through hysteresis or commitment.

The general principle is:

> **Control decisions should account for the cost of changing control decisions.**

---

## 5.23 Controller Contracts

Composition becomes easier when controllers expose clear contracts.

A controller contract may specify:

### Inputs

What observations or commands it accepts.

### Outputs

What actions or references it produces.

### Operating range

What states or values it can handle.

### Timing

How quickly it updates and how quickly it is expected to respond.

### Guarantees

What behavior it promises under valid conditions.

### Failure modes

How it reports or responds when it cannot satisfy the objective.

For example:

```text
VelocityController

Input:
    desired velocity

Observation:
    measured velocity

Output:
    motor torque request

Operating range:
    -5 m/s to +5 m/s

Update rate:
    1 kHz

Guarantee:
    track velocity within tolerance
    under specified load bounds

Failure:
    report saturation or sensor fault
```

This makes the controller usable as a subsystem by higher-level agents.

---

## 5.24 Contracts Hide Internal Complexity

A higher-level controller does not need to know:

```text
PID gains
observer internals
filter coefficients
motor electrical model
```

It needs to know:

```text
what command can I issue?
what result should I expect?
how long should it take?
what failures can occur?
```

This is exactly how software abstractions work.

The controller becomes a semantic interface.

This is another major bridge between computer science and mechanical control.

---

## 5.25 Controllers as Services

For a computer scientist, one useful analogy is to think of a lower-level controller as a service.

The caller provides:

```text
request
```

The controller produces:

```text
effect
```

while maintaining its own internal state.

The caller does not micromanage implementation.

For example:

```text
NavigateTo(Target)
```

may internally use:

```text
path planning
trajectory generation
velocity control
motor control
```

The higher-level controller simply cares about:

```text
Running
Completed
Failed
```

This resembles an asynchronous software operation, except the operation is itself a feedback controller interacting continuously with a changing plant.

---

## 5.26 Completion as Feedback to Higher Layers

Suppose a high-level controller requests:

```text
NavigateTo(Dock)
```

The navigation controller begins operating.

Eventually it may return:

```text
Completed
```

or:

```text
Failed
```

That outcome becomes feedback to the higher-level controller.

So the hierarchy contains nested feedback loops:

```text
High-level controller
    ↓ objective
Lower-level controller
    ↓ actions
Plant
    ↑ observations
Lower-level controller
    ↑ completion/failure
High-level controller
```

Feedback therefore exists both within and between layers.

---

## 5.27 Different Layers See Different Plants

The same physical system may look like a different plant at each level.

For the motor-current controller:

```text
Plant = motor winding and motor mechanics
```

For the velocity controller:

```text
Plant = motor + current controller
```

For the navigation controller:

```text
Plant = mobile robot motion subsystem
```

For the mission planner:

```text
Plant = entire robot
```

For a human operator:

```text
Plant = robot + autonomous control system
```

The plant is therefore partly a matter of abstraction boundary.

This is not a contradiction.

It is composition.

---

## 5.28 Agent and Plant Are Relative Roles

This leads to an important refinement.

Something is not permanently either:

```text
agent
```

or:

```text
plant
```

A subsystem may be an agent from one perspective and part of the plant from another.

For example:

```text
PID controller
```

is the agent when controlling a motor.

But:

```text
PID controller + motor
```

becomes part of the plant seen by a trajectory controller.

Likewise:

```text
Dominatus behavior controller
```

may be an agent relative to navigation.

But it may be part of the plant observed by an LLM mission planner.

Therefore:

> **Agent and plant are roles defined by a control boundary.**

This is a deeper idea than it first appears.

---

## 5.29 Nested Agency

Once controllers can become plants to other controllers, agency becomes nested.

For example:

```text
Human Operator
    ↓
LLM Mission Agent
    ↓
Behavior Controller
    ↓
Navigation Controller
    ↓
Velocity Controller
    ↓
Motor Controller
    ↓
Motor
```

Each layer possesses some agency.

Each layer constrains or directs the layer below.

Each layer sees a different abstraction of the same overall system.

No one layer needs to contain the entire intelligence of the system.

---

## 5.30 Slow Intelligence, Fast Reflexes

Biological systems offer a useful analogy.

A human does not consciously compute every muscle activation required to remain upright.

High-level thought may decide:

```text
walk across the room
```

while lower-level neural systems handle:

```text
balance
posture
coordination
reflexes
```

The hierarchy separates:

```text
slow, expressive reasoning
```

from:

```text
fast, reliable stabilization
```

Modern autonomous systems benefit from the same structure.

An LLM can reason:

```text
The main entrance is blocked.
Use the loading entrance instead.
```

while fast controllers maintain:

```text
balance
velocity
collision avoidance
motor stability
```

The LLM does not need to know how to generate PWM duty cycles.

It should not.

---

## 5.31 LLMs as High-Level Controllers

Large language models are unusually expressive policies.

They can:

* interpret natural language,
* reason across semantic context,
* generate plans,
* use tools,
* adapt to unfamiliar tasks.

But they also have properties that make them poor low-level controllers:

* high latency,
* variable latency,
* nondeterminism,
* high computational cost,
* weaker hard real-time guarantees.

Agentic Control Theory therefore naturally places LLMs where their strengths matter:

```text
high-level deliberation
planning
interpretation
diagnosis
tool selection
```

while lower-level controllers handle:

```text
fast stabilization
precise regulation
hard timing
safety-critical reflex
```

This is not a limitation of LLMs.

It is good control architecture.

---

## 5.32 LLMs as Effects

An LLM does not always need to be the main controller.

A controller may invoke an LLM as an **effect**.

For example:

```text
Behavior Controller
    ↓
"Need interpretation of ambiguous failure"
    ↓
Invoke LLM
    ↓
Receive proposed diagnosis
    ↓
Continue control
```

In this structure, the LLM is a cognitive resource used by another controller.

This gives us two useful patterns:

```text
LLM as controller
```

and:

```text
LLM as cognitive effect
```

The distinction depends on where control authority resides.

---

## 5.33 A Worked Example: Autonomous Robot

Consider an autonomous inspection robot.

Its control hierarchy might be:

```text
Mission Controller
    ↓
Behavior Controller
    ↓
Navigation Controller
    ↓
Trajectory Controller
    ↓
Velocity Controller
    ↓
Motor Controller
    ↓
Robot
```

### Mission Controller

Objective:

```text
Inspect machines 1 through 8.
```

Timescale:

```text
seconds to minutes
```

Policy may include an LLM.

### Behavior Controller

Discrete states:

```text
Navigate
Inspect
Recharge
Recover
ReturnHome
```

Timescale:

```text
hundreds of milliseconds
```

Policy may use stack-based state and utility.

### Navigation Controller

Objective:

```text
reach target location
```

Produces:

```text
path / trajectory objective
```

### Trajectory Controller

Produces:

```text
desired velocity
```

### Velocity Controller

Produces:

```text
desired wheel torque
```

### Motor Controller

Produces:

```text
motor current / voltage
```

Timescale:

```text
milliseconds or faster
```

Every layer is part of the same system.

---

## 5.34 Robot Failure Example

Suppose the mission controller requests:

```text
Inspect machine 4.
```

The behavior controller enters:

```text
Navigate
```

Navigation discovers:

```text
PathBlocked
```

The behavior stack becomes:

```text
[Mission, Navigate, RecoverPath]
```

Recovery attempts an alternate route.

Meanwhile the velocity and motor controllers continue maintaining stable motion.

If recovery fails:

```text
RecoverPath → Failed
```

The behavior layer reports failure upward.

The mission controller may then reason:

```text
Skip machine 4 and continue to machine 5.
```

Notice the separation.

The LLM-level controller handles semantic adaptation.

The lower layers never stop performing fast control.

---

## 5.35 A Worked Example: Software Service

The same architecture exists without any physical machinery.

Consider a web service.

Plant:

```text
running service
request queue
network dependencies
resource usage
```

A low-level rate controller may continuously regulate:

```text
request admission rate
```

A retry controller may decide:

```text
retry after delay
```

A recovery state machine may choose:

```text
Healthy
Degraded
Restarting
Failover
```

A high-level incident agent may reason:

```text
The database dependency is unhealthy.
Fail over traffic to the secondary region.
```

The hierarchy might be:

```text
Incident Agent
    ↓
Recovery Controller
    ↓
Retry / Backoff Controller
    ↓
Rate Controller
    ↓
Service
```

No motors.

Still control.

---

## 5.36 Backoff as Damping

Consider retry behavior.

Without stabilization:

```text
request fails
→ retry immediately
→ fails
→ retry immediately
→ fails
→ ...
```

The controller amplifies load precisely when the plant is already unhealthy.

Add exponential backoff:

```text
retry after 1 s
retry after 2 s
retry after 4 s
retry after 8 s
```

The system becomes less aggressive.

Conceptually, backoff plays a role analogous to damping.

It suppresses runaway feedback.

Again, the representation differs.

The control principle survives.

---

## 5.37 Autoscaling as Hybrid Control

Consider a cloud autoscaler.

Continuous observation:

```text
CPU = 82%
```

Discrete decision:

```text
ScaleUp
```

The action changes the plant:

```text
add instance
```

Then CPU changes continuously afterward.

If thresholds are poorly designed:

```text
scale up
→ CPU drops
→ scale down
→ CPU rises
→ scale up
```

The controller chatters.

Solutions include:

* hysteresis,
* cooldown,
* dwell time,
* predictive control.

This is an excellent example of hybrid agentic control in ordinary software infrastructure.

---

## 5.38 Composition Creates New Failure Modes

A hierarchy introduces problems beyond those of individual controllers.

Examples include:

### Conflicting objectives

One controller wants speed.

Another wants efficiency.

Another wants safety.

### Timescale mismatch

A slow controller changes objectives faster than a lower layer can complete them.

### Authority conflict

Two controllers believe they own the same actuator or decision.

### Feedback delay

Higher layers receive stale information.

### Oscillation between layers

One layer corrects behavior that another layer immediately reverses.

### Hidden saturation

A higher layer requests behavior impossible for lower layers to achieve.

These are composition failures.

They require explicit architecture.

---

## 5.39 Do Not Micromanage Lower Layers

One useful design principle is:

> **Higher-level controllers should specify intent at the highest useful abstraction, not micromanage lower-level action.**

Bad hierarchy:

```text
Mission controller:
    left motor = 47%
    right motor = 52%
```

Better:

```text
Mission controller:
    inspect machine 4
```

Then:

```text
Behavior:
    navigate to machine 4

Navigation:
    follow path

Velocity:
    maintain speed

Motor:
    regulate current
```

Each layer solves the problem it is best suited to solve.

---

## 5.40 Do Not Hide Important Constraints

Abstraction should not mean ignorance.

Suppose the mission planner requests:

```text
Reach target in 1 second.
```

But the lower-level system cannot physically accelerate quickly enough.

The contract must allow that limitation to propagate upward.

Possible response:

```text
Objective infeasible.
Estimated minimum time: 4.2 seconds.
```

A good hierarchy hides implementation detail without hiding capability limits.

---

## 5.41 Control Boundaries

Every composition introduces a **control boundary**.

Across that boundary flow:

```text
objectives downward
observations upward
status upward
constraints downward or upward
```

We can visualize:

```text
Higher Controller
    │
    │ objective
    ▼
Lower Controller
    │
    │ effects
    ▼
Plant

Higher Controller
    ▲
    │ status / observation
    │
Lower Controller
```

The boundary defines what each layer knows and controls.

---

## 5.42 Control Interfaces

A useful control interface may therefore contain:

```text
Objective input
Observation output
Status output
Failure output
Constraint description
Timing expectations
```

For example:

```text
NavigateTo(Target)

Status:
    Running
    Completed
    Failed

Telemetry:
    Position
    DistanceRemaining

Constraints:
    MaxSpeed
    AllowedArea
```

The control interface becomes the contract by which agents compose.

---

## 5.43 Composition Across Machines

Nothing requires all control layers to exist on one processor.

A hierarchy might span:

```text
motor microcontroller
robot computer
edge server
cloud service
human operator
```

For example:

```text
Human
    ↓
Cloud mission planner
    ↓
Robot behavior controller
    ↓
Embedded motion controller
    ↓
Motor controller
```

Network latency now becomes part of the control hierarchy.

The farther a controller is from the fast physical plant, the less appropriate it becomes for tight feedback loops.

This again reinforces timescale separation.

---

## 5.44 Composition Across Agents

Controllers may also coordinate laterally rather than purely hierarchically.

For example:

```text
Robot A
Robot B
Robot C
```

Each has its own local controller.

A coordination agent may assign:

```text
Robot A → inspect north wing
Robot B → inspect south wing
Robot C → recharge
```

The coordination controller treats each robot as an agent and simultaneously as a controllable subsystem.

This gives us multi-agent control.

That subject deserves deeper treatment later.

---

## 5.45 The Unified Agentic Hierarchy

We can now write the general pattern:

```text
Agent
    ↓
controls
    ↓
Agent + Plant subsystem
    ↓
controls
    ↓
Agent + Plant subsystem
    ↓
controls
    ↓
Plant
```

At each level:

* an agent observes,
* maintains state,
* follows an objective,
* applies a policy,
* produces actions,
* receives feedback.

The details change.

The structure repeats.

---

## 5.46 Recursive Control

This repetition means control can be **recursive**.

A controller controls a controller that controls a controller.

For example:

```text
Mission Agent
    controls
Behavior Agent

Behavior Agent
    controls
Navigation Agent

Navigation Agent
    controls
Velocity Controller

Velocity Controller
    controls
Motor Controller

Motor Controller
    controls
Motor
```

The same abstraction appears at every level.

This is one of the strongest reasons to use the word **agent** broadly.

Each layer has agency appropriate to its role.

---

## 5.47 Continuous and Discrete Are Not Opposites

At this point, the distinction between continuous and discrete control should look less absolute.

A real controller may simultaneously contain:

```text
continuous plant state
discrete controller state
continuous utility values
discrete transitions
continuous timers
discrete interrupts
continuous optimization
discrete planning
```

The system is hybrid.

The useful question is no longer:

> Is this a continuous controller or a discrete controller?

It is:

> Which parts of the control problem are continuous, which are discrete, and how do they compose?

---

## 5.48 The Core Synthesis

The synthesis developed in this chapter can be summarized in several claims.

### Continuous and discrete controllers solve different layers of the same problem

Continuous control is particularly good at regulating physical quantities.

Discrete control is particularly good at selecting modes, behaviors, and tasks.

### Controllers can control other controllers

A higher-level controller may change the objective, mode, or authority of a lower-level controller.

### A controlled subsystem can become a plant

A controller plus its plant can be abstracted as a new plant at the next level.

### Timescale is part of architecture

Fast dynamics belong to fast controllers.

Slow semantic decisions belong to slower controllers.

### Authority must be explicit

Controllers need clearly defined domains of control and override behavior.

### Stability belongs to the composition

Individually stable or sensible controllers can interact badly.

### Agent and plant are relative roles

A subsystem may be an agent at one boundary and part of the plant at another.

Together these ideas form the basic architecture of hybrid agentic control.

---

## 5.49 What Agentic Control Theory Adds

Hybrid systems and hierarchical control are not new.

Neither are state machines, PID controllers, supervisors, or multi-rate control.

Agentic Control Theory does not claim otherwise.

The broader contribution is the abstraction under which these concepts can be extended naturally to:

* software controllers,
* utility agents,
* pushdown control,
* planners,
* tool-using agents,
* LLM-based cognition,
* human supervision.

The goal is not to rename existing control theory.

It is to provide a shared framework that allows existing control theory and modern computational agency to compose without treating one side as an awkward exception.

---

## 5.50 Where We Go Next

We now have a control hierarchy.

But composition raises new questions.

Who has authority?

What happens when controllers disagree?

How should objectives flow downward?

How should failure propagate upward?

How should controllers operating at different timescales interact?

When should one controller preempt another?

How can a high-level controller trust a subsystem without knowing its implementation?

These questions lead naturally to the next topic:

**hierarchical agentic control, controller contracts, authority, arbitration, and timescale.**

That is where a collection of controllers becomes a coherent control architecture rather than a pile of clever parts.
