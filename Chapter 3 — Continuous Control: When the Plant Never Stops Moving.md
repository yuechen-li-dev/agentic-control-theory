# Chapter 3 — Continuous Control: When the Plant Never Stops Moving

Computer scientists are usually trained to think in discrete transitions.

A program waits.

An event arrives.

A condition becomes true.

A function is called.

A state changes.

Then the program waits again.

Physical systems are less courteous.

A falling object keeps falling.

A motor keeps spinning.

A hot object keeps cooling.

A vehicle keeps moving.

A spring keeps pulling.

A tank keeps filling.

The plant does not stop evolving just because the controller has not reached the next line of code.

This is the central idea behind continuous control:

> **The state of a physical system evolves continuously through time, whether or not the controller is currently doing anything.**

That single fact changes how control problems must be approached.

In a discrete software system, it is often reasonable to think:

```text
Current state
→ evaluate condition
→ choose next state
```

In a continuous physical system, the more useful question is:

> Given the state right now, and the input being applied right now, how quickly is the state changing?

That question leads directly to differential equations, feedback control, PID, stability, damping, and the rest of classical control theory.

This chapter introduces those ideas through the lens of Agentic Control Theory, with a particular emphasis on making them intuitive to readers coming from computer science.

---

## 3.1 The Physical World Does Not Wait for Your Tick

Consider a simple software state machine.

```text
State = Idle
```

If nothing happens, it may remain in `Idle` forever.

No event occurs.

No transition executes.

Nothing changes.

Now consider a ball falling through the air.

At some instant:

```text
position = 10 m
velocity = -5 m/s
```

If the controller does nothing for the next second, the ball does not remain frozen in place.

Gravity continues acting.

Its velocity changes.

Its position changes.

Its future state evolves continuously.

This is one of the most important differences between ordinary software control and physical control.

A physical plant usually contains its own dynamics.

It is already doing something before the controller intervenes.

The controller does not create all state transitions.

It influences a system whose state is already evolving.

For a computer scientist, one useful mental model is:

> **Reality has its own transition function, and it never stops running.**

---

## 3.2 From Transitions to Rates of Change

In a discrete system, we might write:

$$
x_{t+1}=F(x_t,u_t)
$$

This means:

> Given current state \(x_t\) and input \(u_t\), compute the next state.

For continuous systems, we usually do not jump directly from one state to the next.

Instead, we describe how quickly the state is changing:

$$
\dot{x}=f(x,u)
$$

The dot over \(x\) means:

$$
\dot{x}=\frac{dx}{dt}
$$

In plain language:

> **How fast is \(x\) changing with respect to time?**

This is the continuous equivalent of a transition rule.

If you are more comfortable with programming notation, think approximately:

```text
RateOfChange = Dynamics(CurrentState, ControlInput)
```

Then over a very small amount of time:

```text
State += RateOfChange * DeltaTime
```

That approximation is not the full mathematics of continuous dynamics, but it is a useful intuition.

The physical world is effectively integrating these changes continuously.

---

## 3.3 Derivatives Without the Calculus Ceremony

A derivative is just a rate of change.

If:

$$
x(t)
$$

is position, then:

$$
\dot{x}(t)
$$

is velocity.

If:

$$
v(t)
$$

is velocity, then:

$$
\dot{v}(t)
$$

is acceleration.

Or:

$$
\ddot{x}(t)
$$

is acceleration directly.

So:

```text
position
→ how position changes = velocity
→ how velocity changes = acceleration
```

For programmers, it is useful to think of a derivative as answering:

> “How fast is this variable changing right now?”

A numerical approximation may look familiar:

```text
Velocity ≈ (CurrentPosition - PreviousPosition) / DeltaTime
```

or:

```text
Derivative ≈ (CurrentValue - PreviousValue) / DeltaTime
```

The mathematical derivative is the limiting case as that time interval becomes arbitrarily small.

You do not need to become a calculus monk to understand the control intuition.

You only need to recognize that continuous systems care about both:

* what the state is,
* and how that state is changing.

---

## 3.4 Physical State Contains Momentum and Memory

A subtle but important idea is:

> **A continuous plant may contain memory even when the controller does not.**

Consider a moving car.

If the car is traveling at 30 m/s and the throttle suddenly becomes zero, the car does not instantly stop.

