# craneOntology
OWL ontology and Apache Jena reasoning rules for modelling crane operations in construction projects.


## Ontology (`ontology.owl`)

The **Crane Operation Ontology** is an OWL 2 schema-level ontology that models **mobile crane operations** and their surrounding context. It is designed to support:

- multi-modal fusion of **vision, sensor and structured-text** data  
- ontology-based inference of **transient crane behaviours**  
- similarity-based matching between **as-is executed operations** and **as-planned lifting orders**

---

### Core Concepts

#### Classes

- `Thing`
- `CraneTransientStatus`  
  Crane’s transient operational state derived from low-level signals.
- `Operation`  
  An executed crane operation (e.g., a specific lift).
- `Order`  
  A planned lifting order from the construction schedule.

#### Object Properties

- `belongTo` (`CraneTransientStatus → Operation`)  
  Links a transient status to the operation it belongs to.
- `isSimilarTo` (`Operation ↔ Order`)  
  Relates an executed operation to a planned order that is similar in terms of time, destination and load attributes; declared as a **symmetric** object property.

#### Datatype Properties

- **For `CraneTransientStatus`**  
  - `weightTransientStatus`  
  - `boomTransientStatus`  
  - `slewTransientStatus`  
  - `hookTransientHeightStatus`  
  - `craneTransientBehaviour`  

- **For `Operation`**  
  - `operationDateAndTime`  
  - `operationLoadColourShapeMaterialWeight`  
  - `operationLoadGeometricDimension`  
  - `operationDestinationXYZSpatialPosition`  

- **For `Order`**  
  - `proposedDateAndTime`  
  - `proposedLoadColourShapeMaterialWeight`  
  - `proposedLoadGeometricDimension`  
  - `proposedDestinationAreaAndHeight`  

- **Generic similarity attribute**  
  - `similarityScore` (`xsd:double`) – stores numeric similarity values produced by Jaccard matching algorithms.

---

### Usage and Scope

The current version of `ontology.owl` intentionally contains **only TBox axioms (no individuals)** so that it can be reused across different projects and datasets. Instance data derived from specific sites (operations, orders and their similarity scores) can be generated and linked to this schema using separate scripts.

This ontology has been developed in the context of research on **Knowledge-augmented Multi-modal Data Fusion and Reasoning for Automated Crane Lift Monitoring**, and is intended for reuse in:

- construction informatics  
- semantic sensor fusion  
- ontology-supported reasoning


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
