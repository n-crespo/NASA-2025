# Formally Verifying the Safety of Untrusted Aircraft Trajectories in PVS

> [!NOTE]
> A prerequisite for this work is the [embedding of aircraft trajectories as
> hybrid programs](./dl_flight/dl_flight_trajectories.pvs) in Plaidypvs, which
> was largely developed by Bernny Velasquez and César Muñoz. A valid flight plan
> (`ValidFP`) is an `init(...)` followed by a series of instructions such as
> `accel(...)`, `linear(...)` and `turn(...)`, executed sequentially with `++`.
> The proofs below assume and reason with this structure.

## Goal

Describe an untrusted flight plan as a hybrid program (HP) built from predefined actions
(`init(...)`, `accel(...)`, `turn(...)`, ...) and prove that every run of it
stays inside some predefined operational enclosure:

```pvs
Gamma |- ALLRUNS(init(...) ++ action(...) ++ ... , enclosure(lowerX, upperX, lowerY, upperY))
```

without the user having to expand the semantics of each action by hand.

## My Approach

Each lemma in [`dl_path_action.pvs`](./dl_flight/dl_path_action.pvs) consumes
the action directly after `init(...)` and turns its effect into a new
`init(...)`:

```pvs
% path_accel (simplified)
start and end of accel(a,t) are inside the enclosure AND
(Gamma |- ALLRUNS(SEQ(init(x+dX, y+dY, gs+a*tmin, trk, tmin, t), hp), enclosure(...)))
  IMPLIES
(Gamma |- ALLRUNS(SEQ(init(x, y, gs, trk, t0, tm0), SEQ(accel(a,t), hp)), enclosure(...)))
```

Because every state variable is an argument of `init(...)`, the
enclosure never refers to a value (like `gs` or `trk`) that a later step could
change. Applying the lemma repeatedly shrinks the flight down to a single
`init(...)`.

The [flight-path proofs](./dl_flight/dl_path_action.prf) for `path_accel` and
`path_accel_base` can be used to show that every run of an acceleration action
keeps the vehicle within safety boundaries using the ODE solution lemma
[`accel_sol`](./dl_flight/dl_trajectory_continuous.pvs). The
[`dl-enclosure`](./dl_flight/pvs-strategies) strategy automates proving these
kinds of `ALLRUNS(fp, enclosure(...))` goals by following these steps:

1. rewrite `a ++ b ++ c` into `SEQ(a, SEQ(b, c))`
2. pattern-match on the shape of the dL sequent (`init` followed by an `accel` or `turn`),
3. apply the matching `path_accel`/`path_accel_base` (or `path_turn`/`path_turn_base`) lemma,
4. and recurse until only `init(...)` is left. (note `path_turn` is not yet proven)

> [!NOTE]
> A test bench of safe/unsafe flight examples that can be solved with this
> strategy can be found in
> ([dl_path_action_example.pvs](./dl_flight/dl_path_action_example.pvs)). An
> HTML visualization of the [`path_accel` proof
> tree](./dl_flight/dl_enclosure/path_accel.html) is also available (it may
> require quite a bit of zooming out to be visible).

## My Previous Approach

The [original plan](./notes/backwards-approach.md) peeled actions off the end
of a flight and moved the enclosure backwards through each one. This is the
approach I presented in these
[slides](./slides/Representing_and_Formally_Verifying_Flight_Trajectories_as_Hybrid_Programs.pptx).
Unfortunately I later found that such an approach produces false positives once an
`accel(...)` is followed by anything else. Further explanation
[here](./notes/backwards-approach-false-positives.md).

Artifacts from this approach are still in the repo:

- [proofs](./dl_flight/dl_enclosure/backwards_approach/dl_path_action.prl) for `path_accel`, `path_accel_base`,
  `path_accel_reals`, `intersect_enc` and `intersect_enc_re` (the matching
  `.pvs` spec was replaced by the forward version)
- the backwards [`dl-enclosure`](./dl_flight/dl_enclosure/backwards_approach/pvs-strategies)
  strategy, which used `intersect_enc` to intersect enclosures via `DLAND`
- the [`path_accel_reals` proof tree](./dl_flight/dl_enclosure/backwards_approach/path_accel_reals.html)

## Other Notes

- I wrote proofs for the PVS dynamic logic sequent rules [`dl_orR` and `dl_impliesR`](./dl_flight/dynamic_logic.prl)
- I wrote a [short guide](./notes/pvs-on-windows.md) on running PVS and VSCode-PVS through WSL
- [_old/](./notes/_old/) contains some early weekly notes and proof snippets
