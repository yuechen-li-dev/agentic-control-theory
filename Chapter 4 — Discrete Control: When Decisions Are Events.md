# Chapter 4 — Discrete Control: When Decisions Are Events

Chapter 3 dealt with systems whose state evolves continuously.

A motor does not wait for permission to keep spinning.

A falling object does not wait for the next control tick.

A thermal system does not freeze while the processor is busy.

Continuous control therefore focuses heavily on rates of change, feedback gain, damping, stability, sampling, and the dynamics of plants that are always moving.

But many control problems are different.

Sometimes the important question is not:

> How much force should we apply?

It is:

> What should we do now?

Should the robot:

* keep moving,
* stop,
* recharge,
* avoid an obstacle,
* investigate a fault,
* return home?

Should a game character:

* patrol,
* chase,
* attack,
* flee?

Should a software service:

* retry,
* fail,
* reconnect,
* restart,
* wait?

These are **discrete control decisions**.

Instead of continuously varying one control value, the controller chooses among distinct modes, behaviors, or transitions.

For a mechanical engineer, the conceptual shift is:

> **Continuous control asks how a state evolves. Discrete control often asks which state or mode should be active at all.**

The two are not competitors.

Real systems frequently need both.

---

## 4.1 A Different Kind of State

In Chapter 3, state meant quantities such as:

```text id="y4zbu0"
position
velocity
temperature
pressure
```

These variables may take continuously varying values.

In discrete control, state can instead represent a **mode of behavior**.

For example:

```text id="3kfmp2"
Stopped
Starting
Running
Faulted
```

or:

```text id="1yg8rj"
Idle
Patrol
Chase
Attack
Flee
```

These states are not positions along a continuous scale.

`Patrol` is not halfway between `Idle` and `Attack`.

They represent qualitatively different modes.

A discrete controller asks:

> Which mode am I in?

and:

> Under what conditions should I change modes?

---

## 4.2 The Finite-State Machine

The simplest important model of discrete control is the **finite-state machine**, or FSM.

An FSM consists of:

* a finite set of states,
* rules for transitioning between them,
* conditions determining when transitions occur,
* and optionally actions associated with states or transitions.

A simple motor supervisory controller might be:

```text id="5bzfkn"
Stopped
   │ StartRequested
   ▼
Starting
   │ SpeedReached
   ▼
Running
   │ StopRequested
   ▼
Stopping
   │ SpeedZero
   ▼
Stopped
```

A fault may interrupt several modes:

```text id="969mnb"
Starting ─┐
Running  ─┼─ FaultDetected → Faulted
Stopping ─┘
```

This is already control.

The controller observes the plant, retains its current discrete state, evaluates transition conditions, and selects what should happen next.

---

## 4.3 For Mechanical Engineers: Think Operating Modes

Mechanical systems already contain discrete modes everywhere.

A transmission may be:

```text id="vvdyvv"
Park
Reverse
Neutral
Drive
```

A machine tool may be:

```text id="n6phgy"
Idle
Homing
Ready
Running
Paused
Faulted
EmergencyStop
```

An aircraft controller may have modes such as:

```text id="00nd7p"
AltitudeHold
HeadingHold
Approach
Flare
GoAround
```

So finite-state control is not foreign to mechanical engineering.

The difference is that computer science developed explicit formal and programming models for representing these modes and transitions.

An FSM simply makes operating-mode logic first-class.

---

## 4.4 The Transition

A **transition** moves the controller from one discrete state to another.

Suppose:

```text id="ehfn7o"
State = Patrol
```

and the controller observes:

```text id="7zz6oz"
EnemyVisible = true
```

A transition rule may say:

```text id="4ubqiu"
Patrol --EnemyVisible--> Chase
```

In code-like form:

```text id="6a2tk6"
if State == Patrol
and EnemyVisible:
    State = Chase
```

The controller has changed its own internal state:

$$
m_t \rightarrow m_{t+1}
$$

This is a **controller transition**.

It may then produce effects that influence the plant.

For example:

```text id="407hr2"
State = Chase
→ command movement toward target
```

This distinction matters.

The controller state may change instantaneously in software, while the physical plant still evolves continuously afterward.

---

## 4.5 Guards

The condition permitting a transition is often called a **guard**.

For example:

```text id="i7ee4y"
Patrol
    when EnemyVisible
→ Chase
```

