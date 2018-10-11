---
layout: default
title: PgBouncer features
---

# Features

-   Several levels of brutality when rotating connections:

     Session pooling
     :  Most polite method. When client connects, a server connection
        will be assigned to it for the whole duration it stays
        connected. When client disconnects, the server connection will
        be put back into pool.

     Transaction pooling
     :  Server connection is assigned to client only during a
        transaction. When PgBouncer notices that transaction is over,
        the server will be put back into pool. This is a hack as it
        breaks application expectations of backend connection. You can
        use it only when application cooperates with such usage by not
        using features that can break. See the table below for breaking
        features.

     Statement pooling
     :  Most aggressive method. This is transaction pooling with a twist
        - multi-statement transactions are disallowed. This is meant to
        enforce "autocommit" mode on client, mostly targeted for
        PL/Proxy.

-   Low memory requirements (2k per connection by default). This is due
    to the fact that PgBouncer does not need to see full packet at
    once.

-   It is not tied to one backend server, the destination databases can
    reside on different hosts.

-   Supports online reconfiguration for most of the settings.

-   Supports online restart/upgrade without dropping client connections.

-   Supports protocol V3 only, so backend version must be \>= 7.4.


## SQL feature map for pooling modes

Following table list various PostgreSQL features and whether they are
compatible with PgBouncer pooling modes.  Note that 'transaction'
pooling breaks client expectations of server and can be used only
if application cooperates by not using non-working features.

|----------------------------------+-----------------+---------------------|
| Feature                          | Session pooling | Transaction pooling |
|----------------------------------+-----------------+---------------------|
| Startup parameters               | Yes [^0]        | Yes [^0]            |
| SET/RESET                        | Yes             | Never               |
| LISTEN/NOTIFY                    | Yes             | Never               |
| WITHOUT HOLD CURSOR              | Yes             | Yes                 |
| WITH HOLD CURSOR                 | Yes [^1]        | Never               |
| Protocol-level prepared plans    | Yes [^1]        | No [^2]             |
| PREPARE / DEALLOCATE             | Yes [^1]        | Never               |
| ON COMMIT DROP temp tables       | Yes             | Yes                 |
| PRESERVE/DELETE ROWS temp tables | Yes [^1]        | Never               |
| Cached plan reset                | Yes [^1]        | Yes [^1]            |
| LOAD statement                   | Yes             | Never               |
|----------------------------------+-----------------+---------------------|

[^0]:
    Startup parameters are: **client_encoding**, **datestyle**, **timezone**
    and **standard_conforming_strings**.  PgBouncer can detect their changes
    so it can guarantee they remain consistent for client.  Available
    from PgBouncer 1.1.

[^1]:
    Full transparency requires PostgreSQL 8.3 and PgBouncer 1.1 with
    `server_reset_query = DISCARD ALL`.

[^2]:
    It is possible to add support for that into PgBouncer.

