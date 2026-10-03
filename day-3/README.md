# Day 3 – More About Tables & Queries, Foreign Keys, Joins, I/O, and Database Types

## Learning Objectives

By the end of Day 3, you will understand:
- Foreign keys and how to reference other tables
- Different types of joins (union, plus, inner, as-of, window)
- Loading and saving data from/to files
- Database storage models (splayed, partitioned, splayed-partitioned, segmented)
- When and how to use each database type for optimal performance
- Building real-time data pipelines

---

## Part 9: Foreign Keys

### 9.1 Concept of Foreign Keys

A **foreign key** is a reference from one table to another. It establishes a relationship between tables where one column in a table references the primary key of another table.

**Why use foreign keys?**

1. **Data integrity**: Ensures consistency across related tables
2. **Avoid duplication**: Store shared data once, reference it many times
3. **Efficient queries**: Leverage relationships for complex queries
4. **Real-world modeling**: Represent actual data relationships

**Example: Symbols and Exchanges**

Instead of storing the full exchange name in every trade record:

```q
/ Without foreign keys (redundant)
q) trades: ([]
  sym: `AAPL`MSFT`GOOGL`AAPL;
  exchange: `NASDAQ`NASDAQ`NASDAQ`NASDAQ;
  price: 150 300 2500 151
)

/ With foreign keys (efficient)
q) exchanges: ([sym: `AAPL`MSFT`GOOGL] exchange: `NASDAQ`NASDAQ`NASDAQ)
q) trades: ([]
  sym: `AAPL`MSFT`GOOGL`AAPL;
  exchange: `NASDAQ`NASDAQ`NASDAQ`NASDAQ
)
```

### 9.2 Creating Foreign Keys

Foreign keys are created by referencing an enumeration or a keyed table.

**Enumeration (Most Common):**

An enumeration is a way to compress symbol columns by storing them as integers:

```q
q) syms: `AAPL`MSFT`GOOGL`TSLA`NVDA
q) trades: ([]
  sym: syms?`AAPL`MSFT`AAPL`GOOGL`TSLA;
  price: 150 300 151 2500 800;
  qty: 100 200 150 300 50
)

q) meta trades
c    | t f a
-----| -----
sym  | e syms
price| f
qty  | j
```

Here, `sym` has type `e` (enumeration) with foreign key `syms`.

**Access enumeration value:**

```q
q) trades[0; `sym]
`AAPL

q) sym: `e$0
q) sym
`AAPL
```

**Direct Foreign Key Creation:**

```q
q) instruments: ([id: 1 2 3] symbol: `AAPL`MSFT`GOOGL; exchange: `NYSE`NASDAQ`NASDAQ)

q) trades: ([]
  instrumentID: 1 2 1 3 2;
  price: 150 300 151 2500 301;
  qty: 100 200 150 300 250
)

q) meta trades
c             | t f a
--------------| -----
instrumentID  | j
price         | f
qty           | j
```

**Benefits of Enumeration:**

1. **Memory efficiency**: Stores integers instead of full symbols
2. **Faster joins**: Integer comparison is faster than symbol comparison
3. **Data integrity**: Constrains values to valid symbols

**Using Foreign Keys in Queries:**

```q
q) select sym, price from trades where sym in `AAPL`MSFT
sym   price
----------
AAPL  150
MSFT  300
AAPL  151
MSFT  301
```

---

## Part 10: Joins

**Joins** combine data from multiple tables based on matching criteria. This is fundamental to data analysis.

### 10.1 Union Join (`uj`)

Union join combines all rows from both tables, filling missing values with nulls.

**Syntax:**
```q
t1 uj t2
```

**Example:**

```q
q) trades1: ([]
  sym: `AAPL`MSFT;
  price: 150 300;
  qty: 100 200
)

q) trades2: ([]
  sym: `GOOGL`TSLA;
  price: 2500 800;
  qty: 150 75;
  commission: 1.50 2.00
)

q) trades1 uj trades2
sym   price qty commission
--------------------------
AAPL  150   100
MSFT  300   200
GOOGL 2500  150 1.50
TSLA  800   75  2.00
```

