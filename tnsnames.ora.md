 # tnsnames.ora Configuration Guide

## File Location

Check Oracle version of home:
```bash
echo $ORACLE_HOME
```

Configuration file path:
```
$ORACLE_HOME/network/admin/tnsnames.ora
```

## Syntax

```
<service_name> =
(DESCRIPTION =
(ADDRESS = (PROTOCOL = TCP)(HOST = <HOSTNAME>)(PORT = 1521))
(CONNECT_DATA =
(SERVER = DEDICATED)
(SERVICE_NAME = <service_name>)
)
)
```

## Explanation

`tnsnames.ora` works as a DNS record. This means you define necessary details in this file to connect to the database and then you can call the name of the service to connect.

Usually this config file is on the client machine, or on the server in case of need for remote service resolution.

## SERVER Options

Explanation of possible options for `SERVER`:

1. **DEDICATED**: This is the default and specifies a dedicated server process for each client connection.

2. **SHARED**: Specifies the use of a shared server (multi-threaded server) configuration. In this mode, multiple client connections can share a single server process. It can be more efficient when dealing with many concurrent connections, but it may have some limitations.

3. **POOLED**: Specifies the use of a connection pooling server. In this mode, a connection pool is created, and client connections are assigned from the pool. Connection pooling can improve performance and resource utilization in applications with many short-lived connections.

4. **SHARED_THREAD**: Specifies a shared server configuration with threads instead of processes. This option is available in some Oracle configurations and is designed to provide a balance between dedicated and shared server models.