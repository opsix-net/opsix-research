# A Synchronous Machine After the Switch: Reading Transients Without Losing the Physics

A generator is carrying load when a breaker closes a short circuit. The voltage at its terminals changes almost instantly, but the rotor currents cannot. That mismatch is the start of a transient: stored magnetic energy and winding currents shape the machine's response before it reaches a new steady condition.

The selected section of *Electrical Machines and Drives: Fundamentals and Advanced Modelling* develops several useful ways to reason about this interval. It distinguishes subtransient and transient models, explains why a pre-fault operating condition matters, and treats zero-sequence and negative-sequence behavior. These are not interchangeable descriptions of one simple current waveform. Each model preserves different physical effects and is useful on a different time scale.

## Begin with the operating point, not just the fault

For a sudden short circuit, the terminal voltage after switching is zero. In the section's notation, the voltage used as the input to its transient-current equations depends on the condition just before the fault. If the machine was unloaded, the relevant pre-fault relation reduces to the negative of the internal emf, $-E_o$. If it was carrying current, the voltage expression also includes the stator resistance and reactance terms associated with that current. Thus, two machines with the same excitation emf can have different transient responses if their pre-fault currents differ.

This is a practical modeling lesson: an event does not erase the state that came before it. A short-circuit calculation needs a consistent initial operating point, including voltage, current, and excitation. Using an unloaded-machine initial condition for a loaded generator can give a misleading current prediction.

The section also makes an important accounting distinction. The currents calculated from its transient equations are transient components, not automatically the total current at every instant. To obtain total current, add the appropriate steady-state component when the chosen formulation separates the two. This is superposition within a model, not permission to mix incompatible reference conditions.

## Choose the model for the time window

Synchronous-machine windings are magnetically coupled. After switching, rotor circuits resist rapid changes in the flux linked with them. The subtransient and transient representations simplify that coupled response by treating selected linked fluxes as momentarily constant.

In the **subtransient** picture, induced currents in both the damper windings and the excitation winding momentarily maintain their coupled fluxes. The associated emf is the stator voltage induced by those fluxes. Stator current can alter the rotor-winding flux only through the relevant leakage paths, represented by subtransient reactances such as $X_d''$ and $X_q''$. This is an initial-time approximation, suited to the interval immediately after switching.

In the **transient** picture, the excitation-winding current maintains its coupled flux momentarily, while the damper-winding effect is no longer retained in the same way. The associated emf is again the stator voltage induced by the flux treated as constant, and the relevant leakage paths use transient reactances such as $X_d'$ and $X_q'$. This representation is useful after the subtransient interval but before steady state.

One can also describe a steady state using subtransient reactances by introducing a fictitious emf. In the section's notation,

$$
E_o'' = E_o + j(X_d-X_d'')I_{do} + j(X_q-X_q'')I_{qo}.
$$

Then the corresponding representation is

$$
V_o = E_o'' + jX_d''I_{do} + jX_q''I_{qo}.
$$

The fictitious emf is an algebraic device for keeping the chosen reactances consistent with the operating point. It is not a claim that the physical excitation suddenly changed. Similarly, a transient steady-state emf can be defined using $X_d'$ and $X_q'$ in place of the double-prime reactances. These adjusted sources let a model use one reactance set while reproducing the intended steady operating condition.

The choice is about the question being asked. A rectifier commutation event supplied by a synchronous machine can be so fast that damper and excitation currents have changed little during commutation. The section therefore identifies subtransient reactances and an approximately constant emf as appropriate for that fast analysis. A study of behavior later in the recovery calls for the transient representation instead.

## Worked example: the steady-state adjustment

Consider a simplified per-unit case with equal direct- and quadrature-axis quantities, so the two axis terms can be combined. Let the original reactance be $X=1.20$, the subtransient reactance be $X''=0.30$, and the pre-event current be $I=0.50$ at zero angle. For this illustrative calculation, assume the relevant internal emf is $E_o=1.00$ at zero angle and ignore any axis distinction. The adjustment is

$$
E_o''=E_o+j(X-X'')I.
$$

First, $X-X''=1.20-0.30=0.90$. Multiplying by the current gives $0.90(0.50)=0.45$, so $E_o''=1.00+j0.45$. Its magnitude is

$$
|E_o''|=\sqrt{1.00^2+0.45^2}=\sqrt{1.2025}=1.0966\text{ (approximately)}.
$$

Now reconstruct the same pre-event voltage in the simplified one-axis form. With $V_o=E_o+jXI$, the original representation gives $V_o=1.00+j(1.20)(0.50)=1.00+j0.60$. Using the adjusted source and subtransient reactance gives $E_o''+jX''I=(1.00+j0.45)+j(0.30)(0.50)=1.00+j0.60$. The two expressions agree exactly under the stated assumptions.

This check illustrates what the adjusted emf accomplishes. It preserves the operating-point voltage while changing the reactance used in the model. It does not calculate the complete short-circuit waveform, because that also requires the machine's dynamic equations, parameters, and initial conditions.

## Zero sequence: a simple series picture

For zero-sequence quantities, the section gives the time-domain relation

$$
V_o=R_s I_o+L_o\frac{dI_o}{dt},
$$

and in the Laplace domain,

$$
V_o(p)=R_sI_o(p)+L_opI_o(p).
$$

