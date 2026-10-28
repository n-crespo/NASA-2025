# A Generalized Formalization of Aircraft Trajectories Described as Hybrid Programs

## My Approach

I will refer to the arbitrary step previously referred to as `forward(n)` as the
"action" that will occur after HP.

I want to be able to have a _new_ predicate, P', that I can use to show that
P is true. If we can define a P' such that HP satisfying P' implies that
`HP;action |- P`:

```pvs
DLIMPLIES(
  ALLRUNS( HP, P_prime),
  ALLRUNS( SEQ( HP, action ), P)
)
```

I am thinking of doing similar to `dl-diffinv` and `dl-ghost`, where the user adds some
information to the proof (in the case of the differential invariant this
information was the ghost, or counterbalancing function) that specifies HOW the
action affects the HP in a general way.

### Example: 1D Case

Lets consider the 1 dimensional case of the trajectory. In this case, we have a
hybrid program (HP) that runs in sequence with some action or change that
occurs (let's say) _after_ the HP.

> [!NOTE]
> The fact that we are saying the action is occurring AFTER the initial HP will
> (later) not necessarily be upheld since we plan to add actions like
> `backwards(n)` that _pre_-pend segments to the trajectory HP. This is useful
> when defining a trajectory that lands a plane then takes off again.

Let HP be some trajectory that the aircraft has followed, and let the action
be `forward(10 seconds)`, where `forward(n)` evolves the `x` and `y` variables
in the HP via the rule:

$$
x' = vx \\
y' = vy
$$

where `vx` and `yx` are the velocities in the `x` and `y` direction, respectively.

Thus, we have

$$
[HP;\text{forward(n)}]\ P
$$

or

```pvs
ALLRUNS(
  SEQ( HP, forward(n) ),
  P
)
```

Let P be the predicate that states the aircraft is within some bounds, say:

$$
(|x| < A ) \land (|y| < B)
$$

This creates a square shaped enclosure around the origin with side length $B$,
and $A$.

We want to form some kind of general rule/strategy that will enable the user to
show that for ALL RUNS of `[HP];forward(n)`, some predicate `P` is true. We will
do this by creating a new predicate `P'` that, when the HP satisfies it, implies
that the sequence satisfies `P`.

> [!WARNING]
> This solution my not be exactly what we want. Our initial goal is to prevent
> the user from having to `(expand)` part of this proof to prove each piece
> individually, but this solutions entails asking the user to define some new
> rule/function that describes how the action affects the trajectory HP.
> Furthermore, since this proves inductively, they will also have to specify
> such a function for each "action" segment that _makes up_ HP (unless we can
> already make some assumptions about HP, in which case these are not needed)

In this simple case, since we know that `forward(n)` affects `x` and `y` by
simply increasing their values by `vx * n` and `yx * n`, respectively, we can
say at the end of this action:

$$
(x = x + vx * n)
\land
(y = y + vy * n)
$$

This we can define P' as:

$$
P' = B - \text{max}(|vx*n|, |vy*n|) > 0
$$

Thus, if we start at the boundary and "undo" the action, or subtract the action
rather than add it, we can find a _new_ boundary that the original HP must
start within to.

In more general terms, we must apply the _inverse_ of the action to the first
state of the HP where the predicate `P` is **false** to find a new predicate
`P'` that HP must satisfy BEFORE executing the action for the sequence
`HP;action` to satisfy `P`.

> [!NOTE]
> This approach assumes that all actions on HP are continuous

```pvs
(label "A" -1)
(hide "A")
(reveal "A")
```
