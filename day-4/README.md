# Day 4 – Advanced IPC in Q and Tickerplant Architecture

## Learning Objectives

By the end of Day 4, you will understand:

- How IPC works in KDB+/Q
- Message handlers and synchronous vs asynchronous communication
- The role of callbacks in event-driven processing
- Key system commands and variables used in Q processes
- The `.Q` and `.z` namespaces and their responsibilities
- The architecture of a tickerplant system and its relationship to RDB and HDB
- How log files and replay support recovery and historical reconstruction

---

## Module 13: IPC

### 13.1 What is IPC in KDB+/Q?

Inter-process communication (IPC) is how one KDB+/Q process communicates with another. This is the foundation of real-time systems where many processes work together:

- Feed handlers send market data to a tickerplant
- Tickerplant distributes messages to subscribers
- RDB stores current data for live queries
- HDB stores historical partitions on disk
- Client processes issue queries to live or historical systems

IPC in Q is fast, lightweight, and designed for real-time data movement.

### 13.2 Message Handlers

A message handler is a function that receives and processes messages sent by another process.

In Q, a process can listen on a socket using `hopen` and then respond to messages.

```q
/ Server process listens on port 5001
q -p 5001

/ Client connects
h: hopen `:localhost:5001

/ Server-side handling is usually done via .z.pg or custom function assignments
```

Typical pattern:

```q
/ Simple server function
processMsg:{[msg]
  / msg is incoming payload
  show msg;
  :"ack"
}

/ For synchronous calls
.z.pg: processMsg
```

#### Example: custom handler for request/reply

```q
reply: { [msg]
  :"processed: ", string msg
}

.z.pg: { [msg]
  $[msg ~ `ping; "pong";
    msg ~ `status; "running";
    reply msg]
}
```

#### Key idea

- `.z.pg` handles synchronous requests
- The handler decides what to do with the request
- Response is returned to the calling process

### 13.3 Synchronous and Asynchronous Calls

#### Synchronous calls

A synchronous call blocks until the remote process responds.

```q
h: hopen `:localhost:5001
result: h "1 + 1"
```

This is useful when the caller needs an immediate reply.

Characteristics:

- Blocking behavior
- Caller waits for result
- Simpler flow control
- Useful for request/reply logic

#### Asynchronous calls

An asynchronous call does not block. The sender sends a message and continues.

```q
neg[h; "processTrade[]"]
```

`neg` returns a handle that allows asynchronous communication.

Typical async pattern:

```q
/ Send an async request
neg[h; (`cmd; `getData; `AAPL)]
```

Characteristics:

- Non-blocking
- Higher throughput
- Good for streaming and event-driven systems
- More complex to manage responses

#### When to use which?

Use synchronous calls when:

- immediate feedback is required
- logic depends on the result
- simplicity is more important than throughput

Use asynchronous calls when:

- you are distributing feed updates
- you want low latency and high throughput
- the process is event-driven

### 13.4 Callbacks

Callbacks are functions invoked when an asynchronous task completes or when an event occurs.

Example pattern:

```q
handler:{[x]
  show "callback fired";
  show x
}

asyncTask:{[cb; payload]
  result: payload + 10;
  cb result
}

asyncTask[handler; 5]
```

In real Q systems, callbacks often support event-driven logic:

```q
.onTrade: { [trade]
  show trade
}

