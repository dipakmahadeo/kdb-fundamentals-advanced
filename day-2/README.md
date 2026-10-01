# Day 2 – Functions, Namespaces, Tables, and Queries

## Learning Objectives

By the end of Day 2, you will understand:
- How to write your own functions in Q
- Projection and partial application of functions
- Global and local variable scope
- Iterators and adverbs for list processing
- Namespaces to prevent naming conflicts
- Creating, manipulating, and querying tables
- Functional form queries with grouping and aggregations

---

## 5. Functions

### 5.1 Writing Your Own Function

A function in Q is defined using curly braces `{ }`. Functions are first-class objects and can be assigned to variables.

**Basic Function Syntax:**

```q
functionName: { x + 1 }
```

The parameter `x` is implicit. Call the function:

```q
q) add1: { x + 1 }
q) add1 5
6
q) add1 10
11
```

**Function with Multiple Parameters:**

Use explicit parameters in square brackets:

```q
q) add: { [x; y] x + y }
q) add[2; 3]
5
q) add[10; 20]
30
```

**Function with Multiple Statements:**

Use semicolons to separate statements:

```q
q) processValue: { [x]
  doubled: x * 2;
  result: doubled + 1;
  result
}
q) processValue 5
11
```

**Function Returning a List:**

```q
q) makeList: { [n]
  1 + til n    / til generates list 0 to n-1
}
q) makeList 5
1 2 3 4 5
```

**Conditional Function:**

```q
q) absolute: { [x]
  $[x < 0; -x; x]    / if x < 0 then -x else x
}
q) absolute -5
5
q) absolute 3
3
```

**Function with Default Logic:**

```q
q) greet: { [name]
  "Hello, " , name
}
q) greet "Alice"
"Hello, Alice"
```

**Anonymous Functions:**

```q
q) { x + 1 } 5
6
q) { [x; y] x * y } [3; 4]
12
```

**Function Library Pattern:**

```q
q) math: {
  add: { [x; y] x + y };
  multiply: { [x; y] x * y };
  square: { [x] x * x };
  add
}
q) math[2; 3]
5
```

---

### 5.2 Projection

**Projection** is a powerful Q feature where a function is partially applied. You fix some arguments and let others vary.

**What is Projection?**

A projection creates a new function by binding some parameters of an existing function.

**Example 1: Simple Projection**

```q
q) add: { [x; y] x + y }
q) add5: add[5;]      / create a function that adds 5 to anything
q) add5 10
15
q) add5 20
25
```

Here, `add[5;]` creates a projection where the first argument is fixed to 5.

**Example 2: Creating Multiple Projections**

```q
q) multiply: { [x; y] x * y }
q) double: multiply[2;]     / multiply by 2
q) triple: multiply[3;]     / multiply by 3
q) double 5
10
q) triple 5
15
```

**Example 3: Projection with Three Parameters**

```q
q) f: { [a; b; c] a + b + c }
q) f1: f[10;;]      / fix first param
q) f1[20; 30]
60
q) f2: f[10; 20;]   / fix first two params
q) f2 30
60
```

**Example 4: Using Projection for Configuration**

```q
q) raiseToThePower: { [x; n] x ^ n }
q) square: raiseToThePower[; 2]    / raise to power 2
q) cube: raiseToThePower[; 3]      / raise to power 3
q) square 5
25
q) cube 5
125
```

**Example 5: Real-World Use Case - Discount Calculator**

```q
q) applyDiscount: { [rate; price] price * (1 - rate) }
q) apply10PercentDiscount: applyDiscount[0.10;]
q) apply10PercentDiscount 100
90
q) apply10PercentDiscount 50
45
```

**Advantages of Projection:**

- Reduces code repetition
- Makes code more readable
- Enables function composition
- Useful for callbacks and higher-order functions

---

### 5.3 Global and Local Scope

Variables in Q can have **global scope** (visible everywhere) or **local scope** (visible only within a function).

**Global Variables:**

Variables defined outside a function are global:

