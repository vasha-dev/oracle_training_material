# SGA and PGA Memory Management Guide

## Overview

SGA (System Global Area) and PGA (Program Global Area) are the primary memory structures used by Oracle Database.

## Query Memory Usage

Check total memory usage for SGA and PGA:
```sql
select decode( grouping(nm), 1, 'total', nm ) nm, round(sum(val/1024/1024)) mb
from ( select 'sga' nm, sum(value) val from v$sga union all select 'pga',
sum(a.value) from v$sesstat a, v$statname b where b.name = 'session pga memory'
and a.statistic# = b.statistic# ) group by rollup(nm);
```

List current SGA components usage:
```sql
SET LINESIZE 150
SET PAGESIZE 5000
COLUMN component FORMAT A35
COLUMN current_size FORMAT A15
COLUMN min_size FORMAT A15
COLUMN max_size FORMAT A15
SELECT component,
TO_CHAR(current_size/1024/1024, '999,999,999') || ' MB' AS current_size,
TO_CHAR(min_size/1024/1024, '999,999,999') || ' MB' AS min_size,
TO_CHAR(max_size/1024/1024, '999,999,999') || ' MB' AS max_size
FROM v$sga_dynamic_components
ORDER BY current_size DESC;
```

Check currently allocated buffer cache:
```sql
SELECT NAME, ROUND(BYTES/1024/1024, 2) AS "Size in MB" 
FROM v$sgainfo 
WHERE NAME = 'Buffer Cache Size';
```

Show memory parameters:
```sql
show parameter memory
show parameter sga
show parameter pga
```

## Memory Components

List memory components minimum allocation size:
```sql
SHOW PARAMETER SHARED_POOL_SIZE;
SHOW PARAMETER LARGE_POOL_SIZE;
SHOW PARAMETER JAVA_POOL_SIZE;
SHOW PARAMETER STREAMS_POOL_SIZE;
SHOW PARAMETER DB_CACHE_SIZE;
SHOW PARAMETER DB_KEEP_CACHE_SIZE;
SHOW PARAMETER DB_RECYCLE_CACHE_SIZE;
```

### Component Descriptions

1. **SHARED_POOL_SIZE**: Specifies the size of the shared pool, which stores parsed SQL statements, PL/SQL code, and data dictionary information.

2. **LARGE_POOL_SIZE**: Specifies the size of the large pool, which is used for large memory allocations such as backup and restore operations.

3. **JAVA_POOL_SIZE**: Specifies the size of the Java pool, which is used for all session-specific Java code and data within the Oracle JVM.

4. **STREAMS_POOL_SIZE**: Specifies the size of the Streams pool, which is used by Oracle Streams for capture and apply processes.

5. **DB_CACHE_SIZE**: Specifies the size of the default buffer cache.

6. **DB_KEEP_CACHE_SIZE**: Specifies the size of the KEEP buffer cache, which retains frequently accessed data blocks.

7. **DB_RECYCLE_CACHE_SIZE**: Specifies the size of the RECYCLE buffer cache, which stores less frequently accessed data blocks.

8. **DB_2K_CACHE_SIZE, DB_4K_CACHE_SIZE, DB_8K_CACHE_SIZE, DB_16K_CACHE_SIZE, DB_32K_CACHE_SIZE**: Specify the sizes of the buffer caches for non-default block sizes.

9. **SHARED_IO_POOL_SIZE**: Specifies the size of the shared I/O pool, which is used for large I/O operations.

10. **DATA_TRANSFER_CACHE_SIZE**: Specifies the size of the data transfer cache.

11. **INMEMORY_SIZE**: Specifies the size of the In-Memory area, which is used for the In-Memory Column Store.

12. **ASM_CACHE_SIZE**: Specifies the size of the ASM buffer cache, used by Automatic Storage Management.

## HugePages

HugePages are a memory management feature in Linux that allows the use of larger memory pages than the standard 4KB size. Typically, HugePages are 2MB or larger. HugePages can reduce swapping. When HugePages are enabled, they lock memory, preventing it from being swapped out to disk.

### Check HugePages Size

Formula:
- `HugePages_Total * Hugepagesize` = currently allocated HugePages in GB
- `HugePages_Free * Hugepagesize` = total free in GB

```bash
grep -i huge /proc/meminfo
```

## Backup SPFILE Before Changes

