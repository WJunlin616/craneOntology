# craneOntology
OWL ontology and Apache Jena reasoning rules for modelling crane operations in construction projects.
### Jena reasoning rules

This repository also includes an Apache Jena rule file  
`Apache_Jena_Reasoning_Rules.rules` that is designed to be used together
with `ontology.owl` via Jena’s `GenericRuleReasoner`.

The rule set encodes forward-chaining production rules
that infer **high-level crane status labels** (e.g. `"Loaded_hold"`,
`"Idle"`, `"Moving_unloaded"`) from **low-level transient attributes** such as:

- `StartPointweightCompareWithEmptyRef` and `EndPointweightCompareWithEmptyRef`
  – whether the load is heavier than the empty reference;
- `BoomAngle` and `SlewingAngle` – boom and slewing motion states
  (e.g. steady vs. changing);
- `LiftingHeight` – hook height behaviour (steady / increasing / decreasing);
- `BoomLength_new` – jib length behaviour (steady vs. changing).

By combining these attribute values, the rules derive a consolidated
`CraneStatus` literal for each observation. This provides a symbolic
representation of crane behaviour that can be:

- linked to `CraneTransientStatus` and `Operation` instances in `ontology.owl`;
- used as input to higher-level reasoning, matching and machine-learning models.

To use the rule file with Jena:

```java
Model baseModel = ...; // contains ontology.owl + instance data

List<Rule> rules = Rule.rulesFromURL("Apache_Jena_Reasoning_Rules.rules");
Reasoner reasoner = new GenericRuleReasoner(rules);
reasoner.setDerivationLogging(true);

InfModel infModel = ModelFactory.createInfModel(reasoner, baseModel);
