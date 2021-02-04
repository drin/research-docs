# Code and Repositories

## xhcaDB

The repository [xhcadb][repo-xhcadb] is checked out to:
```bash
    ${zfs-datadir}/code/xhcadb
```

The repository layout is (roughly) as follows:

```bash
    >> tree -L 1 -C
    .
    ├── poetry.lock
    ├── pyproject.toml
    ├── README.md
    ├── R
    ├── resources
    ├── shell-scripts
    ├── sql
    ├── tests
    ├── toolbox
    └── xhcadb

    7 directories, 3 files
```

#### Shell scripts

The shell-scripts directory is a bit chaotic, but has the following layout:
```bash
    >> tree -L 1 -C
    .
    ├── convert-simulated-data.fish
    ├── extract
    ├── infrastructure
    ├── load
    ├── perf
    └── transform

    5 directories, 5 files
```

All shell scripts have an appropriate [shebang][wiki-shebang] to be explicit, but I have coalesced
automation scripts on [bash shell][tool-bash] (instead of [fish shell][tool-fish]).

**Extract, transform, and load functions (ETL).** 

**Database and schema creation.** The bash script `infrastructure/create-xhcadb` will complete 3
tasks to create the database and define the database schema: (1) drop the database if it exists,
(2) create the database, and (3) use `psql` to run `sql/ddl/create_schema.sql`.

**Dataset loading.** A dataset represents a gene expression matrix and consists of three types of
data: (1) cell metadata, (2) gene metadata, and (3) gene expression matrix data. The [MTX file
format][format-mtx] contains a file for each of these data types, whereas database bulk load
utilities tend to best support a single TSV (tab-separated values) per schema. Dataset loading
assumes that the data has already been extracted into a bulk-loadable TSV format, and the
appropriate directory is checked for the file. The `load/load-datasets` bash script implements
logic for loading datasets by calling `load/pgsql-copy.genes`, `load/pgsql-copy.cells-clusters`,
and `load/pgsql-copy.expression` scripts. Because these data types are loaded into separate tables,
the only requirement (assuming foreign keys are enabled) is to load gene and cell metadata before
expression data. If all foreign keys are disabled at the time of load, then this constraint is not
enforced.


**Top-level Scripts.** There are a few scripts that provide functionality by calling other scripts.
The important "top-level" scripts include: `create-xhcadb.fish` and `load/load-datasets`. When
using bash, `load/load-datasets` loads cell annotations, gene annotations, and expression matrix
data for a dataset into an existing database. To create a database from scratch, use the
`infrastructure/db.create.bash` script which takes a database name and runs: (1) dropdb (if db
exists), (2) createdb, and (3) create schema (`psql ... < sql/create_schema.sql`). Note that the
schema load step directly runs a SQL script against the database, rather than invoking another
shell script.

`create-xhcadb.fish` should illustrate how to create the database and load it. Note that I have not setup the database server yet, and the `extract`, `transform`, `load`, and `insfrastructure` directories have bash scripts instead of fish scripts.
simulated-data/ has only 1 dataset (which I am calling "538-celltypes"), but there are various directories and file formats that are produced/used by the ETL process
toolbox/ will contain scripts or binaries, and at the moment has a bash script I use to help manage my local postgres databases, "experimentdb"
This script needs to be adapted for the cloudlab machine
the "bulk load" format are just simple TSVs but they're formatted to be loadable using COPY
I have postgresql 13 installed, cuz my home computer is archlinux (rolling releases)
I use pyenv to manage python interpreters. I am installing 3.9.1 right now and will install 3.7.3 afterwards
I use poetry for dependency management. I have heard it's just okay, but I haven't decided to switch to anything else yet.
I have installed Fish shell (in case you use it), but I will be trying to have shell scripts written in bash so that you don't have to learn fish (plus I'm used to scripting in bash now).

#### Sample Data
You can actually play around with some of the loading scripts and use the data in the resources directory:
https://gitlab.com/xhca/xhcadb/-/tree/develop/resources/simulated-data

Note that the data in that directory is checked in using git LFS. I just tried doing a fresh checkout, and it'll automatically checkout those files from LFS. If you need to re-download a file from LFS you can retrieve it with
`git lfs checkout -- <file>`
and as long as you aren't adding any other files to LFS you probably don't need to know any other commands. Otherwise, docs here if you're interested: https://www.mankier.com/package/git-lfs


<!-- resources -->
[wiki-shebang]: https://en.wikipedia.org/wiki/Shebang_(Unix)

[tool-fish]:    https://fishshell.com/docs/current/index.html
[tool-bash]:    https://www.gnu.org/software/bash/manual/bash.html

[format-mtx]:   https://math.nist.gov/MatrixMarket/formats.html

[repo-xhcadb]:  https://gitlab.com/xhca/xhcadb