Notice:
- All rows from both tables included
- Missing columns filled with nulls
- Common columns (sym, price, qty) preserved

**Use cases:**
- Combining data from different sources
- Merging tables with different schemas
- Data consolidation

### 10.2 Plus Join (`pj`)

Plus join is like left join but **adds numeric values** instead of replacing them.

**Syntax:**
```q
t1 pj t2
```

**Example:**

```q
q) daily1: ([]
  sym: `AAPL`MSFT;
  volume: 1000 2000
)

q) daily2: ([]
  sym: `AAPL`MSFT;
  volume: 500 1500
)

q) daily1 pj daily2
sym   volume
-----------
AAPL  1500
MSFT  3500
```

The volumes are **added**, not replaced.

**Use cases:**
- Combining trade volumes from different markets
- Aggregating metrics from multiple sources
- Accumulating running totals

### 10.3 Inner Join (`ij`)

Inner join returns only rows where keys match in both tables.

**Syntax:**
```q
t1 ij t2
```

**Example:**

```q
q) trades: ([sym: `AAPL`MSFT`GOOGL`TSLA]
  price: 150 300 2500 800;
  qty: 100 200 150 75
)

q) listed: ([sym: `AAPL`MSFT`NVDA]
  exchange: `NYSE`NASDAQ`NASDAQ
)

q) trades ij listed
sym   price qty exchange
------------------------
AAPL  150   100 NYSE
MSFT  300   200 NASDAQ
```

Notice:
- Only `AAPL` and `MSFT` included (both in trades and listed)
- `GOOGL` and `TSLA` excluded (not in listed)
- `NVDA` excluded (not in trades)

**Use cases:**
- Finding common symbols across markets
- Filtering valid instruments
- Finding intersections of datasets

### 10.4 Left Join (`lj`)

Left join returns all rows from the left table, matched with data from the right table.

**Syntax:**
```q
t1 lj t2
```

**Example:**

```q
q) trades: ([]
  sym: `AAPL`MSFT`GOOGL;
  price: 150 300 2500
)

q) exchange_info: ([sym: `AAPL`MSFT]
  exchange: `NYSE`NASDAQ
)

q) trades lj exchange_info
sym   price exchange
-------------------
AAPL  150   NYSE
MSFT  300   NASDAQ
GOOGL 2500
```

Notice:
- All rows from trades included
- `GOOGL` has null exchange (not in exchange_info)

**Use cases:**
- Enriching data with optional information
- Finding unmatched records
- Left table is the source of truth

### 10.5 As-Of Join (`aj`)

**As-of join** finds the most recent row from the right table **at or before** each row in the left table.

This is critical for **time series data**.

**Syntax:**
```q
aj[joinKeys; t1; t2]
```

**Example:**

```q
q) trades: ([]
  sym: `AAPL`AAPL`MSFT;
  time: 09:30:00 09:30:05 09:30:10;
  price: 150.0 150.5 300.0
)

q) quotes: ([]
  sym: `AAPL`AAPL`AAPL`MSFT`MSFT;
  time: 09:30:00 09:30:03 09:30:07 09:30:00 09:30:08;
  bid: 149.9 149.95 150.4 299.5 300.2;
  ask: 150.1 150.05 150.6 299.9 300.5
)

q) aj[`sym`time; trades; quotes]
sym   time     price bid    ask
-------------------------------
AAPL  09:30:00 150.0 149.9  150.1
AAPL  09:30:05 150.5 149.95 150.6
MSFT  09:30:10 300.0 299.5  300.5
```

**How it works:**

For each trade:
- Trade 1: AAPL at 09:30:00 → Quote from AAPL at 09:30:00 (exact match)
- Trade 2: AAPL at 09:30:05 → Quote from AAPL at 09:30:03 (most recent before 09:30:05)
- Trade 3: MSFT at 09:30:10 → Quote from MSFT at 09:30:08 (most recent before 09:30:10)

**Use cases:**
- Matching trades with quotes at the exact time
- Finding prices before timestamp
- Building market data feeds
- Real-time risk monitoring

**Variations:**

```q
/ aj0: keeps the time from matched row
q) aj0[`sym`time; trades; quotes]