`EnemyVisible` is the guard.

A more complicated guard might be:

```text id="t49s7o"
EnemyVisible
&& Distance < 20
&& Health > 30%
```

Guards answer:

> Is this transition currently allowed?

They do not necessarily answer:

> Is this the best transition?

That distinction becomes important when we introduce utility-based control later.

---

## 4.6 State Entry, State Update, and State Exit

Discrete states often have behavior associated with their lifecycle.

Conceptually:

```text id="zrdpqq"
Enter state
    ↓
Run state behavior
    ↓
Transition condition occurs
    ↓
Exit state
    ↓
Enter next state
```

For example:

```text id="tc376y"
Enter Chase:
    acquire target

While Chase:
    move toward target

Exit Chase:
    clear chase-specific state
```

This is very similar to mechanical operating modes.

Different modes may activate different controllers.

For example:

```text id="pa2jpp"
Idle:
    motor disabled

VelocityControl:
    PID velocity loop active

PositionControl:
    trajectory + position controller active

Fault:
    outputs forced safe
```

A discrete controller may therefore determine **which continuous controller is currently in authority**.

This is one of the first places continuous and discrete control naturally meet.

---

## 4.7 Events

A transition may be triggered by an **event**.

Examples:

```text id="jpofaz"
button pressed
message received
timer expired
sensor threshold crossed
task completed
fault detected
```

An event represents something significant enough that the controller should reconsider its discrete state.

Computer systems often organize control around event loops:

```text id="n2u7bw"
wait for event
→ process event
→ update state
→ produce effects
→ wait again
```

For a mechanical engineer, an event can be understood as:

> a discrete observation that indicates the control regime may need to change.

---

## 4.8 Tick-Based Control

Not all discrete controllers wait for explicit events.

Many execute periodically:

```text id="oy0rqy"
every control tick:
    observe
    evaluate transitions
    update state
    produce effects
```

This resembles a sampled continuous controller, but the decision space differs.

A continuous controller might compute:

```text id="bgfyb7"
Torque = 12.7 N·m
```

A discrete controller may compute:

```text id="vfnexu"
Behavior = AvoidObstacle
```

The tick still provides a timescale.

The output is simply more categorical.

---

## 4.9 Deterministic State Machines

A simple FSM often has deterministic transition logic.

Given:

```text id="3zk3w1"
current state
+
current observation
```

there is exactly one next state.

For example:

```text id="kgcq1t"
if LowBattery:
    Recharge
else if EnemyVisible:
    Chase
else:
    Patrol
```

The ordering determines the result.

This is easy to reason about.

It can also become awkward as behavior becomes more complicated.

---

## 4.10 The State Explosion Problem

Suppose a robot can independently be:

```text id="gb7j8o"
Moving / Stationary
Armed / Unarmed
Healthy / Damaged
Connected / Disconnected
Loaded / Empty
```

If every combination becomes its own FSM state, the state count grows rapidly.

For example:

```text id="rc66el"
MovingArmedHealthyConnectedLoaded
MovingArmedHealthyConnectedEmpty
MovingArmedHealthyDisconnectedLoaded
...
```

This is **state explosion**.

Computer science deals with this through techniques such as:

* hierarchical state machines,
* orthogonal state regions,
* composition,
* stacks,
* separate controllers,
* structured state data.

The lesson is:

> A state machine should represent meaningful control modes, not every possible combination of every variable in the system.

---

## 4.11 Hierarchical State Machines

States can themselves contain substates.

For example:

```text id="5igbe0"
Operational
├── Idle
├── Patrol
└── Combat
    ├── Approach
    ├── Attack
    └── Retreat
```

Now common transitions can exist at the higher level.

For example:

```text id="tiqs1k"
Operational
    when CriticalFault
→ Faulted
```

without duplicating that transition for every substate.

Hierarchical states reduce duplication and make the control structure reflect conceptual organization.

---

## 4.12 Why Flat State Machines Become Awkward

Consider a robot performing:

```text id="68ewb7"
InspectMachine
```

During inspection, it encounters an obstacle.

It temporarily needs to perform:

```text id="bi0rai"
AvoidObstacle
```

After avoiding the obstacle, it should resume the inspection exactly where it left off.

A flat FSM must somehow remember:

```text id="jksipn"
What was I doing before AvoidObstacle?
```

One option is to encode transitions explicitly:

```text id="gmpshl"
InspectA → AvoidObstacleFromInspectA
InspectB → AvoidObstacleFromInspectB
InspectC → AvoidObstacleFromInspectC
```

This becomes unpleasant very quickly.

What we really want is:

```text id="h775pp"
Suspend current behavior
→ temporarily perform another behavior
→ return to suspended behavior
```

That requires a richer form of controller memory.

---

## 4.13 Pushdown Automata

A **pushdown automaton** extends a finite-state machine with a stack.

Instead of remembering only:

```text id="knicrz"
CurrentState
```

the controller may remember:

```text id="cnphtr"
ControlStack
```

For example:

```text id="d0jhgd"
[Patrol]
```

The controller encounters something interesting:

```text id="5yxwh5"
[Patrol, Investigate]
```

Then an obstacle interrupts:

```text id="x9r21r"
[Patrol, Investigate, AvoidObstacle]
```

Obstacle avoidance completes:

```text id="dng8pc"
[Patrol, Investigate]
```

Investigation completes:

```text id="jus3qz"
[Patrol]
```

The stack naturally represents nested control contexts.

This is conceptually similar to a function call stack.

A function calls another function.

The caller is suspended.

The callee runs.

When the callee returns, execution resumes where the caller left off.

A pushdown controller applies the same idea to behavior.

---

## 4.14 Why a Stack Is More Than an Implementation Detail

The stack increases the expressive power of the controller.

A finite-state machine has finite memory encoded directly in its state.

A pushdown automaton has an unbounded stack in the theoretical model.

That makes nested behavior naturally representable.

For practical control systems, the important distinction is simpler:

> **A stack gives the controller explicit memory of suspended control contexts.**

This makes behaviors compositional.

One behavior can temporarily invoke another without knowing every detail of how it will execute.

---

## 4.15 State as Control Context

In a pushdown controller, a state is often better understood as a **control context**.

A control context may contain:

* current behavior,
* local memory,
* transition rules,
* timers,
* utility information,
* a resume location.

So instead of thinking only:

```text id="d15o27"
State = Attack
```

we may think:

```text id="uovlmy"
Current control context:
    Attack
    target = Enemy7
    phase = Approach
    elapsed = 1.2 s
```

This makes discrete controllers resemble ordinary structured programs.

And that is not accidental.

A program itself is a control system over computational state.

---

## 4.16 Effects in Discrete Control

Discrete controller states often produce effects.

For example:

```text id="gcw0g6"
Patrol
→ follow patrol path

Chase
→ move toward target

Attack
→ fire weapon

Recharge
→ navigate to charging station
```

The state does not have to perform the low-level physical control directly.

Instead, it may command another controller.

For example:

```text id="caf7rl"
Chase
    ↓
DesiredVelocity = direction_to_target * speed
    ↓
Velocity Controller
    ↓
Motor Controller
```

This gives us hierarchical composition across discrete and continuous control.

The discrete controller chooses **what mode of behavior** should occur.

The continuous controller determines **how to physically realize it**.

---

## 4.17 Transitions Are Also Control Actions

Changing controller state can itself be considered an action.

Suppose:

```text id="c1vd0y"
Patrol → Chase
```

The transition changes future policy.

Before the transition, observations are interpreted according to patrol behavior.

After the transition, they are interpreted according to chase behavior.

A transition therefore alters the controller's future decision-making structure.

This makes state transitions a particularly important class of internal effect.

---

## 4.18 Priority-Based Arbitration

What happens when several transitions are valid at once?

Suppose:

```text id="xbwukg"
EnemyVisible = true
LowHealth    = true
LowAmmo      = true
```

Possible transitions include:

```text id="6my19d"
Attack
Flee
Reload
```

One solution is fixed priority:

```text id="s7677n"
if LowHealth:
    Flee
else if LowAmmo:
    Reload
else if EnemyVisible:
    Attack
```

This is straightforward.

But priority systems can become brittle.

The first valid condition always wins regardless of degree.

For example:

```text id="dtla8h"
Health = 29%
```

may cause `Flee`, while:

```text id="2dnp7g"
Health = 31%
```

causes `Attack`.

A tiny input difference can create a completely different behavior.

This problem should look familiar to mechanical engineers.

It is a switching-control problem.

---

## 4.19 Threshold Chatter

Suppose the rule is:

```text id="8a1tlx"
if Temperature > 100:
    Cooling = On
else:
    Cooling = Off
```

Now suppose the measured temperature fluctuates:

```text id="bcv9ld"
99.9
100.1
99.8
100.2
99.9
100.1
```

The controller responds:

```text id="pqk2y9"
Off
On
Off
On
Off
On
```

This is **chatter**.

Software systems suffer the same problem.

Suppose:

```text id="jvl7oy"
if EnemyDistance < 10:
    Attack
else:
    Chase
```

and distance fluctuates around 10:

```text id="dd6je9"
9.9
10.1
9.8
10.2
```

The agent becomes:

```text id="iprrbz"
Attack
Chase
Attack
Chase
```

Mechanical engineering has dealt with this family of problems for a very long time.

One standard solution is hysteresis.

---

## 4.20 Hysteresis

**Hysteresis** means that the condition for entering a state differs from the condition for leaving it.

Instead of:

```text id="i3b6tu"
Cooling On  if Temperature > 100
Cooling Off if Temperature < 100
```

we use:

```text id="lpznwv"
Cooling On  if Temperature > 102
Cooling Off if Temperature < 98
```

Between 98 and 102, the controller keeps its current state.

The decision therefore depends not only on the current observation but also on controller history.

Graphically:

```text id="mg7rf6"
            turn ON
              →
---------98---------102---------
         ←
       turn OFF
```

The same principle applies beautifully to discrete software behavior.

Instead of:

```text id="xl8ql4"
Attack if AttackUtility > ChaseUtility
```

we may require:

```text id="z5iqeh"
Switch to Attack only if
AttackUtility > ChaseUtility + HysteresisMargin
```

The current behavior receives a small advantage simply because it is already active.

That prevents tiny utility fluctuations from causing constant switching.

In Agentic Control Theory terms:

> **Hysteresis deliberately introduces history dependence into transition policy to suppress unstable switching.**

Mechanical engineers already know the phenomenon.

Software engineers frequently rediscover it after wondering why their AI cannot make up its mind.

---

## 4.21 Minimum Commitment

Hysteresis prevents switching caused by small differences.

But another problem remains.

Suppose a controller changes to a new behavior:

```text id="4cb10u"
Patrol → Investigate
```

One tick later, another behavior becomes slightly more desirable:

```text id="j8g4iz"
Investigate → Patrol
```

Then:

```text id="rlw66f"
Patrol → Investigate
```

Even with hysteresis, large enough noisy changes can still produce thrashing.

A second strategy is **minimum commitment**.

Once a behavior is selected, the controller commits to it for some minimum duration unless an exceptional condition occurs.

For example:

```text id="271qnl"
Enter Investigate

Do not reconsider ordinary transitions
for at least 500 ms.
```

After the commitment period expires, normal arbitration resumes.

This has close analogues in mechanical control:

* minimum actuator dwell times,
* compressor anti-short-cycle delays,
* minimum relay on/off periods,
* switched-system dwell-time constraints.

So although the implementation may look computational:

```text id="f9mlam"
if TimeInState < MinimumCommit:
    Stay
```

the control principle is not uniquely computational at all.

It is another form of stabilization.

> **Minimum commitment prevents a controller from changing modes faster than the controlled process can meaningfully respond.**

That last point is especially important.

If a behavior requires time to produce an effect, switching away before that effect can occur guarantees poor control.

---

## 4.22 Dwell Time

The more general control concept is **dwell time**.

A switched system may require that each mode remain active for some minimum period before another switch occurs.

Why?

Because even if every individual controller is well behaved, switching too rapidly between them can produce undesirable or unstable overall behavior.

This is a major conceptual bridge between classical and discrete control.

The same pattern appears in:

```text id="bf9t8t"
relay controllers
hybrid systems
game AI
distributed systems
autoscaling
task scheduling
agent behavior
```

Different implementation.

Same control problem.

---

## 4.23 Utility-Based Control

Fixed transition priorities are often too rigid.

Instead, we can assign each candidate action or state a **utility**.

For example:

```text id="t25i6l"
Attack     = 0.65
TakeCover  = 0.80
Heal       = 0.90
Flee       = 0.55
```

The controller can choose the highest-scoring valid option.

Formally:

$$
a^\* = \arg\max_{a \in A} U(a,o,m)
$$

Do not let the notation make this look more exotic than it is.

