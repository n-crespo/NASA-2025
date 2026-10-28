Week 3 Notes

I have been making progress on the dl_path_action lemma, including writing
dl_path_action_example in an attempt to verify the correctness of my lemma. The
idea is that if I can't prove dl_path_action_example using the path_action lemma,
there's definitely something wrong. Otherwise, there's a better chance things
are working as expected.

After making some critical fixes to path_action, we were able to prove the
simple example containing `init;accel` using a `grind()` and matching up
specific variables for pvs to understand across the antecedent and sequent. We
improved upon this by using `unroll` to take apart all variables defined in the
antecedent so that a simple `(assert)` can be used to do the final
simplification.

```pvs
%-------------------------------------------
% @formula: example0
% @theory: dl_path_action_example
% @pvsfile: dl_path_action_example.pvs
% @status: ✅ proved
%-------------------------------------------
example0 : PROOF
(then (LEMMA "path_action")(INST? -1)(ASSERT)(HIDE 2)
  (spread (GRIND)
   ((then (EXPAND "semantic_rel")(FLATTEN)(HIDE -2)(UNROLL -1)(ASSERT))
    (then (EXPAND "semantic_rel")(FLATTEN)(HIDE -2)(UNROLL -1)(ASSERT)))))
QED example0
```

We were able to improve upon this by writing a strategy `dl-init` that
continuously `grind` and `unroll`s a lemma of the form:

```pvs
ALLRUNS(init();accel(), enclosure()
```

And can be used to prove our more simple example in one step:

```pvs
%-------------------------------------------
% @formula: example0
% @theory: dl_path_action_example
% @pvsfile: dl_path_action_example.pvs
% @status: ✅ proved
%-------------------------------------------
example0 : PROOF
(then (DL-INIT))
QED example0
```

See the [specification of this strategy](../assets/pvs-strategies).

By first using the `path_action` lemma, we can simplify any flight with >1
action into a flight with 1 less action and a modified enclosure. Once it is
simplified enough, `dl-init` should be able to finish it off:

```pvs
%-------------------------------------------
% @formula: example
% @theory: dl_path_action_example
% @pvsfile: dl_path_action_example.pvs
% @status: ✅ proved
%-------------------------------------------
example : PROOF
(then (LEMMA "path_action")(INST? -1)(ASSERT)(HIDE 2)(DL-INIT))
QED example
```

Without using the strategy, the proof looks like this:
