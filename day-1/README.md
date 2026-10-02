# Day 1 – Learning the Building Blocks of Q Programming

## Learning Objectives

By the end of Day 1, you will understand:
- The fundamentals of Q language and its origins
- How to set up and use the Kdb+ development environment
- Startup options and how to configure Kdb+ processes
- How to start and exit a Q session correctly
- The core datatypes: atoms and lists
- Type numbers and how to inspect types
- Type casting and conversions
- Lists and dictionaries as fundamental data structures
- How to index, manipulate, and amend lists and dictionaries
- Order of evaluation and arithmetic operations
- The `@` (apply) and `.` (dot notation) operators

---

## Part 1: The Basics

### 1.1 Language Background

**What is Q?**

Q is a programming language designed specifically for data manipulation and real-time analytics. It is the query language used by **Kdb+**, a high-performance in-memory time-series database.

**Key Characteristics:**

- **Terse**: One line of Q can replace 10 lines of Python or SQL
- **Vector-oriented**: Operations work on entire lists implicitly
- **Functional**: Functions are first-class values
- **Expression-driven**: Everything evaluates to a value
- **Machine-sympathetic**: Designed for CPU and memory efficiency

**Historical Context:**

Q is a descendant of:
- **APL** (A Programming Language, 1962) – Kenneth Iverson's revolutionary array language
- **Lisp** – McCarthy's functional programming paradigm
- **K** – Arthur Whitney's predecessor to Q (even more terse)

The Q language was created by **Arthur Whitney** at Kx Systems around 1993-1998, building on these foundational languages while optimizing for financial data and real-time systems.

**Why Learn Q?**

1. **Financial Industry Standard**: Used by every major investment bank, trading firm, and hedge fund
2. **Performance**: Can process millions of transactions per second
3. **Conciseness**: Write more with less code
4. **Real-time Analytics**: Built for streaming data and live markets
5. **Time-series Excellence**: Optimized specifically for timestamped data

**Q vs. Other Languages:**

| Aspect | Q | SQL | Python |
|--------|---|-----|--------|
| Vector ops | Native | Not built-in | NumPy-dependent |
| Real-time | Designed for | Limited | Not ideal |
| Time-series | Native | Poor | Needs libraries |
| Performance | Microseconds | Milliseconds | Milliseconds+ |
| Conciseness | 1 line | 5 lines | 10+ lines |

**A Simple Example:**

Calculate the average price of all AAPL trades:

```sql
-- SQL
SELECT AVG(price) FROM trades WHERE sym = 'AAPL';
```

```python
# Python
df[df['sym'] == 'AAPL']['price'].mean()
```

```q
/ Q (query form)
select avg price from trades where sym = `AAPL

