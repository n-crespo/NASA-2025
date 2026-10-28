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