/ ajf: fills nulls from left table
q) ajf[`sym`time; trades; quotes]
```

### 10.6 Window Join (`wj`)

**Window join** aggregates data from the right table within time windows around each left table row.

**Syntax:**
```q
wj[windows; joinKeys; t1; (t2; aggregations)]
```

**Example:**

```q
q) trades: ([]
  sym: `AAPL`AAPL`MSFT`MSFT;
  time: 09:30:00 09:30:10 09:30:05 09:30:15;
  qty: 100 150 200 250
)

q) quotes: ([]
  sym: `AAPL`AAPL`AAPL`AAPL`MSFT`MSFT`MSFT`MSFT;
  time: 09:29:58 09:30:02 09:30:08 09:30:12 09:29:55 09:30:03 09:30:08 09:30:18;
  bid: 149.9 149.95 150.2 150.5 299.5 299.6 299.8 300.2;
  ask: 150.1 150.05 150.3 150.6 299.9 300.0 300.2 300.5
)

/ Define 1-second window (±500ms)
q) windows: (-0D00:00:00.500; 0D00:00:00.500) +\: trades.time

q) wj[windows; `sym`time; trades; (quotes; (max;`bid); (min;`ask))]
sym   time     qty bid    ask
------------------------------
AAPL  09:30:00 100 149.95 150.1
AAPL  09:30:10 150 150.5  150.3
MSFT  09:30:05 200 299.6  299.8
MSFT  09:30:15 250 300.2  300.2
```

**How it works:**

1. For each trade, create a time window (±500ms)
2. Find all quotes within that window
3. Apply aggregations (max bid, min ask)

**Use cases:**
- Finding best bid/ask around trade time
- Calculating spreads during execution
- Building execution analysis
- Market microstructure analysis

---

## Part 11: I/O (Input/Output)

### 11.1 Loading Data from Files

**Reading CSV Files:**

```q
/ Load CSV with header
q) trades: ("SJIFF"; enlist",") 0: `:trades.csv

/ Load CSV without header
q) trades: ("SJIFF"; enlist",") 0: `:trades_noheader.csv
q) trades: 4 # ([] sym: trades[0]; time: trades[1]; qty: trades[2]; price: trades[3])
```

**Reading Q Binary Format:**

Q binary format (`.q`) is much faster than CSV:

```q
/ Save table to binary
q) `trades.q set trades

/ Load binary table
q) trades: get `:trades.q
```

**Reading from Splayed Tables:**

```q
/ Load from splayed directory
q) trades: get `:trades/

/ This loads the entire splayed table
```

### 11.2 Saving Data to Files

**Save as CSV:**

```q
/ Save to CSV
q) `:trades.csv 0: csv trades
q) `:trades.csv 0: .Q.s trades
```

**Save as Q Binary:**

```q
/ Save to binary format (fast)
q) `:trades.q set trades

/ Save just schema (no data)
q) `:trades_schema.q set meta trades
```

**Save as Splayed Table:**

```q
/ Splay the table (each column as separate file)
q) .Q.en[`:.; trades] / enumerate and splay

/ Equivalent to:
q) `:trades_splayed/ set trades
```

**Example: Daily Data Archival**

```q
/ Daily function to save trades
archiveDaily: { [date]
  trades: select from trades where date = date;
  filename: `$"trades_", string[date], ".q";
  filename set trades;
  delete from `trades where date = date;
  `Archived ,string[count trades], ` records for date `, string[date]
}

