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
2. Test (`f validate datapackage.json`) fails
3. Transform data
4. `f validate datapackage.json` passes
5. Repeat

# Anatomy of `datapackage.json`

```bash
cat datapackage.json
# or
jq . datapackage.json
```