```q
q) x: 10             / global variable
q) f: { x + 1 }
q) f[]
11
q) x                 / can access x globally
10
```

**Local Variables:**

Variables defined inside a function with `:` are local:

```q
q) globalX: 100
q) f: { 
  localX: 20;       / local variable (only within f)
  globalX + localX
}
q) f[]
120
q) localX           / localX not accessible outside function
'localX
```

**Shadowing Global Variables:**

A local variable with the same name as a global variable "shadows" it inside the function:

```q
q) x: 10
q) f: {
  x: 20;            / local x shadows global x
  x
}
q) f[]
20
q) x                / global x unchanged
10
```

**Function Parameters are Local:**

```q
q) x: 100
q) f: { [x] x + 1 }  / parameter x is local
q) f 5
6
q) x                 / global x unchanged
100
```

**Best Practices:**

- Use meaningful names to avoid confusion
- Be explicit about parameters
- Avoid shadowing global variables unless intentional
- Use local variables to keep functions self-contained

---

### 5.4 Iterators and Adverbs

Iterators (also called adverbs) are operators that modify how functions behave. They apply functions to lists efficiently.

**The `each` Adverb:**

Applies a function to each element of a list:

```q
q) l: 1 2 3 4 5
q) { x + 1 } each l
2 3 4 5 6
q) { x * 2 } each l
2 4 6 8 10
```

**Using Built-in Functions with `each`:**

```q
q) l: (1 2 3; 4 5 6; 7 8 9)
q) sum each l        / sum each sublist
6 15 24
q) count each l      / count elements in each sublist
3 3 3
```

**The `over` Adverb (Reduce):**

Accumulates a result across a list:

```q
q) l: 1 2 3 4 5
q) { x + y } over l  / (((1+2)+3)+4)+5 = 15
15
q) { x * y } over l  / (((1*2)*3)*4)*5 = 120
120
```

**Starting with an Initial Value:**

```q
q) l: 1 2 3 4 5
q) 0
q) { x + y } over 0, l  / start with 0, then add
15
q) 1
q) { x * y } over 1, l  / start with 1, then multiply
120
```

**The `scan` Adverb:**

Like `over`, but returns intermediate results:

```q
q) l: 1 2 3 4 5
q) { x + y } scan l
1 3 6 10 15
q) { x * y } scan l
1 2 6 24 120
```

**The `prior` Adverb:**

Applies a function to each element and the previous element:

```q
q) l: 1 2 3 4 5
q) { x - y } prior l
0 1 1 1 1
q) { y - x } prior l
0N 1 1 1 1
```

**Multi-Argument Iterators:**

```q
q) a: 1 2 3
q) b: 10 20 30
q) { x + y } each (a; b)
11 22 33
```

**Common Iterator Patterns:**

| Iterator | Purpose | Example |
|----------|---------|---------|
| `each` | Apply to each element | `{x+1} each 1 2 3` → `2 3 4` |
| `over` | Reduce/accumulate | `{x+y} over 1 2 3` → `6` |
| `scan` | Reduce with history | `{x+y} scan 1 2 3` → `1 3 6` |
| `prior` | Apply to consecutive pairs | `{x-y} prior 1 2 3` → `0 1 1` |

**Real-World Example: Calculate Running Sum**

```q
q) prices: 100 102 101 105 103
q) { x + y } scan prices
100 202 303 408 511
```

**Real-World Example: Apply Discount to Each Price**

```q
q) prices: 100 50 75
q) discount: 0.1
q) { x * (1 - y) } each (prices; discount)
90 45 67.5
```

---

## 6. Namespaces

### 6.1 Concept of Namespaces

A **namespace** is a prefix that groups related functions and variables together, preventing name conflicts in large systems.

**Why Use Namespaces?**

In large applications, you might have multiple functions or variables with similar names. Namespaces separate them:

```q
/ Without namespaces - NAME CONFLICT!
q) process: { "Module A" }
q) process: { "Module B" }  / overwrites the first one!

/ With namespaces - NO CONFLICT
q) moduleA.process: { "Module A" }
q) moduleB.process: { "Module B" }
q) moduleA.process[]
"Module A"
q) moduleB.process[]
"Module B"
```

