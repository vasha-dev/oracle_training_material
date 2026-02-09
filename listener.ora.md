# listener.ora Configuration Guide

## File Location

Check Oracle version of home:
```bash
echo $ORACLE_HOME
```

Configuration file path:
```
$ORACLE_HOME/network/admin/listener.ora
```

## Syntax

Basic listener configuration:
```
LISTENER =
(DESCRIPTION_LIST =
(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = <HOSTNAME>)(PORT = 1521))
)
)
)

SID_LIST_LISTENER =
(SID_LIST =
(SID_DESC =
(ORACLE_HOME = /u01/app/oracle/product/19/dbhome_1)
(SID_NAME = ORADB)
)
(SID_DESC =
(ORACLE_HOME = /u01/app/oracle/product/19/dbhome_1)
(SID_NAME = ORADB2)
)
)
```

## Adding a New Listener

### Important Information

One single listener, using the default name of `LISTENER` and the default port of 1521, is quite capable of -- indeed, **WAS DESIGNED TO** -- service multiple databases of multiple versions running from multiple homes. Multiple listeners simply add complications. Only add new listeners based on customer request.

### Creating an Additional Listener

To create an extra listener, it must listen on a unique port. Fill in all `<>` variables:

```
<NAME_OF_NEW_LISTENER> =
(DESCRIPTION_LIST =
(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = <HOSTNAME>)(PORT = 1522))
)
)
)

SID_LIST_<NAME_OF_NEW_LISTENER> =
(SID_LIST =
(SID_DESC =
(ORACLE_HOME = <ORACLE_HOME>)
(SID_NAME = <ORACLE_SID>)
)
(SID_DESC =
(ORACLE_HOME = <ORACLE_HOME>)
(SID_NAME = <ORACLE_SID>)
)
)
```

## Example Configuration

Example of 3 listeners on different ports:

```
JESSE =
(DESCRIPTION_LIST =
(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = walter.dbplatform.local)(PORT = 1521))
)
)
)

SID_LIST_JESSE =
(SID_LIST =
(SID_DESC =
(ORACLE_HOME = /u01/app/oracle/product/19/dbhome_1)
(SID_NAME = JESSE1)
)
)

MIKE =
(DESCRIPTION_LIST =
(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = walter.dbplatform.local)(PORT = 1522))
)
)
)

SID_LIST_MIKE =
(SID_LIST =
(SID_DESC =
(ORACLE_HOME = /u01/app/oracle/product/19/dbhome_1)
(SID_NAME = MIKE1)
)
)

GUS =
(DESCRIPTION_LIST =
(DESCRIPTION =
(ADDRESS_LIST =
(ADDRESS = (PROTOCOL = TCP)(HOST = walter.dbplatform.local)(PORT = 1523))
)
)
)

SID_LIST_GUS =
(SID_LIST =
(SID_DESC =
(ORACLE_HOME = /u01/app/oracle/product/19/dbhome_1)
(SID_NAME = GUS1)
)
)
```

## Protocol Options

### TCP Protocol

Standard communication protocol for network connections:
```
(ADDRESS = (PROTOCOL = TCP)(HOST = ol8-23.localdomain)(PORT = 1521))
```

### IPC Protocol

A local connection protocol. IPC can be used only when the client program and Oracle database are installed on the same system:
```
(ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
```

## Listener Registration

The listener registration process is handled by **LREG**.

### Listener Status Values

1. **Ready**: Instance started in open mode with dynamic registration. Dynamic registration means the instance registers the service automatically to the listener. The PMON process is responsible for registering the service to the listener dynamically. The instance also tells more details to the listener about the instance, such as whether the DB is up or down and more.

2. **Unknown**: Instance started in open mode with static registration. Static registration means the instance service is registered manually in `listener.ora`. Due to manual setup, the listener does not know the detailed state of the instance. This is okay behavior.

3. **Blocked**: Instance is in NOMOUNT mode.

4. **Restricted**: Instance is started with restricted session.