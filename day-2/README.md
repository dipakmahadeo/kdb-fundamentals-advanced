# Day 2 – Functions, Namespaces, Tables, and Queries

## Learning Objectives

By the end of Day 2, you will understand:
- How to write your own functions in Q
- How to modularize tasks within functions
- Projection and partial application of functions
- Global and local variable scope
- Iterators and adverbs for list processing
- Namespaces to prevent naming conflicts and organize code
- Creating and manipulating tables
- Table schema inspection
- CRUD operations (Create, Read, Update, Delete) on tables
- Writing queries in both SQL-like and functional forms
- Grouping and aggregations

---

## Part 5: Functions

### 5.1 Writing Your Own Function

A **function** in Q is defined using curly braces `{ }`. Functions are first-class objects that can be assigned to variables, passed as arguments, and returned from other functions.

**Function Syntax:**

```q
functionName: { expression }
```

The simplest function has an implicit parameter `x`:

```q
q) square: { x * x }
q) square 5
25

q) addOne: { x + 1 }
q) addOne 10
11

q) greet: { "Hello, " , x }
q) greet "Alice"
"Hello, Alice"
```

**Functions with Multiple Parameters:**

Use explicit parameters in square brackets, separated by semicolons:

```q
q) add: { [x; y] x + y }
q) add[2; 3]
5

q) add[100; 50]
150
```

**Three or More Parameters:**

```q
q) sumThree: { [a; b; c] a + b + c }
q) sumThree[10; 20; 30]
60

q) formula: { [x; y; z] (x + y) * z }
q) formula[2; 3; 4]
20
```

---

### 5.2 Modularizing Tasks within Functions

Breaking a function into logical steps makes it more readable and maintainable.

**Multi-Statement Functions:**

Use semicolons to separate multiple statements. The last expression is the return value:

```q
q) processValue: { [x]
  doubled: x * 2;
  result: doubled + 10;
  result
}
q) processValue 5
20
```

Here:
- Line 1: Calculate `doubled`
- Line 2: Calculate `result` using `doubled`
- Line 3: Return `result`

**Real-World Example: Calculate Trade Commission**

```q
q) calcTradeMetrics: { [price; qty; commissionRate]
  / Step 1: Calculate total trade value
  totalValue: price * qty;
  
  / Step 2: Calculate commission
  commission: totalValue * commissionRate;
  
  / Step 3: Calculate net proceeds
  net: totalValue - commission;
  
  / Step 4: Return results as a dictionary
  ([] total: enlist totalValue; commission: enlist commission; net: enlist net)
}

q) calcTradeMetrics[150; 100; 0.001]
total  commission net
100.05 15        99.85
```

**Conditional Logic in Functions:**

Use the `$[condition; true_value; false_value]` operator:

```q
q) absolute: { [x] $[x < 0; -x; x] }
q) absolute -5
5

q) absolute 3
3

q) gradeScore: { [score]
  $[score >= 90; "A";
    score >= 80; "B";
    score >= 70; "C";
    score >= 60; "D";
    "F"]
}

q) gradeScore 85
"B"

q) gradeScore 55
"F"
```

**Error Handling in Functions:**

Use `@[function; args; error_handler]` to handle errors:

```q
q) safeDiv: { [x; y]
  @[{x / y}; (x; y); { "Error: Division by zero" }]
}

q) safeDiv[10; 2]
5f

q) safeDiv[10; 0]
"Error: Division by zero"
```

**Function Composition (Modular Design):**

Create small, reusable functions and combine them:

```q
/ Utility functions
double: { x * 2 }
addTen: { x + 10 }
square: { x * x }

/ Composed function
process: { x | double addTen square . }

q) process 5
/ Equivalent to: double(addTen(square(5)))
/ square(5) = 25
/ addTen(25) = 35
/ double(35) = 70
70
```

**Function Documentation:**

Include comments to explain your functions:

```q
/ Purpose: Calculate the discounted price
/ Parameters:
/   originalPrice: the starting price
/   discountRate: discount as decimal (0.1 = 10%)
/ Returns: discounted price
discountPrice: { [originalPrice; discountRate]
  originalPrice * (1 - discountRate)
}

q) discountPrice[100; 0.15]
85f
```

---

### 5.3 Projection: Partial Application of Functions

**Projection** creates a new function by fixing some parameters of an existing function. This is one of Q's most powerful features.

**Concept:**

You take a function with N parameters and fix some of them, creating a new function with fewer parameters.

**Simple Projection:**

```q
q) add: { [x; y] x + y }
q) add5: add[5;]          / Fix first parameter to 5
q) add5 10                / Now only need one parameter
15

q) add5 20
25

q) add5 -5
0
```

Here, `add[5;]` creates a new function that adds 5 to whatever value you pass.

**Real-World Example: Discount Functions**

```q
q) applyDiscount: { [discountRate; price] price * (1 - discountRate) }

/ Create specific discount functions
discount10: applyDiscount[0.10;]      / 10% discount
discount20: applyDiscount[0.20;]      / 20% discount
discount50: applyDiscount[0.50;]      / 50% discount (clearance)

q) discount10 100
90f

q) discount20 100
80f

q) discount50 100
50f
```

**Projection with Multiple Parameters:**

```q
q) f: { [a; b; c] a + b + c }

/ Fix first parameter
f1: f[10;;]
q) f1[20; 30]
60

/ Fix first and second parameters
f2: f[10; 20;]
q) f2 30
60

/ Fix second parameter (use :: for first)
f3: f[; 100;]
q) f3[1; 50]
151
```

**Creating Configuration-Based Functions:**

```q
/ Base function
raise: { [base; exponent] base ^ exponent }

/ Create specialized functions
square: raise[; 2]
cube: raise[; 3]

q) square 5
25

q) cube 5
125

/ Or
power2: { raise[x; 2] }
power3: { raise[x; 3] }
```

**Projection in List Operations:**

```q
/ Multiply each element by a factor
multiply: { [factor; list] list * factor }
double: multiply[2;]

q) double 1 2 3 4 5
2 4 6 8 10

/ Apply discount to each price
applyDiscount: { [rate; prices] prices * (1 - rate) }
discount15: applyDiscount[0.15;]

q) discount15 100 200 300 400
85 170 255 340
```

**Advantages of Projection:**

1. **Reduces code duplication**: No need to define similar functions
2. **Improves readability**: `discount10` is clearer than `applyDiscount[0.10;]`
3. **Enables function composition**: Combine projections to build complex operations
4. **Configuration management**: Easy to adjust parameters globally

---

### 5.4 Global and Local Scope

Variables in Q can be **global** (visible everywhere) or **local** (visible only within a function).

**Global Variables:**

Variables defined outside any function are global and accessible everywhere:

```q
q) x: 10
q) f: { x + 1 }
q) f[]
11

q) x
10
```

**Local Variables:**

Variables defined inside a function with `:` are local and only exist within that function:

```q
q) globalX: 100
q) f: {
  localX: 20;              / local to f
  globalX + localX         / can access both
}
q) f[]
120

q) localX                  / error: doesn't exist globally
'localX
```

**Function Parameters are Local:**

```q
q) x: 100
q) f: { [x] x + 1 }        / parameter x is local
q) f 5
6

q) x                       / global x unchanged
100
```

**Shadowing (Local Variables Hide Globals):**

A local variable with the same name as a global "shadows" it:

```q
q) price: 100
q) f: {
  price: 50;               / local price shadows global
  price
}
q) f[]
50

q) price                   / global price unchanged
100
```

**Best Practices for Scope:**

1. **Use explicit parameters** for functions:
   ```q
   / Good
   addVal: { [x; val] x + val }
   
   / Bad (relies on global val)
   addVal: { x + val }
   ```

2. **Avoid relying on globals** for function logic:
   ```q
   / Good: discountRate is a parameter
   discount: { [rate; price] price * (1 - rate) }
   
   / Bad: discountRate must be global
   discountRate: 0.1
   discount: { [price] price * (1 - discountRate) }
   ```

3. **Name local variables clearly**:
   ```q
   process: { [data]
     processedData: data + 1;
     finalData: processedData * 2;
     finalData
   }
   ```

---

### 5.5 Iterators and Adverbs

**Iterators** (also called **adverbs**) are operators that modify how functions behave. They allow you to apply a function to each element of a list or accumulate results.