**Creating a Namespace:**

Use dot notation to create namespaced variables:

```q
q) .ns1.a: 10
q) .ns1.b: 20
q) .ns2.a: 100
q) .ns2.b: 200
```

**Accessing Namespace Variables:**

```q
q) .ns1.a
10
q) .ns2.a
100
```

**Nested Namespaces:**

```q
q) .company.finance.budget: 1000000
q) .company.engineering.budget: 500000
q) .company.finance.budget
1000000
q) .company.engineering.budget
500000
```

**Functions in Namespaces:**

```q
q) .math.add: { [x; y] x + y }
q) .math.multiply: { [x; y] x * y }
q) .string.concat: { [a; b] a, b }
q) .math.add[5; 3]
8
q) .string.concat["Hello"; " World"]
"Hello World"
```

**System Namespaces:**

Q has built-in namespaces like `.z` and `.Q`:

```q
q) .z.d           / current date
2024.10.01
q) .z.t           / current time
12:34:56.789
q) .z.h           / hostname
"LOCALHOST"
q) .Q.ind         / index
```

**Listing Namespace Contents:**

```q
q) key .ns1
`a`b
q) .ns1
a| 10
b| 20
```

**Practical Example: Configuration Namespaces**

```q
q) .config.database.host: "localhost"
q) .config.database.port: 5432
q) .config.api.timeout: 30
q) .config.api.retries: 3
q) .config.database.host
"localhost"
q) .config.api.timeout
30
```

---

## 7. Tables

### 7.1 Creating Tables

A **table** is a rectangular data structure with named columns, similar to a spreadsheet or SQL table.

**Creating a Simple Table:**

```q
q) t: ([] sym:`A`B`C; price:100 110 120)
q) t
sym price
---------
A   100
B   110
C   120
```

**Creating a Table with Multiple Columns:**

```q
q) trades: ([]
  sym: `AAPL`MSFT`GOOGL`AAPL;
  time: 09:30:00 09:30:01 09:30:02 09:30:03;
  qty: 100 200 150 50;
  price: 150.5 300.2 2800.1 151.0
)
q) trades
sym   time     qty price
--------------------------
AAPL  09:30:00 100 150.5
MSFT  09:30:01 200 300.2
GOOGL 09:30:02 150 2800.1
AAPL  09:30:03 50  151.0
```

**Creating a Table from Dictionaries:**

```q
q) t: ([id: 1 2 3] name: `Alice`Bob`Charlie; age: 25 30 35)
q) t
id| name    age
--|----------
1 | Alice   25
2 | Bob     30
3 | Charlie 35
```

**Empty Table with Schema:**

```q
q) emptyTrades: ([sym:`$()] time:`time$(); qty:`int$(); price:`float$())
```

---

### 7.2 Schema Inspection

**Using `meta` to View Schema:**

