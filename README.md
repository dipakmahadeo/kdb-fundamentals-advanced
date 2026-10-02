# KDB Fundamentals + Advanced

KDB Fundamentals 

This repository hosts the training material for a structured five-day KDB+/Q learning path covering fundamentals, table/query work, database concepts, and IPC/tickerplant architecture.

## Course Overview

- Duration: 5 days
- Focus: KDB+/Q language, tables, queries, database design, and advanced IPC concepts
- Audience: Developers and data engineers learning KDB+/Q for real-time analytics and market data systems

## Course Roadmap

### Day 1 – Foundations of Q Programming
- Language background
- Development environment
- Kdb+ startup options
- Starting and exiting a session
- Datatypes: atoms, lists, type numbers, casting
- Lists and dictionaries: indexing and manipulation
- Basic operations: evaluation order, arithmetic, @ and .

### Day 2 – Functions, Namespaces, Tables, and Queries
- Writing functions
- Projection
- Global and local scope
- Iterators and adverbs
- Namespaces and conflict prevention
- Tables: creation, schema, manipulation
- Queries: functional form, grouping, aggregations

### Day 3 – Advanced Table and Query Concepts
- Foreign keys
- Joins: union, plus, inner, as-of, window join
- I/O: loading/saving data files
- Database types: splayed, partitioned, splayed-partitioned, segmented

### Day 4 – Advanced IPC and Tickerplant Architecture
- IPC concepts
- Message handlers
- Synchronous and asynchronous calls
- Callbacks
- System commands and variables
- .Q and .z namespaces
- Tickerplant architecture: tickerplant, log files, replay, RDB, HDB

### Day 5 – Assessment and Review
- Concept review
- Practical exercises
- Architecture recap
- Final assessment

## Repository Structure

- `day-1/` – Q basics and language foundations
- `day-2/` – functions, namespaces, tables, and queries
- `day-3/` – joins, I/O, and KDB database models
- `day-4/` – IPC, systems, and tickerplant architecture
- `day-5/` – assessment and revision notes
- `resources/` – setup and quick reference material

## Suggested Learning Flow

1. Start with the language basics in Day 1.
2. Move to functional programming and tables in Day 2.
3. Learn table joins and database layout on Day 3.
4. Understand the real-time architecture on Day 4.
5. Finish with Day 5 review and assessment.

## Quick Setup

Install KDB+/Q and open a terminal session. Example startup commands depend on your platform and license setup.

```bash
q
```

To exit:

```q
\\
```

## Notes

This repository is intended to act as a living training deck. Material can be expanded with code snippets, labs, and assignments as needed.

## License

This content is provided for training and educational use.