/ Q (functional form)
avg select[where sym=`AAPL] price from trades
```

---

### 1.2 The Development Environment

**What You Need:**

1. A terminal or command prompt
2. The Kdb+ executable (`q`)
3. A text editor or IDE (optional but recommended)
4. A working directory for scripts

**System Requirements:**

- **Linux**: 64-bit recommended (Kdb+ is officially 64-bit)
- **macOS**: Intel or Apple Silicon (Intel better supported)
- **Windows**: 64-bit Windows 10+
- **RAM**: Minimum 2GB; 8GB+ recommended for real systems
- **Disk**: 1GB for Kdb+, additional for data

**Installation Paths:**

**Option 1: Download from Kx Systems**
```bash
# Visit code.kx.com/download
# Personal edition is free (64-bit only)
# Extract to a directory, e.g., ~/kdb
```

**Option 2: Package Managers**
```bash
# macOS (Homebrew)
brew install kdb

# Linux (some distributions)
apt-get install kdb  # Debian/Ubuntu
yum install kdb      # RedHat/CentOS
```

**Setting Up Your Environment:**

**Step 1: Navigate to Kdb+ directory**
```bash
cd ~/kdb
ls
```

You should see:
```
q              (the executable)
q.k            (base library)
c.q            (standard library)
README
```

**Step 2: Create a workspace directory**
```bash
mkdir ~/myq
cd ~/myq
```

**Step 3: Start Q**
```bash
/path/to/q/q
```

Or add to your PATH:
```bash
export PATH=$PATH:~/kdb
q              # now works from anywhere
```

**Development Tools:**

- **Text Editors**: VS Code (with Q extension), Vim, Emacs
- **IDEs**: KX Studio, IKnow
- **REPL**: Interactive Q session
- **Notebooks**: Jupyter with kdb+ kernel

**Directory Structure Best Practice:**

```
myproject/
├── scripts/
│   ├── setup.q        # Initialization
│   ├── functions.q    # Custom functions
│   └── queries.q      # Standard queries
├── data/
│   ├── trades.csv     # Sample data
│   └── quotes.csv
└── README.md
```

---

### 1.3 Kdb+ Startup Options

**Basic Startup:**

```bash
q                    # Start interactive session
```

Output:
```
KDB+ 4.0 2024.04.01 (c) 2024 Kx Systems
l64/ 8* 16core 6.246GB RAM ...
Welcome to kdb+ 4.0
q)
```

**Common Startup Flags:**

| Flag | Purpose | Example |
|------|---------|---------|
| `-p PORT` | Listen on TCP port | `q -p 5001` |
| `-q` | Quiet mode (no banner) | `q -q` |
| `-s N` | Secondary threads | `q -s 4` |
| `-l` | Load from script | `q -l script.q` |
| `-u N` | Set user permission | `q -u 1` (read-only) |
| `-T N` | Timer ticks | `q -T 50` |
| `-w N` | Workspace limit (MB) | `q -w 500` |
| `-m N` | Memory limit (MB) | `q -m 1000` |

**Practical Examples:**

**Start as IPC Server:**
```bash
q -p 5001 -q
```

This starts Kdb+ listening on port 5001 in quiet mode. Other processes can connect:

```bash
# From another terminal
q)h: hopen `:localhost:5001
q)h "1 + 1"
2
```

**Load a Script on Startup:**
```bash
# mysetup.q contains:
/ Load trades data
trades: get `:trades.csv
/ Define utility functions
avg_price: {avg x}

# Start with:
q mysetup.q
```

Then the session has `trades` and `avg_price` already defined.

**Start with Secondary Threads for Parallel Execution:**
```bash
q -s 4
```

Starts Q with 4 secondary threads for parallel operations (more on this in advanced sections).

**Read-Only Mode (Safe for HDB Access):**
```bash
q -u 1 /path/to/hdb
```

Prevents accidental writes to the database.

**Set Memory Limit:**
```bash
q -m 2000
```

Limits Kdb+ to 2GB of memory (crashes if exceeded).

---

### 1.4 Starting and Exiting a Session

**Starting a Q Session:**

```bash
q
```

The prompt `q)` indicates readiness for input.

**First Commands:**

```q
q) 2 + 2
4

q) "hello world"
"hello world"

q) `symbol
`symbol
```

**Checking Your Environment:**

```q
q) .z.d                    / Today's date
2024.10.02

q) .z.t                    / Current time (milliseconds)
12:34:56.789

q) .z.h                    / Hostname
"COMPUTERNAME"

q) .z.w                    / Current connection handle (-1 for console)
-1

q) .z.D                    / Date as integer
18566

q) system "pwd"            / Execute system command
"/Users/trader/myq"
```

**Exiting the Session:**

There are several ways to exit:

**Method 1: Backslash (Standard Q Way)**
```q
q) \\
```

Two backslashes (sometimes appears as single `\` depending on display).

**Method 2: Exit Function**
```q
q) exit 0
```

The argument is the exit code (0 = success).

**Method 3: Ctrl+D (Unix/Linux/macOS)**
```
Press Ctrl+D
```

**Method 4: Ctrl+C (may require second press)**
```
Press Ctrl+C twice
```

**Best Practice**: Use `\\` as it's the official Q way:

```q
q) \\
$                    / Back to shell prompt
```

**Verifying Exit:**

```bash
echo $?              # Shows last exit code (0 = success)
```

---

## Part 2: Datatypes

### 2.1 Atoms

An **atom** is a single value in Q. Atoms are the fundamental building blocks.

**Basic Atom Types:**

**Integers:**
```q
q) 42
42

q) -100
-100

q) 0
0
```

**Floats:**
```q
q) 3.14
3.14

q) 2.5
2.5

q) 0.0
0f
```

**Symbols:**
```q
q) `apple
`apple

q) `AAPL
`AAPL

q) `my_variable
`my_variable
```

A symbol is Q's version of an interned string. Symbols are:
- Stored once in memory (no duplication)
- Compared by reference (fast)
- Ideal for categorical data (symbols, tickers, names)

**Strings:**
```q
q) "hello"
"hello"