Its velocity is part of the plant state.

The system carries momentum.

Likewise:

* a capacitor stores charge,
* an inductor stores magnetic energy,
* a spring stores mechanical energy,
* a hot object stores thermal energy,
* a fluid system stores pressure,
* a rotating shaft stores angular momentum.

This means that the plant's future behavior depends not merely on what the controller commands now, but on stored physical state.

In software, programmers are accustomed to persistent variables storing history.

In physical systems, the plant itself often stores history through its dynamics.

That is why physical control cannot usually be understood as:

```text
Input → immediate output
```

Instead:

```text
Current physical state
+ current input
→ evolving future state
```

---

## 3.5 Error: How Wrong Are We?

Most feedback controllers begin with a simple question:

> How far are we from where we want to be?

Let:

$$
r(t)
$$

be the desired value, called the **reference** or **setpoint**.

Let:

$$
y(t)
$$

be the measured output.

Then the error is:

$$
e(t)=r(t)-y(t)
$$

In programmer language:

```text
Error = Desired - Actual;
```

Examples:

```text
desired speed = 1000 RPM
actual speed  = 900 RPM
error         = 100 RPM
```

or:

```text
desired temperature = 22 °C
actual temperature  = 24 °C
error               = -2 °C
```

The sign matters.

Positive and negative error usually imply correction in opposite directions.

This error becomes one of the most common observations used by a controller policy.

---

## 3.6 Proportional Control

The simplest common continuous feedback controller is proportional control.

The rule is:

$$
u(t)=K_Pe(t)
$$

where:

* \(u(t)\) is the control action,
* \(e(t)\) is the error,
* \(K_P\) is the proportional gain.

In plain language:

> **The more wrong you are, the harder you push.**

For a programmer:

```text
Command = Gain * Error;
```

Suppose:

```text
DesiredSpeed = 1000
ActualSpeed  = 900
Error        = 100
Gain         = 0.5
```

Then:

```text
Command = 50
```

If the error doubles, the command doubles.

This is intuitively attractive.

It is also incomplete.

---

## 3.7 Why More Gain Is Not Always Better

A programmer may reasonably think:

> If the system is responding too slowly, increase the gain.

Sometimes that works.

Until it does not.

Physical plants have inertia, delay, and stored energy.

Suppose a controller sees that a moving object is below its target position.

It commands a large positive force.

The object accelerates.

By the time it reaches the target position, it is already moving quickly.

The controller now commands force in the opposite direction.

But the object continues moving due to momentum.

The controller overshoots again.

The result may become:

```text
too low
→ push hard up
→ overshoot
→ push hard down
→ overshoot
→ push harder up
→ ...
```

The controller is now fighting itself.

This is one of the core lessons of continuous control:

> **Aggressive correction can create instability.**

That is very different from many ordinary software systems, where executing a correction more strongly or more quickly does not automatically produce oscillation.

---

## 3.8 Overshoot

Suppose the target is:

```text
1.0
```

The system responds:

```text
0.0
0.4
0.8
1.1
1.2
1.05
1.0
```

The output went beyond the target before returning.

That is **overshoot**.

The maximum amount beyond the target is often described as percentage overshoot.

Overshoot is not always bad.

Some systems tolerate it.

Some systems cannot.

A user-interface animation may look pleasant with slight overshoot.

A precision positioning system may not.

A chemical reactor may absolutely not.

Control quality always depends on the plant and objective.

---

## 3.9 Oscillation

A system may repeatedly cross the target:

```text
0.8
1.2
0.85
1.1
0.93
1.04
0.98
1.01
1.00
```

This is oscillation.

If the oscillations gradually shrink, the system is damped and converging.

If they stay the same size forever, the system may be marginally stable.

If they grow:

```text
0.8
1.2
0.6
1.5
0.2
2.0
-0.5
3.0
```

the system is unstable.

This is the respectable engineering translation of:

> “the line wiggles harder and harder until everything goes pwshhhh.”

---

## 3.10 Damping

**Damping** describes how strongly a system suppresses oscillation and stored motion.

Consider a spring.

Without damping, it may oscillate for a very long time:

```text
left
right
left
right
left
right
...
```

Add friction or a shock absorber, and each oscillation becomes smaller.

