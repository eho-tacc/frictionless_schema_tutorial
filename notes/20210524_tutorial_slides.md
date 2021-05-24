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
2. Test (`frictionless validate vdr.table.yaml`) fails
3. Transform data
4. `frictionless validate vdr.table.yaml` passes
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

## Adding a Simple Constraint (`step-3`)

Schemas support a handful of field constraints, notably:

- `required`
- `unique`
- Maximum / minimum allowed values
- Regular expression matching
- Full specification at https://specs.frictionlessdata.io/table-schema/#constraints

Let's try adding a couple of (failing) constraints on fields `name` and `counts2_t`:

```yaml
- name: name
  type: string
  description: Name of the protein design
  constraints:
    pattern: '^.+sequence.+$'
# ...
- name: counts2_t
  type: number
  description: FACS counts at high concentration of trypsin
  constraints:
    maximum: 300.0
```

Validation fails as expected:

```bash
$ f validate vdr.table.yaml
# ...
constraint-error  The cell "EEHEE_rd1_0043" in row at position "2" and field "name" at position "2" does not conform to a constraint: constraint "pattern" is "^.*sequence.*$"
constraint-error  The cell "534.0" in row at position "3" and field "counts2_t" at position "5" does not conform to a constraint: constraint "maximum" is "300.0"             
constraint-error  The cell "359.0" in row at position "7" and field "counts2_t" at position "5" does not conform to a constraint: constraint "maximum" is "300.0" 
```


## Adding Tabular Relations (`step-4`)

Two steps involved in defining a relationship:

1. Add `primaryKey` for both tables
2. Map `primaryKey` in the lookup table (`stab_scores` here) to a `foreignKey` in the other table (`trypsin_counts`)

Full specification available at https://specs.frictionlessdata.io/table-schema/#primary-key

## `step-4` continued

`primaryKey`s, which must be unique and non-null, are defined in our Data Package like:

```yaml
resources:
  - name: stab_scores
    path: data/trypsin_counts.csv
    schema:
      fields:
      # ...
      primaryKey:
        - dataset
        - name
```

## `step-4` continued

Finally, we tell Frictionless that the `dataset` and `name` fields are the same in both Data Resources:

```yaml
resources:
  - name: stab_scores
    path: data/trypsin_counts.csv
    schema:
      fields:
      # ...
      primaryKey:
        - dataset
        - name
  - name: trypsin_counts
    path: data/trypsin_counts.csv
    schema:
      fields:
      # ...
      primaryKey:
        - dataset
        - name
      foreignKeys:
        fields: [dataset, name]
        reference:
          resource: stab_scores
          fields: [dataset, name]
```

## `step-4` continued

- We only need to define `foreignKeys` for one Data Resource, not both
- `trypsin_counts` will "look up" every value for `dataset` and `name` in `stab_scores`
- Therefore, foreign key values in `stab_scores` must be a superset of values in `trypsin_counts`

# Common Pain Points

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




