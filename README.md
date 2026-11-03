# Quantifying structural impact in the AI threat landscape

A graph-based method for prioritising remediation when a vulnerability affects system components at once.

**[Open the notebook in Colab](https://colab.research.google.com/github/MarwaSal/risk-impact-modelling/blob/main/blast-radius-risk-model.ipynb)**

---

## The problem

In October 2026, two critical vulnerabilities were disclosed in PHP — a language running on roughly three-quarters of the world's web servers, affecting virtually every version released over the last decade. The flaws allowed unauthenticated remote control of the server. The NCSC called it an extremely serious risk, and more than thirty UK government services were taken offline.

But taking everything down at once causes service interruption at scale, with consequences of its own. And if you don't do that, how do you know what to deal with first?

A severity score cannot answer it. Every instance of the same vulnerability carries the same score. It tells you how severe the flaw is, and nothing about its cascading effect.

**The answer has to come from the structure.**

## The role of GenAI

The PHP flaws were found with AI assistance — code that had sat flawed for over a decade, turned into a working exploit. That is the shift this work is built around: as AI shortens the window between a vulnerability being discovered and being exploitable, likelihood moves towards certainty. The useful question becomes consequence — and consequence is computable from structure.

## The questions

1. What services does the product provide, and how critical is each one's data?
2. Which of them are exposed to the internet?
3. If one service goes down, what goes down with it?
4. How do we prioritise what to patch or take offline first?
5. And what fails immediately, rather than eventually?

## The data

| Component | Fields |
|---|---|
| Service | `service_id`, `service_name`, `internet_facing`, `data_tier`, `data_tier_weight` |
| Dependency | `service_id`, `depends_on_service_id` |

The data here is **synthetic** — 40 services and 68 dependencies — and deliberately small so the graph stays readable.

## The model

A directed graph. Nodes are services, carrying exposure and data tier. Edges are dependencies: an edge from A to B means **A depends on B**.

The method is infrastructure-agnostic. It applies to any environment that can be described as components depending on components — cloud, on-premises, or hybrid.

## The method

Risk is **exposure × impact** throughout. What changes between versions is how completely impact is measured.

| Version | Impact is | Result |
|---|---|---|
| **v1** | the service's own data tier | The conventional assessment |
| **v2** | plus how many services depend on it | 5 HIGH, 1 MEDIUM, 34 LOW |
| **v3** | plus how critical those dependents are | 5 HIGH, 35 LOW |

Internet-facing services carry an exposure factor of 1.0; internal services carry 0.3 rather than 0, because an attacker already inside can reach them. That is a stated assumption, and a single number anyone can change.

**Risk concentration** is computed separately — the direct dependents only, weighted by what they hold. It answers what fails *immediately* rather than eventually, and is deliberately not folded into the risk score.

## What the analysis found

**The ranking changes substantially once structure is included.**

| Service | Data tier | Impact | Rank v1 | Rank v3 |
|---|---|---|---|---|
| configuration-service | 2  | 101 | 29 | **2** |
| event-bus | 2  | 101 | 29 | **2** |
| inventory-service | 2  | 14 | 29 | **12** |

**Classifying data tells you what a service holds. It tells you nothing about what you lose when it fails.**

## Validation

The analysis uses a graph library. The notebook recomputes the same answers by a different route — filtering the dependency table and recursing, with no graph functions at all. All 40 services match.

The check is also around 130× slower, because it filters the whole table at every step where the graph has the relationships indexed.

## Assumptions and limitations

- **Likelihood is treated as certain.** This is modelled with the AI threat landscape in mind: AI is shortening the window between a vulnerability being discovered and being exploitable. As that window narrows, likelihood moves towards certainty — so the useful question becomes consequence, and consequence is computable from structure.
- **The dependency data is assumed complete.** The analysis is only as good as it is.
- **Edges are binary.** A weak dependency counts the same as a strong one.
- **Security controls are not modelled**, deliberately: the question is what happens regardless of whether they hold.

Open limitations are tracked as [issues](../../issues).

## Running it

```
pip install -r requirements.txt
```

Open `blast-radius-risk-model.ipynb` and run all cells in order.

## Repository

```
├── README.md
├── requirements.txt      
├── blast-radius-risk-model.ipynb    the analysis
├── services.csv
└── dependencies.csv
```