**The `each` Adverb (Apply to Each Element):**

Applies a function to every element of a list:

```q
q) square: { x * x }
q) square each 1 2 3 4 5
1 4 9 16 25

q) { x + 1 } each 10 20 30
11 21 31

q) { x * 2 } each (1 2 3 4)
2 4 6 8
```

With built-in functions:

```q
q) l: (1 2 3; 4 5 6; 7 8 9)
q) sum each l             / sum each sublist
6 15 24

q) count each l           / count elements in each sublist
3 3 3

q) avg each l             / average of each sublist
2 5 8f
```

**Multi-List `each`:**

```q
q) a: 1 2 3
q) b: 10 20 30
q) { x + y } each (a; b)  / element-wise addition
11 22 33

q) { x * y } each (a; b)  / element-wise multiplication
10 40 90
```

**The `over` Adverb (Reduce/Fold):**

Accumulates a single result by repeatedly applying a function:

```q
q) { x + y } over 1 2 3 4 5
15
/ Equivalent to: (((1 + 2) + 3) + 4) + 5

q) { x * y } over 1 2 3 4 5
120
/ Equivalent to: (((1 * 2) * 3) * 4) * 5
```

Starting with an initial value:

```q
q) { x + y } over 0, 1 2 3 4 5
15
/ Start with 0, then add: 0 + 1 + 2 + 3 + 4 + 5

q) { x * y } over 1, 1 2 3 4 5
120
/ Start with 1, then multiply: 1 * 1 * 2 * 3 * 4 * 5
```

**The `scan` Adverb (Reduce with History):**

Like `over`, but returns all intermediate results:

```q
q) { x + y } scan 1 2 3 4 5
1 3 6 10 15
/ Shows: 1, (1+2)=3, (3+3)=6, (6+4)=10, (10+5)=15

q) { x * y } scan 1 2 3 4 5
1 2 6 24 120
/ Shows: 1, (1*2)=2, (2*3)=6, (6*4)=24, (24*5)=120
```

**The `prior` Adverb (Apply to Consecutive Pairs):**

Applies a function to each element and the previous element:

```q
q) prices: 100 102 101 105 103
q) { y - x } prior prices           / price changes
0N 2 -1 4 -2

q) { x - y } prior 1 2 3 4 5
0 -1 -1 -1 -1
```

**Real-World Examples:**

**Calculate Running Sum (Cumulative):**

```q
q) trades: 100 150 200 50 100
q) { x + y } scan trades
100 250 450 500 600
/ Cumulative sum of trade amounts
```

**Calculate Daily Price Changes:**

```q
q) prices: 100 102 101 105 103 104
q) changes: { y - x } prior prices
0N 2 -1 4 -2 1
/ Each day's price change from previous day
```

**Apply Commission to Each Trade:**

```q
q) amounts: 1000 2000 1500 3000
q) commissionRate: 0.001
q) { x * (1 - y) } each (amounts; commissionRate)
999 1998 1499.5 2997
```

**Find Running Maximum (High Water Mark):**

```q
q) prices: 100 102 101 105 103 106 104
q) { max x, y } scan prices
100 102 102 105 105 106 106
/ Highest price seen so far at each point
```

**Iterator Summary Table:**

| Iterator | Purpose | Example | Output |
|----------|---------|---------|--------|
| `each` | Apply to each element | `{x+1} each 1 2 3` | `2 3 4` |
| `over` | Reduce to single value | `{x+y} over 1 2 3` | `6` |
| `scan` | Reduce with intermediate values | `{x+y} scan 1 2 3` | `1 3 6` |
| `prior` | Apply to consecutive pairs | `{y-x} prior 1 2 3` | `0N 1 1` |

---

## Part 6: Namespaces

### 6.1 Concept of Namespaces

A **namespace** is a named container that groups related functions and variables together. Namespaces prevent name conflicts in large systems and organize code logically.

**Why Use Namespaces?**

In large applications, you often have similarly-named functions from different modules. Without namespaces, they would overwrite each other:

```q
/ Problem: Name conflicts!
q) process: { "Module A logic" }
q) process: { "Module B logic" }
q) process[]
"Module B logic"              / Module A's function was overwritten!
```