/ Use it
q) archiveDaily 2024.10.01
"Archived 1000 records for date 2024.10.01"
```

### 11.3 Practical I/O Workflow

```q
/ Load raw data
raw: ("SSIFF"; enlist",") 0: `:raw_data.csv

/ Transform to proper types
trades: ([sym:`$(raw[0]); time:`t$(raw[1]); date:`d$(raw[1])]
  qty: "J"$ raw[2];
  price: "F"$ raw[3]
)

/ Add computed columns
trades: update total: qty * price from trades

/ Save transformed data
`:trades_processed.q set trades

/ Verify
trades: get `:trades_processed.q
select from trades limit 5
```

---

## Part 12: Database Types

### 12.1 Splayed Tables

A **splayed table** stores each column as a separate file in a directory.

**Structure:**
```
trades/
  sym
  time
  qty
  price
```

Each file contains one column's data.

**Creating a Splayed Table:**

```q
q) trades: ([]
  sym: `AAPL`MSFT`GOOGL`AAPL;
  time: 09:30:00 09:30:01 09:30:02 09:30:03;
  qty: 100 200 150 50;
  price: 150.5 300.2 2800.1 151.0
)

/ Splay to directory
q) `:trades_splayed/ set trades

/ Now we have:
/ trades_splayed/sym
/ trades_splayed/time
/ trades_splayed/qty
/ trades_splayed/price
```

**Advantages:**
- **Fast column access**: Read only needed columns
- **Easy updates**: Update one column without rewriting entire table
- **Compression**: Each column can be compressed independently

**Disadvantages:**
- Slower full table scans
- More complex to manage

**Loading Splayed Table:**

```q
/ Load entire splayed table
q) trades: get `:trades_splayed/

/ Load specific column only
q) sym: get `:trades_splayed/sym
q) price: get `:trades_splayed/price
```

### 12.2 Partitioned Tables

A **partitioned table** is split by a partition key (typically date).

**Structure:**
```
hdb/
  2024.10.01/
    trades/
      sym
      time
      qty
      price
  2024.10.02/
    trades/
      sym
      time
      qty
      price
  ...
```

**Creating Partitioned Data:**

```q
/ Create sample trades for multiple dates
trades: ([]
  date: 2024.10.01 2024.10.01 2024.10.02 2024.10.02;
  sym: `AAPL`MSFT`AAPL`GOOGL;
  time: 09:30:00 09:30:01 09:30:02 09:30:03;
  qty: 100 200 150 50;
  price: 150.5 300.2 151.0 2800.1
)

/ Partition by date and save
.Q.en[`:hdb; trades]
```

This creates:
```
hdb/
  2024.10.01/trades/
  2024.10.02/trades/
```

**Advantages:**
- **Parallelism**: Each partition can be queried in parallel
- **Scalability**: Add partitions incrementally
- **Archival**: Compress old partitions
- **Time-based queries**: Efficient queries on date range

**Disadvantages:**
- Requires partition key
- More complex setup

**Querying Partitioned Data:**

```q
/ Load HDB
\l hdb

/ Query specific date
q) select from trades where date = 2024.10.01

/ Query date range
q) select from trades where date within (2024.10.01; 2024.10.02)

/ All queries are automatically partitioned
```

### 12.3 Splayed-Partitioned Tables

Combines benefits of splaying and partitioning.

**Structure:**
```
hdb/
  2024.10.01/
    trades/
      sym
      time
      qty
      price
  2024.10.02/
    trades/
      sym
      time
      qty
      price
```

**Creating Splayed-Partitioned:**

```q
/ This is the default with .Q.en
.Q.en[`:hdb; trades]
```

**Advantages:**
- All benefits of both splaying and partitioning
- Fast column access within partitions
- Efficient date-range queries

**Use cases:**
- Historical database (HDB)
- Real-time database (RDB) with daily rollover

### 12.4 Segmented Tables

**Segmented tables** extend partitioned tables across multiple directories/servers.

**Structure:**
```
seg1/hdb/2024.10.01/trades/
seg1/hdb/2024.10.02/trades/
seg2/hdb/2024.10.03/trades/
seg2/hdb/2024.10.04/trades/
```

**Use cases:**
- Distributing data across multiple servers
- Load balancing queries
- Managing very large datasets
- Geographic distribution

**Advantages:**
- Scales beyond single server capacity
- Distributes query load
- Enables disaster recovery

---

## Part 13: Real-Time Data Architecture

### 13.1 Typical KDB+ System Architecture

```
Market Data Feed
      ↓
  Tickerplant (TP) → Publish/Subscribe
      ↓
  Real-time DB (RDB) ← In-memory
      ↓ (daily rollover)
  Historical DB (HDB) ← Disk-based