```q
q) trades: ([]
  sym: `AAPL`MSFT`GOOGL;
  time: 09:30:00 09:30:01 09:30:02;
  qty: 100 200 150;
  price: 150.5 300.2 2800.1
)
q) meta trades
c    | t f a
-----| -----
sym  | s
time | t
qty  | j
price| f
```

Explanation:
- `c`: column name
- `t`: column type (s=symbol, t=time, j=long, f=float)
- `f`: foreign key (if any)
- `a`: attribute (if any)

**Getting Column Names:**

```q
q) cols trades
`sym`time`qty`price
```

**Getting Column Types:**

```q
q) `t in cols `meta trades    / not standard; use meta instead
```

**Number of Rows:**

```q
q) count trades
4
```

**Table Shape:**

```q
q) shape: (count trades; count cols trades)
q) shape
4 4
```

---

### 7.3 Table Manipulations (Select, Insert, Delete, Update)

#### **SELECT: Retrieving Data**

**Select All Rows:**

```q
q) select from trades
sym   time     qty price
--------------------------
AAPL  09:30:00 100 150.5
MSFT  09:30:01 200 300.2
GOOGL 09:30:02 150 2800.1
AAPL  09:30:03 50  151.0
```

**Select Specific Columns:**

```q
q) select sym, price from trades
sym   price
----------
AAPL  150.5
MSFT  300.2
GOOGL 2800.1
AAPL  151.0
```

**Select with WHERE Clause:**

```q
q) select from trades where qty > 100
sym   time     qty price
--------------------------
MSFT  09:30:01 200 300.2
GOOGL 09:30:02 150 2800.1
```

**Select with Multiple Conditions:**

```q
q) select from trades where (qty > 100) & (price < 300)
sym   time     qty price
--------------------------
GOOGL 09:30:02 150 2800.1
```

**Select with Column Aliasing:**

```q
q) select symbol: sym, cost: price from trades
symbol cost
----------
AAPL   150.5
MSFT   300.2
GOOGL  2800.1
AAPL   151.0
```

#### **INSERT: Adding Data**

**Insert a Single Row:**

```q
q) `trades insert (`TESLA; 09:30:04; 75; 250.3)
4
q) trades
sym    time     qty price
---------------------------
AAPL   09:30:00 100 150.5
MSFT   09:30:01 200 300.2
GOOGL  09:30:02 150 2800.1
AAPL   09:30:03 50  151.0
TESLA  09:30:04 75  250.3
```

**Insert Multiple Rows:**

```q
q) `trades insert ((`NVDA`AMD; 09:30:05 09:30:06; 60 80; 900.5 140.2))
```

#### **UPDATE: Modifying Data**

**Update a Column Value:**

```q
q) update price: price * 1.05 from `trades where sym = `AAPL
q) trades
sym    time     qty price
---------------------------
AAPL   09:30:00 100 158.025
MSFT   09:30:01 200 300.2
GOOGL  09:30:02 150 2800.1
AAPL   09:30:03 50  158.775
TESLA  09:30:04 75  250.3
```

**Update Multiple Columns:**

```q
q) update qty: qty * 2, price: price * 0.9 from `trades where sym = `MSFT
```

**Update with Computed Values:**

```q
q) update total: qty * price from `trades
q) trades
sym    time     qty price   total
---------------------------------
AAPL   09:30:00 100 158.025 15802.5
MSFT   09:30:01 200 300.2   60040
GOOGL  09:30:02 150 2800.1  420015
AAPL   09:30:03 50  158.775 7938.75
TESLA  09:30:04 75  250.3   18772.5
```

#### **DELETE: Removing Data**

**Delete Rows Based on Condition:**

```q
q) delete from `trades where sym = `TESLA
q) trades
sym   time     qty price
--------------------------
AAPL  09:30:00 100 150.5
MSFT  09:30:01 200 300.2
GOOGL 09:30:02 150 2800.1
AAPL  09:30:03 50  151.0
```

**Delete with Multiple Conditions:**

```q
q) delete from `trades where (qty < 100) & (sym = `AAPL)
```

---

## 8. Queries

### 8.1 Functional Form

Q queries can be written in **SQL-like form** (select ... from ... where) or **functional form**.

**SQL-Like Form:**

```q
q) select sym, price from trades where qty > 100
```

**Functional Form (equivalent):**

The functional form uses the `?` operator:

```q
q) ?[trades; ((> ;`qty; 100)); 0b; `sym`price]
```

This can be more powerful for dynamic queries.

**Components of Functional Query:**

```
?[table; where_clause; by_clause; select_clause]
```

- `table`: the table to query
- `where_clause`: filtering conditions
- `by_clause`: grouping (0b = no grouping)
- `select_clause`: columns to return

**Building Functional Queries:**

```q
/ Simple select
q) ?[trades; (); 0b; `sym`price]
sym   price
----------
AAPL  150.5
MSFT  300.2
GOOGL 2800.1
AAPL  151.0

/ Select with WHERE
q) ?[trades; ((>; `qty; 100)); 0b; `sym`price]
sym   price
----------
MSFT  300.2
GOOGL 2800.1