So, in this sequence, the machine behaves like a resistance in series with a relatively small inductance. The stated physical interpretation is that the zero-sequence inductance corresponds to stator-phase leakage, including slot, air-gap, and end-winding leakage. This is a sequence-specific equivalent, not a complete description of every magnetic effect in every machine construction.

The equation also provides a useful sanity check: for a constant zero-sequence current, $dI_o/dt=0$, so the inductive voltage term vanishes and the remaining drop is $R_s I_o$. During a rapid current change, the derivative term matters. The relative size of the two terms depends on the waveform and parameters, not on a blanket rule that inductance always dominates.

## Negative sequence: the rotor sees twice frequency

Negative-sequence stator voltages rotate opposite to the synchronous rotor. In stationary coordinates, the applied set has the supply frequency. In rotor-oriented $d$ and $q$ coordinates, however, the relative motion makes the voltage components oscillate at twice the supply angular frequency. The section derives this as $V_q$ and $V_d$ terms involving $2\omega t$.

That frequency doubling explains why the negative-sequence steady-state currents in those coordinates are AC, unlike the DC components associated with positive sequence in the synchronous frame. It first neglects rotor resistance relative to reactance at twice supply frequency, then neglects stator resistance for a simplified result. Under that approximation, the time-domain currents are

$$
I_q(t)=\frac{V}{\omega L_q''}\sin(2\omega t+\delta_2+\delta_3),
$$

$$
I_d(t)=\frac{V}{\omega L_d''}\cos(2\omega t+\delta_2+\delta_3).
$$

The angle symbols specify phase relationships in the source's setup. Their exact values are not needed to see the key behavior: the rotor-frame components oscillate at twice the supply frequency, and their amplitudes depend inversely on the relevant subtransient inductances in this approximation.

Saliency, meaning unequal direct- and quadrature-axis reactances, complicates the idea of a single negative-sequence impedance. With equal subtransient reactances, the section obtains a fundamental per-phase reactance $X_2=X''$ and no third harmonic in the three-phase currents. With unequal values, it reports a small third harmonic in addition to the fundamental. It also gives different approximate fundamental impedances for voltage supply and current supply:

$$
Z_2=j\frac{2X_q''X_d''}{X_q''+X_d''}\quad\text{(voltage supply)},
$$

$$
Z_2=j\frac{X_q''+X_d''}{2}\quad\text{(current supply)}.
$$

These are not contradictory measurements of one universal scalar impedance. With saliency, the waveform can contain harmonics, so a single fundamental-frequency impedance is only an approximation. The source condition affects the approximation. If stator resistance is retained, the section says to add $R_s$ to the stated impedance expressions.

## Where these ideas are useful

Power-system engineers use sequence models to assess unbalanced conditions and fault responses. Machine designers and control engineers need the right time-scale model when checking switching events, protection behavior, or converter interaction. Researchers building simulation tools can use the distinctions as validation targets: a model should preserve the initial operating point, show the correct sequence-frequency behavior, and state which rotor fluxes it treats as effectively constant.

The methods are also useful for deciding what not to model. If a switching event is much faster than the relevant rotor-current decay, a subtransient approximation may be appropriate. If the question concerns the later recovery, retaining only the initial subtransient picture will miss the physics of that interval. For negative sequence, a fundamental impedance may be adequate for some engineering estimates, but it is not a full harmonic model for a salient machine.

## Assumptions and common traps

These equivalent circuits and reactances rely on stated simplifications. The excerpt considers constant-speed synchronous-machine transients, uses sequence transformations, and makes particular resistance-neglect assumptions in its negative-sequence derivation. Its simplified negative-sequence equations should not be applied as exact predictions when stator resistance, rotor resistance, saturation, changing speed, or detailed harmonic effects matter.

A few errors are especially easy to make:

- Treating the transient current as the total current without checking whether a steady component must be added.
- Starting a fault calculation from an unloaded state when the machine was actually carrying current.
- Using subtransient reactances for a later recovery interval without explaining the approximation.
- Calling $E_o''$ a newly created physical emf instead of a fictitious source adjusted to preserve the operating point.
- Assigning one exact negative-sequence impedance to a salient machine while ignoring source conditions and harmonic content.

A good engineering report names the time interval, reference frame, initial operating point, and neglected terms. Those details often matter more than a page of unqualified decimal places.

## Exercises with short answers

1. A machine is loaded before a sudden short circuit. Why is an unloaded-machine initial voltage generally insufficient? **Answer:** The pre-fault current changes the voltage relation used to initialize the transient equations, even if excitation emf is unchanged.

2. In the synchronous rotor frame, what frequency appears in negative-sequence voltage components if the supply angular frequency is $\omega$? **Answer:** $2\omega$, because the sequence rotates opposite to the rotor.

3. What does the adjusted subtransient emf accomplish? **Answer:** It lets the subtransient reactances represent the same selected steady operating point; it is a modeling adjustment, not a physical excitation change.

4. Why can a salient machine have a small third harmonic in negative-sequence current? **Answer:** Unequal axis reactances distort the response, so the three-phase waveform need not be purely fundamental.

---

Published 2026-10-19.

**Source:** Electrical Machines and Drives_ Fundamentals and Advanced Modelling, section **30 Transient Phenomena in Electrical Machines**.

This is an original explanatory note; the source book is not redistributed.