```

### 13.2 Building a Simple Real-Time System

**Step 1: Define Schema**

```q
/ Trade schema
trade: ([]
  sym: `symbol$();
  time: `time$();
  qty: `int$();
  price: `float$();
  total: `float$()
)

/ Quote schema
quote: ([]
  sym: `symbol$();
  time: `time$();
  bid: `float$();
  ask: `float$()
)
```

**Step 2: Create Insert Function**

```q
.u.upd: { [tbl; data]
  if[tbl = `trade; data: update total: qty * price from data];
  .[tbl; (); ,; data];
  data
}

/ Usage
.u.upd[`trade; ([] sym: `AAPL; time: 09:30:00; qty: 100; price: 150.5)]
```

**Step 3: Publish Updates**

```q
/ Notify all subscribers
publish: { [tbl; data]
  .u.upd[tbl; data];
  / Notify subscribers (would connect to real system)
}
```

**Step 4: End-of-Day Rollover**

```q
eod: {
  / Save current data to disk
  .Q.en[`:hdb; trade];
  .Q.en[`:hdb; quote];
  
  / Clear in-memory tables
  delete from `trade;
  delete from `quote;
  
  `Trade eod: complete
}
```

---

## Complete Day 3 Example: Multi-Table Query System

```q
/ Create symbol enumeration
syms: `AAPL`MSFT`GOOGL`TSLA`NVDA

/ Create trades
trades: ([]
  sym: syms ? `AAPL`MSFT`AAPL`GOOGL`TSLA;
  date: 2024.10.01 2024.10.01 2024.10.01 2024.10.02 2024.10.02;
  time: 09:30:00 09:30:01 09:30:02 09:35:00 09:35:05;
  qty: 100 200 150 300 250;
  price: 150.5 300.2 151.0 2500.1 800.5
)

/ Create quotes
quotes: ([]
  sym: syms ? `AAPL`AAPL`AAPL`MSFT`MSFT;
  time: 09:30:00 09:30:01 09:30:02 09:30:01 09:30:02;
  bid: 150.2 150.3 150.4 299.8 300.1;
  ask: 150.8 150.9 151.0 300.2 300.5
)

/ Create reference data
instruments: ([sym: syms] 
  exchange: `NYSE`NASDAQ`NASDAQ`NASDAQ`NASDAQ;
  sector: `TECH`TECH`TECH`AUTO`TECH
)

/ Query 1: Trades with matched quotes (as-of join)
q) aj[`sym`time; trades; quotes]
sym   date       time     qty price  bid    ask
-----------------------------------------------
AAPL  2024.10.01 09:30:00 100 150.5  150.2  150.8
MSFT  2024.10.01 09:30:01 200 300.2  299.8  300.2
AAPL  2024.10.01 09:30:02 150 151.0  150.4  151.0
GOOGL 2024.10.02 09:35:00 300 2500.1
TSLA  2024.10.02 09:35:05 250 800.5

