Week 1 Notes

# 2025-06-16

## Overview of the project

- we want to be able to define a hybrid program (HP) that represents a flight
  plan or trajectory by using some set of predefined actions, liks:
  - `forward(20 seconds)` or `forward(10 miles)`
  - `turn(some_turn_rate)`
  - `accelerate(20 seconds)`
- eventually, we want to be able to go backwards in time:
  - `backwards(20 seconds)`
    - this will prepend an action to time = 0
    - (like a landing trajectory, where t=0 is when the plane is on the ground)

The main idea is that if we have SOME hybrid program, and make some initial
assumptions about it, we want to be able to say that `SEQ(HP, action())` is a
will _always_ satisfy some predicate `p` if some other assumptions are true.

# 2025-06-17

- met with Cesar, Lauren and Bernny this morning

## Understanding the theory

> reading [trajectory spec](../assets/dl_flight_trajectories/dl_flight_trajectories.pvs) and
> Plaidypvs User Manual (printed copy)

# 2025-06-18

```pvs
dl_flight_trajectories [(IMPORTING dL@top) T, x, y, vx, vy : dLVar]: THEORY
  BEGIN
    ASSUMING
       list_of_correct_var : list[dLVar] = (: T, x, y, vx, vy :)
```

- updating master note file (readme)
