To prove `not_in_map(...)` in antecedent: (shows that some `dlvar` isn't in a map)

```pvs
(HIDE-ALL-BUT 1)(EXPAND "not_in_map")(SKEEP)(EXPAND "nth")(EXPAND "nth")(TYPEPRED "i")(EXPAND "length")(EXPAND "length")(EXPAND "length")(ASSERT)(HIDE -1)(LIFT-IF)(LEMMA "distinct_vars")(EXPAND "list_of_correct_var")(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "pairwise_distinct_vars?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(EXPAND "distinct_var?")(FLATTEN)(ASSERT)
```
