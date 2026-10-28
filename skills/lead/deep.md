# Deep phases

Read this only when the user has switched a phase to deep. Apply it to that phase only.

- **Deep plan**: read the few files the decision hinges on yourself. Before showing steps to the user, a `Plan` agent on the strongest model critiques them (missing cases, risky ordering, simpler alternatives) in ≤30 lines. Present the plan with that critique resolved.
- **Deep research**: 2–3 researchers on different angles in parallel, cap ≤100 lines each. Facts the plan depends on are checked by a second researcher who did not find them.
- **Deep build**: builder on the strongest model with `effort: high`. Add to the brief: "List the edge cases before coding, test each, and include them in the report."
- **Deep review**: read the full diff, not only risky files. Then an independent reviewer with fresh context (the profile's reviewer if set, else `general-purpose` on the strongest model) gets the acceptance criteria and the branch: "Find defects, ranked by severity, with path:line and a failure scenario. ≤30 lines. No praise; say 'none found' if none." Verify each finding yourself before sending it to the builder.

After a deep phase, tell the user in one line what it caught, or that it found nothing new.