It means:

> Evaluate the desirability of each available action and choose the best one.

In pseudocode:

```text id="nhciis"
BestAction = None;
BestScore = -Infinity;

for action in AvailableActions:
    score = Utility(action);

    if score > BestScore:
        BestAction = action;
        BestScore = score;
```

Utility turns:

```text id="kip0z1"
Can I do this?
```

into:

```text id="0w9jg7"
How much do I want to do this right now?
```

---

## 4.24 Guards and Utility Solve Different Problems

A useful architecture separates **eligibility** from **desirability**.

A guard asks:

> Is this transition allowed?

Utility asks:

> If it is allowed, how desirable is it?

For example:

```text id="7csban"
Attack:
    Guard:
        EnemyVisible
        && Ammo > 0

    Utility:
        ThreatLevel
        * HitProbability
        * Aggression
```

The guard provides a hard constraint.

The utility provides a preference.

This distinction avoids trying to encode everything into one giant score.

---

## 4.25 Utility Without Stability Is a Gremlin

Suppose:

```text id="sxjvg2"
AttackUtility = 0.71
CoverUtility  = 0.70
```

The controller chooses `Attack`.

Next tick:

```text id="yd1gsj"
AttackUtility = 0.70
CoverUtility  = 0.71
```

Now `Cover`.

Next tick they reverse again.

The controller oscillates.

This is the discrete-control equivalent of an underdamped or poorly stabilized switching system.

Utility alone does not solve control.

It merely supplies a richer policy.

We still need concepts such as:

* hysteresis,
* minimum commitment,
* dwell time,
* transition cost,
* priority,
* interruptibility.

This is an important theme in Agentic Control Theory:

> **A more expressive decision function does not eliminate classical control problems. It often recreates them in a new representation.**

---

## 4.26 Transition Cost

Switching modes may itself have a cost.

Suppose two options have utility:

```text id="hfdnwl"
CurrentBehavior = 0.70
Candidate       = 0.74
```

Switching may involve:

* stopping current work,
* discarding progress,
* reconfiguring hardware,
* moving resources,
* reacquiring context,
* mechanical wear,
* latency.

If switching costs:

```text id="p74xxn"
0.10 utility equivalent
```

then changing behavior is not worthwhile.

One can model this as:

$$
U_{\text{effective}}=U_{\text{candidate}}-C_{\text{transition}}
$$

This is another way of introducing resistance to unnecessary mode changes.

Mechanical systems have switching costs.

Software systems do too.

They simply tend to hide them behind abstractions.

---

## 4.27 Interrupts

Minimum commitment should not mean:

> Never react until the timer expires.

Some conditions must override ordinary commitment.

Examples:

```text id="78oo9f"
EmergencyStop
CriticalFault
CollisionImminent
ProcessTerminated
LossOfControl
```

So controllers often distinguish between:

* normal transitions,
* high-priority interrupts.

For example:

```text id="rp9j5t"
if Emergency:
    transition immediately
else if MinimumCommitNotSatisfied:
    stay
else:
    perform normal arbitration
```

This is analogous to safety interlocks in physical systems.

Commitment stabilizes ordinary behavior.

Safety conditions retain authority to interrupt it.

---

## 4.28 Completion

Some states represent activities that naturally finish.

For example:

```text id="fg1of5"
Reload
OpenDoor
NavigateToTarget
PerformInspection
```

These can produce a **completion event**.

A parent controller may then decide what happens next.

This becomes especially useful with pushdown control.

For example:

```text id="8kmfe1"
[Patrol, Investigate]
```

`Investigate` completes.

The stack pops:

```text id="6gxpo4"
[Patrol]
```

Control automatically returns to the suspended context.

This provides a structured analogue to function return.

---

## 4.29 Failure

Activities may also fail.

For example:

```text id="ftgp43"
NavigateToTarget
→ path unavailable
→ failure
```

A structured controller should represent failure explicitly.

A parent might respond:

```text id="hfhafn"
TryAlternativeRoute
```

or:

```text id="17cpfh"
AbortMission
```

or:

```text id="3bh9h4"
AskHigherLevelController
```

This means nested controllers can produce outcomes such as:

```text id="u15v2i"
Running
Completed
Failed
Interrupted
```

The exact model varies.

The broader idea is important:

> **Control contexts can themselves have lifecycle and outcome semantics.**

