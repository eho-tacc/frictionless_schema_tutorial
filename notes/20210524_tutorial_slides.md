# Frictionless Schema Tutorial

## Agenda

1. Assemble a simple Data Package in Frictionless
    - Emphasis on schemas
    - Interruptions are welcome!
2. Review misc. useful features of Frictionless
3. BYOD (Bring Your Own Dataset) one-on-one assistance

## Resources

- Repository for this tutorial: https://github.com/eho-tacc/frictionless_schema_tutorial
- Frictionless Framework documentation: https://framework.frictionlessdata.io/
- Ethan's Frictionless prototypes of Data Converge outputs: https://github.com/eho-tacc/dc_frictionless_prototypes
- Ethan's Frictionless prototypes of Versioned Datasets: https://github.com/eho-tacc/vdr_frictionless_prototypes

## Iterative Data Validation

Approach is similar to test-driven development (TDD) for software

1. Add data or enrich schema
2. Test (`frictionless validate vdr.package.yaml`) fails
3. Transform data
4. `frictionless validate vdr.package.yaml` passes
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

"I fell behind; how do I catch up quickly?"

```bash
git checkout step-4
```

## Anatomy of a Data Package

Minimum viable Data Package contains just two files:

1. Data file: `data/stab_scores.csv`
2. Metadata file: `vdr.package.yaml`
    - `stab_scores` resource was inferred using `f describe data/stab_scores.csv`

## Writing a Simple Schema (`step-1`)

- Frictionless schemas allow one to validate data files such as `stab_scores.csv` against the metadata in `vdr.package.yaml`.
- We can let Frictionless infer a schema using `f describe`, or write one by hand in the `vdr.package.yaml` file:

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

We can now validate our package against this schema using:

```bash
$ f validate vdr.package.yaml
# -----
# valid: data/stab_scores.csv
# -----
```

Since the schema defined in the metadata matches the data file, validation passes.

## Adding another Data Resource (`step-2`)

Let's add another Data Resource to this Data Package. Specifically, we want to add some FACS count data that is contained in another CSV file:

```bash
cat data/trypsin_counts.csv
```

Instead of writing the schema by hand as we did before, we will let Frictionless generate it for us:

```bash
$ f describe data/trypsin_counts.csv
# --------
# metadata: data/trypsin_counts.csv
# --------

encoding: utf-8
format: csv
hashing: md5
name: trypsin_counts
path: data/trypsin_counts.csv
profile: tabular-data-resource
schema:
  fields:
    - name: dataset
      type: string
    - name: name
      type: string
    - name: counts0_t
      type: number
    - name: counts1_t
      type: number
    - name: counts2_t
      type: number
    - name: counts3_t
      type: number
scheme: file
```

...and paste the output into the `resources` section of `vdr.package.yaml`:

```yaml
resources:
  - name: stab_scores
    # ...
  - name: trypsin_counts
    path: data/trypsin_counts.csv
    # ...
```

Finally, we check that the Data Package passes validation with `f validate vdr.package.yaml`

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
$ f validate vdr.package.yaml
# ...
constraint-error  The cell "EEHEE_rd1_0043" in row at position "2" and field "name" at position "2" does not conform to a constraint: constraint "pattern" is "^.*sequence.*$"
constraint-error  The cell "534.0" in row at position "3" and field "counts2_t" at position "5" does not conform to a constraint: constraint "maximum" is "300.0"             
constraint-error  The cell "359.0" in row at position "7" and field "counts2_t" at position "5" does not conform to a constraint: constraint "maximum" is "300.0" 
```

## Exercise: explore validation failure modes

1. Add a new field to the schema that does not exist in the data
2. Add a new field in the data that does not exist in the schema
3. Define a `number` type for values that are `string`s in the data
4. Define a failing `constraint` similar to `step-3` above

_et cetera, et cetera_

## Adding Tabular Relations (`step-4`)

Two steps involved in defining a relationship:

1. Add `primaryKey` for the referenced table (`stab_scores` here)
2. Map `primaryKey` in the referenced table to a `foreignKey` in the referencing table (`trypsin_counts`)

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
      foreignKeys:
        - fields: [dataset, name]
          reference:
            resource: stab_scores
            fields: [dataset, name]
```