**Solution: Use Namespaces**

```q
/ Solution: Each module has its own namespace
q) .moduleA.process: { "Module A logic" }
q) .moduleB.process: { "Module B logic" }
q) .moduleA.process[]
"Module A logic"

q) .moduleB.process[]
"Module B logic"
```

---

### 6.2 Creating and Accessing Namespaces

**Basic Namespace Creation:**

Use dot notation to create namespace variables:

```q
q) .math.pi: 3.14159
q) .math.e: 2.71828
q) .math.phi: 1.61803
```

**Accessing Namespace Variables:**

```q
q) .math.pi
3.14159

q) .math.e
2.71828
```

**Nested Namespaces:**

Namespaces can be arbitrarily nested:

```q
q) .company.finance.budget: 1000000
q) .company.finance.revenue: 1500000
q) .company.engineering.budget: 500000
q) .company.engineering.headcount: 50

q) .company.finance.budget
1000000

q) .company.engineering.headcount
50
```

**Functions in Namespaces:**

```q
q) .math.add: { [x; y] x + y }
q) .math.multiply: { [x; y] x * y }
q) .math.divide: { [x; y] $[y = 0; ::; x / y] }

q) .math.add[10; 20]
30

q) .math.multiply[5; 6]
30

q) .math.divide[100; 4]
25f
```

**Listing Namespace Contents:**

```q
q) key .math
`add`divide`e`multiply`phi`pi

q) value .math
{ [x; y] x + y }
{ [x; y] $[y = 0; ::; x / y] }
2.71828
{ [x; y] x * y }
1.61803
3.14159
```

---

### 6.3 Organizing Code with Namespaces

**Module Pattern:**

Use namespaces to organize related functions:

```q
/ Trading module
.trading.calcCommission: { [total; rate] total * rate }
.trading.netProceeds: { [total; commission] total - commission }
.trading.profitLoss: { [buyPrice; sellPrice; qty] (sellPrice - buyPrice) * qty }

/ Analytics module
.analytics.volatility: { [prices] dev prices }
.analytics.returns: { [prices] (1 drop prices) % (prices: -1 drop prices) }
.analytics.sharpe: { [returns; riskFreeRate] (avg returns - riskFreeRate) / dev returns }

/ Utilities module
.utils.roundToDecimals: { [value; decimals] floor (value * 10 ^ decimals) / (10 ^ decimals) }
.utils.formatCurrency: { [value] "$", string .utils.roundToDecimals[value; 2] }
.utils.percentage: { [value; total] (.utils.roundToDecimals[value * 100 / total; 2]), "%" }
```

**Using Organized Code:**

```q
q) total: 1000
q) .trading.calcCommission[total; 0.001]
1

q) .trading.netProceeds[total; 1]
999

q) prices: 100 102 101 105 103
q) .analytics.volatility[prices]
1.581139
```

---

### 6.4 System Namespaces

Q has built-in namespaces for system functions and variables:

**The `.z` Namespace (System/Environment):**

```q
q) .z.d                / current date
2024.10.02

q) .z.t                / current time in milliseconds
12:34:56.789

q) .z.D                / date as integer (days since 2000.01.01)
18566

q) .z.T                / time as long (milliseconds since midnight)
45296789

q) .z.h                / hostname
"MACHINENAME"

q) .z.u                / username
"trader"

q) .z.w                / current connection handle
-1

q) .z.p                / timestamp (date + time)
2024.10.02T12:34:56.789

q) .z.Z                / UTC timestamp
2024.10.02T16:34:56.789

q) .z.a                / access level (admin, service, user, guest)
`admin
```

**The `.Q` Namespace (Standard Library):**

```q
q) .Q.ty 1 2 3         / type of each element
`j`j`j

q) .Q.j                / JSON encoding/decoding
q) .Q.s                / space-separated string

q) .Q.dpft             / function for partitioned tables

q) .Q.ind              / index
```

---

## Part 7: Tables

### 7.1 Creating Tables

A **table** is a rectangular data structure with named columns, similar to a spreadsheet or database table.

**Simple Table Creation:**

