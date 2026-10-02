# Day 1 – Foundations of Q Programming

## Learning Objectives

By the end of Day 1, you will understand:
- The background and philosophy of Q and Kdb+
- The development environment used for writing and running Q code
- How to start, configure, and exit a Kdb+ session
- The core Q datatypes and how they are represented
- How lists and dictionaries work in Q
- How to index, manipulate, and amend data structures
- The basic rules of evaluation and arithmetic in Q
- How `@` and `.` are used in practical Q code

---

## 1. Language Background

Q is the query language used by Kdb+, a high-performance time-series database designed for real-time analytics and market data processing. It is a terse, expression-based language that is especially well-suited for:

- Streaming data processing
- Real-time analytics
- Financial and market data systems
- Table-driven queries and aggregations
- High-throughput event processing

Q is built around a small set of very powerful ideas:

- Everything is a value
- Lists and tables are first-class data structures
- Functions are data too
- Expressions are evaluated directly
- Code is concise and often built from composable operators

This compact style allows a very small amount of code to do a lot of work, which is one reason Q is used heavily in trading and analytics environments.

### Why Q is different

Unlike many programming languages, Q is strongly oriented around data manipulation and declarative queries. In Q:

- Simple expressions are central
- Data is often represented as lists, dictionaries, and tables
- Queries can be written in concise functional form
- Operators are often applied uniformly across data

This makes Q especially effective when working with large volumes of time-series data and symbol-based market data.

---

## 2. The Development Environment

A typical Q environment includes:

- A terminal or command prompt
- The Kdb+ executable (`q`)
- A working directory for scripts or notebooks
- Optional editor support (VS Code, Vim, etc.)
- Access to local or remote Kdb+ processes

### Basic workflow

A common workflow is:

1. Open a terminal
2. Start the `q` process
3. Write expressions interactively
4. Save reusable logic in `.q` script files
5. Load scripts or define functions as needed

### Example environment setup

On Linux/macOS, start Q from the shell:

```bash
q
```

You will then see the Q prompt:

```q
q)
```

This prompt indicates that Q is ready to accept expressions.

### Practical tips

- Use scripts for reusable logic rather than one-off commands
- Keep functions small and focused
- Prefer readable variable names
- Use comments to explain logic when needed

---

## 3. Kdb+ Startup Options

Q is launched from the command line. Several startup options are available depending on the use case.

### Basic startup

```bash
q
```

This starts an interactive Q session.

### Start on a specific port

```bash
q -p 5001
```

This starts a Q process listening on port `5001`, typically used for IPC or client/server access.

### Start in a script file

```bash
q script.q
```

This runs the contents of `script.q` immediately on startup.

### Quiet mode or non-interactive execution

```bash
q -q
```

This suppresses some startup output and is useful for scripting or automation.

### Start with a specific workspace or custom setup

```bash
q -l
```

This loads the Q environment with a log file or local startup options, depending on installation details.

### Common options summary

- `q` – start Q
- `q -p 5001` – run server on port 5001
- `q script.q` – execute a script on startup
- `q -q` – quiet mode
- `q -l` – custom load setup depending on environment

A real system may also use other flags, but the general idea remains the same: Q is started from the shell and then used interactively or as a service.

---

## 4. Starting and Exiting a Q Session

### Starting a session

Open a terminal and type:

```bash
q
```

You should see something like:

```q
KDB+ 4.0 ...
q)
```

The `q)` prompt means Q is active and waiting for an expression.

### Example expressions

```q
q) 2 + 3
5

q) 10 * 5
50

q) `abc
`abc
```

### Exiting the session

To exit Q, type:

```q
\\
```

This is the standard Q exit command.

You may also quit from a process by using a system-style termination depending on your environment, but in the interactive shell the normal method is:

```q
\\
```

---

## 5. Datatypes

Q has a small but powerful set of datatypes. The most important categories are:

- Atoms
- Lists
- Symbols
- Strings
- Booleans
- Numbers
- Temporal types (dates, times, timestamps)

### 5.1 Atoms

An atom is a single value.

Examples:

```q
q) 5
5
q) 3.14
3.14
q) `abc
`abc
q) "hello"
"hello"
```

Atoms can be numeric, symbol, string, boolean, temporal, or other scalar types.

### 5.2 Lists

A list is an ordered collection of values.

```q
q) 1 2 3 4 5
1 2 3 4 5

q) `a`b`c
`a`b`c

q) "abc"
"abc"
```

Lists are extremely important in Q because they are used for vectors and as the building blocks of tables.

### 5.3 Type numbers

Every Q value has a type, and you can inspect it using `type`.

```q
q) type 5
-7
q) type 1.5
-9
q) type `abc
-11
q) type "abc"
-10
```

The result is a type number. Q has a fixed mapping between type number and datatype. For example:

- `-7` = integer
- `-9` = float
- `-10` = string
- `-11` = symbol
- `-1` = boolean

A full type table is available in Q documentation, but the important idea is that every value belongs to a specific type.

### 5.4 Casting

Casting converts a value from one datatype to another.

```q
q) 10h
10
q) "123"
"123"
q) "I"$"123"
123
```

In Q, casting is done using the notation:

```q
<type>$<value>
```

Examples:

```q
q) `float$5
5f
q) `int$3.9
3
q) `symbol$"abc"
`abc
```

One of the most important ideas in Q is that values can be strongly typed and conversions are explicit.

### 5.5 Common datatypes

```q
q) type 42
-7
q) type 42.5
-9
q) type 01:02:03
-13
q) type 2024.10.02
-14
q) type 1b
-1
```

These types are heavily used in Kdb+ for market and time-series data.

---

## 6. Lists and Dictionaries

### 6.1 Indexing a list

Q uses zero-based indexing for lists.

```q
q) l: 10 20 30 40
q) l[0]
10
q) l[1]
20
q) l[2 3]
30 40
```

The first element is at index `0`.

### 6.2 List slicing

```q
q) l: 10 20 30 40 50
q) l[0 2 4]
10 30 50
q) l[1 2]
20 30
```

You can also use ranges conceptually depending on the expression being used, but the key concept is that lists are ordered and indexable.

### 6.3 Manipulating lists

```q
q) l: 1 2 3
q) 0,l
0 1 2 3
q) l,4
1 2 3 4
q) 1_l
2 3
```

Common operations include:

- concatenation using `,`
- prepending or appending values
- selecting subsets by index
- transforming values with functions

### 6.4 Dictionaries

A dictionary maps keys to values.

```q
q) d: `a`b`c!10 20 30
q) d
a| 10
b| 20
c| 30
```