/ Select with WHERE and multiple conditions
q) ?[trades; ((>; `qty; 100); (<; `price; 300)); 0b; `sym`price]
sym   price
----------
GOOGL 2800.1
```

---

### 8.2 Grouping and Aggregations

**Simple GROUP BY:**

```q
q) select sum qty by sym from trades
sym   | qty
------| ---
AAPL  | 150
GOOGL | 150
MSFT  | 200
```

**Multiple Aggregations:**

```q
q) select count i, sum qty, avg price by sym from trades
sym   | count qty avg price
------| --------------------
AAPL  | 2     150 150.75
GOOGL | 1     150 2800.1
MSFT  | 1     200 300.2
```

**Aggregation Functions:**

| Function | Purpose | Example |
|----------|---------|---------|
| `count` | Count rows | `count i` |
| `sum` | Sum values | `sum qty` |
| `avg` | Average | `avg price` |
| `min` | Minimum | `min price` |
| `max` | Maximum | `max price` |
| `first` | First value | `first sym` |
| `last` | Last value | `last sym` |

**GROUP BY Multiple Columns:**

```q
q) trades: ([]
  sym: `AAPL`AAPL`MSFT`MSFT;
  date: 2024.10.01 2024.10.01 2024.10.01 2024.10.02;
  qty: 100 50 200 150;
  price: 150 151 300 301
)
q) select sum qty by sym, date from trades
sym  date       | qty
-----------------| ---
AAPL 2024.10.01 | 150
MSFT 2024.10.01 | 200
MSFT 2024.10.02 | 150
```

**Aggregation with Having (Filter Groups):**

```q
q) select sum qty by sym from trades where sum qty > 100
sym   | qty
------| ---
AAPL  | 150
MSFT  | 350
```

**Aggregation with Expressions:**

```q
q) select totalValue: sum (qty * price) by sym from trades
sym   | totalValue
------| ----------
AAPL  | 30100
MSFT  | 121150
```

**Real-World Example: Daily Trading Summary**

```q
q) trades: ([]
  sym: `AAPL`AAPL`MSFT`MSFT`GOOGL;
  date: 2024.10.01 2024.10.01 2024.10.01 2024.10.02 2024.10.02;
  qty: 100 50 200 150 300;
  price: 150 151 300 301 2800
)
q) summary: select 
  count i as numTrades,
  sum qty as totalQty,
  avg price as avgPrice,
  min price as minPrice,
  max price as maxPrice
  by sym, date from trades
q) summary
sym   date       | numTrades totalQty avgPrice minPrice maxPrice
-----------------------------------------------------------------
AAPL  2024.10.01 | 2         150      150.5    150      151
GOOGL 2024.10.02 | 1         300      2800     2800     2800
MSFT  2024.10.01 | 1         200      300      300      300
MSFT  2024.10.02 | 1         150      301      301      301
```

---

## Complete Day 2 Example: Building a Trade System

```q
/ Create trades table
trades: ([]
  sym: `AAPL`MSFT`GOOGL`AAPL`MSFT;
  time: 09:30:00 09:30:01 09:30:02 09:30:03 09:30:04;
  qty: 100 200 150 50 75;
  price: 150.5 300.2 2800.1 151.0 301.5
)

/ Define reusable functions
.trading.calcTotal: { [qty; price] qty * price }
.trading.applyCommission: { [total; rate] total * (1 - rate) }
.trading.calculateProfit: { [buyPrice; sellPrice; qty]
  (sellPrice - buyPrice) * qty
}

/ Add commission column
trades: update commission: .trading.applyCommission'[qty * price; 0.001] from trades

/ Create projection for common fee rates
applyStandardFee: .trading.applyCommission[; 0.002]

/ Query: Total volume by symbol
q) select sum qty by sym from trades
sym   | qty
------| ---
AAPL  | 150
GOOGL | 150
MSFT  | 275