q) "This is a string"
"This is a string"

q) ""
""
```

**Booleans:**
```q
q) 1b                 / true
1b

q) 0b                 / false
0b
```

**Dates:**
```q
q) 2024.10.02
2024.10.02

q) 2024.01.01
2024.01.01
```

Format: `YYYY.MM.DD`

**Times:**
```q
q) 12:34:56          / hh:mm:ss
12:34:56

q) 09:30:00
09:30:00

q) 23:59:59
23:59:59
```

**Timestamps (DateTime):**
```q
q) 2024.10.02T12:34:56.789
2024.10.02T12:34:56.789000000

q) 2024.10.02T09:30:00
2024.10.02T09:30:00.000000000
```

**Null Values:**
```q
q) 0N                 / null integer
0N

q) 0.0                / null float (displays as 0)
0f

q) ` or `$           / null symbol
`

q) " "               / null string (space)
" "
```

---

### 2.2 Lists

A **list** is an ordered collection of values of the same type.

**Integer Lists:**
```q
q) 1 2 3 4 5
1 2 3 4 5

q) 10 20 30
10 20 30

q) enlist 42           / single element as a list
,42
```

**Float Lists:**
```q
q) 1.5 2.5 3.5
1.5 2.5 3.5

q) 0.1 0.2 0.3
0.1 0.2 0.3
```

**Symbol Lists:**
```q
q) `AAPL`MSFT`GOOGL
`AAPL`MSFT`GOOGL

q) `alice`bob`charlie
`alice`bob`charlie
```

**String Lists:**
```q
q) "abc"              / equivalent to `a` `b` `c`
"abc"

q) ("hello";"world")  / list of two strings
"hello"
"world"
```

**Boolean Lists:**
```q
q) 1b 0b 1b 1b
1b 0b 1b 1b
```

**Mixed Lists (Generic):**
```q
q) (1; "hello"; `symbol; 2.5)
1
"hello"
`symbol
2.5
```

Mixed lists use parentheses and semicolons.

**Creating Lists:**

**Using Semicolons and Parentheses:**
```q
q) (1; 2; 3; 4; 5)
1 2 3 4 5
```

**Using Comma (Join):**
```q
q) 1 2 3, 4 5
1 2 3 4 5
```

**Using til (Generate Range):**
```q
q) til 5              / 0 to 4
0 1 2 3 4

q) til 10
0 1 2 3 4 5 6 7 8 9

q) 1 + til 5          / 1 to 5
1 2 3 4 5
```

**Using repeat:**
```q
q) 5 # 0              / repeat 0 five times
0 0 0 0 0

q) 3 # `a             / repeat symbol three times
`a`a`a
```

**Using enumerate:**
```q
q) til 3 3            / cartesian product
0 0
0 1
0 2
1 0
1 1
1 2
2 0
2 1
2 2
```

---

### 2.3 Type Numbers

Every Q value has a **type number**. Use the `type` function to inspect:

**Checking Types:**

```q
q) type 42
-7

q) type 3.14
-9

q) type `symbol
-11

q) type "string"
-10

q) type 1b
-1

q) type 2024.10.02
-14

q) type 12:34:56
-13
```

**Type Number System:**

| Type | Code | Example | Bytes |
|------|------|---------|-------|
| Boolean | -1 | `1b` | 1 |
| Byte | -4 | `0x42` | 1 |
| Short | -5 | `42h` | 2 |
| Integer | -7 | `42` | 4 or 8 |
| Long | -7 | `42j` | 8 |
| Float | -9 | `3.14` | 8 |
| Char | -10 | `"a"` | 1 |
| String | -10 | `"abc"` | variable |
| Symbol | -11 | `` `abc `` | 4 or 8 |
| Timestamp | -12 | `2024.10.02T12:34:56` | 8 |
| Timespan | -16 | `00:01:00` (duration) | 8 |
| Time | -13 | `12:34:56` | 4 |
| Date | -14 | `2024.10.02` | 4 |
| Month | -15 | `2024.10m` | 4 |

**List Type Numbers (Positive):**

```q
q) type 1 2 3         / list of integers
7

q) type `a`b`c        / list of symbols
11

q) type "abc"         / list of characters (string)
10

q) type 1b 0b 1b      / list of booleans
1