.onQuote: { [quote]
  show quote
}
```

Callbacks are especially useful in:

- event-driven architectures
- distributed processing
- streaming market-data systems
- asynchronous UI or service workflows

#### Example: async request with callback

```q
req:{[server; query; cb]
  neg[server; (`query; query)];
  cb[]
}
```

In real applications, callback functions often update state, log messages, and trigger downstream processing.

---

## Module 14: System Commands and Variables

Q gives access to many system tools and variables through the `.z` namespace and system commands.

### 14.1 Common system commands

```q
system "ls"
system "pwd"
```

Command examples:

```q
\l myscript.q       / load a script
\cd /path/to/dir    / change directory
\w                  / memory used
\q                  / quit
```

### 14.2 Timing and environment variables

```q
.z.d
.z.t
.z.P
.z.z
.z.h
.z.w
```

Examples:

```q
q).z.d
2024.10.03

q).z.t
12:34:56.789

q).z.h
"host01"

q).z.w
-1
```

### 14.3 Useful system commands for debugging

```q
\d .
\a
\v
\f
```

These are helpful for inspecting the current environment and loaded names.

### 14.4 Why they matter

System variables and commands are crucial in:

- debugging live processes
- monitoring memory and availability
- instrumenting trade systems
- reading host and connection information
- supporting monitoring and operational alerts

---

## Module 15: .Q and .z Namespaces

### 15.1 The `.Q` namespace

The `.Q` namespace contains standard library functions used across Q applications.

Examples:

```q
.Q.ty
.Q.ind
.Q.dpft
.Q.s
.Q.j
```

These functions help with:

- type inspection
- table operations
- file and partition management
- serialization and conversion

Example:

```q
q).Q.ty 1 2 3
`i`i`i
```

### 15.2 The `.z` namespace

The `.z` namespace contains system callbacks and connection hooks.

Important examples:

```q
.z.pg     / synchronous get
.z.ps     / synchronous set
.z.po     / connection open
.z.pc     / connection close
.z.ws     / websocket handler (in some contexts)
.z.pw     / password validation
```

#### Example: log connection open

```q
.z.po:{[x]
  show "connected:", string x
}
```

#### Example: synchronous query handling

```q
.z.pg:{[x]
  value x
}
```

#### Example: custom error logging

```q
.z.pc:{[x]
  show "client disconnected:", string x
}
```

### 15.3 Why namespaces matter

Namespaces help keep system functions distinct from user-defined functions.

- `.Q` = standard library functions
- `.z` = process environment and lifecycle hooks
- custom namespaces like `.trade`, `.tp`, `.rdb`, `.hdb` organize production code logically

Example:

```q
.trade.calcCommission:{[total; rate] total * rate}
.rdb.queryHist:{[sym] select from trades where sym = sym}
```

This avoids name collisions and makes large systems manageable.

---

## Module 16: Tickerplant Architecture

### 16.1 What is a tickerplant?

A tickerplant (`TP`) is the central message distribution component in a KDB+/Q real-time architecture. It accepts updates from feed handlers and publishes them to subscribers.

Typical architecture:

```text
Feeders --> Tickerplant --> RDB --> HDB
              |
              +--> Subscribers / monitoring / analytics
```

### 16.2 Core responsibilities of a tickerplant

The tickerplant usually handles:

- ingesting incoming market data
- logging events to disk
- distributing updates to subscribers
- replaying log files during recovery
- acting as the source of truth for real-time data flow

### 16.3 Log files

Tickerplants write updates to log files for persistence and recovery.

This is essential because:

- a process may crash
- data must be replayed after restart
- historical reconstruction is required

Typical log pattern:

```q
logEntry:(`trade; tradeData)
```

The TP stores messages in an append-only format, preserving chronological order.

### 16.4 Replay

During restart or recovery, the TP replays log data to rebuild state.

Conceptually:

```q
replay:{[logfile]
  / read log file sequentially
  / apply each message to in-memory state
  / rebuild tables or subscriptions
}
```

Replay is important because it allows:

- consistent state restoration
- recovery after process failure
- reconstruction of historical or near-real-time data

### 16.5 RDB (Real-time Database)

The RDB stores the latest in-memory state for current queries.

Responsibilities:

- receive live updates from the tickerplant
- maintain recent market state
- answer low-latency queries
- persist to disk periodically or end-of-day

Typical flow:

```q
/ RDB receives live updates
.z.pg:{[msg]
  if[msg[0]=`trade; trades,: msg[1]]
}
```

The RDB is optimized for speed and current-state access.

### 16.6 HDB (Historical Database)

The HDB stores historical data on disk, typically partitioned by date.

Common layout:

```text
/data/hdb/2024.10.01/
/data/hdb/2024.10.02/
/data/hdb/2024.10.03/
```

The HDB is used for:

- historical analysis
- backtesting
- long-term queries
- end-of-day reporting

### 16.7 Full architecture flow

```text
            +-------------------+
            | Feed Handlers     |
            | (market data)     |
            +---------+---------+
                      |
                      v
            +-------------------+
            | Tickerplant       |
            | - receives data   |
            | - logs updates    |
            | - distributes     |
            +----+----------+---+
                 |          |
                 v          v
             +-----+     +------+
             | RDB |     | Subs |
             | live|     | live |
             +--+--+     +------+
                |
                v
             +-----+
             | HDB |
             | disk|
             +-----+
```

### 16.8 Simple tickerplant example

```q
/ Example TP state
quotes: ([] time:`time$(); sym:`symbol$(); bid:`float$(); ask:`float$())
subs: ()

publish:{[msg]
  {neg[x; msg]} each subs
}

handleQuote:{[quote]
  quotes,: quote;
  publish quote
}

subscribe:{[h]
  subs,: h
}
```

This is a minimal example, but it captures the core idea:

- new data comes in
- it is appended to state
- it is broadcast to subscribers

### 16.9 Why the TP architecture matters

This pattern is the standard design for financial real-time systems because it separates:

- ingestion
- distribution
- live analytics
- historical storage

This separation improves scalability, fault tolerance, and operational clarity.

---

## Module 17: Assessment

### 17.1 Knowledge check

Answer these questions to confirm understanding:

1. What is IPC in KDB+/Q?
2. What is the difference between synchronous and asynchronous calls?
3. Why are message handlers important in distributed Q systems?
4. What is the role of callbacks in real-time processing?
5. What are the primary responsibilities of `.z` and `.Q`?
6. What is a tickerplant and why is it central to the architecture?
7. What is the difference between RDB and HDB?
8. Why are log files important in a tickerplant-based system?
9. How does replay rebuild system state after a restart?
10. Where would you place the responsibility for live queries and historical queries in the architecture?

### 17.2 Practical exercises

#### Exercise 1: Create a simple async message flow

```q
server:{[msg]
  :"received: ", string msg
}

/ Simulate sending a message to a server process
show server `ping
```

#### Exercise 2: Use `.z.pg` for simple request handling

```q
.z.pg:{[msg]
  $[msg like "sum*"; "ok"; "unknown"]
}
```

#### Exercise 3: Design a minimal tickerplant

Create a simple Q process that:

- accepts incoming trade messages
- logs them
- publishes them to subscribers
- allows replay from log files

#### Exercise 4: Explain architecture

Describe the relationship between:

- feed
- tickerplant
- RDB
- HDB
- subscribers

### 17.3 Suggested project outcome

By the end of Day 4, a student should be able to explain:

- how Q processes communicate
- how a real-time market-data system is organized
- how tickerplant, RDB, and HDB work together
- why replay and persistence matter in production systems

---

## Day 4 Summary

Day 4 bridges the gap between basic Q programming and real-time system design.

Key takeaways:

- IPC enables communication between KDB+/Q processes
- Message handlers are essential for request/reply and event processing
- Async communication is useful for throughput-intensive systems
- `.z` and `.Q` provide critical runtime behaviors and utilities
- A tickerplant is the central hub of a live market-data architecture
- RDB and HDB separate current-state and historical storage
- Log replay is a core resilience mechanism

This is the foundation for building modern KDB+/Q high-performance data-processing systems.

---

## Additional Reading Suggestions

- KX documentation on `.z` namespace
- KX documentation on IPC and client/server models
- Tickerplant architecture guides from KX and production implementations
- Q examples for async feeds and real-time market data distribution

---

## End of Day 4

This completes the Day 4 module of the KDB Fundamentals + Advanced course.

The next stage is a review and assessment-focused wrap-up to consolidate understanding before moving deeper into real-world system implementation.