/ Query: Average price by symbol
q) select avg price by sym from trades
sym   | price
------| -------
AAPL  | 150.75
GOOGL | 2800.1
MSFT  | 300.85

/ Use namespace for module organization
.analytics.dailySummary: {
  select sum qty, avg price, max price by sym from trades
}
q) .analytics.dailySummary[]
sym   | qty avg price max price
------| ------------------------
AAPL  | 150 150.75   151
GOOGL | 150 2800.1   2800.1
MSFT  | 275 300.85   301.5
```

---

## Practice Exercises

### Exercise 1: Writing Functions
```q
/ 1. Write a function that calculates the square of a number
q) square: { [x] x * x }
q) square 5
25

/ 2. Write a function that returns the larger of two numbers
q) max2: { [x; y] $[x > y; x; y] }
q) max2[10; 20]
20

/ 3. Write a function that sums three numbers
q) sum3: { [a; b; c] a + b + c }
q) sum3[1; 2; 3]
6
```

### Exercise 2: Using Projection
```q
/ Create a projection of add for adding specific values
q) add: { [x; y] x + y }
q) add10: add[10;]
q) add10 5
15

/ Create a projection for multiplication
q) mul: { [x; y] x * y }
q) double: mul[2;]
q) double 7
14
```

### Exercise 3: Namespaces
```q
/ Organize utilities in namespaces
q) .math.sum3: { [a; b; c] a + b + c }
q) .math.average: { [a; b; c] (.math.sum3[a; b; c]) % 3 }
q) .string.concat: { [a; b] a, b }

q) .math.sum3[10; 20; 30]
60
q) .string.concat["Hello"; " World"]
"Hello World"
```

### Exercise 4: Table Operations
```q
/ Create a stocks table
q) stocks: ([]
  symbol: `AAPL`MSFT`GOOGL`TSLA;
  price: 150 300 2800 250;
  volume: 1000000 500000 200000 300000
)

/ Select stocks with price > 200
q) select from stocks where price > 200
symbol | price volume
-------| ------
MSFT   | 300   500000
GOOGL  | 2800  200000
TSLA   | 250   300000

/ Group by price range (simple aggregation)
q) select sum volume by symbol from stocks
symbol | volume
-------| ------
AAPL   | 1000000
GOOGL  | 200000
MSFT   | 500000
TSLA   | 300000
```

### Exercise 5: Complex Query
```q
/ Create orders table
q) orders: ([]
  orderID: 1 2 3 4 5;
  symbol: `AAPL`MSFT`AAPL`GOOGL`MSFT;
  quantity: 100 50 200 150 75;
  price: 150 300 151 2800 301
)

/ Query: Total value by symbol
q) select total: sum (quantity * price) by symbol from orders
symbol | total
-------| ------
AAPL   | 45150
GOOGL  | 420000
MSFT   | 37575

/ Query: Average price and total quantity
q) select avgPrice: avg price, totalQty: sum quantity by symbol from orders
symbol | avgPrice totalQty
-------| -----------
AAPL   | 150.5    300
GOOGL  | 2800     150
MSFT   | 300.5    125
```

---

## Key Takeaways

1. **Functions are first-class objects** – Assign them to variables and pass them around.
2. **Projection is powerful** – Create specialized functions from general ones.
3. **Scope matters** – Understand global vs. local variables to avoid bugs.
4. **Iterators are essential** – Use `each`, `over`, `scan` for list processing.
5. **Namespaces organize code** – Use them to prevent naming conflicts in large systems.
6. **Tables are central** – Learn select, insert, update, delete operations well.
7. **Queries are flexible** – Use both SQL-like and functional forms as needed.
8. **Aggregations are powerful** – Group, sum, average, and count efficiently.

---

## Next Steps

- Practice writing functions and using projections in a q session.
- Create tables and experiment with all CRUD operations.
- Write complex queries with multiple groupings and aggregations.
- Build a small project using namespaces to organize your code.
- Move to Day 3 to learn joins and advanced database concepts.
"