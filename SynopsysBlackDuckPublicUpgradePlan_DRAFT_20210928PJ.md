<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Preface](#preface)
- [Customer details](#customer-details)
  - [Contacts](#contacts)
  - [Customer Upgrade Schedule](#customer-upgrade-schedule)
  - [Customer Background Information](#customer-background-information)
- [Plan the Upgrade](#plan-the-upgrade)
  - [Determine Upgrade and production requirements](#determine-upgrade-and-production-requirements)
    - [Documentation](#documentation)
    - [Checklist](#checklist)
  - [Write post-upgrade validation test plan](#write-post-upgrade-validation-test-plan)
    - [Checklist](#checklist-1)
  - [Document Database/API Connections](#document-databaseapi-connections)
  - [Schedule the Upgrades (Staging and Production)](#schedule-the-upgrades-staging-and-production)
  - [Open a Synopsys SalesForce Case](#open-a-synopsys-salesforce-case)
- [Perform Upgrade-Preparation Activities](#perform-upgrade-preparation-activities)
  - [Resolve Performance and Networking Issues](#resolve-performance-and-networking-issues)
    - [Implement Best Practices](#implement-best-practices)
    - [Sage](#sage)
    - [system_check.sh](#system_checksh)
    - [sar](#sar)
    - [Other Customer monitoring tools, such as zenoss?](#other-customer-monitoring-tools-such-as-zenoss)
      - [SynopsysGatherServerSpecs.bash](#synopsysgatherserverspecsbash)
      - [sysbench](#sysbench)
      - [pgbench](#pgbench)
      - [pg_test_fsync](#pg_test_fsync)
      - [bonnie++](#bonnie)
      - [dd](#dd)
  - [Clean up Black Duck Projects and Scans](#clean-up-black-duck-projects-and-scans)
  - [Clean up Databases](#clean-up-databases)
    - [bdio Database](#bdio-database)
    - [bds_hub_report Database](#bds_hub_report-database)
    - [bds_hub, postgresql, template0, and template1 Database](#bds_hub-postgresql-template0-and-template1-database)
    - [Trim the Notification and audit_event logs](#trim-the-notification-and-audit_event-logs)
      - [Notification Logs](#notification-logs)
      - [Audit_Events](#audit_events)
    - [Remove any orphaned large objects](#remove-any-orphaned-large-objects)
      - [vacuumlo](#vacuumlo)
    - [Run PostgreSQL tuning utility](#run-postgresql-tuning-utility)
      - [Pgtune](#pgtune)
    - [Full Vacuum of Database](#full-vacuum-of-database)
      - [Make sure the database pgdata partition has enough space](#make-sure-the-database-pgdata-partition-has-enough-space)
      - [Perform Full Vacuum](#perform-full-vacuum)
  - [Upgrade O/S, Kernel, Docker, Postgresql](#upgrade-os-kernel-docker-postgresql)
    - [O/S and/or Kernel upgrade](#os-andor-kernel-upgrade)
    - [Docker Upgrades](#docker-upgrades)
    - [PostgreSQL Upgrades](#postgresql-upgrades)
  - [Replicate Production Database over into Staging Environment](#replicate-production-database-over-into-staging-environment)
  - [Duplicate Production Environment in Staging Environment](#duplicate-production-environment-in-staging-environment)
  - [Download images, files, scripts](#download-images-files-scripts)
    - [Download upgrade images](#download-upgrade-images)
    - [Download Orchestration Files](#download-orchestration-files)
      - [Download updated YML Files](#download-updated-yml-files)
    - [Download blackduck_migrator yaml file](#download-blackduck_migrator-yaml-file)
    - [Download Upgrade-Monitoring Scripts](#download-upgrade-monitoring-scripts)
  - [Update stakeholders about near-term upgrade Schedule](#update-stakeholders-about-near-term-upgrade-schedule)
- [Perform Black Duck Upgrade](#perform-black-duck-upgrade)
  - [Perform Pre-upgrade steps](#perform-pre-upgrade-steps)
    - [Check Memory](#check-memory)
    - [Check Storage](#check-storage)
  - [Perform Upgrade, Day of Upgrade (Staging first, then Production)](#perform-upgrade-day-of-upgrade-staging-first-then-production)
    - [Stop scanning and external connections](#stop-scanning-and-external-connections)
      - [Stop Scanning](#stop-scanning)
      - [Stop External Connections](#stop-external-connections)
      - [Backup database](#backup-database)
    - [Start the actual upgrade](#start-the-actual-upgrade)
      - [Bring down Black Duck Docker Application](#bring-down-black-duck-docker-application)
      - [Start Black Duck with the new version of Black Duck](#start-black-duck-with-the-new-version-of-black-duck)
      - [Start the abbreviated Database Migration YML](#start-the-abbreviated-database-migration-yml)
      - [Monitor upgrade docker containers health.](#monitor-upgrade-docker-containers-health)
      - [Restore database with pre-upgrade version of hub_db_migrate.sh (TODO: confirm)](#restore-database-with-pre-upgrade-version-of-hub_db_migratesh-todo-confirm)
      - [Monitor the Black Duck database during Database Migration](#monitor-the-black-duck-database-during-database-migration)
      - [Start the official release of Black Duck](#start-the-official-release-of-black-duck)
      - [Monitor upgrade docker containers health.](#monitor-upgrade-docker-containers-health-1)
      - [Docker service replicas](#docker-service-replicas)
      - [Vacuum audit_event table](#vacuum-audit_event-table)
  - [Perform Post-Upgrade Steps](#perform-post-upgrade-steps)
    - [Wait...](#wait)
    - [Run upgrade-validation tests.](#run-upgrade-validation-tests)
    - [Run benchmark tests.](#run-benchmark-tests)
    - [Announce upgrade completion (Staging or Production) to stakeholders](#announce-upgrade-completion-staging-or-production-to-stakeholders)
- [Contingency: Fallback Steps](#contingency-fallback%C2%A0steps)
  - [Restore steps, during Fallback](#restore-steps-during-fallback)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->





# Preface

This document is provided as-is, without warranty or liability.

This document is a draft, informal, test, work in progress.

- last updated  October 4, 2021, pjalajas. 

This document is intended to be used by very large (databases on the order of 1 TB) enterprise Black Duck customers to help their Black Duck upgrades go smoothly.  

This document is only for on-premise installations; hosted/SaaS Black Duck instances are upgraded by Black Duck SaaS team.  

Use github Issues and Discussions and Pull Requests for any corrections or suggestions you may have.  

TODO:  search this document for "TODO" and __________ and consider them.

Document filename: SynopsysBlackDuckPublicUpgradePlan_DRAFT_20210928PJ.md

This document has been reviewed by: 

-  
-  


# Customer details
TODO:  Clean up this section

## Contacts

- Customer: 

- Case(s): 
    - Current SalesForce Case number:
    - Prior related cases for reference: 

- Customer: 
    - Name:
    - email:
    - phone:
    - timezone:

- Synopsys Support Staff member(s): 
    - Name(s):  
    - email(s):
    - phone(s):
    - timezone(s):

- Support links: 
    - website: Synopsys Software Integrity Community: https://community.synopsys.com/s/
    - email:  software-integrity-support@synopsys.com


## Customer Upgrade Schedule

- Staging: 
    - Date: 
    - From version:
    - To version:

- Production:
    - Only after successful completion of the upgrade on Staging, and
    resolution of any findings from that upgrade, Customer is currently
    expected to schedule upgrading Production on 
    - Date:

## Customer Background Information

Customer to prepare private documentation of the following system information:

- Describe current Customer Black Duck instances: 
    - Customer to download desire upgrade target Black Duck release
        - https://github.com/blackducksoftware/hub/releases
        - From that release archive, Customer to review release documentation:



```

find hub-2021.8.3/ | grep -e en_US.*pdf -e md

hub-2021.8.3/docker-swarm/README.md
hub-2021.8.3/docs/en_US/getting_started.pdf
hub-2021.8.3/docs/en_US/install_kubernetes.pdf
hub-2021.8.3/docs/en_US/install_openshift.pdf
hub-2021.8.3/docs/en_US/install_swarm.pdf
hub-2021.8.3/docs/en_US/release_notes.pdf
hub-2021.8.3/Important_Upgrade_Announcement.md
hub-2021.8.3/kubernetes/blackduck/README.md
hub-2021.8.3/kubernetes/README.md
hub-2021.8.3/README.containers.md
hub-2021.8.3/README.md

```


- Customer to run 


`./docker-swarm/bin/system_check.sh` 


- Customer to decide whether to use docker swarm kubernetes.
- Customer to work with Black Duck Support team to configure container services.


```

grep -E -e image -e HUB_MAX_MEMORY -e limits -e reservations -e cpus -e memory -e replicas ./hub-2021.8.3/docker-swarm/docker-compose.yml
    image: blackducksoftware/blackduck-postgres:9.6-1.1
	limits: {memory: 3072M}
	reservations: {cpus: '1', memory: 3072M}

    image: blackducksoftware/blackduck-authentication:2021.8.3
       HUB_MAX_MEMORY: 512m
	limits: {cpus: '1', memory: 1024M}
	reservations: {memory: 1024M}

    image: blackducksoftware/blackduck-webapp:2021.8.3
      HUB_MAX_MEMORY: 2048m
	limits: {cpus: '1', memory: 2560M}
	reservations: {cpus: '1', memory: 2560M}

    image: blackducksoftware/blackduck-scan:2021.8.3
      HUB_MAX_MEMORY: 2048m
	limits: {cpus: '1', memory: 2560M}
	reservations: {cpus: '1', memory: 2560M}

    image: blackducksoftware/blackduck-jobrunner:2021.8.3
       HUB_MAX_MEMORY: 4096m
	limits: {cpus: '1', memory: 4608M}
	reservations: {cpus: '1', memory: 4608M}

    image: blackducksoftware/blackduck-cfssl:1.0.3
	limits: {memory: 640M}
	reservations: {memory: 640M}

    image: blackducksoftware/blackduck-logstash:1.0.10
	limits: {memory: 1024M}
	reservations: {memory: 1024M}

    image: blackducksoftware/blackduck-registration:2021.8.3
	limits: {memory: 640M}
	reservations: {memory: 640M}

    image: blackducksoftware/blackduck-nginx:2.0.6
	limits: {memory: 512M}
	reservations: {memory: 512M}

    image: blackducksoftware/blackduck-webui:2021.8.3
	limits: {cpus: '1', memory: 640M}
	reservations: {cpus: '0.5', memory: 640M}

    image: blackducksoftware/blackduck-documentation:2021.8.3
	limits: {memory: 512M}
	reservations: {memory: 512M}

      image: blackducksoftware/blackduck-upload-cache:1.0.18
	  limits: {memory: 512M}
	  reservations: {memory: 512M}

    image: blackducksoftware/blackduck-redis:2021.8.3
	limits: {memory: 1024M}
	reservations: {memory: 1024M}

    image: blackducksoftware/blackduck-bomengine:2021.8.3
      HUB_MAX_MEMORY: 4096m
	  limits: {memory: 4608M}
	  reservations: {memory: 1536M}

    image: blackducksoftware/blackduck-matchengine:2021.8.3
	limits: { memory: 4608M, cpus: '1' }
	reservations: { memory: 1536M, cpus: '1' }
      HUB_MAX_MEMORY: 4096m

    image: blackducksoftware/rabbitmq:1.2.3
	  limits: {memory: 1024M}
	  reservations: {memory: 1024M}

```


Resources to consider include: 

- Number of container replicas:
- HUB_MAX_MEMORY
- Resource reservations and limits
    - ram
    - cpu
- Database: in container or external? 
- Does installation include Synopsys Alert?
    - Is Alert database in container or external database?
- How are database disks connected?
   - Example: over iSCSI over 2-NIC mpath ethernet elsewhere in the same datacenter. 
- Any previously Synopsys-reported infrastructure performance issues with their internal sysadmins, including possible proxy/firewall issues?
- Customer has Staging and Production environments. 
    - Does customer Staging exactly replicate Production in terms of size and networking configuration?
    - Does customer environment include complex proxy/firewall configurations?
    - During a recent upgrade event, did Staging testing underestimate Production upgrade task durations? 
- During a recent upgrade event, were Customer docker container memory, cpu, and replica settings reported to have been misconfigured and had to
        be fixed under a P1 Case? 
- Customer Production bds_hub database was recently __________ TB.
- The upgrade is planned to be performed using a simplified Upgrade
    Docker stack implemented by deploying new Synopsys-supplied
    deployment (.yml) files. These Upgrade .yml files will be different
    from the Production versions of the files.
- Any significant database migration steps expected?  Any other potential causes of long delays during upgrade?
- The upgrade is expected to take about __________ hours. 
    - If Fallback is required, an additional __________ hours would be
        required.


# Plan the Upgrade

Planning for an upgrade should occur days or weeks prior to an
upgrade. This is not about actually *changing* anything in the Customer Black Duck
Environment. It is only about *planning* on what to do prior to the Black Duck upgrade.


## Determine Upgrade and production requirements

In this section, the areas that need to be checked prior to upgrading
Black Duck are described. This includes server setup (RAM, CPUs, Storage), O/S
versions, and application versions.

### Documentation

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

Customer must also review the README and Announcments files provided with the target Black Duck release.

As these documents are updated over time, it is important to keep up
on the changes.

Those documents are the primary resources the customer should follow to perform a Black Duck upgrade;
this Upgrade Support Plan document tries to augment, not replicate,
them. If something is unclear in this upgrade document, refer to the
primary documents or ask Black Duck Support. 

In particular, Chapter 6 of the Installing Black Duck using Docker
Swarm (Upgrading Black Duck) should be reviewed. This document indicates
the supported versions of the target operating system, kernel, java,
docker, and PostgreSQL. NOTE: since certain versions of PostgreSQL are
supported in certain versions, the Black Duck server must be at that
minimum version prior to upgrading PostgreSQL. 

For instance, if the
Installation Guide indicates that PostgreSQL 11.7 is now supported for
external databases as part of the 2020.6.x release, you cannot upgrade
PostgreSQL to 11.7 until the Black Duck server is at that release
(minimum).

For example, as part of the 2020.6.0 release, the following are
supported (please refer to your Install Guide for the actual versions
for your release):

  |O/S or Application|Version|
  |----------------------|-----------| 
  |Red Hat Linux|7.3|
  |Docker|Community 19.03|
  |PostgreSQL (external database)|9.6.x or 11.7|


### Checklist

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
    -   CPUs sufficient for the server (Check documentation and the same files as the RAM check)?
    -   Check Storage (df -hl on the Black Duck server)
-   Customer to review those documents and resolve with Synopsys any
    questions or issues discovered with Synopsys.
-   Customer needs to determine if any server upgrades need to occur and if
    they should be done prior to the Black Duck upgrade.



## Write post-upgrade validation test plan

Customer is required to have a set of major stress and regression tests, that very closely mimic Production load, that
are run in Staging, prior to upgrading Black Duck in Production. This set of regression tests might include
verifying that the same projects are visible, that the results
of key BOMs of project-versions are OK, that admins can access license
management, policy management, or other sections.

In addition to this, there will be new functionality with the new
release. If your organization is looking to take advantage of these
changes, then you will need to make sure that these changes fit into
your policies and procedures. Having tests to check these features will
provide validation specific to your installation.

You may have cases open with Synopsys concerning your use of the
product that are slated to be fixed in this release. RFEs you have
requested may also be part of this release. There should be tests
to verify that these fixes and improvements work as expected in this
release.

In this section, the areas that need to be checked prior to upgrading
Black Duck are listed. This includes server setup (RAM, CPUs, Storage), O/S
versions, and application versions.

### Checklist

-   Review Regression Tests to see if changes in functionality impact
    the test cases.
    -   Make changes as necessary for target release
-   Review Release Notes for areas to test (at least both Chapters 1
    and 2, for all applicable Black Duck versions that will be spanned by this Black Duck upgrade event).
    -   Create Test Cases for the functionality that you will use or
        may impact your use
-   Review issues opened with Synopsys (or recent issues not tracked
    yet) and determine if any may be fixed in the target release.
    -   Create Test Cases for the functionality that you will use or
        may impact your use.
-   Review Scripts based on Black Duck APIs.
    -   Create Test Cases to verify that interface is still working
        the same way after the upgrade.



## Document Database/API Connections

It is important for the Customer know unequivocally what connections are made, by Customer users, between the various
tools that may interact with the Black Duck Database and/or REST API.
These connections will need to be blocked during the upgrade, to minimize impact to the database
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



## Schedule the Upgrades (Staging and Production)

If Synopsys Support is required, then the customer should schedule the
start of an upgrade to occur during the morning of a normal business day
(Eastern Time Zone). This request should occur at least 1 week prior to
the upgrade. This will maximize the chance and duration of Synopsys
Support help.

Should the customer require Synopsys support, they should open a case
requesting help.

## Open a Synopsys SalesForce Case 

You can create a SalesForce case by going to
<https://community.synopsys.com> and logging in with your Community site
username and password. If you do not have one, request one from your
sales and/or support contact.

Select Support -> Create a Support case.

The case should be set to *P3 -- Medium*. Should any issues be
encountered during the upgrade, the Customer will update the priority of
that case to *P1 -- Critical*. Customer will then call Synopsys at
800-873-7793 to have the on-call Technical Support Engineer (TSE)
engaged.



# Perform Upgrade-Preparation Activities

The preceding section described *Planning* actions that did not change the customer environment
in preparation for the Black Duck upgrade.  By contrast, this section describes *changes* to 
be made in the Customer Black Duck environment *prior to* implementing the actual upgrade. 

Prior to the target upgrade (be it test/staging or production), the
following activities must be performed to make sure that the environment is ready. 

This activity should be performed weeks or months before the upgrade. 



## Resolve Performance and Networking Issues

TODO:  be sure to elaborate on resolving networking issues, such as proxies, firewalls, tls certificates, etc, etc.

It is important that any performance issues be resolved or mitigated
prior to an upgrade in order to keep the upgrade duration 
minimized. 

These are a few of the documents/tools that can help determine if
there are any issues.

If any issues are discovered, resolve them prior to performing the Black Duck upgrade.


### Implement Best Practices 

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

### system_check.sh

(./hub_<version>/docker-swarm/bin/system_check.sh)

Run this on the Black Duck Server as well as on the Database Server if applicable. Check to see if there
are any issues that need to be dealt with prior to the upgrades

### sar

Check out the iowait values on the Database Server. 

```
06:00:01 PM CPU %user %nice %system %iowait %steal %idle
06:10:01 PM all 2.04 0.00 1.12 0.02 0.00 96.83
06:20:01 PM all 2.21 0.00 1.02 0.02 0.00 96.76
```

Should there be any issues, then you need to talk to your server,
network, and database admins to see what you can do to remove any
bottlenecks. Synopsys Support may be able to help.

NOTE: Should you need to look at the SAR results from the other day,
go to /var/log/sa and type "sar -f sa\#\#" where \#\# is the day of the
month.



### Other Customer monitoring tools, such as zenoss?

Customer to document plans to use any such monitoring tools.

#### SynopsysGatherServerSpecs.bash

TODO: Pete to test and update this script if needed.

This optional ad hoc script captures a
lot of system data including top, cpu, mem, application versions, etc.
It has several psql queries that need to be modified for your setup. The
end of the script does latency testing that may need to be configured if
there are any partitions you do not wish tested.

Synopsys can provide the script:

TODO: is this is a different script, needs own header, or did the filename change?

```shell
export PGPASSWORD='<PSQL Database Password>'
./SynopsysMonitorDbActivity_202007.bash > SynopsysMonitorDbActivity_202007.out 2>&1
```
TODO: should use ~/.pgpass







#### sysbench

sysbench provides benchmarking capabilities for Linux. sysbench
supports testing CPU, memory, file I/O, mutex performance, and even
[MySQL](https://wiki.gentoo.org/wiki/MySQL) benchmarking.

If sysbench is not installed on the server (yum list installed | grep
sysbench), you can install it as root by running:

```
sudo su - yum install sysbench
```

To run sysbench, run the following as root:

```
date ; date --utc ; hostname -f ; pwd ; whoami ; nproc ; free -g ;
time for mthreads in 4 8 16 ; do echo test threads \$mthreads ; sysbench
fileio --file-test-mode=rndrw --threads=\$mthreads run | grep -e
"\(read|writ\).*/" ; done
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



#### pgbench

pgbench is a simple program for running benchmark tests on
PostgreSQL.

Caution: pgbench -i creates four tables pgbench_accounts,
pgbench_branches, pgbench_history, and pgbench_tellers, destroying
any existing tables of these names. Be very careful to use another
database if you have tables having these names!

If pgbench is not installed on the server (yum list installed | grep
pgbench), you can install it as root by running:

```
sudo su -
yum install postgresql-contrib
```

To run pgbench, run the following as non-root user:

Example:
```
date ; hostname ; time pgbench -h hub-stg-db -U blackduck -p 5432 -d bds_hub -s 10000 -c 200 -j 100 -M prepared -t 1000 2> /dev/null
```    

Example output:
```
Fri Jul 24 13:50:59 EDT 2020
sup-pjalajas-hub.dc1.lan
pghost: sup-pjalajas-2 pgport: 55436 nclients: 200 nxacts: 1000
dbName: bds_hub
transaction type: TPC-B (sort of)
scaling factor: 1
query mode: prepared
number of clients: 200
number of threads: 100
number of transactions per client: 1000
number of transactions actually processed: 200000/200000
tps = 439.798734 (including connections establishing)
tps = 440.029375 (excluding connections establishing)

real 7m34.812s
user 0m23.666s
sys 0m49.548s
```




#### pg_test_fsync

pg_test_fsync is intended to give you a reasonable idea of what the fastest
[wal_sync_method](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-SYNC-METHOD)
is on your specific system, as well as supplying diagnostic information
in the event of an identified I/O problem.

```
date --utc ; hostname -f ; pg_test_fsync
```

```
Fri Jul 24 19:56:41 UTC 2020
sup-pjalajas-hub.dc1.lan
2 seconds per test
O_DIRECT supported on this platform for open_datasync and
open_sync.
Compare file sync methods using one 8kB write:
(in wal_sync_method preference order, except fdatasync
is Linuxs default)
        open_datasync                    1828.665 ops/sec
        fdatasync                        1839.337 ops/sec
        fsync                             538.328 ops/sec
        fsync_writethrough                            n/a
        open_sync                         442.936 ops/sec
Compare file sync methods using two 8kB writes:
(in wal_sync_method preference order, except fdatasync
is Linuxs default)
        open_datasync                     945.146 ops/sec
        fdatasync                        1461.665 ops/sec
        fsync                             531.997 ops/sec
        fsync_writethrough                            n/a
        open_sync                         330.452 ops/sec
Compare open_sync with different write sizes:
(This is designed to compare the cost of writing 16kB
in different write open_sync sizes.)
         1 * 16kB open_sync write         595.042 ops/sec
         2 *  8kB open_sync writes        229.368 ops/sec
         4 *  4kB open_sync writes        121.647 ops/sec
         8 *  2kB open_sync writes         83.208 ops/sec
        16 *  1kB open_sync writes         39.609 ops/sec
Test if fsync on non-write file descriptor is honored:
(If the times are similar, fsync() can sync data written
on a different descriptor.)
        write, fsync, close               633.979 ops/sec
        write, close, fsync               486.340 ops/sec
Non-Synced 8kB writes:
        write                           244921.142 ops/sec
```




#### bonnie++

Bonnie++ allows you to benchmark how your file systems perform with
respect to data read and write speed, the number of seeks that can be
performed per second, and the number of file metadata operations that
can be performed per second.
```
date --utc ; hostname -f ; time bonnie++ # need 2 x RAM to be free on disk 
```

```
Fri Jul 24 20:53:01 UTC 2020
sup-pjalajas-hub.dc1.lan
Writing a byte at a time...done
Writing intelligently...done
Rewriting...done
Reading a byte at a time...done
Reading intelligently...done
start em...done...done...done...done...done...
Create files in sequential order...done.
Stat files in sequential order...done.
Delete files in sequential order...done.
Create files in random order...done.
Stat files in random order...done.
Delete files in random order...done.
Version  1.97       ------Sequential Output------
--Sequential Input- --Random-
Concurrency   1     -Per Chr- --Block-- -Rewrite- -Per Chr-
--Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec
%CP  /sec %CP
sup-pjalajas 80112M   668  99 993827  96 553909  59  1411  99
1341920  79  8210 150
Latency             20730us   98013us    1412ms    8035us  
41587us   15082us
Version  1.97       ------Sequential Create------
--------Random Create--------
sup-pjalajas-hub.dc -Create-- --Read--- -Delete-- -Create--
--Read--- -Delete--
              files  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec
%CP  /sec %CP
                 16 +++++ +++ +++++ +++ +++++ +++ +++++ +++ +++++
+++ +++++ +++
Latency              1262us    6290us     660us    3386us    
228us     626us
1.97,1.97,sup-pjalajas-hub.dc1.lan,1,1595640930,80112M,,668,99,993827,96,553909,59,1411,99,1341920,79,8210,150,16,,,,,+++++,+++,+++++,+++,+++++,+++,+++++,+++,+++++,+++,+++++,+++,20730us,98013us,1412ms,8035us,41587us,15082us,1262us,6290us,660us,3386us,228us,626us

real    5m9.032s
user    0m56.024s
sys     2m53.775s
```






#### dd

You can use dd to create a large file as quickly as possible to
see how long it takes. It is a very basic test and not very customisable
however it will give you a sense of the performance of the file system.
You must make sure this file is larger than the amount of RAM you have
on your system to avoid the whole file being cached in memory.

```
time sh -c "dd if=/dev/zero of=[PATH] bs=[BLOCK_SIZE]k count=[LOOPS] && sync"
```

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

```
time sh -c "dd if=/dev/zero of=/mnt/mount1/test.tmp bs=4k count=2000000 && sync"
2000000+0 records in
2000000+0 records out
8192000000 bytes transferred in 159.062003 secs (51501929 bytes/sec)

real 2m41.618s
user 0m0.630s
sys 0m14.998s
```




## Clean up Black Duck Projects and Scans

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




## Clean up Databases

From time-to-time, Databases used by Black Duck become obsolete. If an
upcoming release requires the removal of a database as part of the
Database Migration scripts (part of the upgrade), then this should be
explicitly tested as part of the staging testing. 

Contact your Synopsys Support to make sure you understand what might
be impacted by the database removal. 

Make sure your verification tests
can verify that that data migrated correctly and that the database is
removed.

Since Synopsys will tend to migrate the data from a database in one
release and then remove the database in another release (to make sure
that there are no issues prior to deleting the data), this test may need
to be done in each release.


### bdio Database

The BDIO Database was removed as part of the 2020.2.0 upgrade (TODO: confirm).
One of the Database Migration scripts removed that database. Should your
database be on the order of 1 TB or more, then there may be timeout issues when running
that script. Make sure you thoroughly test that migration in your Staging environment with a full copy of the
production database.


### bds_hub_report Database

In Black Duck version 2019.10.0, to make it easier for users to
quickly create, use, back up, and restore the reporting database, the
tables from the separate reporting database (bds_hub_report) have been
moved to the *reporting* schema, with periodically updated materialized
views, in the Black Duck database, bds_hub.

So, as part of the 2020.6.0 release, the bds_hub_report database was
removed as part of the Database Migration scripts. If the Customer has
not generated any reports using the bds_hub_report database, the
Database Migration script to remove that database should be quick.


### bds_hub, postgresql, template0, and template1 Database

These databases are still valid as of the 2020.6.1 release.  (TODO:  update/confirm)





### Trim the Notification and audit_event logs

Depending on the activity on the Black Duck server, the number of
records taken up by the logging could be in the millions. It is
important to trim those logs periodically and specifically prior to an
upgrade.


#### Notification Logs

Notification logs are sometimes used by integration tools to trigger
alerts or the generation/update of JIRA cases so they do need to stick
around for a bit.

In the blackduck-config.env file, the Customer can indicate how long
the notifications logs should be retained, by default that duration is
30 days:
```
BLACKDUCK_HUB_NOTIFICATIONS_DELETE_DAYS=7
```


#### Audit_Events

TODO:  confirm:    
There is no automatic cleanup of Audit Events as of 2020.6.1. These
events are required for a few days but other logging has now been added
to the Black Duck tool so these Events are no longer needed as logging.
They do not need to be kept longer than 10 days.

As such, a psql command should be run periodically to clean up the
Audit events:
```
delete from st.audit_event where event_timestamp < now() - interval '10 days';
```






### Remove any orphaned large objects

If the Customer generates many default reports, this could result in
"orphaned" large objects
(<https://www.postgresql.org/docs/9.5/vacuumlo.html>).

The customer should run "vacuumlo" to remove any orphaned large
objects.

#### vacuumlo

vacuumlo is a simple utility program that will remove any
"orphaned" large objects from a PostgreSQL database. An orphaned
large object (LO) is considered to be any LO whose OID does not appear
in any oid or lo data column of the database.

Example:

```
vacuumlo bds_hub (TODO:  Verify)
```



### Run PostgreSQL tuning utility

The customer should run pgtune (PostgreSQL Config Tuner) or a similar
PostgreSQL tuning utility on the PostgreSQL database. This is not
required but highly recommended. Detailed instructions are available
upon request from Synopsys support.

#### Pgtune

pgtune takes the wimpy default postgresql.conf and expands the
database server to be as powerful as the hardware it is being deployed
on. It is available from <https://github.com/gregs1104/pgtune/>.

Example: 

```
date ; hostname ; pgtune --type OLTP --connections=800 -i /tmp/postgresql.conf -o /tmp/postgresql.conf.pgtune
diff /tmp/postgresql.conf /tmp/postgresql.conf.pgtune
```

If this has been done as part of the database server setup, then this
does not need to be redone unless the server configuration has changed
(e.g. memory, cores, storage)




### Full Vacuum of Database

Due to the vestiges that are left behind by notifications, alerts, and
scans and project-versions that are created and deleted, Synopsys
recommends periodic Full Vacuums. These will clean up the unused space
that autovacuums will not.

You should also do a Full Vacuum prior to an upgrade if you have not done a full vacuum 
recently.

NOTE: It takes about 24 hours to Auto Vacuum 1 TB of space.


#### Make sure the database pgdata partition has enough space

A Full Vacuum copies the table being vacuumed to a new table name. It
cleans that new table, swaps it with the production table and then
deletes the temp table. As such, it is recommended that the storage
holding the database has 150% of the size of the largest table (100% +
buffer).

You can find the table sizes by using the \l+ and \dt+ psql
commands:

```
psql -h 127.0.0.1 -p 55436 -U blackduck -d bds_hub -c "\l+" 
psql -h 127.0.0.1 -p 55436 -U blackduck -d bds_hub -c "\dt+ st.*"
df -hPT
```

You can also get a sorted list by running the following. NOTE: It
might to help to save the script in a file and run it using the -f
option.

```
SELECT
    relname AS "relation",
    pg_size_pretty (
        pg_total_relation_size (C .oid)
    ) AS "total_size"
FROM
    pg_class C
LEFT JOIN pg_namespace N ON (N.oid = C .relnamespace)
WHERE
    nspname NOT IN (
        'pg_catalog',
        'information_schema'
    )
AND C .relkind <> 'i'
AND nspname !~ '^pg_toast'
ORDER BY
    pg_total_relation_size (C .oid) DESC;
```

The results look like:
```
relation | total_size
--------------------------------------------+------------
scan_file | 59 GB
scan_composite_leaf | 40 GB
scan_composite_element | 7639 MB
scan_match_node | 835 MB
```

For vacuuming, ensure available disk space is at least the 1.2 *
the size of the largest database table. If you are planning on
vacuuming tables in parallel, you should have 1.5* the total database
size in available disk space.

Refer to the "Installing Black Duck using Docker Swarm" document,
 Chapter 6: "Upgrading Black Duck", "Upgrading from an existing Docker
 architecture" section.

 The data migration will temporarily require an additional free disk
 space at approximately 2.5 times your original database volume. Add in
 stuff on pg_dump being to a different partition. Pg 61 of the install
 guide 6.1

 pg 61 install guide: The data migration will temporarily require an
 additional free disk space at approximately 2.5 times youroriginal
 database volume size to hold the database dump and the new database
 volume. As a rule-of-thumb, if the volume upon which your database
 resides is at least 60% free, there should be enough diskspace. ????

#### Perform Full Vacuum

Make sure that any monitoring software (e.g. Zenoss) has been
terminated.

<TODO:  Fill in Steps>

Monitor the vacuum to see if it is still going

select current_timestamp - query_start as
runtime,pid,datname,usename,query from pg_stat_activity where query !=
'<IDLE>' and query not in ('COMMIT','ROLLBACK')order by 1
desc;

Look for "VACUUM" in the results

```
      runtime         |  pid   | datname  | usename  |

                                                      query
------------------------+--------+----------+----------+-----------------------------------
-------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------
---------------------------------------------------------
 2 days 04:31:37.696708 |  39364 | repmgr   | repmgr   |  INSERT
INTO repmgr.events (
        node_id,              event,              successful,          
   details
    )       VALUES (\$1, \$2, \$3, \$4)    RETURNING event_timestamp
 2 days 04:23:14.553972 |  42093 | bds_hub  | postgres |
VACUUM(FULL, ANALYZE, VERBOSE)  st
.scan_file
 00:00:03.262079        |  37681 | repmgr   | repmgr   | INSERT INTO
repmgr.monitoring_hist
ory            (primary_node_id,             standby_node_id,      
      last_monitor_time
,             last_apply_time,            
last_wal_primary_location,             last_wal_
standby_location,             replication_lag,             apply_lag
)      VALUES(1,
       2,             '2020-08-10 21:12:41.657707-04'::TIMESTAMP WITH
TIME ZONE,
  '2020-08-10 21:12:36.640652-04'::TIMESTAMP WITH TIME ZONE,          
  '6B5E/18FC058',
          '6B5E/18FC058',             0,             0)
```





























## Upgrade O/S, Kernel, Docker, Postgresql

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



### O/S and/or Kernel upgrade

Working with your server admins for the downtime required to upgrade
the O/S and/or kernel. Sometimes the upgrade occurs because a patch
(security or bugfix) is required. Sometimes it is because Black Duck
will no longer be supporting the version you are using.

Work with your system admins to perform the upgrade.



### Docker Upgrades

At this point, it looks like there is no "upgrade" per se. The
currently supported version is 18.03. However 19.03 of Docker is
available. So, to "upgrade", the old Docker installation would have to
be uninstalled and the new Docker installation would have to be
installed.

<Need to test in order to verify that images and cert secrets do not have to be reinstalled>



### PostgreSQL Upgrades

Work with your Database Administrators to upgrade your Postgresql
instance. 9.6.x was previously supported in both the container and
external databases. 11.7.x is supported in the external databases as of
Black Duck 2020.6. But, since 11.7.x was not supported until Black Duck
2020.6.x, you need to upgrade Black Duck to at least 2020.6.0 before
upgrading Postgresql.


















## Replicate Production Database over into Staging Environment

Sometimes issues arise due to scale. Unless you have created a test
database with the possible scaling issues of the production database,
issues may not be detected until the upgrade occurs in production.

So, one way to do that is to build the staging database with similar
data as in production.

Another way is to replicate the production data in the staging
environment. This is easier with an external database.






## Duplicate Production Environment in Staging Environment

Just as with the creation of a similar/exact copy of the production
database in the test/staging environment, the same should be done with
the hardware (CPU Cores, RAM, storage access (iSCSI vs NFS), storage
type (spinning disk vs flash), amount of storage, and networking (including proxies and firewalls). The versions of
Linux, Docker, and PostgreSQL should also be the same.

Example: if Staging environment cannot "phone home", but Production can, that discrepancy should be resolved prior to upgrading Black Duck. 



## Download images, files, scripts

Download and review all required files.  Resolve any quuestions or issues.


### Download upgrade images

Any new docker images used by the Black Duck application should be
downloaded to the target server prior to the upgrade. Downloading and
adding the new images can be done any time prior to the upgrade.

These are available at a location specified by your Synopsys Support
person.

Once they are on the target server, you can load them into docker:

```
ls | while read image; do docker load -i \$image; done 
```


### Download Orchestration Files

The Orchestration .yml and .env determine how the Black Duck
application will startup with which functionality and resources.

These need to be downloaded from github
(<https://github.com/blackducksoftware/hub>) and updated to include
customer-specific changes as indicated in the orchestration files for
the previous release.

This may take a while so be careful with the updates to these files.

#### Download updated YML Files

If Synopsys Support is providing updated YML files, then they will be
dropped on the Synopsys sharefile or in the Support case


### Download blackduck_migrator yaml file 

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



### Download Upgrade-Monitoring Scripts

Test scripts may be provided by Synopsys to help with monitoring
upgrades.

For instance, SynopsysMonitorDbActivity_202007.bash is used to
monitor PostgreSQL activity at the database/table level. It is to be
used if the upgrade processing seems to have stalled (presumably due to
a very large database table upgrade/migration).







## Update stakeholders about near-term upgrade Schedule

Given that the Black Duck server will be up and down during the
upgrade, it is important that Customer non-Black-Duck external scripts, scans, etc should not be
running during the upgrade.

Planning with Synospys Support should be made sufficiently ahead of
time so that they can plan support during your upgrade. In addition,
those Customer teams using Black Duck need to plan their release schedules to
will want to work around the schedule and put the downtime into their
schedules. 











# Perform Black Duck Upgrade

Once you environment is where you want it, you need to actually
perform the upgrade. The steps are the same for staging as for
production

## Perform Pre-upgrade steps

### Check Memory

As with all application it is important to make sure that there is
sufficient RAM and storage for the upgrade. Changes to orchestration,
additions of services, etc may result in more usage or RAM and/or
storage. The new images will take up more space in docker. 

```
free -g
```
### Check Storage

<TODO:  elaborate.   check storage. Add in comment on pruning containers,
volumes, and system/images?> /opt/docker/..... copy and paste here.>

```
docker volume prune TODO: elaborate
```
















## Perform Upgrade, Day of Upgrade (Staging first, then Production)

The upgrades normally take enough time to bring down Black Duck,
restart Black Duck, and run through any Database Migration scripts.
Upgrading from __________ to ___________ is expecting to take under __________
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

Wait for the scan jobs to finish before terminating external
connections in the next step

After the scan jobs are complete, the customer should terminate all
external queries, utilities, scripts, and SDK calls prior to starting
the actual upgrade.

You can verify this by doing the following psql command:

```
psql -h hub-stg-db -U blackduck -p 5432 -d bds_hub -c "SELECT * FROM pg_stat_activity WHERE datname = 'bds_hub' and state = 'active';"

bds_hub=\# SELECT * FROM pg_stat_activity WHERE datname = 'bds_hub' and state = 'active';
```
```
datid | datname | pid | usesysid | usename | application_name |
client_addr | client_hostname | client_port | backend_start
| xact_start | query_start | state_change | wait_event_type
| wait_event | state | ba
ckend_xid | backend_xmin | query
-------+---------+------+----------+-----------+------------------+---------------+-----------------+-------------+----------------------------
---+-------------------------------+-------------------------------+-------------------------------+-----------------+------------+--------+---
----------+--------------+--------------------------------------------------------------------------------
16441 | bds_hub | 8722 | 16440 | blackduck | psql |
10.251.22.151 | | 35794 | 2020-08-04 22:56:43.585215-
04 | 2020-08-04 22:56:59.671193-04 | 2020-08-04 22:56:59.671193-04
| 2020-08-04 22:56:59.671215-04 | | | active |
| 94647657 | SELECT * FROM pg_stat_activity WHERE datname =
'bds_hub' and state = 'active';
```


#### Stop External Connections

Customer to ensure all Black Duck and PostgreSQL server activities are
as quiet as possible before starting upgrade. So, to that end, the
customer should ensure that no external connections to the database will
interfere with upgrade. This is because external database connections
can block upgrade database actions.

To ensure this:

-   Make backup of pg_hba.conf file.
-   Edit pg_hba.conf to permit connections only from
    localhost/127.0.0.1 and the upgrading Black Duck server. 
-   Set most "host" settings to "reject" or comment them out. 
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
-   If not already done, backup external database with pre-upgrade-version of  
    hub_create_data_dump.sh



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

List the Docker services if you do not know the name:
```
docker stack ls
NAME SERVICES
hub 10
```

Stop the Docker services associated with the existing Black Duck
instance

```
docker stack rm <hub stack name>
```

In order to watch the Docker Services until they all shutdown, do the
following:
```
watch docker ps 
```

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

```
docker stack deploy -c blackduck_migrator-<target-upgrade-version>.yml -c docker-compose.local-overrides.migrator.yml hub
```

Run the script (SynopsysMonitorDbActivity_202007.bash) to see when
the Database Migrations are done. In the output (see below), look at the
"v_hub script status". From top down are the latest Database Migration
scripts that were run. Check the "version" column for the script \# and
the success column to see if it successfully completed (t). Synopsys
Support can tell you what the latest Database Migration script version
that is run with the target release.

```
v_hub script status...
now | inet_server_addr | cmd | loop | installed_rank | version
| description
| type | script | checksum | installed_by | insta
lled_on | execution_time | success
-------------------------------+------------------+---------------------+------+----------------+---------------+----------------------------------------
-----------------+------+-------------------------------------------------------------------------------------+-------------+--------------+-------------
---------------+----------------+---------
2020-08-05 17:27:25.254976-04 | 10.251.22.104 |
v_hub_script_status | 1 | 185 | 2020.06.0.011 | clear stale policies from central release | SQL |
hub_V2020_06_0_011__clear_stale_policies_from_central_release.sql.vpp
| -1470604476 | blackduck | 2020-08-05 1
9:26:12.882272 | 18 | t
```


#### Monitor upgrade docker containers health.

Verify that Black Duck starts up correctly. Once all the Upgrade
docker containers are healthy, the upgrade installation is complete. 

You can monitor the services as follows:

```
watch docker ps
```





#### Restore database with pre-upgrade version of hub_db_migrate.sh (TODO: confirm)


#### Monitor the Black Duck database during Database Migration

During large database table migration steps, the system may appear
quiet or stuck for many hours. 

To confirm progress, do the following:


##### Tail the container logs

```
for mcontainer in \$(docker ps -q) ; do docker ps | grep \$mcontainer ; docker logs --tail 2 \$mcontainer |& cut -c 1-150 ; done 
```


##### Check PostgreSQL activity

```
export PGPASSWORD='<PSQL Database Password>'

./SynopsysMonitorDbActivity_202007.bash > SynopsysMonitorDbActivity_202007.out 2>&1
```




#### Start the official release of Black Duck

Once the Database Migration is complete, bring the Docker containers
back down and start the official release:

Stop Docker

```
docker stack rm hub
```

Local Database:

```
docker stack *deploy* -c docker-compose.yml -c docker-compose.local-overrides.yml <stack name>
```

e.g.:
```
docker stack *deploy* -c docker-compose.yml -c docker-compose.local-overrides.yml hub
```


External Database:

```
docker stack *deploy* -c docker-compose.externaldb.yml -c docker-compose.local-overrides.yml <stack name>
```

e.g. 
```
docker stack *deploy* -c docker-compose.externaldb.yml -c docker-compose.local-overrides.yml hub
```



#### Monitor upgrade docker containers health.

Verify that Black Duck starts up correctly. Once all the Upgrade
docker containers are healthy, the upgrade installation is complete.
NOTE that the nginx service will show up and disappear until all of the
other services are up and healthy and then it will startup correctly.

You can monitor the services as follows:

```
watch docker ps
```

#### Docker service replicas

Customers should implement container replica counts in their .yml files.   If customer is
using replicas, but not via the .yml files, then Customr should increase replicas manually.




#### Vacuum audit_event table 

TODO: is this redundant here?:   When the migration script is finished, and if Customer has not done
"vacuum full analyze" prior to the upgrade, or if Customer has not done a database restore,
then Synopsys strongly recommends
that you run the VACUUM command on the audit_event table to optimize
PostgreSQL performance. See "Installing Black Duck using Docker Swarm",
Chapter 6: "Upgrading Black Duck", "Migration script to purge unused
rows in the audit event table". 







## Perform Post-Upgrade Steps

### Wait...

After an upgrade, Black Duck performs a tremendous amount of work on the database behind the scenes.   
IMPORTANT:  wait for system to quiesce before continuing.  
This could take hours on a very large Black Duck installation. 


### Run upgrade-validation tests.

Customer run the tests required to verify that the existing projects
and process were not impacted unexpectedly by the upgrade.


### Run benchmark tests. 

Customer run the benchmark tests that they ran prior to the upgrade
(see above)

For example, sysbench:

```
date ; date --utc ; hostname -f ; pwd ; whoami ; nproc ; free -g ;
time for mthreads in 4 8 16 ; do echo test threads \$mthreads ; sysbench
fileio --file-test-mode=rndrw --threads=\$mthreads run | grep -e
"\(read|writ\).*/" ; done
```


### Announce upgrade completion (Staging or Production) to stakeholders

When the upgrade is successfully complete, Customer will update their
internal customers, as well as the Synopsys Salesforce Community Case,
with the appropriate status to provide an "all clear".





# Contingency: Fallback Steps

We need to talk about these.  (TODO:  needs work)

- If at any point in the migrations, an unrecoverable error occurs,
    the plan is to fallback to version the previous version.
- In order to fall back to the pre-upgrade version, first remove the volumes.
    - Start up pre-upgrade version, and THEN Run 'docker volume prune' after
        verifying no other volumes are in use. 
    - Alternatively, 'docker volume ls' to list and docker volume rm
        <volume name> each related volume. 
- Using the restore commands to re-establish the pre-upgrade version of the backup
    taken during upgrade steps
    - Expect about _________ hours to fallback to the pre-upgrade version.




## Restore steps, during Fallback

TODO:  augment this section

- To restore the PostgreSQL data during a fallback 
- Use the docker-compose.dbmigrate.yml file located in the
    docker-swarm directory. That starts the containers and volumes
    needed to migrate the database. 

```
docker stack deploy -c docker-compose.dbmigrate.yml hub 
```

- Note that there are some versions of Docker where if the images
    live in a private repository, docker stack will not pull them unless
    the following flag is added to the above command: 

```
--with-registry-auth 
```

- After the DB container has started, run the migration script
    located in the docker-swarm directory. This script restores the data
    from the existing database dump file. 
    - This has a duration on the order of hours, so perform it in a
        screen multiplexer like screen or tmux, or as a detached
        background process.

```
./bin/hub_db_migrate.sh <path to dump file>
```
