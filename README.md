# automigrate
Automate migration of non-CDB Oracle databases to Multitenant (CDB/PDB) architecture.
- Tested on source database versions 10.1, 10.2, 11.2, 12.1, 12.2, 18.3
- Tested on target database versions 19.7, 19.8 

OVERVIEW
--------
Migrating to a new version of Oracle database is invariably a costly and risky affair, which most organizations avoid for as long as possible. However, at the time of writing (July 2020) there are a number of factors that make it increasingly incumbent on Oracle customers to migrate to the current terminal release - version 19:
- Oracle no longer supports the non-Multitenant architecture, i.e. non-CDB, as of version 20
- Oracle offers 3 PDBs per version 19 CDB license-free
- version 19 is supported until 2027
- through sharing infrastructure resources, adoption of Multitenant significantly lowers cost of ownership
- version 19 enables limited use of features like in-Memory at no extra license cost

Essentially, all pre-19 databases need to be migrated before these fall out of support (e.g. version 11.2.0.4 ends Premier support December 2020).   

Where the business demands minimal application downtime, the utility can transfer large databases over an extended period during which the application remains fully available.

BACKGROUND
----------
Starting with v20, Oracle will stop all further development of the NON-CDB architecture and have announced that NON-CDB will be de-supported in a future release. The CDB architecture, however, represents a radical departure from NON-CDB and many sites have cautiously stayed with their familiar NON-CDB databases. Compounding the problem, many other sites continue to maintain old database versions that are well past their extended support date. In order to encourage their clients to change, Oracle now (since 2020) permit 3 PDBs to run license-free per v19 CDB.

TECHNICAL DESCRIPTION
---------------------
With the cross platform transportable tablespace feature becoming available starting with v10, Oracle database migration has become a more efficient process; copying physical database data files will always be quicker than logically exporting / importing rows of data.

The advent of Datapump, particularly with the network link option, plus the DBMS_FILE_TRANSFER utility which automatically converts to target database endianness, together with the Transportable Database feature available starting 11.2.0.3 have all enabled database migrations to be more streamlined. Potentially.

The issue is often one of making the right choice for the situation at hand. Even experienced DBAs can be unaware of modern migration methods, preferring trusted approaches even when these are manifestly no longer appropriate. For example, I've seen a highly respected DBA recently build a suite of complex application metadata extract / import scripts, unaware that starting with 11.2.0.3 there is an option in DATAPUMP to migrate both data and metadata at the same time by specifying just 2 parameters - i.e. `impdp TRANSPORTABLE=ALWAYS FULL=YES` ... replacing 1000's of lines of complex code to migrate an application's views, PL/SQL packages, database links, triggers etc. And then there are grants to SYS-owned objects which are not exported by DATAPUMP - these need to be migrated as well if application code is referencing objects like SYS.V$SESSION for example.

Application owners like to "see" a guarantee that their databases have been fully migrated. We therefore need to include a comprehensive reconciliation process in the migration. What about statistics? Should we export these? Or re-gather them in the migrated database? What about fixed table stats? Dictionary stats? And what about database links? These will be imported but probably won't work if the connectivity is not enabled in the target database - application owners need to "see" the application database links - maybe they aren't needed anymore? Maybe they should be replaced by some other data transfer technique, especially if the migration is to Cloud infrastructure. In all cases, the migration not only has to succeed technically, but must provide an in-depth report of where post-migration tasks need to be carried out. What about loosely-coupled object references - i.e. where database link references are included in dynamic SQL within application code? 

Of course, migrated directory objects need to be reviewed for much the same reasons as database links. And then we have potential issues about setting transported tablespaces to their pre-migration status. Suppose we migrated a 100TB read only tablespace - we won't be very happy if this gets backed up by our RMAN scripts after the migration .... and yet, we have to remember that Transportable Database automatically sets all migrated tablespaces to READ WRITE.

The point is, migrations are complex, involve a lot of steps, and we haven't even discussed pre-11.2.0.3 options (hint: even more complex). Beyond the technical complexities involved, it may well be necessary to constrain a migration to a limited time frame of application downtime. The utility is based on the Transportable Tablespace technique which unavoidably requires that source tablespaces be set to read only *before* they are copied to the target system. However, if the source database is 10TB in size and our effective network bandwith is 100GB/hour, the business may not be able to tolerate 4 days' application downtime needed to transport the data. To resolve this problem, the utility can transport data over an extended period during which the application remains fully online and available. It does this be taking incremental backups of file image copies that are automatically transferred to the target system where they are applied according to a frequency that minimizes the final backup elapsed time.

In this way, all migrations starting from version 10 can be automated using at most a total of 3 source and target interventions depending on whether data is transferred by incremental backup. The utility guarantees that each migration is optimal for the source database version and uses a minimum of resources and intrusion. Most importantly, each migration is consistent and complete. It is noted that I.T. management, being inherently risk-averse, will always prefer an Oracle-supported solution - indeed, you can obtain from Oracle MOS a collection of files and scripts to enable database file transfer by applying incremental backup - however, these require no small amount of effort to install and involve more steps where cross platform migration is involved. By contrast, this utility provides enhanced functionality within a single SQL script.