q) type (1; "hello"; `sym)  / mixed/generic list
0
```

**Null Type:**
```q
q) type 0N            / null integer
-7

q) type (::)          / generic null
101
```

---

### 2.4 Type Casting

**Type casting** converts a value from one type to another using the `$` operator.

**Syntax:**
```
`targetType$value
```

**Integer to Float:**
```q
q) `float$10
10f

q) `float$5
5f
```

**Float to Integer (Truncates):**
```q
q) `int$3.9
3

q) `int$4.2
4

q) `int$-2.7
-2
```

**String to Symbol:**
```q
q) `symbol$"apple"
`apple

q) `symbol$"AAPL"
`AAPL
```

**Symbol to String:**
```q
q) `string$`apple
"apple"

q) `string$`AAPL
"AAPL"
```

**Character/Byte Casting:**
```q
q) `char$65           / ASCII 65 = 'A'
"A"

q) `byte$0x42         / byte literal
0x42
```

**List Casting:**
```q
q) `int$1 2 3.5 4.9
1 2 3 4

q) `float$1 2 3
1 2 3f

q) `symbol$"a" "b" "c"
`a`b`c
```

**Casting with Errors:**

```q
q) `int$"not_a_number"
'type

q) `symbol$123        / integers don't cast to symbols
'type
```

Invalid casts signal an error (notice the single quote `'`).

**Safe Casting Pattern:**

Use conditionals to test before casting:

```q
q) s: "42"
q) $[all s in "0123456789"; `int$s; `symbol$s]
42
```

---

## Part 3: Lists and Dictionaries

### 3.1 List Indexing

Q uses **zero-based indexing**, where the first element is at index 0.

**Single Element Access:**

```q
q) l: 10 20 30 40 50
q) l[0]                / first element
10

q) l[1]                / second element
20

q) l[4]                / fifth element
50
```

**Negative Indexing (From End):**

```q
q) l: 10 20 30 40 50
q) l[-1]               / last element
50

q) l[-2]               / second from end
40

q) l[-3]               / third from end
30
```

**Multiple Elements:**

```q
q) l: 10 20 30 40 50
q) l[0 2 4]            / elements at indices 0, 2, 4
10 30 50

q) l[1 1 1]            / can repeat indices
20 20 20

q) l[0 1 2]
10 20 30
```

**Using Variables for Indices:**

```q
q) l: `a`b`c`d`e
q) idx: 1 3
q) l[idx]
`b`d

q) idx: 0 2 4
q) l[idx]
`a`c`e
```

**Out of Bounds:**

```q
q) l: 1 2 3
q) l[10]               / index beyond list length
'index
```

This signals an index error.

---

### 3.2 List Slicing

Use `take` (from the beginning) and `drop` (from the end) to create sublists.

**Take (First N Elements):**

```q
q) l: `a`b`c`d`e
q) 3 take l            / first 3 elements
`a`b`c

q) 2 take l
`a`b

q) 5 take l            / take all (equal to list length)
`a`b`c`d`e

q) 10 take l           / take more than length = pad with nulls
`a`b`c`d`e
```

**Drop (Remove First N Elements):**

```q
q) l: `a`b`c`d`e
q) 2 drop l            / drop first 2, keep rest
`c`d`e

q) 1 drop l
`b`c`d`e

q) 5 drop l            / drop all
`

q) 10 drop l           / drop more than length = empty
`
```

**Negative Take (Last N Elements):**

```q
q) l: `a`b`c`d`e
q) -2 take l           / last 2 elements
`d`e

q) -3 take l
`c`d`e

q) -1 take l
`e
```

**Negative Drop (Remove Last N Elements):**

```q
q) l: `a`b`c`d`e
q) -1 drop l           / remove last 1
`a`b`c`d

q) -2 drop l
`a`b`c

q) -3 drop l
`a`b
```

**Range Slicing (Using `til`):**

```q
q) l: 10 20 30 40 50
q) l[1 + til 3]        / indices 1,2,3 (next 3 starting at 1)
20 30 40

q) l[0 + til count l]  / all elements
10 20 30 40 50
```

---

### 3.3 List Manipulation and Amendment

**Concatenation (Join):**

```q
q) 1 2 3, 4 5
1 2 3 4 5

q) `a`b, `c`d
`a`b`c`d

q) (1 2), (3 4 5)
1 2 3 4 5
```

**Prepend (Add to Beginning):**

```q
q) (enlist 0), 1 2 3   / add 0 to front
0 1 2 3