In control theory, a commonly used second-order model is:

$$
\ddot{x}+2\zeta\omega_n\dot{x}+\omega_n^2x=0
$$

where:

* \(\omega_n\) is the natural frequency,
* \(\zeta\) is the damping ratio.

The important cases are:

### Underdamped

$$
0<\zeta<1
$$

The system oscillates while settling.

### Critically damped

$$
\zeta=1
$$

The system returns quickly without oscillation.

### Overdamped

$$
\zeta>1
$$

The system does not oscillate, but responds more slowly.

### Undamped

$$
\zeta=0
$$

Oscillation continues indefinitely in the ideal model.

The exact mathematics will matter later.

For now, the practical intuition is:

> **Damping removes energy from unwanted motion.**

---

## 3.11 Rise Time, Settling Time, and Steady-State Error

Several terms describe how a controller responds to a change in target.

### Rise time

How quickly the output approaches the target.

### Settling time

How long it takes before the output remains sufficiently close to the target.

### Overshoot

How far the response exceeds the target.

### Steady-state error

The remaining error after transient behavior has settled.

These quantities often conflict.

A very aggressive controller may reduce rise time while increasing overshoot.

A very conservative controller may avoid overshoot but take too long to settle.

Control design is often about choosing acceptable tradeoffs.

---

## 3.12 Why Proportional Control Can Leave Persistent Error

Suppose a motor must apply constant torque just to support a load.

A proportional controller computes:

$$
u=K_Pe
$$

If error becomes zero:

$$
e=0
$$

then:

$$
u=0
$$

But the plant may require nonzero input merely to hold the desired state.

Therefore the system may settle slightly away from the target.

It needs some persistent error to generate the control effort required to oppose the load.

This is called **steady-state error**.

One solution is integral control.

---

## 3.13 Integral Control

The integral of error is:

$$
I(t)=\int_0^t e(\tau)\,d\tau
$$

Do not let the notation frighten you.

In programmer language:

```text
Integral += Error * DeltaTime;
```

The integral term remembers accumulated error over time.

The control contribution is:

$$
u_I(t)=K_I I(t)
$$

Its intuition is:

> **You have been wrong for too long. Increase the correction.**

If a small error persists, the integral term keeps growing until enough control effort is produced to eliminate it.

This makes integral control very effective at removing steady-state error.

It also creates new ways to misbehave.

---

## 3.14 Integrator Windup

Real actuators have limits.

Suppose a controller asks a motor for:

```text
500 N·m
```

but the motor can only produce:

```text
20 N·m
```

The actuator is saturated.

The actual output is clamped:

```text
CommandRequested = 500
CommandApplied   = 20
```

But the integral term may continue accumulating error:

```text
Integral += Error * DeltaTime;
```

So while the actuator is already fully saturated, the controller keeps building a larger and larger internal correction.

When the plant finally approaches the target, the controller may still contain an enormous accumulated integral term.

It then overshoots spectacularly.

This is **integrator windup**.

For a programmer, the bug pattern should look very familiar:

> A state variable keeps accumulating while downstream output is already clamped.

The controller's internal state becomes disconnected from what the actuator can actually do.

**Anti-windup** mechanisms exist to prevent this.

---

## 3.15 Derivative Control

The derivative term considers how quickly the error is changing:

$$
u_D(t)=K_D\frac{de(t)}{dt}
$$

A numerical approximation might look like:

```text
Derivative =
    (CurrentError - PreviousError)
    / DeltaTime;
```

The intuition is:

> **If the error is changing rapidly, account for where the system is heading.**

Suppose the system is still below the target, but moving toward it very quickly.

A proportional controller sees:

```text
still below target
→ keep pushing
```

A derivative term sees:

```text
approaching target very quickly
→ reduce push before overshoot
```

Derivative control therefore often contributes damping.

A useful informal summary is:

* **P** cares about how wrong you are now.
* **I** cares about how long you have been wrong.
* **D** cares about how quickly the error is changing.

---

## 3.16 PID

Combine all three terms:

$$
u(t)=K_Pe(t)+K_I\int_0^t e(\tau)d\tau+K_D\frac{de(t)}{dt}
$$

This is the classic PID controller.

In programmer-style pseudocode:

```text
Error = Target - Measurement;

Integral += Error * DeltaTime;

Derivative =
    (Error - PreviousError)
    / DeltaTime;

Command =
      Kp * Error
    + Ki * Integral
    + Kd * Derivative;

PreviousError = Error;
```

That is much less mystical than the acronym makes it sound.

PID is simply a stateful control policy combining:

* current error,
* accumulated historical error,
* change in error.

---

## 3.17 PID as an Agent

Now map PID back into the framework from Chapter 2.

### Plant

The physical system being controlled.

### Observation

Measured output.

### Objective

Reference or setpoint.

### Controller state

Typically:

```text
Integral
PreviousError
```

### Policy

The PID equation.

### Action

Control command.

### Effect

Physical actuation changes plant behavior.

### Feedback

The resulting plant output is measured again.

Therefore PID fits directly into the minimal agentic model:

$$
(m_{t+1},a_t) = \pi(o_t,m_t,r_t)
$$

where controller memory contains the accumulated integral and previous error.

PID is not intelligent in any general sense.

It does not reason.

It does not plan.

But it is an agent under the definition used in this book because it observes, retains state, selects a control action, and affects a plant.

> **PID is an agent with an extremely small, fixed policy.**

---

## 3.18 Stability

A controller is useful only if its behavior remains controlled.

The intuitive idea of **stability** is:

> Small disturbances should not cause the system to run away indefinitely.

Suppose the system is near equilibrium.

Something bumps it.

A stable system tends to return toward acceptable behavior.

An unstable system may amplify the disturbance.

For example:

```text
small position error
→ correction
→ overshoot
→ stronger opposite correction
→ larger overshoot
→ even stronger correction
→ divergence
```

The feedback loop has become self-amplifying.

This is not unique to mechanical systems.

Computer scientists have seen analogous phenomena:

```text
temporary slowdown
→ retry more requests
→ server becomes more overloaded
→ more requests fail
→ retry even more
```

That is a **retry storm**.

Or:

```text
high load
→ add capacity
→ load drops
→ remove capacity
→ load rises
→ add capacity
→ ...
```

That is an oscillating autoscaler.

Or:

```text
queue grows
→ scheduler changes priority aggressively
→ queue drains
→ scheduler reverses priority
→ queue grows again
```

These are control problems even when no motor is involved.

---

## 3.19 Asymptotic, Marginal, and Unstable Behavior

Several stability concepts are useful.

### Asymptotically stable

After a disturbance, the system returns toward equilibrium.

Symbolically:

$$
x(t)\rightarrow x^\*
$$

as time increases.

### Marginally stable

The system remains bounded but may never settle.

A perfect undamped oscillator is the classic example.

### Unstable

The state moves increasingly far from equilibrium.

Oscillations may grow.

Errors may diverge.

The machine may, in sufficiently entertaining cases, attempt to become aerospace hardware.

---

## 3.20 Linear Systems

Much of classical control focuses on **linear systems**.

A linear system has a special mathematical structure that makes analysis much easier.

One common continuous state-space form is:

$$
\dot{x}=Ax+Bu
$$

and:

$$
y=Cx+Du
$$

where:

* \(x\) is the state vector,
* \(u\) is the control input,
* \(y\) is the output,
* \(A\) describes internal plant dynamics,
* \(B\) describes how inputs affect state,
* \(C\) describes how state becomes output,
* \(D\) describes any direct input-to-output effect.

For a programmer, the first equation can be read approximately as:

```text
StateRate =
      InternalDynamics * State
    + InputDynamics    * Control;
```

The second:

```text
Output =
      StateProjection * State
    + DirectInput     * Control;
```

This representation should feel much closer to ordinary computational state-transition thinking than transfer functions usually do.

---

## 3.21 Continuous State-Space and the Agentic Model

Recall the discrete plant model from Chapter 2:

$$
x_{t+1}=F(x_t,a_t,d_t)
$$

Continuous control replaces the explicit next-state step with a rate of change:

$$
\dot{x}=f(x,u,d)
$$

These are two forms of the same underlying idea.

Discrete:

> Given current state and action, what state comes next?

Continuous:

> Given current state and action, how is state changing right now?

This is one of the central bridges between computer science and mechanical control.

The representation differs.

The control structure does not.

---

