# Chapter 2 - PostgreSQL DB

## Overview

This chapter discusses `xhcaDB`--a reference [PostgreSQL][web-postgres] database for XHCA data.
This database is designed and implemented from scratch, to analyze single-cell expression data.
This work is done in coordination with [Josh Stuart's lab][group-jstuart], specifically with
Bianca Xue, Hongxu Ding, and Josh Stuart.

Due to this effort being done on our own for prototyping and learning, it is possible that some
research will need to be done on existing databases that address similar use cases. For timing
reasons and the need to have infrastructure that we fully control and understand, we are choosing
this route.

This database is intended to be both:
* A reference implementation
* A baseline for future performance benchmarks


## Database Schema

Our schema contains 6 entities (shown in the ERD diagram, below) which represent the core logical
entities of XHCA data and their relationships.

![Diagram for relationships between entities for single cell sequencing
data](assets/Single-Cell-ERD.png)

The ERD diagram has some color coding for convenience:
* Blue entities are primary entities (have no foreign keys)
* Green entities are used for set membership
  * `genesets` represent sets of genes
  * `clusters` represent sets of cells
* Yellow entities are logically identical matrices for core expression data
  * `expression` represents a matrix of values for each `cell` and each `gene`
  * `centroids` represents a matrix of values for each `cluster` and each `gene`, where a
    `centroid` is essentially a representative cell for the cluster.
* Red relationships are logical constraints rather than physical, meaning they are not defined as
  foreign key constraints in the DDL.
  * We want to support a one-to-many relationship between clusters and centroids: a cluster may
    have many centroids, but each centroid can only be associated with one cluster
  * In `cluster_methods`, each `centroid_id` is unique, but `cluster_id` may be duplicated


### NOTE:

Genesets is currently grey, but I think it will be made green to reflect that it is metadata for
tuples in the `geneset_membership` entity. It's closest equivalent is `cluster_methods`, as it
represents groups of genes in a way that metadata can be attached (e.g.  `geneset_group`). It is
not yet clear that it is a core entity, though it seems likely to become one.


## SQL Queries

There are a variety of queries that represent the analysis use cases we want to satisfy. We can
broadly categorize these into the following types of queries, where *many* just refers to *more
than one* (e.g. 2 is "many"):

1. scan (selection only)
2. filtration (includes many predicates in `where` clause)
3. projection (projects a proper subset of columns)
4. aggregation (includes many aggregation operations)

The first use case, scan, may not be very interesting since there are very few ways of
speeding it up. However, I list it here (until there is reason to remove it) so that we can
consider the throughput of data access for various storage architectures and devices.

We expect the common case for single-cell expression analysis to consist of some mix of the
remaining use cases.

## DBMS Tuning

References to curate later:
* [Foreign tables][repo-skyhookfdw]
* [PostgreSQL FDW docs][postgres-fdw]
* [PostgreSQL Foreign Table][postgres-foreigntable]
* [Arrow FDW for PG-strom][pgstrom-fdw]
* [PostgreSQL on ZFS][docs-postgres-zfs]
* [Tuning PostgreSQL on ZFS][docs-postgres-zfs2]
* [Blog - PostgreSQL on ZFS][blog-postgres-zfs]
* [OpenZFS - Workload Tuning][docs-openzfs]
* [ZFS on ArchLinux][docs-zfs-archlinux]
* [Ars Technica - ZFS storage][article-zfs]
* [pthree Blog - ZFS Administration][blog-zfs-admin]

<!-- Note: this likely can be moved to a blog or research-oriented writing of some sort -->
<!--
We expect the common case for single-cell expression analysis to be made up of the remaining use
cases, and we have particular techniques in mind that we believe will provide performance gains in
storage systems: adaptive physical design management, dynamic pushdowns, and speculative
materialization.
-->


<!-- Scratch
I started the postgres database (systemctl controls the default install, experimentdb should make
it easy to specifically manage the one that uses a data directory on the ZFS pool).

 -->

<!-- resources -->
[web-postgres]:          https://www.postgresql.org/
[postgres-fdw]:          https://www.postgresql.org/docs/12/sql-createforeigndatawrapper.html
[postgres-foreigntable]: https://www.postgresql.org/docs/10/sql-createforeigntable.html

[pgstrom-fdw]:           https://heterodb.github.io/pg-strom/arrow_fdw/
[docs-postgres-zfs]:     https://wiki.postgresql.org/images/8/86/PostgreSQL_on_ZFS.pdf
[docs-postgres-zfs2]:    https://pg.uptrace.dev/zfs/
[docs-openzfs]:          https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Workload%20Tuning.html
[docs-zfs-archlinux]:    https://wiki.archlinux.org/index.php/ZFS#Storage_pools
[blog-postgres-zfs]:     https://evol-monkey.blogspot.com/2017/08/postgresql-on-zfs.html
[blog-zfs-admin]:        https://pthree.org/2012/12/10/zfs-administration-part-v-exporting-and-importing-zpools/
[article-zfs]:           https://arstechnica.com/information-technology/2020/05/zfs-101-understanding-zfs-storage-and-performance/

[repo-skyhookfdw]:       https://github.com/uccross/skyhookdm-ceph/blob/skyhook-luminous/src/progly/cloudlab_postgres_bench/postgres-skyhook-example.sql

[group-jstuart]:         https://sysbiowiki.soe.ucsc.edu/
