# automigrate
Automate migration of Oracle databases into PDB

OVERVIEW
--------
This project aims to simplify the process of migrating Oracle databases into a target Multitenant architecture (in 2020 this would be version 19, the terminal release).
Database migrations are often a complex orchestration of multiple different tasks carried out on the source and target systems. Optimal migration will depend on a number of factors, including source database version, database size,  availability requirements as well as any cross platform requirements and network capacity constraints.

This utility simplifies the process by applying the optimal migration method for the given source database, whilst automating the data transfer process through intra-database communication. Where the business demands minimal application downtime, the utility can transfer large databases over an extended period during which the application remains fully available.

BACKGROUND
----------
Starting with v20, Oracle will stop all further development of the NON-CDB architecture and have announced that NON-CDB will be de-supported in a future release. The CDB architecture, however, represents a radical departure from NON-CDB and many sites have cautiously stayed with their familiar NON-CDB databases. Compounding the problem, many other sites continue to maintain old database versions that are well past their extended support date. In order to encourage their clients to change, Oracle now (since 2020) permit 3 PDBs to run license-free per v19 CDB - n.b the Multitenant option costs about 12000 USD per core per year and allows up to 

This project was motivated by the need to rapidly and consistently migrate some 40 Production Oracle databases, versions 10 through 12 running on AIX to v19 PDBs running on Linux RHEL. Manually migrating this many databases 

; the announcement by 

TECHNICAL DESCRIPTION
---------------------
With the cross platform transportable tablespace feature becoming available starting with v10, Oracle database migration has become a simpler process, if only in the sense that efficient migration choices became more limited. The advent of Datapump in v9 to replace the antiquated exp/imp utilities has further simplified the available toolset, particularly with the network link option.

What this means is that unless you want to change the database characterset, usually you can now migrate by moving physical tablespace datafiles instead of logically exporting/importing table rows between source and target. However, this addresses only the need to migrate data segments (tables, indexes, clusters); it does not include metadata like PLSQL packages, views, sequences and other schema objects.

Starting with v 11.2.0.3 it is possible to migrate both tablespaces and metadata in a single datapump operation. This is called Transportable Database and is a tremendous facility. Prior to v 11.2.0.3 there is Transportable Tablespace which covers the migration of data segments but does not cater for metadata. Both migration methods start by placing source application tablespaces into read only mode. Often the biggest constraint in these migrations is the time spent transferring the datafiles to the soure database server. Particularly where network capacity is limited, transporting terabytes of data can take days to complete during which time applications hosted on the source database remain read only and unavailable.

To resolve this problem, the data can be transported by a process of restoring source tablespace datafiles on the target and continuously applying incremental backups; during this "recovery" period the source tablespaces remain fully online and available. Depending on how frequently incremental backups are taken and how much data is generated by the source applications, a final backup will be a relatively small amount of data, taking a proportionally small amount of time to be rolled forward into the target database. To ensure consistency between source and target, the final backup needs to be taken when the application tablespaces are set to read only - this would be when downtime starts.

In this way, all migrations from versions 10, 11, 12 and beyond are performed using the same transportable tablespace feature. Where downtime must be minimized, we have the option to transfer datafiles as a process rather than a one-off operation; whilst this unavoidably complicates the migration, it represents only a small addition to the code base, since it only effects the data transfer process. The rest of the integration process remains identical, since the only prerequisite is that all original tablespace data files have been fully transferred and are read only.