## 3.22 Eigenvalues and Natural Modes

A linear system:

$$
\dot{x}=Ax
$$

contains natural response modes determined by the eigenvalues of \(A\).

You do not need the full linear-algebra derivation yet.

The useful intuition is:

> An eigenvalue describes how one natural mode of the system grows, decays, or oscillates.

For continuous systems:

### Negative real part

$$
\Re(\lambda)<0
$$

The mode decays.

Good.

### Zero real part

The mode may persist indefinitely.

Possibly marginal.

### Positive real part

$$
\Re(\lambda)>0
$$

The mode grows.

Bad.

Very roughly:

```text
negative → dies away
zero     → persists
positive → grows
```

If an eigenvalue also has an imaginary component, the mode oscillates.

So a complex eigenvalue with negative real part corresponds roughly to:

```text
oscillate
while gradually shrinking
```

and one with positive real part:

```text
oscillate
while gradually exploding
```

There is your formal version of the increasingly violent wiggle.

---

## 3.23 Transfer Functions

Another common representation of a linear system is the **transfer function**.

$$
G(s)=\frac{Y(s)}{U(s)}
$$

A transfer function describes the relationship between input and output after transforming the system into the Laplace domain.

The important conceptual trick is that the Laplace transform converts differential equations into algebraic relationships.

For example, differentiation in time becomes multiplication by \(s\).

That means ugly expressions involving:

```text
derivative of x
second derivative of x
third derivative of x
```

can become polynomial expressions involving:

```text
s
s²
s³
```

This makes linear-system analysis dramatically easier.

For a computer scientist, a transfer function is not best thought of as an ordinary function call.

It is closer to:

> **A compact behavioral description of how a linear dynamical system transforms input signals into output signals.**

---

## 3.24 Poles

Suppose:

$$
G(s)=\frac{N(s)}{D(s)}
$$

The values of \(s\) that make:

$$
D(s)=0
$$

are the **poles**.

Poles correspond closely to the natural modes of the system.

This gives us a useful stability rule for ordinary continuous linear systems:

> **Poles in the left half of the complex plane correspond to decaying modes.**

> **Poles in the right half correspond to growing modes.**

Therefore:

```text
left half-plane  → generally stable
right half-plane → unstable
```

Poles on the imaginary axis require more care.

This is why control engineers spend so much time asking where the poles are.

They are effectively asking:

> What natural modes does this system contain, and do those modes decay or explode?

---

## 3.25 Zeros

The values of \(s\) that make the numerator:

$$
N(s)=0
$$

are called **zeros**.

Zeros influence how inputs excite or suppress system response.

They matter greatly in detailed system behavior, but poles are usually the first thing to understand when reasoning about stability.

A crude but useful introductory distinction is:

> Poles tell you a great deal about what the system naturally wants to do.

> Zeros tell you a great deal about how inputs interact with that behavior.

That is incomplete, but useful enough for now.

---

## 3.26 Frequency Response

Physical systems respond differently to inputs that vary at different speeds.

Consider pushing a heavy mass.

A slowly varying force may move it easily.

A rapidly oscillating force may barely move it at all.

Likewise, a controller may track slowly changing commands very well while failing to follow rapid changes.

This leads to **frequency response**.

Instead of asking only:

> What does the system do to this input?

we ask:

> How does the system respond to signals at different frequencies?

This becomes fundamental for understanding:

* filtering,
* bandwidth,
* resonance,
* noise rejection,
* stability margins.

---

## 3.27 Bandwidth

A controller has a limited range of frequencies over which it can respond effectively.

This is loosely called its **bandwidth**.

A high-bandwidth controller can respond to faster changes.

A low-bandwidth controller reacts only to slower variations.

Bandwidth is not simply:

> faster is better.

Higher bandwidth may make the controller more sensitive to:

* measurement noise,
* model uncertainty,
* delay,
* unmodeled dynamics.

This matters tremendously when integrating controllers operating at different timescales.

---

## 3.28 Delay

Delay is one of the nastiest sources of instability.

Suppose the controller observes state at time:

$$
t
$$

but its action does not affect the plant until:

$$
t+\tau
$$

where \(\tau\) is delay.

By the time the correction arrives, the plant may already have moved.

The controller is acting on stale information.

For a programmer, imagine:

```text
ReadState();
Wait(500 ms);
ApplyCorrectionForOldState();
```

If the plant is changing quickly, this is dangerous.

Delay contributes phase lag and can turn a stable controller into an unstable one.

---

## 3.29 Why LLMs Do Not Belong in Fast Control Loops

This is where continuous-control intuition becomes especially important for modern AI systems.

Suppose a motor controller needs to update at:

```text
1000 Hz
```

That means one decision every:

```text
1 ms
```

Now suppose an LLM takes:

```text
2 seconds
```

to reason and respond.

The motor state may have changed thousands of times before the LLM's answer arrives.

The issue is not that the LLM is unintelligent.

The issue is:

> **The controller is operating at the wrong timescale.**

An LLM may be excellent for:

```text
choose mission
diagnose failure
replan route
interpret ambiguous instruction
```

while a lower-level controller handles:

```text
stabilize velocity
control torque
maintain attitude
```

This naturally produces hierarchical control:

```text
slow, expressive controller
        ↓
faster supervisory controller
        ↓
fast stabilizing controller
        ↓
plant
```

Agentic Control Theory treats this as composition across timescales.

---

## 3.30 Digital Controllers Are Actually Discrete

There is a useful irony in modern control systems:

The plant may be continuous, but the controller is usually implemented on a digital computer.

That means the controller samples the plant:

```text
measure
compute
apply command
wait
measure
compute
apply command
...
```

The continuous world is being controlled by a discrete program.

This gives us a sampled system.

Instead of continuously observing:

$$
y(t)
$$

the controller observes:

$$
y[k]
$$

at discrete sample indices.

The controller then computes:

$$
u[k]
$$

and often holds that input until the next sample.

This is sometimes called a **zero-order hold**.

---

## 3.31 Sampling Rate

The **sampling rate** describes how often the controller observes and updates.

If:

$$
\Delta t=0.001\text{ s}
$$

then the controller operates at:

$$
1000\text{ Hz}
$$

Choosing the sampling rate matters.

Too slow, and important plant behavior may occur between samples.

The controller may effectively be blind to fast dynamics.

This can cause:

* poor tracking,
* oscillation,
* aliasing,
* instability.

One of the most important lessons for programmers is:

> **The physical plant keeps evolving between software ticks.**

Your loop frequency is not merely a performance setting.

It is part of the control design.

---

## 3.32 Aliasing

If you sample a signal too slowly, fast motion can appear to be slower motion or even motion in the wrong direction.

This is **aliasing**.

A familiar visual example is a wheel in a video appearing to rotate backward.

The wheel is moving continuously.

The camera samples it at discrete times.

The samples create a false apparent motion.

Sensors and digital controllers suffer the same problem.

Sampling does not merely produce less information.

It can produce misleading information.

---

## 3.33 Nyquist Intuition

The Nyquist sampling theorem states, roughly, that to reconstruct a band-limited signal, the sampling frequency must be greater than twice the highest frequency contained in the signal.

The formal details matter in signal processing.

The practical control intuition is simpler:

> **If important plant behavior happens faster than your sampling loop can observe, you cannot reliably control it from those samples.**

In real control systems, the desired sampling rate is often substantially higher than the bare Nyquist limit because control requires useful phase and response margin, not merely theoretical signal reconstruction.

---

## 3.34 Saturation

Physical actuators have limits.

A controller may compute:

```text
DesiredTorque = 100;
```

while the actuator can provide only:

```text
MaximumTorque = 20;
```

The command becomes:

```text
ActualTorque = clamp(DesiredTorque, -20, 20);
```

This is **saturation**.

For programmers, the concept is easy.

The consequences are not.

Once saturation occurs, the linear assumptions used to design the controller may stop being valid.

Combined with integral control, saturation may create windup.

Combined with aggressive feedback, it may create surprising transient behavior.

Physical limits are part of the system.

They cannot be abstracted away merely because the control equation produces larger numbers.

---

## 3.35 Rate Limits

Some actuators are limited not only in magnitude but also in how quickly they can change.

For example:

```text
maximum steering angle = 30°
maximum steering rate  = 100°/s
```

Even if the controller requests an immediate jump from:

```text
-30°
```

to:

```text
+30°
```

the mechanism physically requires time to move.

