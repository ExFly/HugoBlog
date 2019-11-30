---
title: "Concurrency Control Theory"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: ["db"]
date: 2019-11-30T13:21:25+08:00
draft: true
---

文章简介：并发控制理论

<!--more-->

# ACID

- Atomicity: All actions in the transaction happen, or none happen.
  “All or Nothing”
- Consistency: If each transaction is consistent and the database is consistent at the beginning of the
  transaction, then the database is guaranteed to be consistent when the transaction completes.
  “It looks correct to me...”
- Isolation: The execution of one transaction is isolated from that of other transactions.
  “As if alone”
- Durability: If a transaction commits, then its effects on the database persist.
  “The transaction’s changes can survive failures...”

> [ACID](https://15445.courses.cs.cmu.edu/fall2019/notes/16-concurrencycontrol.pdf)

## ATOMICITY

### Approach #1: Logging

- DBMS logs all actions so that it can undo the actions of aborted transactions.
- Maintain undo records both in memory and on disk.
- Think of this like the black box in airplanes…

### Approach #2: Shadow Paging

- DBMS makes copies of pages and txns make changes to
  those copies. Only when the txn commits is the page
  made visible to others.
- Originally from System R.

CouchDB, OpenLDAP

## Consistency

### Database Consistency:

- The database accurately represents the real world entity it is modeling and follows integrity constraints.
- Transactions in the future see the effects of transactions committed in the past inside of the database.

### Transaction Consistency:

- If the database is consistent before the transaction starts, it will also be consistent after.
- Ensuring transaction consistency is the application’s responsibility.

## Isolation

两种 并发控制 方式：乐观，悲观

• Read-Write Conflicts (“Unrepeatable Reads”): A transaction is not able to get the same value when
reading the same object multiple times.
• Write-Read Conflicts (“Dirty Reads”): A transaction sees the write effects of a different transaction
before that transaction committed its changes.
• Write-Write conflict (“Lost Updates”): One transaction overwrites the uncommitted data of another
concurrent transaction

## Durability

All of the changes of committed transactions must be durable (i.e., persistent) after a crash or restart. The
DBMS can either use logging or shadow paging to ensure that all changes are durable.

# Two-Phase Locking ((2PL)

> [A Survey of B-Tree Locking Techniques](https://15721.courses.cs.cmu.edu/spring2019/papers/06-indexes/a16-graefe.pdf)

## 2PL Deadlock Handling

## Approach #1: Deadlock Detection

The DBMS creates a waits-for graph: Nodes are transactions, and edge from Ti
to Tj if transaction Ti
is
waiting for transaction Tj to release a lock. The system will periodically check for cycles in waits-for graph
and then make a decision on how to break it.
• When the DBMS detects a deadlock, it will select a “victim” transaction to rollback to break the cycle.
• The victim transaction will either restart or abort depending on how the application invoked it
• There are multiple transaction properties to consider when selecting a victim. There is no one choice
that is better than others. 2PL DBMSs all do different things:

1. By age (newest or oldest timestamp).
2. By progress (least/most queries executed).
3. By the # of items already locked.
4. By the # of transactions that we have to rollback with it.
5. # of times a transaction has been restarted in the past
   • Rollback Length: After selecting a victim transaction to abort, the DBMS can also decide on how
   far to rollback the transaction’s changes. Can be either the entire transaction or just enough queries to
   break the deadlock.

## Approach #2: Deadlock Prevention

When a transaction tries to acquire a lock, if that lock is currently held by another transaction, then perform
some action to prevent a deadlock. Assign priorities based on timestamps (e.g., older means higher priority).
These schemes guarantee no deadlocks because only one type of direction is allowed when waiting for a
lock. When a transaction restarts, its (new) priority is its old timestamp.
• Wait-Die (“Old waits for Young”): If T1 has higher priority, T1 waits for T2. Otherwise T1 aborts
• Wound-Wait (“Young waits for Old”): If T1 has higher priority, T2 aborts. Otherwise T1 waits.

# Timestamp Ordering Concurrency Control

# Multi-version concurrency control (MVCC)

 Writers don’t block the readers. Readers don’t block the writers.
Read-only transactions can read a consistent snapshot without acquiring locks. Timestamps are used to
determine visibility.
Multi-versioned DBMSs can support time-travel queries that can read the database at a point-in-time snapshot.
There are four important MVCC design decisions:

1. Concurrency Control Protocol (T/O, OCC, 2PL, etc).
2. Version Storage
3. Garbage collection
4. Index Management

## Concurrency Control Protocol

- Timestamp Ordering
- Optimistic Concurrency
- Two-Phase Locking

## Version Storage

- Append-Only Storage
- Time-Travel Storage
- Delta Storage

## Garbage Collection

Approach #1: Tuple Level Garbage Collection – Find old versions by examining tuples directly
• Background Vacuuming: Separate threads periodically scan the table and look for reclaimable versions, works with any version storage scheme.
• Cooperative Cleaning: Worker threads identify reclaimable versions as they traverse version chain.
Only works with O2N.

Approach #2: Transaction Level – Each transaction keeps track of its own read/write set. When a transaction completes, the garbage collector can use that to identify what tuples to reclaim. The DBMS determines
when all versions created by a finished transaction are no longer visible.

## Index Management

# 事务隔离级别

通过不同的处理 冲突策略，即可实现事务隔离界别

# Logging Protocols + Schemes

## Write-Ahead Logging

The DBMS records all the changes made to the database in a log file (on stable storage) before the change is
made to a disk page. The log contains sufficient information to perform the necessary undo and redo actions
to restore the database after a crash. This is an example of a STEAL + NO-FORCE system.
Almost every DBMS uses write-ahead logging (WAL) because it has the fastest runtime performance. But
the DBMS’s recovery time with WAL is slower than shadow paging because it has to replay the log.

**implementation**
• All log records pertaining to an updated page are written to non-volatile storage before the page itself
is allowed to be overwritten in non-volatile storage.
• A transaction is not considered committed until all its log records have been written to stable storage.
• When the transaction starts, write a <BEGIN> record to the log for each transaction to mark its starting
point.
• When a transaction finishes, write a <COMMIT> record to the log and make sure all log records are
flushed before it returns an acknowledgment to the application.
• Each log entry contains information about the change to a single object:
– Transaction ID.
– Object ID.
– Before Value (used for UNDO).
– After Value (used for REDO).
• Log entries to disk should be done when transaction commits. You can use group commit to batch
multiple log flushes together to amortize overhead.

# Crash Recovery Algorithms

> [Algorithms for Recovery and Isolation Exploiting Semantics](https://en.wikipedia.org/wiki/Algorithms_for_Recovery_and_Isolation_Exploiting_Semantics)

# references

- https://15445.courses.cs.cmu.edu/fall2019/schedule.html 视频 笔记 slice
- https://catkang.github.io/2018/09/19/concurrency-control.html
