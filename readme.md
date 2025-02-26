# Mermerd

Create [Mermaid-Js](https://mermaid-js.github.io/mermaid/#/entityRelationshipDiagram) ERD diagrams from existing tables.

# Contents

<ul>
  <li><a href="#installation">Installation</a></li>
  <li><a href="#features">Features</a></li>
  <li><a href="#why-would-i-need-it--why-should-i-care">Why would I need it / Why should I care?</a></li>
  <li><a href="#how-does-it-work">How does it work</a></li>
  <li><a href="#parametersflags">Parameters/Flags</a></li>
  <li><a href="#global-configuration-file">Global configuration file</a></li>
  <li><a href="#use-a-predefined-run-configuration-eg-for-cicd">Use a predefined run configuration (e.g. for CI/CD)</a></li>
  <li><a href="#example-usages">Example usages</a></li>
  <li><a href="#connection-strings">Connection strings</a></li>
  <li><a href="#how-can-i-writeupdate-mermaid-js-diagrams">How can I write/update Mermaid-JS diagrams?</a></li>
  <li><a href="#how-does-mermerd-determine-the-constraints">How does mermerd determine the constraints?</a></li>
  <li><a href="#roadmap">Roadmap</a></li>
</ul>

## Installation

Just head over to the [Releases](https://github.com/KarnerTh/mermerd/releases) page and download the right executable
for your operating system. To be able to use it globally on your system, add the executable to your path.

## Changes from 0.0.2 to 0.0.3

* `.mermerd` configuration file is not automatically created on first use anymore
* the parameter for the connection string suggestions (previously `connectionStrings`) was renamed
  to `connectionStringSuggestions`
* the flag `-ac` was replaced with `--showAllConstraints`

## Features

* Supports PostgreSQL and MySQL
* Select from available schemas
* Select only the tables you are interested in
* Show only the constraints that you are interested in
* Interactive cli (multiselect, search for tables and schemas, etc.)
* Use it in CI/CD pipeline via a run configuration

## Why would I need it / Why should I care?

Documenting stuff and keeping it updated is hard and tedious, but having the right documentation can help to make the
right decisions. Mermerd was designed to be able to export an existing database schema in a format that can be used to
prototype and plan new features based on the existing schema. The resulting output is an ERD diagram
in [Mermaid-Js](https://mermaid-js.github.io/mermaid/#/entityRelationshipDiagram) format that can easily be updated and
extended.

## How does it work

1. Specify the connection string (via parameter or interactive cli)
2. Specify the schema that should be used (via parameter or interactive cli)
3. Select the tables that you are interested in (multiselect, at least 1)
4. Enjoy your current database schema in Mermaid-JS format

https://user-images.githubusercontent.com/22556363/149669994-bd5cfd8d-670c-4f64-9fe9-4892866d6763.mp4

## Parameters/Flags

Some configurations can be set via command line parameters/flags. The available options can also be viewed
via `mermerd -h`

```
  -c, --connectionString string   connection string that should be used
  -h, --help                      help for mermerd
  -o, --outputFileName string     output file name (default "result.mmd")
      --runConfig string          run configuration (replaces global configuration)
  -s, --schema string             schema that should be used
      --showAllConstraints        show all constraints, even though the table of the resulting constraint was not selected
      --useAllTables              use all available tables
```

If the flag `--showAllConstraints` is provided, mermerd will print out all constraints of the selected tables, even when
the resulting constraint is not in the list of selected tables. These tables do not have any column info and are only
present via their table name.

## Global configuration file

Mermerd uses a yaml configuration file in your home directory called `.mermerd` (needs to be created, an example is
shown below). You can set options that you want by default (e.g. enabling the `showAllConstraints` for all runs) or
provide connection string suggestions for the cli.

```yaml
showAllConstraints: true
outputFileName: "my-db.mmd"

# These connection strings are available as suggestions in the cli (use tab to access)
connectionStringSuggestions:
  - postgresql://user:password@localhost:5432/yourDb
  - mysql://root:password@tcp(127.0.0.1:3306)/yourDb
```

## Use a predefined run configuration (e.g. for CI/CD)

You can specify all parameters needed for generating the ERD via a run configuration. Just create a yaml file (example
shown below) and start mermerd via `mermerd --runConfig yourRunConfig.yaml`

```yaml
# Connection properties
connectionString: "postgresql://user:password@localhost:5432/dvdrental"
schema: "public"

# Define what tables should be used
useAllTables: true
# or
selectedTables:
  - city
  - customer

# Additional flags
showAllConstraints: true
outputFileName: "my-db.mmd"
```

## Example usages

```bash
# all parameters are provided via the interactive cli
mermerd

# same as previous one, but show all constraints even though the table of the resulting constraint was not selected
mermerd --showAllConstraints

# ERD is created via the provided run config
mermerd --runConfig yourRunConfig.yaml

# specify all connection properties so that only the table selection is done via the interactive cli
mermerd -c "postgresql://user:password@localhost:5432/dvdrental" -s public

# same as previous one, but use all available tables without interaction
mermerd -c "postgresql://user:password@localhost:5432/dvdrental" -s public --useAllTables
```

## Connection strings

Examples of valid connection strings:

* `postgresql://user:password@localhost:5432/yourDb`
* `mysql://root:password@tcp(127.0.0.1:3306)/yourDb`

## How can I write/update Mermaid-JS diagrams?

* All information can be found here: [Mermaid-JS](https://mermaid-js.github.io/mermaid/#/entityRelationshipDiagram)
* I also recommend using an IDE with an Mermaid-JS extension,
  e.g. [VS Code](https://marketplace.visualstudio.com/items?itemName=tomoyukim.vscode-mermaid-editor)

## How does mermerd determine the constraints?

The table constraints are analysed and interpreted as listed:

| Nr. | Constraint type                        | Criteria                                                                 |
|-----|----------------------------------------|--------------------------------------------------------------------------|
| 1   | <code>a &#124;o--&#124;&#124; b</code> | If table a has a FK to table b and that column is the only PK of table a |
| 2   | <code>a }o--&#124;&#124; b</code>      | Same as 1, but table a has multiple PK                                   |
|     |                                        | Same as 1, but the FK is not a PK                                        |
| 3   | <code>a }o--o&#124; b</code>           | Same as 2, but the FK is nullable                                        |

## Roadmap

* [ ] Unit tests
* [ ] Support `}o--o|` relation (currently displayed as `}o--||`)
* [ ] Take unique constraints into account
* [ ] Support ERD Attributes for FK and PK