q) `x, `a`b`c          / add symbol to front
`x`a`b`c
```

**Append (Add to End):**

```q
q) 1 2 3, (enlist 4)   / add 4 to end
1 2 3 4

q) `a`b`c, `z          / add symbol to end
`a`b`c`z
```

**Remove Duplicates (Distinct):**

```q
q) distinct 1 2 1 3 2 3 3
1 2 3

q) distinct `a`b`a`c`b
`a`b`c
```

**Sort (Ascending/Descending):**

```q
q) asc 3 1 4 1 5 9     / ascending order
1 1 3 4 5 9

q) desc 3 1 4 1 5 9    / descending order
9 5 4 3 1 1

q) asc `c`a`b
`a`b`c

q) desc `c`a`b
`c`b`a
```

**Count Elements:**

```q
q) count 1 2 3 4 5
5

q) count `a`b`c
3

q) count ""            / empty string
0
```

**Amendment (Modify Elements):**

```q
q) l: 10 20 30 40
q) l[1]: 99            / change element at index 1
q) l
10 99 30 40
```

**Amend Multiple Elements:**

```q
q) l: 10 20 30 40 50
q) l[1 3]: 88 77       / change indices 1 and 3
q) l
10 88 30 77 50
```

**Amend with Expression:**

```q
q) l: 10 20 30 40 50
q) l: l + 5            / add 5 to all elements
q) l
15 25 35 45 55

q) l[where l > 30]: 999 / amend elements > 30
q) l
15 25 35 999 999
```

---

### 3.4 Dictionaries

A **dictionary** is a key-value data structure. The `!` operator creates a dictionary.

**Creating Dictionaries:**

**Basic Dictionary:**
```q
q) d: `a`b`c! 10 20 30
q) d
a| 10
b| 20
c| 30
```

**Symbol Keys with Various Values:**
```q
q) prices: `AAPL`MSFT`GOOGL! 150 300 2500
q) prices
AAPL | 150
MSFT | 300
GOOGL| 2500
```

**Mixed-Type Dictionary:**
```q
q) data: `name`age`city! ("Alice"; 30; "NYC")
q) data
name| Alice
age | 30
city| NYC
```

**From Lists:**
```q
q) keys: `x`y`z
q) values: 1 2 3
q) d: keys! values
q) d
x| 1
y| 2
z| 3
```

---

### 3.5 Dictionary Access

**Get Value by Key:**

```q
q) d: `a`b`c! 10 20 30
q) d[`a]
10

q) d[`b]
20

q) prices: `AAPL`MSFT`GOOGL! 150 300 2500
q) prices[`AAPL]
150
```

**Multiple Keys:**

```q
q) d: `a`b`c! 10 20 30
q) d[`a`c]
10
30

q) d[`b`a`b]
20
10
20
```

**Get All Keys:**

```q
q) d: `a`b`c! 10 20 30
q) key d
`a`b`c

q) key prices
`AAPL`MSFT`GOOGL
```

**Get All Values:**

```q
q) d: `a`b`c! 10 20 30
q) value d
10 20 30

q) value prices
150 300 2500
```

**Check Key Existence:**

```q
q) d: `a`b`c! 10 20 30
q) `a in key d
1b

q) `z in key d
0b
```

---

### 3.6 Dictionary Amendment (Modification)

**Change an Existing Value:**

```q
q) d: `a`b`c! 10 20 30
q) d[`a]: 99
q) d
a| 99
b| 20
c| 30
```

**Add a New Key:**

```q
q) d: `a`b`c! 10 20 30
q) d[`d]: 40
q) d
a| 10
b| 20
c| 30
d| 40
```

**Update Multiple Keys:**

```q
q) d: `a`b`c! 10 20 30
q) d[`a`c]: 99 88
q) d
a| 99
b| 20
c| 88
```

**Remove a Key (Set to Null):**

```q
q) d: `a`b`c! 10 20 30
q) d[`b]: ::             / :: is null
q) d
a| 10
b|
c| 30
```

**Merge Dictionaries:**

```q
q) d1: `a`b! 1 2
q) d2: `c`d! 3 4
q) d1, d2               / merge
a| 1
b| 2
c| 3
d| 4
```

---

## Part 4: Basic Operations

### 4.1 Order of Evaluation

Q follows standard **operator precedence**. Evaluation order (highest to lowest priority):

1. **Parentheses** `()`
2. **Exponentiation** `^`, `xexp`
3. **Multiplication/Division** `*`, `%`, `/`, `div`
4. **Addition/Subtraction** `+`, `-`
5. **Comparison** `<`, `>`, `<=`, `>=`, `=`, `<>`
6. **Boolean AND** `and`, `&`
7. **Boolean OR** `or`, `|`
8. **Assignment** `:`

**Examples:**

```q
q) 2 + 3 * 4           / multiply first, then add
14
/ This is: 2 + (3 * 4) = 2 + 12 = 14

