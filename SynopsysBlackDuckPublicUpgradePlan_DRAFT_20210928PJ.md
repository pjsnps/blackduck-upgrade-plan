This document is provided as-is, without warranty or liability.

**Contents**
============

[**Contacts**](#contacts)

[**Introduction**](#introduction)

[**Status of This Document**](#status-of-this-document)

[**Background Information**](#upgrade-schedule)

[**Upgrade Schedule**](#_Toc47127850)

[**Overview**](#overview)

[**Upgrade planning**](#upgrade-planning)

[**Determining Upgrade and production requirements**](#determining-upgrade-and-production-requirements)

[**Documentation**](#documentation)

[**Checklist:**](#checklist)

[**Write post-upgrade validation test plan**](#write-post-upgrade-validation-test-plan)

[**Checklist:**](#checklist-1)

[**Document Database/API Connections**](#document-databaseapi-connections)

[**Scheduling Upgrades**](#scheduling-upgrades)

[**Opening a Synopsys SalesForce Request**](#opening-a-synopsys-salesforce-request)

[**Pre-Upgrade Activities**](#pre-upgrade-activities)

[**Performance Issues**](#performance-issues)

[**Best Practices**](#best-practices)

[**Sage**](#sage)

[**System\_check.sh**](#system_check.sh)

[**Sar**](#sar)

[**Zenoss**](#zenoss)

[**SynopsysGatherServerSpecs\_202007.bash**](#synopsysgatherserverspecs_202007.bash)

[**Resolving Performance Issues**](#resolving-performance-issues)

[**Sysbench:**](#sysbench)

[**Pgbench**](#pgbench)

[**pg\_test\_fsync:**](#pg_test_fsync)

[**bonnie++:**](#bonnie)

[**dd:**](#dd)

[**Cleaning up Black Duck Projects and Scans**](#cleaning-up-black-duck-projects-and-scans)

[**Cleaning up Databases**](#cleaning-up-databases)

[**bdio Database**](#bdio-database)

[**bds\_hub\_report Database**](#bds_hub_report-database)

[**bds\_hub, postgresql, template0, and template1 Database**](#bds_hub-postgresql-template0-and-template1-database)

[**Trimming the Notification and audit\_event logs**](#trimming-the-notification-and-audit_event-logs)

[**Notification Logs**](#notification-logs)

[**Audit\_Events**](#audit_events)

[**Database Cleanup**](#database-cleanup)

[**Remove any orphaned large objects**](#remove-any-orphaned-large-objects)

[**Run PostgreSQL tuning utility**](#run-postgresql-tuning-utility)

[**Upgrade O/S, Kernel, Docker, Postgresql**](#upgrade-os-kernel-docker-postgresql)

[**Full Vacuum of Database**](#full-vacuum-of-database)

[**Duplicating Production Database in Staging Environment**](#duplicating-production-database-in-staging-environment)

[**Production Database Replication**](#production-database-replication)

[**Duplicating Production Environment in Staging Environment**](#duplicating-production-environment-in-staging-environment)

[**Perform Black Duck Upgrade**](#perform-black-duck-upgrade)

[**Pre-upgrade steps**](#pre-upgrade-steps)

[**Check Memory**](#check-memory)

[**Download upgrade images**](#download-upgrade-images)

[**Download Orchestration Files**](#download-orchestration-files)

[**blackduck\_migrator-2020.6.1.yml**](#blackduck_migrator-2020.6.1.yml)

[**Schedule upgrade**](#schedule-upgrade)

[**Perform Upgrade, Day of Upgrade (Staging then Production)**](#perform-upgrade-day-of-upgrade-staging-then-production)

[**Stop scanning and external connections**](#stop-scanning-and-external-connections)

[**Start the actual upgrade**](#start-the-actual-upgrade)

[**Post-Upgrade Steps**](#post-upgrade-steps)

[**Run upgrade-validation tests.**](#section-3)

[**Re-run benchmark tests.**](#re-run-benchmark-tests.)

[**Announce upgrade completion to stakeholders**](#announce-upgrade-completion-to-stakeholders)

[**Contingency: Fallback Steps**](#contingency-fallback-steps)

[**Restore steps, during Fallback**](#restore-steps-during-fallback)

**Contacts**
============

1.  Customer: 

2.  Case(s): 
    - Prior related cases for reference: 

3.  Customer: 
    - Name:
    - email:
    - phone:
    - timezone:

4.  Synopsys Support Staff member(s): 
    - Name(s):  
    - email(s):
    - phone(s):
    - timezone(s):

5.  Support links: 
    - website: Synopsys Software Integrity Community: https://community.synopsys.com/s/
    - email:  software-integrity-support@synopsys.com

**Introduction**
================

**Status of This Document**
---------------------------

1.  This document is a Draft, informal, test, work in progress.
    - last updated September 28, 2021. 
    - Contact Pete Jalajas pjalajas@synopsys.com with any questions, suggestions, or corrections. 

2.  Document filename: SynopsysBlackDuckPublicUpgradePlan_DRAFT_20210928PJ.md

3.  This document has been reviewed by:
    -  
    -  

**Upgrade Schedule**
--------------------

1.  Staging: 
    - Date: 
    - From version:
    - To version:

2.  Production:
    - Only after successful completion of the upgrade on Staging, and
    resolution of any findings from that upgrade, Customer is currently
    expected to schedule upgrading Production on 
    - Date:

**Background Information**
--------------------------

1.  Customer Black Duck instances: 

    1.  Run in Docker compose stack. 
        - Number of container replicas:
        - HUB_MAX_MEMORY
        - Resource reservations and limits
            - ram
            - cpu

    2.  Database is external. 

    2.  Includes Synopsys Alert.
        - Alert database in external database.

    3.  Database disks are connected over iSCSI over 2-NIC mpath
        ethernet elsewhere in the same datacenter. 

    1.  Customer investigating Synopsys-reported infrastructure
        performance issues with their internal sysadmins, including possible proxy/firewall issues. 

2.  Customer has Staging and Production environments. Production
    environment is highly secure, with limited access. During a recent
    upgrade event, Staging underestimated Production upgrade task
    durations. With the smaller database and footprint, the Staging
    environment did not run into the same issues with the Database
    Migration scripts that the Production server did.

    - During a recent upgrade event, Customer docker container
        memory, cpu, and replica settings were misconfigured and had to
        be fixed under a P1 Case. 

3.  Customer Production bds\_hub database was recently 2.3 TB.

4.  The upgrade is planned to be performed using a simplified Upgrade
    Docker stack implemented by deploying new Synopsys-supplied
    deployment (.yml) files. These Upgrade .yml files will be different
    from the Production versions of the files.

5.  Any significant database migration steps expected?  Any other potential causes of long delays during upgrade?

6.  The upgrade is expected to take about \_\_\_\_\_ hours. 

    - If Fallback is required, an additional \_\_\_\_ hours would be
        required.

**Overview**
============

**This section provides a brief overview of the upgrade plan.**

1.  **Planning**

    1.  **Identify upgrade and production requirements**

    2.  **Write post-upgrade validation test plan**

    3.  **Inventory database connections**

    4.  **Schedule Staging upgrade**

    5.  **Open Synopsys SalesForce Case**

2.  **First, In Staging**

    6.  **Prepare Environment**

        1.  **Test and resolve performance issues**

            1.  **Implement guidance from Best Practices, Sage,
                system\_check.sh, sar, Zenoss **

            2.  **Customer IT team to test and resolve issues**

                1.  **Consider: dd, pgbench, sysbench, bonnie++**

        2.  **Remove unused data; delete, truncate/drop, cull, configure
            settings**

        3.  **Vacuum analyze full bds\_hub**

        4.  **Run PostgreSQL tuning utility**

        5.  **Backup database**

        6.  **Create Production-like Staging environment**

        7.  **Ensure disk space for db vacuum, migration**

        8.  **Check memory**

        9.  **Download upgrade-related files**

        10. **Schedule Staging upgrade**

    7.  **Perform Upgrade**

        11. **Stop scanning and external connections**

        12. **Bring down docker stack**

        13. **If not already done, backup external database with
            ~~2020.4.1~~ prior-version hub\_create\_data\_dump.sh**

        14. **Optional: upgrade OS, kernel **

        15. **Optional: upgrade Docker with yum**

        16. **Optional: satisfy other requirements, objectives. **

        17. **Deploy docker migration stack**

        18. **Restore database with ~~2020.4.1~~ prior-version
            hub\_db\_migrate.sh**

        19. **Vacuum audit\_event table**

        20. **Optional, upgrade PostgreSQL**

            3.  **Bring down docker stack**

            4.  **Upgrade PostgeSQL**

        21. **Deploy Black Duck with 2020.6.1 Production deployment .yml
            files**

    8.  **Post-Upgrade Steps**

        22. **Run upgrade-validation tests**

        23. **Re-run benchmark tests**

        24. **Announce upgrade completion to stakeholders**

3.  **Then, In Production**

    9.  **Schedule and repeat "In Staging" steps as above**

**Upgrade planning**
====================

Planning for an upgrade should occur days or weeks prior to an
upgrade. This is not about actually changing everything. It is only
about planning on what to do prior to the Black Duck upgrade.**


**Determining Upgrade and production requirements**
---------------------------------------------------

In this section, the areas that need to be checked prior to upgrading
Black Duck. This includes server setup (RAM, CPUs, Storage), O/S
versions, and application versions.

### **Documentation**

Synopsys provides the following documents to help with the
determination of what the RAM and CPUs that should be available to the
server. It should be noted that as Black Duck expands its functionality,
it may require additional memory or CPU cores. It is at this time that
the Documentation used in setting up your server should be revisited to
see if it still meets your needs.

The following documents are normally only available when the Black
Duck release is available. Your Synopsys support person may be able to
get you an advance copy. These documents include:

-   Black Duck Release Notes
    (<https://community.synopsys.com/s/article/Black-Duck-Release-Notes>)

-   Black Duck Install Guide
    (<https://community.synopsys.com/s/article/Black-Duck-Installing-Black-Duck-using-Docker-Swarm>)

-   Black Duck Scanning Best Practices
    (<https://community.synopsys.com/s/article/Black-Duck-Scanning-Best-Practices>)

As these documents are updated over time, it is important to keep up
on the changes.

Those documents are the primary resources the customer should follow;
this Upgrade Support Plan document tries to augment, not replicate,
them. If something is unclear in this upgrade document, refer to the
primary documents or ask Black Duck Support. 

In particular, Chapter 6 of the Installing Black Duck using Docker
Swarm (Upgrading Black Duck) should be reviewed. This document indicates
the supported versions of the target operating system, kernel, java,
docker, and PostgreSQL. NOTE: since certain versions of PostgreSQL are
supported in certain versions, the Black Duck server must be at that
minimum version prior to upgrading PostgreSQL. For instance, if the
Installation Guide indicates that PostgreSQL 11.7 is now supported for
external databases as part of the 2020.6.x release, you cannot upgrade
PostgreSQL to 11.7 until the Black Duck server is at that release
(minimum).

For example, as part of the 2020.6.0 release, the following are
supported (please refer to your Install Guide for the actual versions
for your release):

  |**O/S or Application**|**Version**|
  |----------------------|-----------| 
  |Red Hat Linux|7.3|
  |Docker|Community 19.03|
  |PostgreSQL (external database)|9.6.x or 11.7|

### **Checklist:**

Depending on how many projects and versions of those projects your
organization is planning on supporting, it is important to make sure
that your Black Duck server has sufficient power to support your
plans.

-   Customer to ensure all system requirements are or will be met, as
    listed in Chapter 2, Installation Planning, of the Installing Black
    Duck using Docker Swarm. 

    -   O/S Version OK? If not, when should an upgrade occur?

    -   Docker Version OK? If not, when should an upgrade occur?

    -   PostgreSQL Version OK? If not, when should an upgrade occur?

    -   RAM sufficient for the server (Check documentation and
        docker-compose.externaldb.yml or docker-compose.yml, and
        docker-compose.local-overrides.yml on the Black Duck server)?

    -   CPUs sufficient for the server (Check documentation and the
        same files as the RAM check)?

    -   Check Storage (df -hl on the Black Duck server)

-   Customer to review those documents and resolve with Synopsys any
    questions or issues discovered with Synopsys.

-   Customer needs to determine if any upgrades need to occur and if
    they should be done prior to the Black Duck upgrade.

**Write post-upgrade validation test plan**
-------------------------------------------

It is highly recommended that you have a set of regression tests that
you run when you upgrade Black Duck. If there was a hiccup that caused
database corruption, then this will help provide validation that the
upgrade worked correctly. This set of regression tests could be as
simple as verifying that the same projects are visible, that the results
of key BOMs of project-versions are OK, that admins can access license
management, policy management, or other sections.

In addition to this, there will be new functionality with the new
releases. If your organization is looking to take advantage of these
changes, then you will need to make sure that these changes fit into
your policies and procedures. Having tests to check these features will
provide validation specific to your installation.

You may have cases open with Synopsys concerning your use of the
product that are slated to be fixed in this release. RFEs you have
requested may also be part of this release. There should be some tests
to verify that these fixes and improvements work as expected in this
release.

In this section, the areas that need to be checked prior to upgrading
Black Duck. This includes server setup (RAM, CPUs, Storage), O/S
versions, and application versions.

### **Checklist:**

-   Review Regression Tests to see if changes in functionality impact
    the test cases.

    -   Make changes as necessary for target release

-   Review Release Notes for areas to test (at least both Chapters 1
    and 2, for all applicable Black Duck versions).

    -   Create Test Cases for the functionality that you will use or
        may impact your use

-   Review issues opened with Synopsys (or recent issues not tracked
    yet) and determine if any may be fixed in the target release

    -   Create Test Cases for the functionality that you will use or
        may impact your use

-   Review Scripts based on Black Duck APIs

    -   Create Test Cases to verify that interface is still working
        the same way after the upgrade

**Document Database/API Connections**
-------------------------------------

It is important to see what connections are made between the various
tools that may interact with the Black Duck Database and/or REST API.
These will be used during the upgrade to minimize impact to the database
and server.

Customer to create an inventory of all the different kinds of Black
Duck and PostgreSQL server connections from all kinds of clients, so
that they may be efficiently and promptly terminated before starting the
upgrade (guidance provided below) including:

-   External Queries using SQL or Rest API

    -   Utilities

    -   Scripts

    -   SDK Calls

-   Scan Results (Scans interacting with Black Duck Server)

-   Infrastructure Queries (e.g. server/docker monitoring)

-   Server Logins (to either Black Duck or database server (if
    external))

**Scheduling Upgrades**
-----------------------

If Synopsys Support is required, then the customer should schedule the
start of an upgrade to occur during the morning of a normal business day
(Eastern Time Zone). This request should occur at least 1 week prior to
the upgrade. This will maximize the chance and duration of Synopsys
Support help.

Should the customer require Synopsys support, they should open a case
requesting help.

### **Opening a Synopsys SalesForce Request**

You can create a SalesForce case by going to
<https://community.synopsys.com> and logging in with your Community site
username and password. If you don't have one, request one from your
sales and/or support contact.

Select Support -\> Create a Support case.

The case should be set to *P3 -- Medium*. Should any issues be
encountered during the upgrade, the Customer will update the priority of
that case to *P1 -- Critical*. Customer will then call Synopsys at
800-873-7793 to have the on-call Technical Support Engineer (TSE)
engaged.

**Pre-Upgrade Activities**
==========================

Prior to the target upgrade (be it test/staging or production), the
following activities must be performed to make sure that the server(s)
is(are) ready.

**Performance and Networking Issues**
----------------------

TODO:  be sure to elaborate on resolving networking issues, such as proxies, firewalls, tls certificates, etc, etc.

It is important that any performance issues be resolved or mitigated
prior to an upgrade in order to keep the upgrade time required
minimized. 

These are a few of the documents/tools that can help determine if
there are any issues.

### Best Practices 

(<https://community.synopsys.com/s/article/Black-Duck-Scanning-Best-Practices>)

These are the practices for project setup and scans as recommended by
Synopsys. These are updated over time and should be reviewed prior to an
upgrade. If there are any changes that apply, these should be assessed
prior to the upgrade. If a change would reduce the footprint of the
database prior to the upgrade, that should be rolled out first.

### Sage 

(<https://github.com/blackducksoftware/sage>)

This tool looks at the status of the projects in the target Black Duck
server. If there are issues flagged, then they should be resolved prior
to the upgrade:

-   Unmapped Scans

-   Projects with Too Many Versions

-   Projects with No Owners

-   Project Versions with Excessive Scans

-   Project Versions with No Scans

### System\_check.sh

(.../hub/hub\_docker/hub\_\<version\>/docker-swarm/bin/system\_check.sh)

Run on Black Duck Server and Database Server. Check to see if there
are any issues that need to be dealt with prior to the upgrades

### Sar

Check out the iowait values on the Database Server. 

06:00:01 PM CPU %user %nice %system %iowait %steal %idle

06:10:01 PM all 2.04 0.00 1.12 0.02 0.00 96.83

06:20:01 PM all 2.21 0.00 1.02 0.02 0.00 96.76

Should there be any issues, then you need to talk to your server,
network, and database admins to see what you can do to remove any
bottlenecks. Synopsys Support may be able to help.

NOTE: Should you need to look at the SAR results from the other day,
go to /var/log/sa and type "sar -f sa\#\#" where \#\# is the day of the
month.

### Zenoss

Not sure what this gives us.

### SynopsysGatherServerSpecs\_202007.bash

TODO: Pete to test and update this script if needed.This captures a
lot of system data including top, cpu, mem, application versions, etc.
It has several psql queries that need to be modified for your setup. The
end of the script does latency testing that may need to be configured if
there are any partitions you don't wish tested.

Synopsys can provide the script:

TODO:is this is a different script, needs own header, or did the
filename change?

bash

export PGPASSWORD='\<PSQL Database Password\>'

./SynopsysMonitorDbActivity\_202007.bash \>
SynopsysMonitorDbActivity\_202007.out 2\>&1

**Resolving Performance and Networking Issues**
-----------------------------------------------
TODO:  add networking, proxy, firewall and other related improvements here

Using the output of the sar/ksar command and the
SynopsysGatherServerSpecs\_202007.bash script (and possibly the Zenoss
output), determine if you have any Performance issues.

This could potentially include memory, network performance between
database server and iSCI disks (especially "read"), nfs-mounted database
directories

Get help from the Customer Admins to resolve the Performance Issues

The Server, network, and database admins should be able to help the
customer in resolving any detected performance issues. Using the data
from the previous section (e.g. sar, cpus, mem) combined with the Best
Practices and the following tools, the Customer should work with their
Admins to determine the appropriate actions:

-   Increase RAM of the Black Duck and/or Database Server (if
    needed)

-   Increase CPU Cores of the Black Duck and/or Database Server (if
    needed)

-   Make sure sufficient storage is available for database,
    application, and log partitions (if needed)

-   Make sure that the connection to the database (e.g. NFS, iSCSI,
    fibre-connected) has sufficient bandwidth to avoid iowaits, caching,
    and latency issues.

Some of the following benchmarking tools (e.g. dd, pgbench, sysbench,
or bonnie++) may be used to collect a performance baseline for
comparison with a separate run after the upgrade, in both Staging and
Production.

### sysbench

sysbench provides benchmarking capabilities for Linux. sysbench
supports testing CPU, memory, file I/O, mutex performance, and even
[MySQL](https://wiki.gentoo.org/wiki/MySQL) benchmarking.

If sysbench is not installed on the server (yum list installed \| grep
sysbench), you can install it as root by running:

```
sudo su - yum install sysbench
```

To run sysbench, run the following as root:

```
date ; date \--utc ; hostname -f ; pwd ; whoami ; nproc ; free -g ;
time for mthreads in 4 8 16 ; do echo test threads \$mthreads ; sysbench
fileio \--file-test-mode=rndrw \--threads=\$mthreads run \| grep -e
\"\\(read\\\|writ\\).\*/\" ; done
```
```
Mon Jul 27 11:17:08 EDT 2020
Mon Jul 27 15:17:08 UTC 2020
sup-pjalajas-hub.dc1.lan
/home/pjalajas/Documents/dev/customers/customer
pjalajas
8
              total        used        free      shared  buff/cache  
available
Mem:             39          12          14           2         
12          23
Swap:             1           1           0
test threads 4
    reads/s:                      7368.02
    writes/s:                     4912.01
    read, MiB/s:                  115.13
    written, MiB/s:               76.75
test threads 8
    reads/s:                      13405.58
    writes/s:                     8936.89
    read, MiB/s:                  209.46
    written, MiB/s:               139.64
test threads 16
    reads/s:                      19459.82
    writes/s:                     12973.21
    read, MiB/s:                  304.06
    written, MiB/s:               202.71
real    0m30.055s
user    0m5.440s
sys     0m13.697s
```
### Pgbench

pgbench is a simple program for running benchmark tests on
PostgreSQL.

Caution: pgbench -i creates four tables pgbench\_accounts,
pgbench\_branches, pgbench\_history, and pgbench\_tellers, destroying
any existing tables of these names. Be very careful to use another
database if you have tables having these names!

If pgbench is not installed on the server (yum list installed \| grep
pgbench), you can install it as root by running:

sudo su -

yum install postgresql-contrib

It will be dropped into /bin or /usr/bin/pgbench

To run pgbench, run the following as yourself:

Example:

-   date ; hostname ; time pgbench -h hub-stg-db -U blackduck -p 5432
    -d bds\_hub -s 10000 -c 200 -j 100 -M prepared -t 1000 2\> /dev/null
    

> Example output:
>
> Fri Jul 24 13:50:59 EDT 2020
>
> sup-pjalajas-hub.dc1.lan
>
> pghost: sup-pjalajas-2 pgport: 55436 nclients: 200 nxacts: 1000
> dbName: bds\_hub
>
> transaction type: TPC-B (sort of)
>
> scaling factor: 1
>
> query mode: prepared
>
> number of clients: 200
>
> number of threads: 100
>
> number of transactions per client: 1000
>
> number of transactions actually processed: 200000/200000
>
> tps = 439.798734 (including connections establishing)
>
> tps = 440.029375 (excluding connections establishing)
>
> real 7m34.812s
>
> user 0m23.666s
>
> sys 0m49.548s

### pg\_test\_fsync

pg\_test\_fsync is intended to give you a reasonable idea of what the
fastest
[wal\_sync\_method](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-SYNC-METHOD)
is on your specific system, as well as supplying diagnostic information
in the event of an identified I/O problem.

-   date \--utc ; hostname -f ; pg\_test\_fsync

> Fri Jul 24 19:56:41 UTC 2020
>
> sup-pjalajas-hub.dc1.lan
>
> 2 seconds per test
>
> O\_DIRECT supported on this platform for open\_datasync and
> open\_sync.
>
> Compare file sync methods using one 8kB write:
>
> (in wal\_sync\_method preference order, except fdatasync
>
> is Linux\'s default)
>
>         open\_datasync                    1828.665 ops/sec
>
>         fdatasync                        1839.337 ops/sec
>
>         fsync                             538.328 ops/sec
>
>         fsync\_writethrough                            n/a
>
>         open\_sync                         442.936 ops/sec
>
> Compare file sync methods using two 8kB writes:
>
> (in wal\_sync\_method preference order, except fdatasync
>
> is Linux\'s default)
>
>         open\_datasync                     945.146 ops/sec
>
>         fdatasync                        1461.665 ops/sec
>
>         fsync                             531.997 ops/sec
>
>         fsync\_writethrough                            n/a
>
>         open\_sync                         330.452 ops/sec
>
> Compare open\_sync with different write sizes:
>
> (This is designed to compare the cost of writing 16kB
>
> in different write open\_sync sizes.)
>
>          1 \* 16kB open\_sync write         595.042 ops/sec
>
>          2 \*  8kB open\_sync writes        229.368 ops/sec
>
>          4 \*  4kB open\_sync writes        121.647 ops/sec
>
>          8 \*  2kB open\_sync writes         83.208 ops/sec
>
>         16 \*  1kB open\_sync writes         39.609 ops/sec
>
> Test if fsync on non-write file descriptor is honored:
>
> (If the times are similar, fsync() can sync data written
>
> on a different descriptor.)
>
>         write, fsync, close               633.979 ops/sec
>
>         write, close, fsync               486.340 ops/sec
>
> Non-Sync\'ed 8kB writes:
>
>         write                           244921.142 ops/sec

### bonnie++

Bonnie++ allows you to benchmark how your file systems perform with
respect to data read and write speed, the number of seeks that can be
performed per second, and the number of file metadata operations that
can be performed per second.

-   date \--utc ; hostname -f ; time bonnie++ \# need 2 x RAM to be
    free on disk 

> Fri Jul 24 20:53:01 UTC 2020
>
> sup-pjalajas-hub.dc1.lan
>
> Writing a byte at a time\...done
>
> Writing intelligently\...done
>
> Rewriting\...done
>
> Reading a byte at a time\...done
>
> Reading intelligently\...done
>
> start \'em\...done\...done\...done\...done\...done\...
>
> Create files in sequential order\...done.
>
> Stat files in sequential order\...done.
>
> Delete files in sequential order\...done.
>
> Create files in random order\...done.
>
> Stat files in random order\...done.
>
> Delete files in random order\...done.
>
> Version  1.97       \-\-\-\-\--Sequential Output\-\-\-\-\--
> \--Sequential Input- \--Random-
>
> Concurrency   1     -Per Chr- \--Block\-- -Rewrite- -Per Chr-
> \--Block\-- \--Seeks\--
>
> Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec
> %CP  /sec %CP
>
> sup-pjalajas 80112M   668  99 993827  96 553909  59  1411  99
> 1341920  79  8210 150
>
> Latency             20730us   98013us    1412ms    8035us  
> 41587us   15082us
>
> Version  1.97       \-\-\-\-\--Sequential Create\-\-\-\-\--
> \-\-\-\-\-\-\--Random Create\-\-\-\-\-\-\--
>
> sup-pjalajas-hub.dc -Create\-- \--Read\-\-- -Delete\-- -Create\--
> \--Read\-\-- -Delete\--
>
>               files  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec
> %CP  /sec %CP
>
>                  16 +++++ +++ +++++ +++ +++++ +++ +++++ +++ +++++
> +++ +++++ +++
>
> Latency              1262us    6290us     660us    3386us    
> 228us     626us
>
> 1.97,1.97,sup-pjalajas-hub.dc1.lan,1,1595640930,80112M,,668,99,993827,96,553909,59,1411,99,1341920,79,8210,150,16,,,,,+++++,+++,+++++,+++,+++++,+++,+++++,+++,+++++,+++,+++++,+++,20730us,98013us,1412ms,8035us,41587us,15082us,1262us,6290us,660us,3386us,228us,626us
>
> real    5m9.032s
>
> user    0m56.024s
>
> sys     2m53.775s

### dd:

You can use dd to create a large file as quickly as possible to
see how long it takes. It's a very basic test and not very customisable
however it will give you a sense of the performance of the file system.
You must make sure this file is larger than the amount of RAM you have
on your system to avoid the whole file being cached in memory.

time sh -c \"dd if=/dev/zero of=\[PATH\] bs=\[BLOCK\_SIZE\]k
count=\[LOOPS\] && sync\"

A break down of the command is as follows:

-   time -- times the overall process from start to finish

-   of= this is the path which you would like to test. The path must
    be read/ writable.

-   bs= is the block size to use. If you have a specific load which
    you are testing for, make this value mirror the write size which you
    would expect.

-   sync -- forces the process to write the entire file to disk before
    completing. Note, that dd will return before completing but the time
    command will not, therefore the time output will include the sync to
    disk.

Example:

time sh -c \"dd if=/dev/zero of=/mnt/mount1/test.tmp bs=4k
count=2000000 && sync\"

2000000+0 records in

2000000+0 records out

8192000000 bytes transferred in 159.062003 secs (51501929 bytes/sec)

real 2m41.618s

user 0m0.630s

sys 0m14.998s

**Cleaning up Black Duck Projects and Scans**
---------------------------------------------

Using the output of the Sage script, you can determine what scans,
project-versions, and whole projects can be removed.

For instance, the Customer Third Party Verification process may only
require that the last 3 releases of data be maintained on a Black Duck
Server. So, prior to the upgrade, make sure that only those
project-versions related to those releases are maintained.

Unmapped scans may occur because there was an error in a scan, a scan
was manually generated without indicating a target project and version,
or a project-version linked to the scans may have been deleted
(resulting in an unmapped scan). These should be removed prior to a full
vacuum and/or an application upgrade.

**Cleaning up Databases**
-------------------------

From time-to-time, Databases used by Black Duck become obsolete. If an
upcoming release requires the removal of a database as part of the
Database Migration scripts (part of the upgrade), then this should be
explicitly tested as part of the staging testing. 

Contact your Synopsys Support to make sure you understand what might
be impacted by the database removal. Make sure your verification tests
can verify that that data migrated correctly and that the database is
removed.

Since Synopsys will tend to migrate the data from a database in one
release and then remove the database in another release (to make sure
that there are no issues prior to deleting the data), this test may need
to be done in each release.

### bdio Database

The BDIO Database was removed as part of the 2020.2.0 upgrade (???).
One of the Database Migration scripts removed that database. Should your
database be greater than 1 TB, there may be timeout issues when running
that script. Make sure you either test that migration with a copy of the
production database or make sure Synopsys Support is present during the
upgrade or both.

### bds\_hub\_report Database

In Black Duck version 2019.10.0, to make it easier for users to
quickly create, use, back up, and restore the reporting database, the
tables from the separate reporting database (bds\_hub\_report) have been
moved to the *reporting* schema, with periodically updated materialized
views, in the Black Duck database, bds\_hub.

So, as part of the 2020.6.0 release, the bds\_hub\_report database was
removed as part of the Database Migration scripts. If the Customer has
not generated any reports using the bds\_hub\_report database, the
Database Migration script to remove that database should be quick.

### bds\_hub, postgresql, template0, and template1 Database

These databases are still valid as of the 2020.6.1 release.

**Trimming the Notification and audit\_event logs**
---------------------------------------------------

Depending on the activity on the Black Duck server, the number of
records taken up by the logging could be in the millions. It is
important to trim those logs periodically and definitely prior to an
upgrade.

### Notification Logs

Notification logs are sometimes used by integration tools to trigger
alerts or the generation/update of JIRA cases so they do need to stick
around for a bit.

In the blackduck-config.env file, the Customer can indicate how long
the notifications logs should be retained, by default that duration is
30 days:

BLACKDUCK\_HUB\_NOTIFICATIONS\_DELETE\_DAYS=30

### Audit\_Events

There is no automatic cleanup of Audit Events as of 2020.6.1. These
events are required for a few days but other logging has now been added
to the Black Duck tool so these Events are no longer needed as logging.
They don't need to be kept longer than 10 days.

As such, a psql command should be run periodically to clean up the
Audit events:

delete from st.audit\_event where event\_timestamp \< now() - interval
\'10 days\';

**Database Cleanup**
--------------------

### Remove any orphaned large objects

If the Customer generates many default reports, this could result in
"orphaned" large objects
(<https://www.postgresql.org/docs/9.5/vacuumlo.html>).

The customer should run "vacuumlo" to remove any orphaned large
objects.

#### vacuumlo

vacuumlo is a simple utility program that will remove any
\"orphaned\" large objects from a PostgreSQL database. An orphaned
large object (LO) is considered to be any LO whose OID does not appear
in any oid or lo data column of the database.

Example:

Vacuumlo bds\_hub (????)

### Run PostgreSQL tuning utility

The customer should run pgtune (PostgreSQL Config Tuner) or a similar
PostgreSQL tuning utility on the PostgreSQL database. This is not
required but highly recommended. Detailed instructions are available
upon request from Synopsys support.

#### Pgtune

pgtune takes the wimpy default postgresql.conf and expands the
database server to be as powerful as the hardware it\'s being deployed
on. It is available from <https://github.com/gregs1104/pgtune/>.

Example: 

> date ; hostname ; pgtune \--type OLTP \--connections=800 -i
> /tmp/postgresql.conf -o /tmp/postgresql.conf.pgtune
>
> diff /tmp/postgresql.conf /tmp/postgresql.conf.pgtune

If this has been done as part of the database server setup, then this
does not need to be redone unless the server configuration has changed
(e.g. memory, cores, storage)

### Upgrade O/S, Kernel, Docker, Postgresql

As time passes, the minimum requirements for Black Duck changes. There
tends to be a transitional period across multiple releases where one
version is supported, then two, then just the latest.

So, most upgrades of the O/S, Kernel, Docker, and Postgresql are done
between Black Duck upgrades.

Take a look at the supported versions in the next release
(installation guide) and determine if the O/S, kernel, docker, or
PostgreSQL need to be upgraded. NOTE that some require the Black Duck to
be upgraded before you can upgrade the O/S, Kernel, Docker, or
PostgreSQL. This is because the current version may not support the new
version until after the upgrade.

#### O/S and/or Kernel upgrade

Working with your server admins for the downtime required to upgrade
the O/S and/or kernel. Sometimes the upgrade occurs because a patch
(security or bugfix) is required. Sometimes it is because Black Duck
will no longer be supporting the version you are using.

Work with your system admins to perform the upgrade.

#### Docker Upgrades

At this point, it looks like there is no "upgrade" per se. The
currently supported version is 18.03. However 19.03 of Docker is
available. So, to "upgrade", the old Docker installation would have to
be uninstalled and the new Docker installation would have to be
installed.

\<Need to test in order to verify that images and cert secrets don't
have to be reinstalled\>

#### PostgreSQL Upgrades

Work with your Database Administrators to upgrade your Postgresql
instance. 9.6.x is currently supported in both the container and
external databases. 11.7.x is supported in the external databases as of
Black Duck 2020.6. But, since 11.7.x is not supported until Black Duck
2020.6.x, you need to upgrade Black Duck to at least 2020.6.0 before
upgrading Postgresql.

### 

### Full Vacuum of Database

Due to the vestiges that are left behind by notifications, alerts, and
scans and project-versions that are created and deleted, Synopsys
recommends periodic Full Vacuums. These will clean up the unused space
that autovacuums will not.

You should also do a Full Vacuum prior to an upgrade if you haven't
done one recently.

NOTE: It takes about 24 hours to Auto Vacuum 1 TB of space.

#### Make sure the database pgdata partition has enough space

A Full Vacuum copies the table being vacuumed to a new table name. It
cleans that new table, swaps it with the production table and then
deletes the temp table. As such, it is recommended that the storage
holding the database has 150% of the size of the largest table (100% +
buffer).

You can find the table sizes by using the \\l+ and \\dt+ psql
commands:

psql -h 127.0.0.1 -p 55436 -U blackduck -d bds\_hub -c \"\\l+\" 

psql -h 127.0.0.1 -p 55436 -U blackduck -d bds\_hub -c \"\\dt+ st.\*\"
\| less

df -hPT

You can also get a sorted list by running the following. NOTE: It
might to help to save the script in a file and run it using the -f
option.

SELECT

    relname AS \"relation\",

    pg\_size\_pretty (

        pg\_total\_relation\_size (C .oid)

    ) AS \"total\_size\"

FROM

    pg\_class C

LEFT JOIN pg\_namespace N ON (N.oid = C .relnamespace)

WHERE

    nspname NOT IN (

        \'pg\_catalog\',

        \'information\_schema\'

    )

AND C .relkind \<\> \'i\'

AND nspname !\~ \'\^pg\_toast\'

ORDER BY

    pg\_total\_relation\_size (C .oid) DESC;

The results look like:

relation \| total\_size

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\--

scan\_file \| 59 GB

scan\_composite\_leaf \| 40 GB

scan\_composite\_element \| 7639 MB

scan\_match\_node \| 835 MB

> For vacuuming, ensure available disk space is at least the 1.2 \*
> the size of the largest database table. If you are planning on
> vacuuming tables in parallel, you should have 1.5\* the total database
> size in available disk space.
>
> Refer to the "Installing Black Duck using Docker Swarm" document,
> Chapter 6: "Upgrading Black Duck", "Upgrading from an existing Docker
> architecture" section.
>
> The data migration will temporarily require an additional free disk
> space at approximately 2.5 times your original database volume. Add in
> stuff on pg\_dump being to a different partition. Pg 61 of the install
> guide 6.1
>
> pg 61 install guide: The data migration will temporarily require an
> additional free disk space at approximately 2.5 times youroriginal
> database volume size to hold the database dump and the new database
> volume. As a rule-of-thumb, if the volume upon which your database
> resides is at least 60% free, there should be enough diskspace. ????

#### Perform Full Vacuum

Make sure that any monitoring software (e.g. Zenoss) has been
terminated.

\<Fill in Steps\>

Monitor the vacuum to see if it is still going

select current\_timestamp - query\_start as
runtime,pid,datname,usename,query from pg\_stat\_activity where query !=
\'\<IDLE\>\' and query not in (\'COMMIT\',\'ROLLBACK\')order by 1
desc;

Look for "VACUUM" in the results

      runtime         \|  pid   \| datname  \| usename  \|

 

\
                                                      query

 

 

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--\
 2 days 04:31:37.696708 \|  39364 \| repmgr   \| repmgr   \|  INSERT
INTO repmgr.events (\
        node\_id,              event,              successful,          
   details\
    )       VALUES (\$1, \$2, \$3, \$4)    RETURNING event\_timestamp\
 2 days 04:23:14.553972 \|  42093 \| bds\_hub  \| postgres \|
VACUUM(FULL, ANALYZE, VERBOSE)  st\
.scan\_file\
 00:00:03.262079        \|  37681 \| repmgr   \| repmgr   \| INSERT INTO
repmgr.monitoring\_hist\
ory            (primary\_node\_id,             standby\_node\_id,      
      last\_monitor\_time\
,             last\_apply\_time,            
last\_wal\_primary\_location,             last\_wal\_\
standby\_location,             replication\_lag,             apply\_lag
)      VALUES(1,\
       2,             \'2020-08-10 21:12:41.657707-04\'::TIMESTAMP WITH
TIME ZONE,\
  \'2020-08-10 21:12:36.640652-04\'::TIMESTAMP WITH TIME ZONE,          
  \'6B5E/18FC058\',\
          \'6B5E/18FC058\',             0,             0)

**Duplicating Production Database in Staging Environment**
----------------------------------------------------------

Sometimes issues arise due to scale. Unless you have created a test
database with the possible scaling issues of the production database,
issues may not be detected until the upgrade occurs in production.

So, one way to do that is to build the staging database with similar
data as in production.

Another way is to replicate the production data in the staging
environment. This is easier with an external database.

### Production Database Replication

If the production database is going to be duplicated in the
staging/test environment, then the following steps must be performed.

**Duplicating Production Environment in Staging Environment**
-------------------------------------------------------------

Just as with the creation of a similar/exact copy of the production
database in the test/staging environment, the same should be done with
the hardware (CPU Cores, RAM, storage access (iSCSI vs NFS), storage
type (spinning disk vs flash), and amount of storage. The versions of
Linux, Docker, and PostgreSQL should also be the same.

**Perform Black Duck Upgrade**
==============================

Once you environment is where you want it, you need to actually
perform the upgrade. The steps are the same for staging as for
production

**Pre-upgrade steps**
---------------------

### Check Memory

As with all application it is important to make sure that there is
sufficient RAM and storage for the upgrade. Changes to orchestration,
additions of services, etc may result in more usage or RAM and/or
storage. The new images will take up more space in docker. 

> free -g

### Check Storage

\<blah blah check storage. Add in comment on pruning containers,
volumes, and system/images?\> /opt/docker/..... copy and paste here.
???

**'docker volume prune'**

### Download upgrade images

Any new docker images used by the Black Duck application should be
downloaded to the target server prior to the upgrade. Downloading and
adding the new images can be done any time prior to the upgrade.

These are available at a location specified by your Synopsys Support
person.

Once they are on the target server, you can load them into docker:

ls \| while read image; do docker load -i \$image; done 

### Download Orchestration Files

The Orchestration .yml and .env determine how the Black Duck
application will startup with which functionality and resources.

These need to be downloaded from github
(<https://github.com/blackducksoftware/hub>) and updated to include
customer-specific changes as indicated in the orchestration files for
the previous release.

This may take a while so be careful with the updates to these files.

#### Updated YML Files

If Synopsys Support is providing updated YML files, then they will be
dropped on the Synopsys sharefile or in the Support case

### blackduck\_migrator-2020.6.1.yml

This is similar to docker-compose.yml and
docker-compose.externaldb.yml. It starts up the required services.
However, only those services required to do the Database Migration
Scripts.

Copy it into your docker-swarm location with the other orchestration
files. BUT, make sure that the services setup matches the number of
available cores and memory for your target server. By default, it is
using up 69 GB of RAM and 8 cores. For example, if you have 8 cores and
32 GB of RAM, you may want to reduce the \# of cores used by "services"
to 2.

**Upgrade Monitoring Scripts**

Test scripts may be provided by Synopsys to help with monitoring
upgrades.

For instance, SynopsysMonitorDbActivity\_202007.bash is used to
monitor PostgreSQL activity at the database/table level. It is to be
used if the upgrade processing seems to have stalled (presumably due to
a very large database table upgrade/migration).

### Schedule upgrade

Given that the Black Duck server will be up and down during the
upgrade, it is important that external scripts, scans, etc should not be
running during the upgrade.

Planning with Synospys Support should be made sufficiently ahead of
time so that they can plan support during your upgrade. In addition,
those teams using Black Duck need to plan their release schedules to
will want to work around the schedule and put the downtime into their
schedules. 

**Perform Upgrade, Day of Upgrade (Staging then Production)**
-------------------------------------------------------------

The upgrades normally take enough time to bring down Black Duck,
restart Black Duck, and run through any Database Migration scripts.
Upgrading from 2020.4.1 to 2020.6.1 is expecting to take under 2
hours.

Remember that you should be doing all of these steps on the
test/staging server to make sure that all the steps work for the current
update prior to doing the upgrade on the production server!

### Stop scanning and external connections 

#### Stop Scanning

The customer should terminate performing scans many hours before
beginning the upgrade installation. This is to allow the scans to
complete prior to the upgrade while minimizing the large process that
need to unwind and rollback during the Black Duck shutdown.

> Wait for the scan jobs to finish before terminating external
> connections in the next step
>
> After the scan jobs are complete, the customer should terminate all
> external queries, utilities, scripts, and SDK calls prior to starting
> the actual upgrade.
>
> You can verify this by doing the following psql command:

psql -h hub-stg-db -U blackduck -p 5432 -d bds\_hub -c "SELECT \* FROM
pg\_stat\_activity WHERE datname = \'bds\_hub\' and state =
\'active\';"

bds\_hub=\# SELECT \* FROM pg\_stat\_activity WHERE datname =
\'bds\_hub\' and state = \'active\';

datid \| datname \| pid \| usesysid \| usename \| application\_name \|
client\_addr \| client\_hostname \| client\_port \| backend\_start

\| xact\_start \| query\_start \| state\_change \| wait\_event\_type
\| wait\_event \| state \| ba

ckend\_xid \| backend\_xmin \| query

\-\-\-\-\-\--+\-\-\-\-\-\-\-\--+\-\-\-\-\--+\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\--+\-\--

\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

16441 \| bds\_hub \| 8722 \| 16440 \| blackduck \| psql \|
10.251.22.151 \| \| 35794 \| 2020-08-04 22:56:43.585215-

04 \| 2020-08-04 22:56:59.671193-04 \| 2020-08-04 22:56:59.671193-04
\| 2020-08-04 22:56:59.671215-04 \| \| \| active \|

\| 94647657 \| SELECT \* FROM pg\_stat\_activity WHERE datname =
\'bds\_hub\' and state = \'active\';

#### Stop External Connections

Customer to ensure all Black Duck and PostgreSQL server activities are
as quiet as possible before starting upgrade. So, to that end, the
customer should ensure that no external connections to the database will
interfere with upgrade. This is because external database connections
can block upgrade database actions.

To ensure this:

-   Make backup of pg\_hba.conf file.

-   Edit pg\_hba.conf to permit connections only from
    localhost/127.0.0.1 and the upgrading Black Duck server. 

-   Set most "host" settings to "reject" or comment them out. 

<!-- -->

-   Restart PostgreSQL (to reload .conf changes, but to also
    disconnect any currently present external connections). 

NOTE: Customer may have other means available to effect this
control.

#### Backup database

-   Customer to backup the database. 

-   Customer to move Production database dump file to separate machine
    if possible. 

-   Customer to balance timing of Production backup with the reality
    that subsequent scans will not be in the backup. 

-   If not already done, backup external database with 2020.4.1
    hub\_create\_data\_dump.sh

### Start the actual upgrade

#### Bring down Black Duck Docker Application

The following steps assume that any docker and/or PostgreSQL upgrades
have been completed already or may be done sometime after the Black Duck
upgrade (as in the case of newly supported versions of PostgreSQL). If
Docker upgrades are occurring at the same time, then the steps mentioned
above (e.g. stopping Docker, uninstalling, reinstalling) should be done
in conjunction with the following steps. ???

Upon stopping Black Duck, be very patient and confirm applications
have come to a complete halt before proceeding. 

When bringing up the Black Duck application in Docker, the command to
start includes the name of the docker stack. For the purposes of this
document, we will call it "hub".

List the Docker services if you don't know the name:

> docker stack ls
>
> NAME SERVICES
>
> hub 10

Stop the Docker services associated with the existing Black Duck
instance

> docker stack rm \<hub stack name\>

In order to watch the Docker Services until they all shutdown, do the
following:

> watch docker ps 

#### Start Black Duck with the new version of Black Duck

As part of the prep work, you will have already downloaded the new
Orchestration files for the new release and updated them with the same
changes you made in the previous release (and additional changes if
required with this release).

Take a minute to make sure that these are the correct files for the
server type (production vs staging) and that the changes are correct.

This includes certs, secrets, external db mount, additional options,
proxy, etc.

Make sure the local directory is where the orchestration files are and
start the Black Duck Migration YML file:

#### Start the abbreviated Database Migration YML

You can start the Database Migration by typing

docker stack deploy -c blackduck\_migrator-2020.6.1.yml -c
docker-compose.local-overrides.migrator.yml hub

Run the script (SynopsysMonitorDbActivity\_202007.bash) to see when
the Database Migrations are done. In the output (see below), look at the
"v\_hub script status". From top down are the latest Database Migration
scripts that were run. Check the "version" column for the script \# and
the success column to see if it successfully completed (t). Synopsys
Support can tell you what the latest Database Migration script version
that is run with the target release.

v\_hub script status\...

now \| inet\_server\_addr \| cmd \| loop \| installed\_rank \| version
\| description

\| type \| script \| checksum \| installed\_by \| insta

lled\_on \| execution\_time \| success

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\--

\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\--

2020-08-05 17:27:25.254976-04 \| 10.251.22.104 \|
v\_hub\_script\_status \| 1 \| 185 \| 2020.06.0.011 \| clear stale
policies from central relea

se \| SQL \|
hub\_V2020\_06\_0\_011\_\_clear\_stale\_policies\_from\_central\_release.sql.vpp
\| -1470604476 \| blackduck \| 2020-08-05 1

9:26:12.882272 \| 18 \| t

#### Monitor upgrade docker containers health.

Verify that Black Duck starts up correctly. Once all the Upgrade
docker containers are healthy, the upgrade installation is complete. 

You can monitor the services as follows:

watch docker ps

> Restore database with 2020.4.1 hub\_db\_migrate.sh ??? 

#### Monitor the Black Duck database during Database Migration

During large database table migration steps, the system may appear
quiet or stuck for many hours. 

To confirm progress, do the following:

##### Tail the container logs

for mcontainer in \$(docker ps -q) ; do docker ps \| grep \$mcontainer
; docker logs \--tail 2 \$mcontainer \|& cut -c 1-150 ; done 

##### Check PostgreSQL activity

**bash**

export PGPASSWORD='\<PSQL Database Password\>'

./SynopsysMonitorDbActivity\_202007.bash \>
SynopsysMonitorDbActivity\_202007.out 2\>&1

#### Start the official release of Black Duck

Once the Database Migration is complete, bring the Docker containers
back down and start the official release:

**Stop Docker**

docker stack rm hub

Local Database:

docker stack *deploy* -c docker-compose.yml -c
docker-compose.local-overrides.yml \<stack name\>

e.g. docker stack *deploy* -c docker-compose.yml -c
docker-compose.local-overrides.yml hub

External Database:

docker stack *deploy* -c docker-compose.externaldb.yml -c
docker-compose.local-overrides.yml \<stack name\>

e.g. docker stack *deploy* -c docker-compose.externaldb.yml -c
docker-compose.local-overrides.yml hub

#### Monitor upgrade docker containers health.

Verify that Black Duck starts up correctly. Once all the Upgrade
docker containers are healthy, the upgrade installation is complete.
NOTE that the nginx service will show up and disappear until all of the
other services are up and healthy and then it will startup correctly.

You can monitor the services as follows:

watch docker ps

#### Start Additional Services

If your target configuration has multiple instances of services (e.g.
3 jobrunner services and 3 scan services, you need to start those.

#### docker service scale hub\_jobrunner=3

#### docker service scale hub\_scan=3

#### 

Confirm with output of commands in step 2.2.7.6.1. above. Ooops.???

#### Vacuum audit\_event table 

When the migration script is finished and If Customer has not done
"vacuum full analyze" prior to the upgrade, Synopsys strongly recommends
that you run the VACUUM command on the audit\_event table to optimize
PostgreSQL performance. See "Installing Black Duck using Docker Swarm",
Chapter 6: "Upgrading Black Duck", "Migration script to purge unused
rows in the audit event table" (Page 55). 

**Post-Upgrade Steps**
----------------------

### 

### Run upgrade-validation tests.

Customer run the tests required to verify that the existing projects
and process were not impacted unexpectedly by the upgrade.

### Re-run benchmark tests. 

Customer run the benchmark tests that they ran prior to the upgrade
(see above)

For example, sysbench:

date ; date \--utc ; hostname -f ; pwd ; whoami ; nproc ; free -g ;
time for mthreads in 4 8 16 ; do echo test threads \$mthreads ; sysbench
fileio \--file-test-mode=rndrw \--threads=\$mthreads run \| grep -e
\"\\(read\\\|writ\\).\*/\" ; done

### Announce upgrade completion to stakeholders

When the upgrade is successfully complete, Customer will update their
internal customers, as well as the Synopsys Salesforce Community Case,
with the appropriate status to provide an "all clear".

**Contingency: Fallback Steps**
===============================

We need to talk about these.???

1.  If at any point in the migrations, an unrecoverable error occurs,
    the plan is to fallback to version the previous version (for this
    example, 2020.4.1). 

2.  In order to fall back to 2020.4.1, first remove the volumes.

    1.  Start up 2020.4.1 and THEN Run 'docker volume prune' after
        verifying no other volumes are in use. 

    2.  Alternatively, 'docker volume ls' to list and docker volume rm
        \<volume name\> each related volume. 

3.  Using the restore commands to re-establish the 2020.4.1 backup
    taken during upgrade steps

    3.  Expect about \_\_\_\_ hours to fallback to 2020.4.1.

**Restore steps, during Fallback ????**
---------------------------------------

1.  To restore the PostgreSQL data during a fallback 

2.  Use the docker-compose.dbmigrate.yml file located in the
    docker-swarm directory. That starts the containers and volumes
    needed to migrate the database. 

> docker stack deploy -c docker-compose.dbmigrate.yml hub 

1.  Note that there are some versions of Docker where if the images
    live in a private repository, docker stack will not pull them unless
    the following flag is added to the above command: 

> \--with-registry-auth 

3.  After the DB container has started, run the migration script
    located in the docker-swarm directory. This script restores the data
    from the existing database dump file. 

    2.  This has a duration on the order of hours, so perform it in a
        screen multiplexer like screen or tmux, or as a detached
        background process.

> ./bin/hub\_db\_migrate.sh \<path to dump file\>
