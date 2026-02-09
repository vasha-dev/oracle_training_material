# Database Creation Guide (DBCA)

## Environment Setup

### Set Oracle Home

Export the Oracle Home directory:
```bash
export ORACLE_HOME=/oracle/app/oracle/product/19/dbhome_1
```

### Define Database Name

Set the SID and global database name:
```bash
export SID_AND_GNAME=<SID_AND_GLOBAL_NAME>
```

### Set Passwords

Set passwords for SYS and SYSTEM users:
```bash
read -p "Input sys/system password:" SYS_PWD
```

### Configure Memory

Total amount of allocated memory (in MB). Oracle will split the allocation between SGA and PGA:
```bash
export ORA_MEMORY=8000
```

### Configure Redo Logs

Redo log size (in MB):
```bash
export REDO=200
```

### Storage Configuration

Define storage type. Options: `FS` or `ASM`:
```bash
export STORAGE_TYPE=FS
```

Oracle Managed Files (OMF). Options: `TRUE` or `FALSE`:
```bash
export USE_OMF=FALSE
```

### Define File Locations

Data location:
```bash
export DATA_DIR=/oracle/oradata
```

Recovery location (FRA):
```bash
export RECO_DIR=/oracle/oraarch
```

## Character Set Configuration

### Check Current Character Set

Query the current character set settings:
```sql
select parameter,value from v$nls_parameters 
where parameter='NLS_CHARACTERSET' or parameter='NLS_NCHAR_CHARACTERSET';
```

### Available Character Sets

1. **AL32UTF8**: Unicode character set that supports all languages. Recommended for new databases.

2. **WE8MSWIN1252**: Western European character set.

3. **UTF8**: Unicode character set that supports a wide range of characters.

4. **AL16UTF16**: Unicode character set used for NCHAR, NVARCHAR2, and NCLOB data types.

5. **US7ASCII**: 7-bit ASCII character set.

6. **WE8ISO8859P1**: ISO 8859-1 Western European character set.

7. **JA16SJIS**: Japanese Shift-JIS character set.

8. **ZHS16GBK**: Simplified Chinese character set.

9. **KO16MSWIN949**: Korean character set.

10. **TH8TISASCII**: Thai character set.

Set the character set:
```bash
export CHAR_SET=WE8MSWIN1252
```

### National Character Set (NCHAR)

**Note:** NCHAR character set can ONLY be set on DBCA GUI. `AL16UTF16` is generally the default option if not otherwise specified in DBCA silent mode.

#### Available NCHAR Options

1. **AL16UTF16**: This is the default national character set for Oracle databases. It is a Unicode character set that uses 2 bytes for each character.

2. **UTF8**: This is a variable-width character set that can represent any character in the Unicode standard. It uses 1 to 4 bytes per character.

3. **AL32UTF8**: This is another Unicode character set that is often used for the database character set but can also be used for the national character set.

## Create Non-CDB Database

Run the DBCA command to create a non-container database:
```bash
nohup $ORACLE_HOME/bin/dbca -silent -createDatabase \
-templateName General_Purpose.dbc \
-gdbname ${SID_AND_GNAME} -sid ${SID_AND_GNAME} \
-responseFile NO_VALUE \
-characterSet ${CHAR_SET} \
-sysPassword ${SYS_PWD} \
-systemPassword ${SYS_PWD} \
-databaseType MULTIPURPOSE \
-automaticMemoryManagement false \
-totalMemory ${ORA_MEMORY} \
-storageType ${STORAGE_TYPE} \
-useOMF ${USE_OMF} \
-datafileDestination ${DATA_DIR} \
-recoveryAreaDestination ${RECO_DIR} \
-redoLogFileSize ${REDO} \
-emConfiguration NONE \
-ignorePreReqs > dbca_${SID_AND_GNAME}_creation_$(date +'%Y_%m_%d_%H_%M').log 2>&1 &
```

## Create CDB Database

### Additional CDB Variables

Define extra variables for CDB creation:
```bash
export PDB_NAME=PDB1
read -p "Input password for PDB admin:" PDB_ADMIN_PASSWORD
```

### Run CDB Creation

Run the DBCA command to create a container database with a pluggable database:
```bash
nohup $ORACLE_HOME/bin/dbca -silent -createDatabase \
-templateName General_Purpose.dbc \
-gdbname ${SID_AND_GNAME} -sid ${SID_AND_GNAME} \
-responseFile NO_VALUE \
-characterSet ${CHAR_SET} \
-sysPassword ${SYS_PWD} \
-systemPassword ${SYS_PWD} \
-databaseType MULTIPURPOSE \
-automaticMemoryManagement false \
-totalMemory ${ORA_MEMORY} \
-storageType ${STORAGE_TYPE} \
-useOMF ${USE_OMF} \
-datafileDestination ${DATA_DIR} \
-recoveryAreaDestination ${RECO_DIR} \
-redoLogFileSize ${REDO} \
-emConfiguration NONE \
-createAsContainerDatabase true \
-numberOfPDBs 1 \
-pdbName ${PDB_NAME} \
-pdbAdminPassword ${PDB_ADMIN_PASSWORD} \
-ignorePreReqs > dbca_${SID_AND_GNAME}_creation_$(date +'%Y_%m_%d_%H_%M').log 2>&1 &
```