q) (2 + 3) * 4         / parentheses change order
20
/ This is: (2 + 3) * 4 = 5 * 4 = 20

q) 10 - 3 - 2          / left-to-right (not right-to-left)
5
/ This is: (10 - 3) - 2 = 7 - 2 = 5
/ Not: 10 - (3 - 2) = 10 - 1 = 9

q) 16 % 2 ^ 3          / exponentiation before modulo
0
/ This is: 16 % (2 ^ 3) = 16 % 8 = 0

q) 5 > 3 and 2 = 2     / comparisons before boolean
1b
/ This is: (5 > 3) and (2 = 2) = 1b and 1b = 1b

q) 2 = 2 or 5 < 3      / OR has lower priority than AND
1b
/ This is: (2 = 2) or (5 < 3) = 1b or 0b = 1b
```

**Use Parentheses for Clarity:**

```q
/ Bad (ambiguous)
select from trades where price > 100 and qty > 50 or sym = `AAPL

/ Better (explicit)
select from trades where (price > 100 and qty > 50) or sym = `AAPL
```

---

### 4.2 Arithmetic Operations

**Basic Operators:**

```q
q) 5 + 3               / addition
8

q) 10 - 4              / subtraction
6

q) 8 * 2               / multiplication
16

q) 15 / 3              / division
5f

q) 15 % 4              / modulo (remainder)
3

q) 2 ^ 3               / exponentiation
8

q) 9 ^ 0.5             / square root (power 0.5)
3f
```

**Mathematical Functions:**

```q
q) abs -5              / absolute value
5

q) sqrt 16             / square root
4f

q) sin 0               / trigonometric (radians)
0f

q) cos 3.14159         / close to -1
-1f

q) exp 1               / e^1
2.718282

q) log 10              / natural logarithm
2.302585

q) floor 3.7           / round down
3f

q) ceiling 3.2         / round up
4f

q) round 3.5           / round to nearest
4f
```

**Vector Arithmetic (Implicit Broadcasting):**

Q automatically applies operations element-wise to lists:

```q
q) (1 2 3) + 10        / add 10 to each element
11 12 13

q) (1 2 3) * 2         / multiply each by 2
2 4 6

q) (1 2 3) + (10 20 30) / element-wise addition
11 22 33

q) (1 2 3 4 5) - 2     / subtract 2 from each
-1 0 1 2 3

q) (10 20 30) % 7      / modulo on each
3 6 2

q) 100 / (1 2 4 5)     / divide 100 by each
100 50 25 20
```

**Min/Max Functions:**

```q
q) min 3 1 4 1 5 9
1

q) max 3 1 4 1 5 9
9

q) min (1 2 3 4 5)
1

q) max (10 20 30)
30
```

**Sum and Product:**

```q
q) sum 1 2 3 4 5       / total
15

q) prd 1 2 3 4 5       / product (multiply all)
120

q) avg 10 20 30        / average
20f

q) var 1 2 3 4 5       / variance
2.5

q) dev 1 2 3 4 5       / standard deviation
1.581139
```

---

### 4.3 The `@` Operator (Apply)

The `@` operator applies a function to a value or list.

**Syntax:**
```
function @ value
{expression} @ value
```

**Simple Examples:**

```q
q) {x + 1} @ 5         / apply function to 5
6

q) {x * 2} @ 10
20

q) abs @ -5            / apply abs to -5
5

q) sqrt @ 16
4f
```

**Apply to Lists:**

```q
q) {x + 1} @ (1 2 3 4)
2 3 4 5

q) {x * x} @ (1 2 3 4 5)
1 4 9 16 25

q) sqrt @ (4 9 16 25)
2 3 4 5f
```

**Using with Operators:**

```q
q) sum @ (1 2 3 4 5)
15