Dictionary keys can be symbols, dates, or other Q types. Values are arbitrary Q values.

### 6.5 Accessing dictionary values

```q
q) d[`a]
10
q) d[`b]
20
```

### 6.6 Updating a dictionary

```q
q) d[`a]: 99
q) d
a| 99
b| 20
c| 30
```

This is an example of amendment, where a value is replaced at a particular key.

### 6.7 Amendment of lists

```q
q) l: 10 20 30
q) l[1]: 99
q) l
10 99 30
```

This is one of the simplest examples of changing a value in an existing structure.

### 6.8 Important idea: structure + indexing

Q heavily relies on the idea that data structures are manipulated by position or key. This is fundamental to both basic data processing and large table queries.

---

## 7. Basic Operations

### 7.1 Order of evaluation

Q follows standard expression evaluation, where arithmetic and function application occur in a predictable order.

Example:

```q
q) 2 + 3 * 4
14
```

This is computed as:

```q
2 + (3 * 4)
```

So multiplication happens before addition unless parentheses change the order.

Another example:

```q
q) (2 + 3) * 4
20
```

Parentheses force evaluation first.

### 7.2 Arithmetic operations

```q
q) 5 + 3
8
q) 10 - 4
6
q) 8 * 2
16
q) 15 % 4
3
q) 9 % 2
1
```

Q also supports common arithmetic expressions on lists:

```q
q) 1 2 3 + 10
11 12 13
```

This is a key Q feature: many operations work element-wise across lists.

### 7.3 Logical and comparison operators

```q
q) 5 > 3
1b
q) 5 < 3
0b
q) (1 = 1)
1b
q) (1 <> 2)
1b
```

Boolean results are represented as `1b` or `0b`.

### 7.4 `@` and `.`

#### `@` - apply or use a function on a value

The `@` operator is used for applying a function or expression to data in a compact way.

```q
q) {x + 1} @ 5
6
```

This means: take the function `{x + 1}` and apply it to `5`.

This is especially useful in functional programming contexts when you want to apply a function directly.

#### `.` - namespace access / dot notation

The dot `.` is used to access names in namespace-like structures.

```q
q) .z.p
```

This is a built-in namespace value in Q. The dot is used to reach names such as `.z`, `.Q`, and others.

Examples:

```q
q) .Q.q
```

The . operator is central in Q for organizing functions and variables into logical groups.

This is especially important later when we discuss namespaces and `.z` / `.Q` systems.

---

## 8. Practical Examples

### Example 1: Basic arithmetic

```q
q) a: 10
q) b: 3
q) a + b
13
q) a * b
30
```

### Example 2: Lists and indexing

```q
q) nums: 5 10 15 20
q) nums[0]
5
q) nums[1 3]
10 20
```

### Example 3: Dictionary lookup

```q
q) prices: `AAPL`MSFT`GOOG!150 300 2500
q) prices[`AAPL]
150
```

### Example 4: Amendment

```q
q) values: 10 20 30
q) values[2]: 99
q) values
10 20 99
```

---

## Practice Exercises

### Exercise 1: Start a Q session

```bash
q
```

Then in the prompt:

```q
q) 2 + 2
4
```

### Exercise 2: Identify datatypes

```q
q) type 7
q) type 7.5
q) type `abc
q) type "abc"
```

### Exercise 3: Cast values

```q
q) `float$10
q) `int$4.9
q) `symbol$"apple"
```

### Exercise 4: Work with list indexing

```q
q) x: 10 20 30 40
q) x[0]
q) x[1 3]
```

### Exercise 5: Update a dictionary or list

```q
q) d: `a`b!1 2
q) d[`a]: 50
q) d

q) l: 1 2 3
q) l[1]: 99
q) l
```

### Exercise 6: Evaluate arithmetic expressions

```q
q) 2 + 3 * 4
q) (2 + 3) * 4
```

---

## Summary

Day 1 focuses on the essential building blocks of Q programming:

- Q is an expression-driven language that is optimized for data manipulation
- The Kdb+ environment is started from the command line and used interactively
- Values are typed and can be cast explicitly between types
- Lists and dictionaries are core structures in Q
- Indexing and amendment are central to working with data
- Arithmetic and evaluation order are simple but powerful
- `@` and `.` are foundational operators used throughout Q

These concepts are the foundation for everything that follows in Days 2–5.

---

## Quick Revision Checklist

Before moving to Day 2, make sure you can:
- Start and exit a Q session
- Use basic arithmetic expressions
- Identify types with `type`
- Cast values explicitly
- Create and index lists
- Create and update dictionaries
- Understand the basics of `@` and `.`
- Explain why Q is useful for time-series and market data work