```q
q) t: ([] sym: `AAPL`MSFT`GOOGL; price: 150 300 2500)
q) t
sym   price
-----------
AAPL  150
MSFT  300
GOOGL 2500
```

The syntax `[]` indicates an unnamed row index; you're creating columns with equal-length lists.

**Table with Multiple Columns:**

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

**Table with Keyed Rows:**

Use `[keyColumn]` to designate a primary key:

```q
q) employees: ([id: 1 2 3 4]
  name: `Alice`Bob`Charlie`David;
  department: `engineering`sales`engineering`marketing;
  salary: 120000 80000 115000 85000
)

q) employees
id| name    department  salary
--|---------------------------
1 | Alice   engineering 120000
2 | Bob     sales       80000
3 | Charlie engineering 115000
4 | David   marketing   85000
```

**Empty Table with Schema:**

```q
q) emptyTrades: ([]
  sym: `symbol$();
  time: `time$();
  qty: `int$();
  price: `float$()
)

q) emptyTrades
sym time qty price
```

---

### 7.2 Schema Inspection

Understanding table structure is critical for data manipulation.

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
- `c`: Column name
- `t`: Data type (s=symbol, t=time, j=long/integer, f=float, etc.)
- `f`: Foreign key (if applicable)
- `a`: Attributes (s=sorted, p=parted, etc.)

**Getting Column Names:**

```q
q) cols trades
`sym`time`qty`price
```

**Getting Column Count and Row Count:**

```q
q) count trades                / rows
4

q) count cols trades           / columns
4

q) shape: (count trades; count cols trades)
q) shape
4 4
```

**Accessing Specific Columns:**

```q
q) trades[`sym]
`AAPL`MSFT`GOOGL

q) trades[`price]
150.5 300.2 2800.1
```

---

### 7.3 Table Manipulations: SELECT, INSERT, UPDATE, DELETE

**SELECT: Retrieve Data**

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
sym  time     qty price
-----------------------
MSFT 09:30:01 200 300.2
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

**INSERT: Add Data**

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

q) trades
sym    time     qty price
---------------------------
AAPL   09:30:00 100 150.5
MSFT   09:30:01 200 300.2
GOOGL  09:30:02 150 2800.1
AAPL   09:30:03 50  151.0
TESLA  09:30:04 75  250.3
NVDA   09:30:05 60  900.5
AMD    09:30:06 80  140.2
```

**UPDATE: Modify Data**

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

q) trades
sym    time     qty price
---------------------------
AAPL   09:30:00 100 158.025
MSFT   09:30:01 400 270.18
GOOGL  09:30:02 150 2800.1
AAPL   09:30:03 50  158.775
TESLA  09:30:04 75  250.3
```

**Update with Computed Values:**

```q
q) update total: qty * price from `trades
q) trades
sym    time     qty price   total
---------------------------------
AAPL   09:30:00 100 158.025 15802.5
MSFT   09:30:01 400 270.18  108072
GOOGL  09:30:02 150 2800.1  420015
AAPL   09:30:03 50  158.775 7938.75
TESLA  09:30:04 75  250.3   18772.5
```

**DELETE: Remove Data**

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

## Part 8: Queries

### 8.1 Functional Form

Q queries can be written in **SQL-like form** (readable, similar to SQL) or **functional form** (programmatic, dynamic).

**SQL-Like Form:**

```q
q) select sym, price from trades where qty > 100
```

This is easy to read but fixed at parse time.

**Functional Form:**

```q
q) ?[trades; ((>; `qty; 100)); 0b; `sym`price]
```

This is more complex but allows dynamic query construction.

**Why Functional Form?**

1. **Dynamic queries**: Build queries at runtime based on user input
2. **Generic functions**: Create query functions that work on any table
3. **Parameterized**: Table names, columns, and conditions become values

**Functional Query Syntax:**

```
?[table; where_clause; by_clause; select_clause]
```

- **table**: Table name or variable
- **where_clause**: List of conditions (empty list `()` = no filter)
- **by_clause**: Grouping columns (0b = no grouping, or a dictionary)
- **select_clause**: Columns to return (empty list = all)

**Building Functional Queries:**

**Simple SELECT (All Rows and Columns):**

```q
q) ?[trades; (); 0b; ()]
sym   time     qty price
--------------------------
AAPL  09:30:00 100 150.5
MSFT  09:30:01 200 300.2
GOOGL 09:30:02 150 2800.1
AAPL  09:30:03 50  151.0
```

**SELECT with WHERE Clause:**

```q
q) ?[trades; ((>; `qty; 100)); 0b; `sym`price]
sym   price
----------
MSFT  300.2
GOOGL 2800.1
```

**SELECT with Multiple WHERE Conditions:**

```q
q) ?[trades; ((>; `qty; 100); (<; `price; 300)); 0b; `sym`price]
sym   price
----------
MSFT  300.2
```

**SELECT Specific Columns:**

```q
q) ?[trades; (); 0b; `sym`qty]
sym   qty
--------
AAPL  100
MSFT  200
GOOGL 150
AAPL  50
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

