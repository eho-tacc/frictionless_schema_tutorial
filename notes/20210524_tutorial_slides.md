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

## Iterative Data Validation

Approach is similar to test-driven development (TDD) for software

1. Add data or enrich schema
2. Test (`f validate vdr.table.yaml`) fails
3. Transform data
4. `f validate vdr.table.yaml` passes
5. Repeat

## Install Frictionless CLI

```bash
python3 -m pip install --user frictionless
frictionless --help
```

## Setting Up Tutorial Repository

```bash
git clone https://github.com/eho-tacc/frictionless_schema_tutorial.git
cd frictionless_schema_tutorial
git checkout step-0
```

## Quality of Life Commands

```bash
alias f=frictionless
```

"What did he do on that slide again?..."

```bash
git diff 'step-0..step-1'
```

## Anatomy of a Data Package

Minimum viable Data Package contains just two files: data and metadata

```bash
cat data/stab_scores.csv
cat vdr.table.yaml
```

## Writing a Simple Schema (`step-1`)

```yaml
# ...
resources:
  - name: stab_scores
    # ...
    schema:
      fields:
        - name: dataset
          type: string
          description: Name of the dataset
        - name: name
          type: string
          description: Name of the protein design
        - name: stabilityscore
          type: number
          description: Experimental stability score of the protein
```

## Adding another Data Resource (`step-2`)

```bash
cat data/trypsin_counts.csv
```

```yaml
# ...
resources:
  - name: stab_scores
    # ...
  - name: trypsin_counts
    path: data/trypsin_counts.csv
    # ...
```

## `step-2` continued

We copy the schema from the `stab_scores` resource as a starting point:

```yaml
# ...
resources:
  - name: stab_scores
    # ...
  - name: stab_scores
    path: data/trypsin_counts.csv
    # ...
    schema:
      fields:
        - name: dataset
          type: string
          description: Name of the dataset
        - name: name
          type: string
          description: Name of the protein design
        - name: stabilityscore
          type: number
          description: Experimental stability score of the protein
```

## `step-2` continued

As expected, this schema fails validation (`f validate vdr.table.yaml`), and we modify the schema to accommodate the new fields:

```yaml
# ...
resources:
  - name: stab_scores
    # ...
  - name: stab_scores
    path: data/trypsin_counts.csv
    # ...
    schema:
      fields:
        - name: dataset
          type: string
          description: Name of the dataset
        - name: name
          type: string
          description: Name of the protein design
        - name: counts0_t
          type: number
          description: FACS counts at lowest concentration of trypsin
        - name: counts1_t
          type: number
          description: FACS counts at lowest concentration of trypsin
        # ...
```

The modified schema passes validation:

```bash
$ f validate vdr.package.yaml
# -----
# valid: data/stab_scores.csv
# -----
# -----
# valid: data/trypsin_counts.csv
# -----
```

## Adding Tabular Relations (`step-3`)

Two steps involved in defining a relationship:

1. Add `primaryKey` for both tables
2. Map `primaryKey` in the lookup table (`stab_scores` here) to a `foreignKey` in the other table (`trypsin_counts`)

## Adding a Simple Constraint (`step-4`)

# Frequent Pain Points

## Data Resource is missing a field (`step-5`)

## Data Resource column order is different from schema's (`step-6`)

# Misc. Useful Features

## JSON instead of YAML

Instead of `*.package.yaml`, define Package in `datapackage.json`:

```bash
jq . datapackage.json
# or
cat datapackage.json
```

## Validate via hash

## Python API

The [Python API](https://framework.frictionlessdata.io/docs/tutorials/working-in-python) supports any entity or operation mentioned thus far:

| Entity/operation  | Python API equivalent | `frictionless` CLI equivalent |
| ------------- | ------------- | ------------- |
| Describe  | [`describe`](https://framework.frictionlessdata.io/docs/guides/describing-data) | `frictionless describe` |
| Validate  | [`validate`](https://framework.frictionlessdata.io/docs/guides/validation-guide#validation-report) | `frictionless validate` |
| Extract  | [`extract`](https://framework.frictionlessdata.io/docs/guides/extracting-data) | `frictionless extract` |
| Data Package  | [`Package`](https://framework.frictionlessdata.io/docs/guides/framework/package-guide) | N/A |
| Data Resource  | [`Resource`](https://framework.frictionlessdata.io/docs/guides/framework/resource-guide) | N/A |
| Schema  | [`Schema`](https://framework.frictionlessdata.io/docs/guides/framework/schema-guide) | N/A |
| Field  | [`Field`](https://framework.frictionlessdata.io/docs/guides/framework/field-guide) | N/A |




