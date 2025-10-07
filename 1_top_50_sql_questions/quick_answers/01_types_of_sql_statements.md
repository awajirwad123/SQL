# 1. Types of SQL Statements (DDL, DML, DCL, TCL, DQL)

Q: What are the five broad categories of SQL statements?
A: DDL (define structure), DML (manipulate data), DCL (control access), TCL (manage transactions), DQL (query/read data).

Q: Give one example each.
A: DDL: CREATE TABLE; DML: UPDATE; DCL: GRANT; TCL: COMMIT; DQL: SELECT.

Q: Why separate DDL vs DML?
A: DDL changes metadata (schema) and usually auto-commits; DML changes row contents inside transactions.

Q: Is SELECT part of DML?
A: No—it's DQL (read-only); confusion arises because many group it under DML historically.

Q: Difference DELETE vs TRUNCATE?
A: DELETE = row-by-row, logs each row, can filter; TRUNCATE = deallocate pages, fast, resets metadata, no WHERE.