This groups rows by symbol and sums the quantities.

**Multiple Aggregations:**

```q
q) select count i, sum qty, avg price by sym from trades
sym   | count qty avg price
------| --------------------
AAPL  | 2     150 150.75
GOOGL | 1     150 2800.1
MSFT  | 1     200 300.2
```

Aggregation functions:
- `count i`: Count rows
- `sum qty`: Sum of quantity column
- `avg price`: Average price
- `min`, `max`, `first`, `last`: Other aggregators

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

**Aggregation Functions Reference:**

| Function | Purpose | Example |
|----------|---------|---------|
| `count` | Count rows | `count i` |
| `sum` | Sum values | `sum qty` |
| `avg` | Average | `avg price` |
| `min` | Minimum | `min price` |
| `max` | Maximum | `max price` |
| `first` | First value | `first sym` |
| `last` | Last value | `last sym` |
| `wavg` | Weighted average | `wavg[price; qty]` |
| `var` | Variance | `var price` |
| `dev` | Std deviation | `dev price` |

**Aggregation with Expressions:**

```q
q) select totalValue: sum (qty * price) by sym from trades
sym   | totalValue
------| ----------
AAPL  | 15050
MSFT  | 60200
```

**Real-World Example: Daily Trading Summary**

```q
q) trades: ([]
  sym: `AAPL`AAPL`MSFT`MSFT`GOOGL;
  date: 2024.10.01 2024.10.01 2024.10.01 2024.10.02 2024.10.02;
  qty: 100 50 200 150 300;
  price: 150 151 300 301 2800
)

q) select 
  numTrades: count i,
  totalQty: sum qty,
  avgPrice: avg price,
  minPrice: min price,
  maxPrice: max price,
  totalValue: sum (qty * price)
  by sym, date 
  from trades

sym   date       | numTrades totalQty avgPrice minPrice maxPrice totalValue
---------------------------------------------------------------------------
AAPL  2024.10.01 | 2         150      150.5    150      151      15050
GOOGL 2024.10.02 | 1         300      2800     2800     2800     840000
MSFT  2024.10.01 | 1         200      300      300      300      60000
MSFT  2024.10.02 | 1         150      301      301      301      45150
```

---

## Complete Day 2 Project: Building a Trading System

```q
/ Create a trades table with market data
trades: ([]
  sym: `AAPL`MSFT`GOOGL`AAPL`MSFT`GOOGL;
  date: 2024.10.01 2024.10.01 2024.10.01 2024.10.02 2024.10.02 2024.10.02;
  time: 09:30:00 10:15:00 11:45:00 09:35:00 10:20:00 14:00:00;
  qty: 100 200 150 75 250 300;
  price: 150.5 300.2 2800.1 151.0 301.5 2820
)

/ Define trading utility functions in a namespace
.trading.calcTotal: { [qty; price] qty * price }
.trading.applyCommission: { [total; rate] total * (1 - rate) }
.trading.calculateProfit: { [buyPrice; sellPrice; qty] (sellPrice - buyPrice) * qty }

/ Create projections for common operations
apply001Commission: .trading.applyCommission[; 0.001]
apply005Commission: .trading.applyCommission[; 0.005]

/ Add computed columns
trades: update total: qty * price from trades
trades: update netProceeds: apply001Commission each total from trades

/ Query 1: Volume by symbol
volumeBySymbol: select sum qty by sym from trades
volumeBySymbol