As usual, we check for issues, such as uniqueness of `primaryKey` in this case, using the `validate` command.

## `step-4` continued: comments on `foreignKeys`

- We only need to define `foreignKeys` for one Data Resource, not both
- `trypsin_counts` will "look up" every value for `dataset` and `name` in `stab_scores`
- Therefore, foreign key values in `stab_scores` must be a superset of values in `trypsin_counts`

# Common Pain Points

## Appending data to a Data Resource (`step-5`)

The following Package definition would concatenate two CSV files. This allows one to describe multiple physical files using a single schema. In this case, we want to add more stability scores to the `stab_scores` resource:

```yaml
resources:
  - name: stab_scores
    path:
      - data/stab_scores.csv
      - data/more_stab_scores.csv
```

## `step-5` continued

This fails validation and provides some informative error messages:

```bash
$ f validate vdr.package.yaml
# -------
# invalid: ['data/stab_scores.csv', 'data/more_stab_scores.csv']
# -------

===  =====  =================  ==================================================================================
row  field  code               message                                                                           
===  =====  =================  ==================================================================================
  8      3  type-error         Type error in the cell "EEHEE" in row "8" and field "stabilityscore" at position "3": type is "number/default"
  8         primary-key-error  Row at position "8" violates the primary key: the same as in the row at position 2                            
  9      3  type-error         Type error in the cell "EEHEE" in row "9" and field "stabilityscore" at position "3": type is "number/default"
===  =====  =================  ==================================================================================
```

## `step-5` continued

If we look at the physical CSV file, we can see why:

```bash
$ cat data/more_stab_scores.csv
dataset,name,sequence
TwoSix_100K,EEHEE_rd1_0043,EEHEE
TwoSix_100K,EEHEE_rd1_0043_another1,EEHEE
```

There are **three** errors here:

A. The value `EEHEE_rd1_0043` already appears in the `stab_scores` Data Resource (in `data/stab_scores.csv`), resulting in a non-unique primaryKey error. 
B. `more_stab_scores.csv` has a field `sequence` that is not defined in the schema.
C. `more_stab_scores.csv` is missing cells for field `stabilityscore`. 

## `step-5a`: Frictionless Pipelines

- We could edit the physical CSV files by hand. 
- Alternatively, Frictionless provides a [toolkit for transforming data](https://framework.frictionlessdata.io/docs/guides/basic-examples#transforming-data). 
- It allows us to write declarative, compostable data transformation pipelines:

```bash
cat pipelines/more_stab_scores.pipeline.yaml
```

Running the pipeline (and pointing our `path` to the output file) resolves the `primary-key-error`:

```bash
f transform pipelines/step_5.pipeline.yaml
cat data/more_stab_scores_transformed.csv
f validate vdr.package.yaml
```

## `step-5b`

Let us say that we are not interested in including `sequence` in the Data Package. We can remove this column by adding a `field-remove` step to our pipeline, fixing issue B above.

```yaml
- code: field-remove
  description: Remove field sequence (step-5b)
  names: 
    - sequence
```

## `step-5c`

Similarly, we can fix issue C with a `field-add` step:

```yaml
- code: field-add
  description: Add stabilityscore field (step-5c)
  name: stabilityscore
  type: number
```

# Misc. Useful Features

## JSON instead of YAML

Instead of `*.package.yaml`, define Package in `datapackage.json`:

```bash
jq . datapackage.json
# or
cat datapackage.json
```

## Validate via hash

```bash
f describe data/stab_scores.csv --stats
```

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

# Questions