This is a **rate limit**.

Rate limits effectively add dynamics and delay.

Again:

> The action the controller requests is not necessarily the effect the plant immediately experiences.

---

## 3.36 Dead Zones and Friction

Some plants do not respond to very small commands.

For example:

```text
command = 0.01
```

may produce no movement because static friction dominates.

Only when the command exceeds some threshold does motion begin.

This is a **dead zone**.

Real systems may contain:

* backlash,
* friction,
* hysteresis,
* saturation,
* nonlinear stiffness,
* actuator delay.

These effects make physical systems less tidy than ideal linear models.

Linear control remains enormously useful because many systems can be approximated linearly near an operating point.

But the approximation has limits.

---

## 3.37 Linearization

Many real plants are nonlinear.

Their dynamics may be:

$$
\dot{x}=f(x,u)
$$

where \(f\) is not linear.

However, near some operating point, the system may behave approximately linearly.

Then we can derive a local model:

$$
\dot{x}=Ax+Bu
$$

after shifting coordinates appropriately.

This is **linearization**.

The intuition is similar to approximating a curved surface with a tangent plane.

Nearby, the approximation may be excellent.

Far away, it may become terrible.

This is why a controller can work beautifully around one operating condition and fail badly elsewhere.

---

## 3.38 Controllability

A plant may have states that the controller simply cannot influence.

Suppose a spacecraft has no thruster capable of producing torque around one axis.

No clever control law can directly command rotation around that axis.

This leads to **controllability**.

For a linear system:

$$
\dot{x}=Ax+Bu
$$

controllability asks:

> Given the available inputs, can we move the system through the states we care about?

The classical test uses the controllability matrix:

$$
\mathcal{C}
=
[B\ AB\ A^2B\ \ldots\ A^{n-1}B]
$$

If it has full rank, the linear system is controllable.

The details can wait.

The conceptual lesson is universal:

> **A controller cannot achieve objectives for which it lacks effective actions.**

This will remain true later for discrete and LLM-based agents.

---

## 3.39 Observability

Likewise, the controller may be unable to determine important state from its measurements.

This leads to **observability**.

For:

$$
y=Cx
$$

observability asks:

> Given the available outputs over time, can we determine the internal state that matters?

The classical observability matrix is:

$$
\mathcal{O}
=
\begin{bmatrix}
C\\
CA\\
CA^2\\
\vdots\\
CA^{n-1}
\end{bmatrix}
$$

Full rank indicates observability for the linear system.

Again, the intuition matters more than the matrix right now:

> **If you cannot observe enough about the plant, no policy can reliably respond to information it does not have.**

---

## 3.40 Observers

When important state cannot be measured directly, the controller may estimate it.

That estimation mechanism is called an **observer**.

Suppose you can measure position but need velocity.

The observer combines:

* previous estimates,
* plant dynamics,
* new measurements,

to estimate hidden state.

We represent estimated state as:

$$
\hat{x}
$$

The observer therefore becomes another computational component in the loop:

```text
Sensor measurements
        ↓
State estimator
        ↓
Estimated state
        ↓
Controller
```

This maps directly onto the distinction from Chapter 2 between:

* plant state,
* observation,
* estimated state.

---

## 3.41 The Kalman Filter

One famous observer is the **Kalman filter**.

Despite the word *filter*, its deeper role is state estimation.

It combines:

* a model of plant dynamics,
* uncertain measurements,
* estimates of process noise,
* estimates of measurement noise,

to maintain an estimate of system state.

In simplified conceptual form:

```text
Predict where the state should be
        ↓
Receive new measurement
        ↓
Compare prediction and measurement
        ↓
Correct estimate according to uncertainty
```

The Kalman filter is valuable because it formalizes an idea that appears throughout agentic systems:

> **The agent may need to construct its own best estimate of reality from incomplete and noisy observations.**

---

## 3.42 Robustness

A controller is usually designed using a model.

Reality is not the model.

Suppose we design using:

$$
G_{\text{model}}
$$

but the actual plant is:

$$
G_{\text{real}}
$$

Differences may come from:

* manufacturing tolerances,
* temperature,
* wear,
* changing payload,
* uncertain parameters,
* external disturbances.

A controller is **robust** if it continues behaving acceptably despite such mismatch.