/ Query 2: Daily trading summary
dailySummary: select
  numTrades: count i,
  totalVolume: sum qty,
  avgPrice: avg price,
  highPrice: max price,
  lowPrice: min price,
  totalValue: sum total
  by sym, date
  from trades

/ Query 3: Symbols trading on both dates
activeSymbols: select distinct sym by date from trades

/ Query 4: Largest trades (qty > 200)
largeTrades: select from trades where qty > 200
```

---

## Practice Exercises

### Exercise 1: Writing and Modularizing Functions

**1.1 Simple Functions**
```q
/ 1. Square function
q) square: { [x] x * x }
q) square 7
49

/ 2. Average of three numbers
q) avg3: { [a; b; c] (a + b + c) / 3 }
q) avg3[10; 20; 30]
20f

/ 3. Conditional function
q) isPositive: { [x] $[x > 0; "positive"; x = 0; "zero"; "negative"] }
q) isPositive 5
"positive"
```

**1.2 Modular Function**
```q
q) calculateSale: { [price; qty; taxRate]
  subtotal: price * qty;
  tax: subtotal * taxRate;
  total: subtotal + tax;
  ([] subtotal: enlist subtotal; tax: enlist tax; total: enlist total)
}

q) calculateSale[100; 5; 0.1]
subtotal tax total
50        5   55
```

### Exercise 2: Projection

**2.1 Basic Projection**
```q
q) add: { [x; y] x + y }
q) add10: add[10;]
q) add10 5
15

q) add10 25
35
```

**2.2 Discount Projection**
```q
q) applyDiscount: { [rate; price] price * (1 - rate) }
q) discount15: applyDiscount[0.15;]
q) discount15 100
85f

q) discount15 200
170f
```

### Exercise 3: Namespaces

**3.1 Organize Functions in Namespaces**
```q
q) .math.add: { [x; y] x + y }
q) .math.subtract: { [x; y] x - y }
q) .math.multiply: { [x; y] x * y }

q) .string.upper: { upper x }
q) .string.concat: { [a; b] a, b }

q) .math.add[10; 20]
30

q) .string.concat["Hello"; " World"]
"Hello World"
```

### Exercise 4: Tables and Queries

**4.1 Create and Query**
```q
q) employees: ([]
  id: 1 2 3 4;
  name: `Alice`Bob`Charlie`David;
  salary: 120000 80000 115000 85000;
  department: `engineering`sales`engineering`marketing
)

q) select name, salary from employees where salary > 100000

q) select avg salary by department from employees

q) select count i as headcount by department from employees
```

**4.2 CRUD Operations**
```q
/ INSERT
q) `employees insert (5; `Eve; 95000; `sales)

/ UPDATE
q) update salary: salary * 1.1 from `employees where department = `sales

/ DELETE
q) delete from `employees where id = 5

/ SELECT with complex aggregation
q) select count i as num, sum salary as payroll by department from employees
```

### Exercise 5: Iterators and Adverbs

**5.1 Using each**
```q
q) square: { x * x }
q) square each 1 2 3 4 5
1 4 9 16 25

q) prices: 100 200 300
q) discount: 0.1
q) { x * (1 - y) } each (prices; discount)
90 180 270
```

**5.2 Using over and scan**
```q
q) { x + y } over 1 2 3 4 5
15

q) { x + y } scan 1 2 3 4 5
1 3 6 10 15
```

---

## Summary

**Day 2 covers the core building blocks for data manipulation:**

✓ **Functions**: Writing, modularizing, and calling functions  
✓ **Projection**: Creating specialized functions from general ones  
✓ **Scope**: Understanding global and local variables  
✓ **Iterators**: Applying functions across lists (`each`, `over`, `scan`, `prior`)  
✓ **Namespaces**: Organizing code and preventing conflicts  
✓ **Tables**: Creating and understanding table structure  
✓ **CRUD Operations**: SELECT, INSERT, UPDATE, DELETE  
✓ **Queries**: SQL-like and functional query forms  
✓ **Aggregations**: GROUP BY and summary statistics  

These are the tools you need to build real-time data systems.

---

## Next Steps

In Day 3, you'll learn:
- Joins: Combining data from multiple tables
- I/O: Loading and saving data
- Database models: Splayed, partitioned, and segmented tables
- Optimization techniques