---

## 4.30 Behavior Trees

Another popular model of discrete control is the **behavior tree**.

A behavior tree organizes tasks using nodes such as:

* sequences,
* selectors,
* conditions,
* actions.

A selector might mean:

```text id="0pc2ug"
Try:
    Flee
    Heal
    Attack
    Patrol

Use the first option that can run successfully.
```

A sequence might mean:

```text id="my4fr0"
Acquire target
→ Move to target
→ Perform action
```

Behavior trees provide a structured alternative to giant state machines.

They are especially common in game AI and robotics.

From the Agentic Control Theory perspective, they are another policy representation.

The abstract loop does not change.

---

## 4.31 FSM, Behavior Tree, and Utility Are Policy Representations

It is easy to treat:

```text id="o5fo7z"
FSM
behavior tree
utility AI
planner
```

as competing philosophical camps.

For our purposes, that is not useful.

They are different ways of representing policy.

An FSM emphasizes:

```text id="kndakk"
explicit modes and transitions
```

A behavior tree emphasizes:

```text id="4vsayh"
hierarchical task evaluation
```

Utility control emphasizes:

```text id="1xqxox"
comparative desirability
```

A planner emphasizes:

```text id="mwe0mg"
future action sequences
```

A sophisticated controller may combine all of them.

---

## 4.32 Hybrid Policy

A practical discrete controller might say:

```text id="pp4qp3"
Current stack:
    Mission
    Combat

Within Combat:
    utility selects:
        Attack
        Cover
        Heal
        Retreat

Within Attack:
    state machine controls:
        Acquire
        Aim
        Fire
        Recover
```

There is no requirement that one policy representation govern the entire system.

Controllers compose.

This is exactly analogous to using different continuous controllers for different physical subsystems.

---

## 4.33 Discrete Controllers Can Command Continuous Controllers

Suppose a robot has states:

```text id="ft94pe"
Idle
Navigate
Dock
EmergencyStop
```

In `Navigate`:

```text id="xza17f"
Discrete Controller
    ↓
Desired trajectory
    ↓
Trajectory Controller
    ↓
Desired velocity
    ↓
Velocity PID
    ↓
Motor command
```

In `Dock`:

```text id="vimw04"
Discrete Controller
    ↓
Docking controller
    ↓
Low-speed position control
```

The discrete controller changes which continuous control regime is active.

This is a **hybrid control system**.

It combines continuous dynamics and discrete mode transitions.

Such systems already exist throughout mechanical engineering.

Agentic Control Theory generalizes the same compositional idea to richer computational agents.

---

## 4.34 Continuous State Can Drive Discrete Transitions

The flow also goes the other direction.

A continuous variable may trigger a discrete transition.

For example:

$$
T > 100^\circ C
$$

causes:

```text id="ofhujc"
Normal → OverheatProtection
```

or:

$$
Battery < 10\%
$$

causes:

```text id="595tix"
Mission → ReturnToCharge
```

Continuous plant state therefore feeds discrete policy.

This creates a system with:

* continuous dynamics inside modes,
* discrete transitions between modes.

Again, this is hybrid control.

---

## 4.35 Discrete Oscillation Is Still Oscillation

Suppose:

```text id="3k8fnn"
State A → State B → State A → State B → ...
```

That is not continuous oscillation in the classical sense.

But structurally, the controller is still exhibiting undesirable repeated switching.

This behavior may be called:

* chatter,
* thrashing,
* mode oscillation,
* transition oscillation.

Agentic Control Theory treats these as related control pathologies.

The representation differs.

The practical question is still:

> Is feedback causing the controller to repeatedly overreact instead of converging to useful behavior?

---

## 4.36 Deadlock

Discrete systems also introduce failure modes that are less prominent in ordinary continuous control.

One is **deadlock**.

A controller may reach a state from which no valid transition exists even though its objective has not been completed.

For example:

```text id="m5gzju"
State = WaitingForPermission
```

but the system responsible for granting permission is:

```text id="orwvq6"
WaitingForCompletion
```

Neither progresses.

Continuous systems tend to fail through divergence, instability, saturation, or equilibrium at undesirable states.

Discrete controllers can additionally fail because their transition structure provides nowhere useful to go.

---

## 4.37 Livelock

A related problem is **livelock**.

The controller keeps transitioning, but makes no meaningful progress.