q) avg @ (10 20 30 40)
25f

q) max @ (3 1 4 1 5)
5

q) min @ (3 1 4 1 5)
1
```

**Function Composition:**

```q
q) {sqrt abs x} @ -16
4f
/ Equivalent to: sqrt @ (abs @ -16)

q) {x * 2 + 1} @ 5
11
/ Multiple operations in one function
```

**Why Use `@`?**

1. **Clarity**: Makes it explicit that you're applying a function
2. **Composition**: Chain operations clearly
3. **Higher-order functions**: Pass functions as arguments

---

### 4.4 The `.` Operator (Dot Notation)

The `.` operator accesses namespace variables and nested structures.

**System Namespaces:**

The `.z` namespace contains system information:

```q
q) .z.d                / current date
2024.10.02

q) .z.t                / current time (milliseconds)
12:34:56.789

q) .z.D                / date as integer (days since 2000.01.01)
18566

q) .z.T                / time as long (milliseconds since midnight)
45296789

q) .z.h                / hostname
"MACHINENAME"

q) .z.u                / username
"trader"

q) .z.w                / current handle (-1 for console, else connection handle)
-1

q) .z.p                / current timestamp
2024.10.02T12:34:56.789000000

q) .z.Z                / UTC timestamp
2024.10.02T16:34:56.789000000
```

**The `.Q` namespace:**

Utility functions:

```q
q) .Q.ind[`a`b`c]      / index function
0
1
2

q) .Q.nv[]             / list of names and values (first 20)
`...

q) .Q.fmt              / format function
q) .Q.dd               / date arithmetic
q) .Q.ty[1 2 3]        / type of each element
`j`j`j
```

**Accessing Nested Variables:**

```q
q) config: ([] name:`server`port`user; value:"localhost"; 5001; "trader")
q) config.name         / dot notation (if column name)
`server`port`user
```

**Dictionary Access (Similar to `.`):**

```q
q) d: `a`b`c! 10 20 30
q) d.a                 / same as d[`a]
10

q) d.b
20

q) d.c
30
```

But note: only works with symbol keys, not numeric keys.

**Function Calls via `.`:**

```q
q) .Q.fmt["hello"]     / call function in namespace
"hello"
```

---

## Practice Exercises

### Exercise Set 1: Types and Casting

**1.1 Type Inspection**
```q
/ Check the types of these values:
q) type 42
q) type 3.14
q) type `symbol
q) type "string"
q) type 1b
q) type 2024.10.02
q) type 12:34:56
```

**Expected Output**: `-7, -9, -11, -10, -1, -14, -13`

**1.2 List Types**
```q
q) type 1 2 3          / list of integers
q) type `a`b`c         / list of symbols
q) type "abc"          / string
q) type 1b 0b 1b       / list of booleans
```

**Expected Output**: `7, 11, 10, 1`

**1.3 Type Casting**
```q
q) `float$10           / int to float
q) `int$4.9            / float to int
q) `symbol$"apple"     / string to symbol
q) `string$`symbol     / symbol to string
q) `int$1.1 2.9 3.5    / list casting
```

**Expected Output**: `10f, 4, `apple, "symbol", 1 2 3`

---

### Exercise Set 2: Lists and Indexing

**2.1 Create and Index**
```q
q) x: 10 20 30 40 50
q) x[0]                / first element
q) x[2]                / third element
q) x[-1]               / last element
q) x[1 3]              / elements at indices 1,3
q) x[0 2 4]            / elements at indices 0,2,4
```

**Expected Output**: `10, 30, 50, 20 40, 10 30 50`

**2.2 Slicing**
```q
q) l: `a`b`c`d`e
q) 3 take l            / first 3
q) -2 take l           / last 2
q) 1 drop l            / drop first 1
q) -1 drop l           / drop last 1
```

**Expected Output**: `` `a`b`c, `d`e, `b`c`d`e, `a`b`c`d ``

**2.3 Manipulation**
```q
q) l: 1 2 3
q) l, 4 5              / join lists
q) (enlist 0), l       / prepend
q) l, (enlist 6)       / append
q) distinct 1 2 1 3 2  / remove duplicates
q) asc 5 2 8 1         / sort ascending
```

**Expected Output**: `1 2 3 4 5, 0 1 2 3, 1 2 3 6, 1 2 3, 1 2 5 8`

**2.4 Amendment**
```q
q) l: 10 20 30 40
q) l[1]: 99
q) l
q) l[0 2]: 88 77
q) l
```

**Expected Output**: After amendments: `10 99 30 40`, then `88 99 77 40`

---

### Exercise Set 3: Dictionaries

**3.1 Create and Access**
```q
q) d: `a`b`c! 10 20 30
q) d[`a]               / get value for key a
q) d[`b`c]             / get values for keys b,c
q) key d               / get all keys
q) value d             / get all values
```

**Expected Output**: `10, 20 30, `a`b`c, 10 20 30`