This distinction gives us:

### Nominal performance

How the controller behaves under the assumed model.

### Robust performance

How it behaves when the real system differs from the model.

A controller that performs perfectly only under laboratory assumptions may be practically useless.

---

## 3.43 Continuous Control Is Already Agentic

We can now map continuous control directly into the model from Chapter 2.

| Agentic concept  | Continuous-control interpretation               |
| ---------------- | ----------------------------------------------- |
| Plant            | physical dynamical system                       |
| Plant state      | position, velocity, temperature, pressure, etc. |
| Observation      | sensor measurement                              |
| Estimated state  | observer output                                 |
| Controller state | integral state, filters, estimates              |
| Objective        | reference or setpoint                           |
| Policy           | control law                                     |
| Action           | control input                                   |
| Effect           | physical plant response                         |
| Disturbance      | external physical influence                     |
| Feedback         | sensor result returned to controller            |
| Timescale        | sample rate and controller bandwidth            |

The mathematics of classical control is highly specialized because continuous physical systems have particular problems:

* inertia,
* stored energy,
* delay,
* oscillation,
* instability,
* noise,
* actuator limits.

But the abstract control structure is the same one introduced earlier.

---

## 3.44 The Important Lesson for Computer Scientists

The biggest conceptual shift is this:

> **A continuous plant is not a passive data structure waiting for the controller to mutate it.**

It is an evolving dynamical system.

Its state may continue changing while:

* your program is blocked,
* your scheduler is busy,
* your controller is computing,
* your network packet is delayed,
* your LLM is thinking.

Time is not just metadata.

Time is part of the system.

That is why concepts such as:

* latency,
* sampling rate,
* damping,
* bandwidth,
* stability,
* phase,
* actuator limits,

are not mere implementation details.

They determine whether the controller works at all.

---

## 3.45 The Agentic Interpretation

From the Agentic Control Theory perspective, continuous control gives us an especially important class of agent.

A continuous controller typically has:

* limited observations,
* tightly bounded actions,
* highly predictable policy,
* fast update rates,
* explicit physical objectives,
* mathematically analyzable behavior.

Its intelligence is low.

Its reliability may be extremely high.

That combination is exactly why continuous controllers should not be replaced casually by more expressive agents.

A more sophisticated agent should usually sit above them.

For example:

```text
LLM planner
    ↓
behavior controller
    ↓
trajectory controller
    ↓
PID controller
    ↓
motor
```

Each layer solves a different control problem.

Each has a different timescale.

Each requires a different kind of policy.

---

## 3.46 Continuous Control Is Not the Whole Story

Classical control theory is extraordinarily powerful.

But it is not naturally suited to every kind of decision.

Consider:

```text
Should the robot continue searching,
return to charge,
or investigate an alarm?
```

Those are not merely continuous corrections around a setpoint.

They are discrete decisions between qualitatively different modes.

Likewise:

```text
Patrol
→ Chase
→ Attack
→ Flee
```

is not naturally expressed as a PID equation.

This does not mean classical control has failed.

It means we have reached another region of the control problem.

Continuous control excels at questions like:

> How much?

> How fast?

> In which direction?

> How strongly should we correct?

Discrete control often deals with questions like:

> Which mode?

> Which behavior?

> Which task?

> Which transition?

The next chapter turns to that world.

---

## 3.47 Where We Go Next

In this chapter we saw that continuous control fits naturally inside Agentic Control Theory.

A continuous plant evolves according to:

$$
\dot{x}=f(x,u,d)
$$

A controller observes the plant, maintains internal state, follows an objective, and computes a control action.

PID is therefore one specific policy architecture.

State-space control is another.

Observers provide estimated state.

Feedback closes the loop.

Stability tells us whether errors decay or grow.

Bandwidth and sampling tell us how quickly the controller may react.

None of these concepts disappear in Agentic Control Theory.

They remain exactly as important as before.

What changes is their position in the larger picture.

Classical continuous control becomes one highly developed specialization of a broader theory of agents influencing state over time.

The next chapter crosses the bridge in the other direction.

We leave systems whose state flows continuously and turn toward systems whose behavior changes through explicit computational decisions:

**finite-state machines, pushdown automata, guards, transitions, utility selection, and hierarchical discrete control.**
