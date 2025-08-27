# Notes

## Proving `not_in_map` in antecedent

To prove `not_in_map(...)` in antecedent: (shows that some `dlvar` isn't in a map)

```pvs
(HIDE-ALL-BUT 1)(EXPAND "not_in_map")(SKEEP)(EXPAND "nth")(EXPAND "nth")(TYPEPRED "i")(EXPAND "length")(EXPAND "length")(EXPAND "length")(ASSERT)(HIDE -1)(LIFT-IF)(LEMMA "distinct_vars")(EXPAND "list_of_correct_var")(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(ASSERT)
```

## Problems with RealExpr

```pvs

[-1]   lowerX0(EEE) - EEE(dlvar_index(gs)) * sin(EEE(dlvar_index(trk))) * t - 1/2 * (sin(EEE(dlvar_index(trk))) * t ^ 2 * a)
       <= env1(dlvar_index(sx))

[-2]   env1(dlvar_index(sx)) <=
       upperX0(EEE) - EEE(dlvar_index(gs)) * sin(EEE(dlvar_index(trk))) * t - 1/2 * (sin(EEE(dlvar_index(trk))) * t ^ 2 * a)

[-3]   lowerY0(EEE) - EEE(dlvar_index(gs)) * cos(EEE(dlvar_index(trk))) * t - 1/2 * (cos(EEE(dlvar_index(trk))) * t ^ 2 * a)
       <= env1(dlvar_index(sy))

[-4]   env1(dlvar_index(sy)) <=
      upperY0(EEE) - EEE(dlvar_index(gs)) * cos(EEE(dlvar_index(trk))) * t - 1/2 * (cos(EEE(dlvar_index(trk))) * t ^ 2 * a)

[-5]   lowerX0(EEE) <= env1(dlvar_index(sx))
[-6]   env1(dlvar_index(sx)) <= upperX0(EEE)
[-7]   lowerY0(EEE) <= env1(dlvar_index(sy))
[-8]   env1(dlvar_index(sy)) <= upperY0(EEE)
  ├───────
{1}   envFinal = env1
[2]   (lowerX0(EEE) <= envFinal(dlvar_index(sx)) AND
        envFinal(dlvar_index(sx)) <= upperX0(EEE))
       AND
       lowerY0(EEE) <= envFinal(dlvar_index(sy)) AND
        envFinal(dlvar_index(sy)) <= upperY0(EEE)

>>
```

## Why I'm getting false positives

Some Examples:

```pvs
  % `dl_path_action_example.pvs`

  % QED
  %! BUG: False positive, should reach 3.5 but stops at 2.5
  accel_then_linear_example: LEMMA
    %                                   +1.5          +2
    (: :) |- ALLRUNS(init(0,0,1,0) ++ accel(1,1) ++ linear(1),enclosure(0,0,0,2.5))

  %! BUG: False negative, should reach 4 but stops at 3
  accel2_example: LEMMA
    %                                   +1.5        +2.5
    (: :) |- ALLRUNS(init(0,0,1,0) ++ accel(1,1) ++ accel(1,1),enclosure(0,0,0,3))

  %! BUG: False positive, should reach 7.5 but stops at 4.5
  three_accels_example: LEMMA
    (: :) |- ALLRUNS(init(0,0,1,0) ++ accel(1,1) ++ accel(1,1) ++ accel(1,1),enclosure(0,0,0,4.5))

  %! BUG: False positive, should reach 4
  linear_then_accel2a_example: LEMMA
    %                                   +1          +1.5            +2.5
    (: :) |- ALLRUNS(init(0,0,1,0) ++ linear(1) ++ accel(1,1) ++ linear(1), enclosure(0,0,0,3.5))

  %! BUG: False positive, should reach 5 but stops at 4
  linear_then_accel2b_example: LEMMA
    %                                   +1          +1.5            +2.5
    (: :) |- ALLRUNS(init(0,0,1,0) ++ linear(1) ++ accel(1,1) ++ accel(1,1),enclosure(0,0,0,3.9))


```

This only happens when there is at least an `accel(...)` with ANYTHING after it.
The reason why is quite subtle. My current `path_accel` lemma, which is used to
prove these examples, has turns something of this form:

```pvs
(: :) |- ALLRUNS(fp ++ action(..), enclosure())
```

Into this:

```pvs
(: :) |- ALLRUNS(fp, enclosure() AND enclosure_new())
```

> Note that this is a simplification of how it actually works, some redundancy
> is required for PVS's pattern matching to automatically apply the lemma properly.

The lemma does this transformation with an implication (`IMPLIES`):

```pvs
(: :) |- ALLRUNS(fp, enclosure(...) AND enclosure_new(...))
  IMPLIES
(: :) |- ALLRUNS(fp ++ action(...), enclosure(...))
```

Now, the `enclosure_new(...)` is a requirement that is calculated within the
lemma, and FULLY depends on the `action()` that the lemma also (conveniently)
removes from the equation.

However, the place where `enclosure_new(...)` is defined is in the second
argument of an `ALLRUNS(...)`, meaning it is a requirement on **the end state
of whatever program is inside the `ALLRUNS`**. This also means that any dL
variables that the `enclosure_new` requirement accesses are evaluated _in the
final environment of the program inside the ALLRUNS_.

Notably, the `enclosure_new(...)` is defined with math that depends on the
evaluation of the `val(gs)` and `val(trk)`. This is obvious looking at the
kinematic equation of motion that is rearranged to calculate the bounds of the
new enclosure:

$$
x_f = x_i + v_i t +\frac{1}{2} at^2 \\
$$

in other words...

$$
\Delta x = v_i t +\frac{1}{2} at^2
$$

Since we know the ground speed, not the velocity, we must also add a coefficient
of sin to the mix:

$$
\Delta x = (v_i t +\frac{1}{2} at^2) (sin(\Theta))
$$

We want to subtract shift our enclosure by this $\Delta x$, so we do this:

$$
\text{lowerX} - \Delta x
$$
expanding...

$$
\text{lowerX} - (v_i t +\frac{1}{2} at^2) (sin(\Theta))
$$
We have some unknowns in this expression, however:

| Unknown       | Reprentation   | Explanation                                                         |
| ------------- | -------------- | --------------                                                      |
| `lowerX`      | `lowerX`       | a `real` var (provided)                                             |
| `t`           | `t`            | time elapsed during the `action(...)` (provided)                    |
| `a`           | `a`            | acceleration that occurs in  the `action(...)` (provided)           |
| `v_i`         | `val(gs)`      | ground speed, **depends on end state of program in the `ALLRUNS`**  |
| `Theta`       | `val(trk)`     | track/heading, **depends on end state of program in the `ALLRUNS`** |

The first three unknowns are provided by the user and are trivially extracted
from the lemma. The final two, although _initially provided_ in the `init(...)`,
could have changed if the flight ever underwent acceleration or turned. Since we
know that the `action(...)` in this case was an `accel(...)`, we can ignore
`val(trk)` for now, and here the problem becomes clear:

We are approaching and unraveling this flight _backwards_, so the **end state of
the program inside the `ALLRUNS`**, which is exactly what `gs` depends on, means
something different from our point of view and the flight's point of view.

In essence, the problem exists because this flight, when reversed, is **not the
same!**

```pvs
%                      ★
init(0,0,1,0) ++    accel(1,1) ++ linear(1) ++ linear(1)
```

```txt
x=0       ★        x=1.5         x=3.5          x=5.5
       ------→            ---->          ---->
v=1                v=2           v=2            v=2
```

And when reversed:

```pvs
%                                                ★
init(0,0,1,0) ++    linear(1) ++ linear(1) ++ accel(1,1)
```

```txt
x=0       ★        x=1           x=2              x=3.5
       ------→            ---->          ------→
v=1                v=1           v=1              v=2
```