For example:

```text id="2q0l2h"
A → B → C → A → B → C → ...
```

Everything is active.

Nothing is accomplished.

This is particularly relevant to utility and agent systems.

High activity does not imply effective control.

---

## 4.38 Starvation

A lower-priority behavior may remain valid forever but never receive control because higher-priority behaviors continually win arbitration.

This is **starvation**.

For example:

```text id="cjf1vn"
Maintenance task:
    always eligible

Urgent requests:
    continuously arrive
```

The maintenance task may never execute.

Control therefore involves not only deciding what is best immediately but sometimes preserving longer-term fairness or progress.

---

## 4.39 Determinism and Reproducibility

Discrete controllers can often be made exactly deterministic.

Given:

```text id="t90nuo"
same initial controller state
same observations
same timing
```

the controller produces:

```text id="x1kw02"
same transitions
same actions
```

This property is extremely valuable for:

* debugging,
* testing,
* simulation,
* replay,
* safety analysis.

A controller trace might look like:

```text id="rejdzy"
t=0   Patrol
t=12  Investigate
t=15  AvoidObstacle
t=19  Investigate
t=31  Patrol
```

Such traces expose the internal logic of the controller directly.

As controllers become more sophisticated, preserving inspectable control state becomes increasingly valuable.

---

## 4.40 The Discrete Controller as an Agent

Map a finite-state or pushdown controller into the model from Chapter 2.

### Plant

The system being controlled.

### Observation

Events, sensor values, messages, telemetry.

### Controller state

Current state, hierarchy, or control stack.

### Memory

Timers, previous choices, cooldowns, stack frames, commitment state.

### Objective

Desired behavior, goal, utility structure.

### Policy

Transition rules, guards, utility arbitration, behavior trees.

### Action

State transition or requested external operation.

### Effect

Changes to the plant or activation of subordinate controllers.

### Feedback

New observations on the next event or tick.

### Timescale

Event-triggered or periodically evaluated.

Again, the minimal model survives intact.

---

## 4.41 Continuous and Discrete Control Compared

The contrast can now be summarized.

| Continuous control                           | Discrete control                              |
| -------------------------------------------- | --------------------------------------------- |
| State often varies continuously              | State often contains categorical modes        |
| Control output often numerical               | Control output may select behavior or mode    |
| Dynamics described by differential equations | Dynamics often described by transitions       |
| Stability includes convergence/divergence    | Stability also includes chatter/thrashing     |
| Damping suppresses oscillation               | Hysteresis/commitment suppress mode switching |
| Sampling determines observation frequency    | Ticks/events determine decision opportunities |
| Saturation bounds actuator output            | Guards constrain allowable transitions        |
| Setpoint expresses desired value             | Goal/utility expresses desired behavior       |
| Controller law computes magnitude            | Policy selects mode/action                    |

The categories overlap rather than divide cleanly.

Many real systems are hybrid.

---

## 4.42 The Mechanical Engineer Already Knows More of This Than It Seems

State machines may sound like a computer-science invention.

But mechanical and control engineers already work with many equivalent concepts:

```text id="4o6tx0"
operating modes
relay logic
supervisory control
switched systems
hybrid systems
safety interlocks
hysteresis
dwell time
fault modes
startup/shutdown sequencing
```

Computer science gives these systems explicit computational representations.

Likewise, computer scientists often rediscover mechanical-control ideas when their discrete systems begin oscillating.

For example:

```text id="rucntn"
AI changes behavior every frame
→ add hysteresis
```

or:

```text id="cwsevn"
scheduler keeps switching strategy
→ add minimum dwell time
```

or:

```text id="12hbnl"
service keeps restarting
→ add restart backoff
```

These are not unrelated hacks.

They are different expressions of stabilization.

---

## 4.43 The Computer Scientist Also Already Knows More Control Theory Than It Seems

Conversely, software developers routinely construct:

* observers,
* policies,
* state estimators,
* feedback loops,
* schedulers,
* supervisory controllers,
* hierarchical control structures.

They simply use different names.

The divide between the disciplines is therefore partly linguistic.

Mechanical engineering became exceptionally good at reasoning about continuous dynamics.

Computer science became exceptionally good at representing complex discrete decision structures.

Agentic Control Theory needs both.

---

## 4.44 Dominatus as a Discrete Controller

