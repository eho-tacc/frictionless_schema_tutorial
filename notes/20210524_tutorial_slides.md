# Frictionless Schema Tutorial

## Agenda

1. Assemble a simple Data Package in Frictionless
    - Emphasis on schemas
    - Interruptions are welcome!
2. Review misc. useful features of Frictionless
3. BYOD (Bring Your Own Dataset) one-on-one assistance

## Resources to Follow Along

- Repository for this tutorial: https://github.com/eho-tacc/frictionless_schema_tutorial
- Frictionless Framework documentation: https://framework.frictionlessdata.io/
- Ethan's Frictionless prototypes of Data Converge outputs: https://github.com/eho-tacc/dc_frictionless_prototypes
- Ethan's Frictionless prototypes of Versioned Datasets: https://github.com/eho-tacc/vdr_frictionless_prototypes

## Setting Up Tutorial Repo

```bash
git clone https://github.com/eho-tacc/frictionless_schema_tutorial.git
cd frictionless_schema_tutorial
git checkout step-0

# ...quality of life
alias f=frictionless
```

## Iterative Data Validation

Approach is similar to test-driven development (TDD) for software

1. Add data or enrich schema
2. Test (`f validate vdr.table.yaml`) fails
3. Transform data
4. `f validate vdr.table.yaml` passes
5. Repeat

## Anatomy of a Data Package

Minimum viable Data Package contains just two files: data and metadata
```bash
cat vdr.table.yaml
cat data/stab_scores.csv
```

## Write a Simple Schema (`step-1`)

## Adding Tabular Relations (`step-2`)

## Adding a Simple Constraint (`step-3`)

# Frequent Pain Points

## Data Resource is missing a field

## Data Resource column order is different from schema's

## Data Resource column order is different from schema's

# Misc. Useful Features

## JSON instead of YAML

## Python API