Always backup the SPFILE before making any memory configuration changes:

```bash
export PFILE_TEMP_LOC_VAR="$ORACLE_HOME/dbs/pfile_${ORACLE_SID}_$(date +'%Y_%m_%d_%H_%M').backup"
sqlplus / as sysdba <<EOF
CREATE PFILE='$PFILE_TEMP_LOC_VAR' FROM SPFILE;
EXIT;
EOF
```

Alternative method:
```sql
CREATE PFILE='/home/oracle/pfile_<>_backup' FROM SPFILE;
```

## SGA Configuration (ASMM)

**Note:** Reboot required for changes to take effect.

### Get SGA Target Advice

Query advisory information to determine optimal SGA size:

```sql
SELECT sga_size, sga_size_factor, estd_db_time_factor
FROM v$sga_target_advice
ORDER BY sga_size ASC;
```

#### Column Explanations

- **SGA_SIZE**: This column represents the size of the System Global Area (SGA) in megabytes. The SGA is a shared memory area that contains data and control information for an Oracle database.

- **SGA_SIZE_FACTOR**: This column shows a factor that represents the size of the SGA relative to its current size. For example, a factor of 1.0 indicates the current size, while a factor of 1.5 would indicate a proposed increase to 150% of the current size.

- **ESTD_DB_TIME_FACTOR**: This column provides an estimated factor of database time, which reflects the expected impact on database performance due to changes in the SGA size. A lower factor suggests better performance, while a higher factor indicates potential performance degradation.

### Check Current SGA Settings

If `SGA_TARGET` is set to 0, ASMM is not enabled. If `SGA_TARGET` is set to a non-zero value, ASMM is used (Doc ID 295626.1).

```sql
SHOW PARAMETER SGA_MAX_SIZE;
SHOW PARAMETER SGA_TARGET;
```

### Change SGA Size

**Reboot required** for these changes to take effect:

```sql
ALTER SYSTEM SET SGA_MAX_SIZE = 10G SCOPE=SPFILE;
ALTER SYSTEM SET SGA_TARGET = 10G SCOPE=SPFILE;
```

Bounce the database to apply changes from SPFILE.

## PGA Configuration (ASMM)

**Note:** No reboot needed for PGA changes.

### Check PGA Target Advice

Query advisory information to determine optimal PGA settings:

```sql
SELECT
ROUND(pga_target_for_estimate/1024/1024) target_mb,
estd_pga_cache_hit_percentage cache_hit_perc,
estd_overalloc_count
FROM
V$PGA_TARGET_ADVICE;
```

#### Column Explanations

- **target_mb**: This converts the estimated PGA target from bytes to megabytes and rounds it to the nearest whole number.

- **cache_hit_perc**: This column shows the estimated PGA cache hit percentage for the given PGA target. A higher cache hit percentage indicates more efficient use of the PGA memory.

- **estd_overalloc_count**: This column provides the estimated number of times the system will need to allocate more PGA memory than the target. A lower number here is better, as it indicates fewer overallocations.

### Check Current PGA Settings

```sql
SHOW PARAMETER PGA_AGGREGATE_LIMIT;
SHOW PARAMETER PGA_AGGREGATE_TARGET;
```

### Change PGA Size

No reboot required (SCOPE=BOTH allows dynamic changes):

```sql
ALTER SYSTEM SET PGA_AGGREGATE_LIMIT = 13G SCOPE=BOTH;
ALTER SYSTEM SET PGA_AGGREGATE_TARGET = 6G SCOPE=BOTH;
```

## Automatic Memory Management (AMM)

**Important:** Don't use this feature! Although on the surface it looks like an improvement from a simplicity perspective, it is a bad feature. Instead, use the `SGA_TARGET` and `PGA_AGGREGATE_TARGET` parameters to manage your memory. Even Oracle has distanced themselves from it. In later releases it can't be selected in the DBCA for memory configurations in excess of 4G.

## Useful Links

- [Automatic Memory Management 11gR1](https://oracle-base.com/articles/11g/automatic-memory-management-11gr1)
- [Oracle Memory Architecture Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/memory-architecture.html#GUID-1CB2BA23-4386-46DA-9146-5FE0E4599AC6)
- [Additional Reference Document](https://docs.google.com/document/d/17n9kfDxQwdsBjT-5ksk15SUDBr-Ex-GGurgsABJwI84/edit)