**3.2 Amendment**
```q
q) d: `a`b! 1 2
q) d[`a]: 99
q) d
q) d[`c]: 3            / add new key
q) d
```

**Expected Output**: After first: `a| 99`, `b| 2`. After second: `a| 99`, `b| 2`, `c| 3`

---

### Exercise Set 4: Arithmetic and Operators

**4.1 Order of Evaluation**
```q
q) 2 + 3 * 4           / should be 14
q) (2 + 3) * 4         / should be 20
q) 10 - 3 - 2          / should be 5 (left-to-right)
q) 16 % 2 ^ 3          / should be 0
```

**4.2 Vector Arithmetic**
```q
q) (1 2 3) + 10        / add 10 to each
q) (1 2 3) * 2         / multiply each by 2
q) (1 2 3) + (10 20 30) / element-wise addition
q) 100 / (1 2 4 5)     / divide 100 by each
```

**Expected Output**: `11 12 13, 2 4 6, 11 22 33, 100 50 25 20`

**4.3 Mathematical Functions**
```q
q) abs -5              / absolute value
q) sqrt 16             / square root
q) sum 1 2 3 4 5       / sum all
q) avg 10 20 30        / average
q) max 3 1 4 1 5       / maximum
```

**Expected Output**: `5, 4f, 15, 20f, 5`

**4.4 Apply Operator**
```q
q) {x + 1} @ 5         / apply to single value
q) {x * 2} @ (1 2 3 4) / apply to list
q) sqrt @ 16           / apply built-in
q) sum @ (1 2 3 4 5)   / apply sum
```

**Expected Output**: `6, 2 4 6 8, 4f, 15`

---

## Summary

**Day 1 Building Blocks:**

✓ **The Basics**: Q language history, development environment, startup options, starting/exiting sessions

✓ **Datatypes**: Atoms (integers, floats, symbols, strings, booleans, dates, times), lists, type numbers, casting

✓ **Lists & Dictionaries**: Indexing (zero-based, negative), slicing (take/drop), manipulation (join, distinct, sort, count), amendment, dictionary creation/access/amendment

✓ **Basic Operations**: Order of evaluation, arithmetic (+, -, *, /, %, ^), vector arithmetic, mathematical functions, `@` (apply), `.` (dot notation)

**These are the building blocks for:**
- Day 2: Functions, namespaces, tables, queries
- Day 3: Joins, I/O, database models
- Day 4: IPC and tickerplant architecture
- Day 5: Assessment and real-world systems

---

## Quick Reference

**Starting Q:**
```bash
q                      # Start interactive
q -p 5001              # Start on port 5001
q script.q             # Load script
```

**Exiting Q:**
```q
\\                     # Exit (standard way)
exit 0                 # Exit with code
```

**Type Checking:**
```q
type x                 / Get type number
```

**List Operations:**
```q
l[0]                   / First element
l[-1]                  / Last element
l[1 2 3]               / Multiple indices
3 take l               / First 3 elements
-2 take l              / Last 2 elements
l, m                   / Join lists
distinct l             / Remove duplicates
```

**Dictionary Operations:**
```q
d: `a`b! 1 2           / Create
d[`a]                  / Get value
d[`a]: 99              / Amend
```

**Arithmetic:**
```q
2 + 3 * 4              / Order of precedence matters
(1 2 3) + 10           / Vector arithmetic
{x + 1} @ 5            / Apply function
.z.d                   / System info
```

---

## Next Steps

You now have the foundational knowledge of Q programming. In Day 2, we'll build on these basics to explore:
- Writing and using functions
- Function projection
- Variable scope
- Iterators and adverbs
- Namespaces for code organization
- Table creation and manipulation
- Queries and aggregations

Recommended Review: Revisit the Practice Exercises and ensure you can comfortably work with lists, dictionaries, and basic arithmetic before proceeding.