/ Query 2: Enriched trades with reference data
q) trades lj instruments
sym   date       time     qty price  exchange sector
-----------------------------------------------------
AAPL  2024.10.01 09:30:00 100 150.5  NYSE    TECH
MSFT  2024.10.01 09:30:01 200 300.2  NASDAQ  TECH
AAPL  2024.10.01 09:30:02 150 151.0  NYSE    TECH
GOOGL 2024.10.02 09:35:00 300 2500.1 NASDAQ  TECH
TSLA  2024.10.02 09:35:05 250 800.5  NASDAQ  AUTO

/ Query 3: Grouped summary
q) select sum qty, avg price by sym from trades
sym   | qty price
------| -----------
AAPL  | 250 150.75
GOOGL | 300 2500.1
MSFT  | 200 300.2
TSLA  | 250 800.5

/ Query 4: Daily volume by sector
q) select sum qty by date, sector from (trades lj instruments)
date       sector | qty
------------------| ---
2024.10.01 TECH   | 450
2024.10.02 AUTO   | 250
2024.10.02 TECH   | 300

/ Save trades
`:trades.q set trades

/ Load trades
trades: get `:trades.q
```

---

## Practice Exercises

### Exercise 1: Foreign Keys and Enumerations

```q
/ Create symbol list
syms: `AAPL`MSFT`GOOGL

/ Create trades with enumeration
trades: ([]
  sym: syms ? `AAPL`MSFT`AAPL;
  price: 150 300 151;
  qty: 100 200 150
)

/ Verify foreign key
q) meta trades
c    | t f a
-----| -----
sym  | e syms
price| f
qty  | j
```

### Exercise 2: Inner Join

```q
/ Create two tables with matching keys
t1: ([id: 1 2 3] name: `Alice`Bob`Charlie)
t2: ([id: 2 3 4] salary: 100000 120000 95000)

/ Inner join - only matching ids
q) t1 ij t2
id| name    salary
--|---------------
2 | Bob     100000
3 | Charlie 120000
```

### Exercise 3: As-Of Join

```q
/ Trades table
trades: ([]
  sym: `AAPL`AAPL;
  time: 09:30:00 09:30:05;
  qty: 100 150
)

/ Quotes table
quotes: ([]
  sym: `AAPL`AAPL`AAPL;
  time: 09:30:00 09:30:03 09:30:07;
  bid: 150.0 150.2 150.5
)

/ As-of join
q) aj[`sym`time; trades; quotes]
sym  time     qty bid
--------------------
AAPL 09:30:00 100 150.0
AAPL 09:30:05 150 150.5
```

### Exercise 4: I/O Operations

```q
/ Create table
trades: ([] sym: `AAPL`MSFT; price: 150 300; qty: 100 200)

/ Save to binary
q) `:trades.q set trades

/ Load from binary
q) trades: get `:trades.q

/ Verify
q) select from trades
sym   price qty
-----------
AAPL  150   100
MSFT  300   200
```

### Exercise 5: Partitioned Data

```q
/ Create trades with dates
trades: ([]
  date: 2024.10.01 2024.10.01 2024.10.02 2024.10.02;
  sym: `AAPL`MSFT`AAPL`GOOGL;
  price: 150 300 151 2500;
  qty: 100 200 150 75
)

/ Partition by date
q) .Q.en[`:hdb; trades]

/ Load partitioned
q) \l hdb
q) trades: get `:2024.10.01/trades
```

---

## Summary

Day 3 covers advanced table concepts critical for production systems:

✓ **Foreign Keys**: Reference integrity and compression  
✓ **Joins**: Multiple join types for combining tables  
✓ **As-Of Join**: Time-series matching  
✓ **Window Join**: Aggregation within time windows  
✓ **I/O**: Loading and saving data efficiently  
✓ **Database Types**: Splayed, partitioned, splayed-partitioned  
✓ **Real-time Architecture**: Building live systems  

These concepts form the foundation of real Kdb+ systems handling billions of records.

---

## Next Steps

In Day 4, you'll learn:
- IPC (Inter-Process Communication): Connecting processes
- Message handlers: Building servers
- Synchronous and asynchronous calls
- The .z and .Q namespaces
- Tickerplant architecture and design