The architecture motivating this book, Dominatus, sits primarily on the discrete side of this spectrum.

At its core, a Dominatus controller can be understood as a stateful agent with:

* explicit control state,
* stack-structured control contexts,
* guarded transitions,
* utility-based transition selection,
* explicit effects,
* hysteresis,
* minimum commitment,
* hierarchical composition.

A simplified conceptual structure is:

```text id="a8a7vd"
Observe
    ↓
Current control stack
    ↓
Find eligible transitions
    ↓
Evaluate utility
    ↓
Apply hysteresis / commitment rules
    ↓
Choose transition or remain
    ↓
Produce effects
    ↓
Repeat
```

The important point is not the implementation details yet.

Those will come later.

The important point is that Dominatus is not outside control theory.

It is a computational control architecture operating primarily through discrete state and transition selection.

---

## 4.45 From Transition Logic to Agentic Control

A simple FSM may have:

$$
m_{t+1}=T(m_t,o_t)
$$

where:

* \(m_t\) is current controller state,
* \(o_t\) is observation,
* \(T\) is the transition function.

A utility controller may instead choose:

$$
m_{t+1}=\arg\max_{m'\in E(m_t,o_t)}U(m',o_t,m_t)
$$

where \(E\) is the set of eligible transitions.

In plain language:

> Find which transitions are allowed, score them, and choose the most desirable one.

Add hysteresis:

> Require the new candidate to be sufficiently better than the current behavior.

Add minimum commitment:

> Do not reconsider ordinary transitions until the current state has had enough time to act.

Add a stack:

> Allow one control context to suspend itself while another runs and later resumes.

We now have something significantly richer than a flat state machine while still preserving explicit control semantics.

---

## 4.46 The Key Lesson for Mechanical Engineers

The most important conceptual shift is:

> **A computational controller can have dynamics of its own.**

Those dynamics may not involve momentum or stored mechanical energy.

Instead they arise from:

* internal state,
* stack structure,
* timers,
* priorities,
* utility,
* commitment,
* transition history.

A poorly designed computational controller can therefore:

* oscillate,
* chatter,
* stall,
* deadlock,
* livelock,
* starve tasks,
* repeatedly undo its own work.

These are control pathologies.

They simply inhabit a discrete state space.

---

## 4.47 The Key Lesson for Computer Scientists

The complementary lesson is:

> **Your state machine is not exempt from control theory merely because it contains `if` statements instead of differential equations.**

If feedback affects future decisions, control phenomena appear.

Thresholds can chatter.

Policies can oscillate.

Latency matters.

Switching has cost.

Commitment can stabilize behavior.

Observations may be noisy.

Actions may fail.

Plants may continue evolving between ticks.

The mathematics may differ from classical continuous control.

The systems problem is still control.

---

## 4.48 Two Traditions, One Problem

We can now see two powerful traditions.

Classical continuous control became exceptionally good at:

* stability,
* feedback,
* dynamics,
* estimation,
* robustness,
* frequency response,
* disturbance rejection.

Computational discrete control became exceptionally good at:

* explicit modes,
* hierarchy,
* composition,
* conditional transitions,
* procedural memory,
* symbolic objectives,
* structured decision logic.

Neither tradition subsumes the other cleanly.

A PID equation is an awful way to represent:

```text id="r3c2ec"
Investigate fault,
unless battery is critical,
then return home,
but resume the inspection afterward.
```

A state machine is an awful way to implement:

```text
maintain motor velocity at 3000 RPM
despite continuously varying load.
```

The useful system uses each where it belongs.

---

## 4.49 Where We Go Next

We now have both halves of the problem.

Continuous control gives us:

```text id="trf73v"
feedback
dynamics
stability
damping
estimation
bandwidth
```

Discrete control gives us:

```text
states
transitions
guards
stacks
utility
commitment
hierarchy
```

The obvious question is now:

> Why should these remain separate theories inside one system?

A modern autonomous system may need:

```text
continuous stabilization
+
discrete behavior selection
+
hierarchical objectives
+
planning
+
semantic reasoning
```

These components operate at different timescales and levels of abstraction, but they all participate in one feedback structure.

The next step is therefore the central synthesis of this book:

> **How do continuous and discrete controllers compose as agents inside one control hierarchy?**

That is where Agentic Control Theory begins to become more than a shared vocabulary.

It becomes an architecture for constructing controllers from other controllers.
