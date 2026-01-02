# CQEngine Next - Maintained Fork

[![Java](https://img.shields.io/badge/Java-21-blue.svg)](https://openjdk.org/projects/jdk/21/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)
[![Maven Central](https://img.shields.io/maven-central/v/io.github.msaifasif/cqengine.svg?label=Maven%20Central)](https://central.sonatype.com/artifact/io.github.msaifasif/cqengine)
[![Status](https://img.shields.io/badge/Status-Maintained%20Fork-brightgreen)](https://github.com/MSaifAsif/cqengine)
![Open Issues](https://img.shields.io/github/issues/MSaifAsif/cqengine)
![Java Version](https://img.shields.io/badge/Java-8+-blue?logo=openjdk)
![Last Commit](https://img.shields.io/github/last-commit/MSaifAsif/cqengine-next)
![Maintained](https://img.shields.io/badge/Maintained%3F-yes-green.svg)
![Stars](https://img.shields.io/github/stars/MSaifAsif/cqengine?style=social)

> **Note**: This is a maintained fork of the original [CQEngine](https://github.com/npgall/cqengine) by Niall Gallagher. This version is published under the GroupId `io.github.msaifasif` to provide continued support and updates for Java 21+.

---

## 📋 Table of Contents

- [What is CQEngine?](#what-is-cqengine)
- [The Limits of Iteration](#the-limits-of-iteration)
- [CQEngine Overview](#cqengine-overview)
- [Maintenance Notice](#maintenance-notice)
- [Quick Start](#quick-start)
- [Migration from Original CQEngine](#migration-from-original-cqengine)
- [What's New in This Fork](#whats-new-in-this-fork)
- [Docker-Based Testing](#docker-based-testing)
- [Building from Source](#building-from-source)
- [Complete Example](#complete-example)
- [String-Based Queries: SQL and CQN Dialects](#string-based-queries-sql-and-cqn-dialects)
- [Feature Matrix for Included Indexes](#feature-matrix-for-included-indexes)
- [Attributes](#attributes)
- [Joins](#joins)
- [Persistence: On-Heap, Off-Heap, Disk](#persistence-on-heap-off-heap-disk)
- [Result Sets](#result-sets)
- [Deduplicating Results](#deduplicating-results)
- [Ordering Results](#ordering-results)
- [Using CQEngine with Hibernate / JPA / ORM Frameworks](#using-cqengine-with-hibernate--jpa--orm-frameworks)
- [Using CQEngine in Scala, Kotlin, or Other JVM Languages](#using-cqengine-in-scala-kotlin-or-other-jvm-languages)
- [Related Projects](#related-projects)
- [Project History](#project-history)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## What is CQEngine?

**CQEngine** (Collection Query Engine) is a NoSQL indexing and query engine for Java collections with ultra-low latency. It provides SQL-like queries on Java collections with minimal overhead.

### Key Features

- **Ultra-fast queries** - Orders of magnitude faster than iterating collections
- **SQL-like syntax** - Familiar query syntax with complex boolean logic
- **Multiple index types** - Hash, NavigableSet, Radix Tree, Suffix Tree, and more
- **Lazy evaluation** - Results computed on-demand with iterator fusion
- **Persistence options** - On-heap, off-heap, disk, and SQLite backends
- **Zero dependencies** - Core library has no external dependencies
- **Thread-safe** - Concurrent reads and writes supported

### Use Cases

- In-memory caching with complex queries
- Real-time analytics on live data
- High-performance data filtering and aggregation
- Embedded databases for desktop/mobile applications
- Standing queries on streaming data

### Real-World Usage

CQEngine is used in production by several organizations:
- [excelian.com](http://www.excelian.com/exposure-and-counterparty-limit-checking) - Exposure and counterparty limit checking
- [gravity4.com](http://gravity4.com/welcome-gravity4-engineering-blog/) - Marketing intelligence platform
- [snapdeal.com](http://engineering.snapdeal.com/how-were-building-a-system-to-scale-for-billions-of-requests-per-day-201601/) - Processing 3-5 billion requests/day

### Reviews and Articles

- [dzone.com: Comparing the search performance of CQEngine with standard Java collections](https://dzone.com/articles/comparing-search-performance)
- [dzone.com: Getting started with CQEngine: LINQ for Java, only faster](https://dzone.com/articles/getting-started-cqengine-linq)

---

## The Limits of Iteration

The classic way to retrieve objects matching some criteria from a collection is to iterate through the collection and apply tests to each object. If the object matches the criteria, it's added to a result set. This is repeated for every object in the collection.

**Conventional iteration is hugely inefficient**, with time complexity O(_n_ _t_). It can be optimized, but requires statistical knowledge of the makeup of the collection. [Read more: The Limits of Iteration](documentation/TheLimitsOfIteration.md)

### Benchmark: CQEngine vs. Iteration

Even with optimizations applied to conventional iteration, CQEngine can outperform it by wide margins. Here is a benchmark for a range-type query:

![quantized-navigable-index-carid-between.png](documentation/images/quantized-navigable-index-carid-between.png)

**Results:**
- **1,116,071 queries per second** (on a single 1.8GHz CPU core)
- **0.896 microseconds per query**
- CQEngine is **330,187.50% faster** than naive iteration
- CQEngine is **325,727.79% faster** than optimized iteration

See the [Benchmark](documentation/Benchmark.md) page for details of this test and other tests with various query types.

---

## CQEngine Overview

CQEngine solves the scalability and latency problems of iteration by making it possible to build _indexes_ on the fields of objects stored in a collection, and applying algorithms based on the rules of set theory to _reduce time complexity_ of accessing them.

### Indexing and Query Plan Optimization

- **Simple Indexes** can be added to any number of individual fields, allowing queries on just those fields to be answered in O(1) time complexity
- **Multiple indexes on the same field** can be added, each optimized for different types of query (equality, numerical range, string starts with, etc.)
- **Compound Indexes** can span multiple fields, allowing queries referencing several fields to be answered in O(1) time complexity
- **Nested Queries** are fully supported, such as the SQL equivalent of `WHERE color = 'blue' AND (NOT (doors = 2 OR price > 53.00))`
- **Standing Query Indexes** allow _arbitrarily complex queries_ or _nested query fragments_ to be answered in O(1) time complexity, regardless of the number of fields referenced
- **Statistical Query Plan Optimization** - when several fields have suitable indexes, CQEngine uses statistical information to internally make a query plan which selects the indexes with minimum time complexity
- **Iteration fallback** - if no suitable indexes are available, CQEngine evaluates queries via iteration using lazy evaluation. CQEngine can always evaluate every query, even without indexes
- **Full concurrency** - objects can be added/removed at runtime; CQEngine updates all registered indexes in realtime
- **Type-safe** - nearly all errors in queries result in _compile-time_ errors instead of runtime exceptions; all indexes and queries are strongly typed using generics at both object-level and field-level
- **On-heap/off-heap/disk** - objects can be stored on-heap (like a conventional Java collection), off-heap (in native memory), or persisted to disk

### IndexedCollection Implementations

Several implementations supporting various concurrency and transaction isolation levels:

- [ConcurrentIndexedCollection](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/ConcurrentIndexedCollection.html) - lock-free concurrent reads and writes with no transaction isolation
- [TransactionalIndexedCollection](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/TransactionalIndexedCollection.html) - lock-free concurrent reads, and sequential writes for full [transaction isolation](documentation/TransactionIsolation.md) using Multi-Version Concurrency Control (MVCC)
- [ObjectLockingIndexedCollection](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/ObjectLockingIndexedCollection.html) - lock-free concurrent reads, and some locking of writes for object-level transaction isolation

For more details see [TransactionIsolation](documentation/TransactionIsolation.md).

---

## Maintenance Notice

This fork was created to:

✅ **Update to Java 21** - Full support for modern Java features and performance improvements  
✅ **Modernize dependencies** - Updated all dependencies to latest versions with security fixes  
✅ **Docker-based testing** - OS-independent integration tests using Testcontainers  
✅ **Fix compatibility issues** - Resolved EqualsVerifier, SQLite native library, and lambda type erasure issues  
✅ **Continue development** - Active maintenance and new feature development

**Original Project**: [https://github.com/npgall/cqengine](https://github.com/npgall/cqengine)  
**This Fork**: [https://github.com/MSaifAsif/cqengine-next](https://github.com/MSaifAsif/cqengine-next)

---

## Quick Start

### Maven Dependency

```xml
<dependency>
    <groupId>io.github.msaifasif</groupId>
    <artifactId>cqengine</artifactId>
    <version>1.0.0</version>
</dependency>
```

### Basic Example

```java
import com.googlecode.cqengine.*;
import com.googlecode.cqengine.query.Query;
import static com.googlecode.cqengine.query.QueryFactory.*;

// Create an indexed collection
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<>();

// Add indexes for fast queries
cars.addIndex(NavigableIndex.onAttribute(Car.CAR_ID));
cars.addIndex(HashIndex.onAttribute(Car.MANUFACTURER));
cars.addIndex(NavigableIndex.onAttribute(Car.PRICE));

// Add data
cars.add(new Car(1, "Ford", "Focus", Car.Color.BLUE, 5, 9000.50));
cars.add(new Car(2, "Honda", "Civic", Car.Color.RED, 5, 5000.00));

// Query with SQL-like syntax
Query<Car> query = and(
    equal(Car.MANUFACTURER, "Ford"),
    lessThan(Car.PRICE, 10000.0)
);

ResultSet<Car> results = cars.retrieve(query);
for (Car car : results) {
    System.out.println(car);
}
```

---

## Migration from Original CQEngine

**100% API Compatible** - Only the Maven coordinates have changed!

### Old Configuration (Original CQEngine)

```xml
<dependency>
    <groupId>com.googlecode.cqengine</groupId>
    <artifactId>cqengine</artifactId>
    <version>3.6.0</version>
</dependency>
```

### New Configuration (CQEngine Next)

```xml
<dependency>
    <groupId>io.github.msaifasif</groupId>
    <artifactId>cqengine</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

### What Changes?

- ✅ **Maven GroupId**: `com.googlecode.cqengine` → `io.github.msaifasif`
- ✅ **Minimum Java version**: Java 8 → Java 21
- ❌ **No code changes required** - All package names (`com.googlecode.cqengine.*`) remain the same

---

## What's New in This Fork

### Java 21 Support

- Compiled and tested with JDK 21
- Modern Java features available
- Improved performance with latest JVM optimizations

### Updated Dependencies (2025-12-17)

| Dependency | Old Version | New Version | Notes |
|------------|-------------|-------------|-------|
| ByteBuddy | 1.9.10 | 1.14.11 | Java 21 bytecode support |
| EqualsVerifier | 3.15.4 | 3.16.1 | Java 21 compatibility |
| SQLite JDBC | 3.42.0.0 | 3.45.0.0 | CVE-2023-32697 fixed, ARM64 support |
| Kryo | 4.0.0 | 5.0.0-RC1 | Java 21 serialization |
| Javassist | 3.29.0-GA | 3.30.2-GA | Latest stable |

### Docker-Based Integration Tests (2025-12-18)

**30 integration tests** using Testcontainers for OS-independent testing:

- ✅ **SQLite persistence** (7 tests) - Database creation, indexing, concurrency, large datasets
- ✅ **Disk persistence** (8 tests) - File I/O, compaction, WAL mode, concurrent access
- ✅ **Off-heap memory** (8 tests) - Native memory management, container limits, expansion
- ✅ **Composite persistence** (7 tests) - Multi-tier architecture (on-heap + off-heap + disk)

**Test Results**: 29 passing, 1 skipped (known limitation documented)

### Fixed Issues

1. **EqualsVerifier** - Java 21 bytecode compatibility resolved
2. **SQLite Native Library** - ARM64 Mac support (no more `aarch64` errors)
3. **Lambda Type Erasure** - Generic type resolution in Java 21
4. **ReflectiveAttribute** - Equality verification with field comparison

### Build System

- **maven-assembly-plugin 3.6.0** - Replaced shade plugin (infinite loop fix)
- Creates fat jar with all dependencies (16 MB) in ~5 seconds
- Separate source and javadoc jars for Maven Central

---

## Docker-Based Testing

### Prerequisites

- **Docker Desktop** or **Docker Engine** must be installed and running
- Verify: `docker ps` should work without errors

### Running Docker Tests

```bash
# Run all Docker integration tests
mvn test -Dtest=Docker*IntegrationTest

# Run specific test class
mvn test -Dtest=DockerSQLiteIntegrationTest
mvn test -Dtest=DockerDiskPersistenceIntegrationTest
mvn test -Dtest=DockerOffHeapPersistenceIntegrationTest
mvn test -Dtest=DockerCompositePersistenceIntegrationTest
```

---

## Building from Source

### Requirements

- **JDK 21** or higher
- **Maven 3.8+**
- **Docker** (optional, for integration tests)

### Build Commands

```bash
# Clean build (skip tests)
mvn clean package -DskipTests

# Build with unit tests only
mvn clean package

# Build with all tests including Docker
mvn clean package -P docker-tests

# Create fat jar with dependencies
mvn clean package -DskipTests
# Output: target/cqengine-1.0.0-SNAPSHOT-jar-with-dependencies.jar (16 MB)

# Install to local Maven repository
mvn clean install
```

### Build Artifacts

- `cqengine-1.0.0-SNAPSHOT.jar` - Main library (1 MB)
- `cqengine-1.0.0-SNAPSHOT-jar-with-dependencies.jar` - Fat jar (16 MB)
- `cqengine-1.0.0-SNAPSHOT-sources.jar` - Source code
- `cqengine-1.0.0-SNAPSHOT-javadoc.jar` - API documentation

---

## Complete Example

In CQEngine, applications mostly interact with [IndexedCollection](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/IndexedCollection.html), which is an implementation of [java.util.Set](http://docs.oracle.com/javase/6/docs/api/java/util/Set.html), and it provides two additional methods:

* [addIndex(SomeIndex)](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/engine/QueryEngine.html#addIndex(com.googlecode.cqengine.index.Index)) allows indexes to be added to the collection
* [retrieve(Query)](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/engine/QueryEngine.html#retrieve(com.googlecode.cqengine.query.Query)) accepts a [Query](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/query/Query.html) and returns a [ResultSet](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html) providing objects matching that query. `ResultSet` implements [java.lang.Iterable](http://docs.oracle.com/javase/6/docs/api/java/lang/Iterable.html), so accessing results is achieved by iterating the result set, or accessing it as a Java 8+ Stream

Here is a **complete example** of how to build a collection, add indexes and perform queries. It does not discuss _attributes_, which are discussed below.

**STEP 1: Create a new indexed collection**
```java
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>();
```

**STEP 2: Add some indexes to the collection**
```java
cars.addIndex(NavigableIndex.onAttribute(Car.CAR_ID));
cars.addIndex(ReversedRadixTreeIndex.onAttribute(Car.NAME));
cars.addIndex(SuffixTreeIndex.onAttribute(Car.DESCRIPTION));
cars.addIndex(HashIndex.onAttribute(Car.FEATURES));
```

**STEP 3: Add some objects to the collection**
```java
cars.add(new Car(1, "ford focus", "great condition, low mileage", Arrays.asList("spare tyre", "sunroof")));
cars.add(new Car(2, "ford taurus", "dirty and unreliable, flat tyre", Arrays.asList("spare tyre", "radio")));
cars.add(new Car(3, "honda civic", "has a flat tyre and high mileage", Arrays.asList("radio")));
```

**STEP 4: Run some queries**

Note: add import statement to your class: _`import static com.googlecode.cqengine.query.QueryFactory.*`_

* *Example 1: Find cars whose name ends with 'vic' or whose id is less than 2*

  Query:
  ```java
    Query<Car> query1 = or(endsWith(Car.NAME, "vic"), lessThan(Car.CAR_ID, 2));
    cars.retrieve(query1).forEach(System.out::println);
  ```
  Prints:
  ```
    Car{carId=3, name='honda civic', description='has a flat tyre and high mileage', features=[radio]}
    Car{carId=1, name='ford focus', description='great condition, low mileage', features=[spare tyre, sunroof]}
  ```
  
* *Example 2: Find cars whose flat tyre can be replaced*

  Query:
  ```java
    Query<Car> query2 = and(contains(Car.DESCRIPTION, "flat tyre"), equal(Car.FEATURES, "spare tyre"));
    cars.retrieve(query2).forEach(System.out::println);
  ```
  Prints:
  ```
    Car{carId=2, name='ford taurus', description='dirty and unreliable, flat tyre', features=[spare tyre, radio]}
  ```
  
* *Example 3: Find cars which have a sunroof or a radio but are not dirty*

  Query:
  ```java
    Query<Car> query3 = and(in(Car.FEATURES, "sunroof", "radio"), not(contains(Car.DESCRIPTION, "dirty")));
    cars.retrieve(query3).forEach(System.out::println);
  ```
   Prints:
  ```
    Car{carId=1, name='ford focus', description='great condition, low mileage', features=[spare tyre, sunroof]}
    Car{carId=3, name='honda civic', description='has a flat tyre and high mileage', features=[radio]}
  ```

Complete source code for these examples can be found [here](https://github.com/MSaifAsif/cqengine-next/blob/master/code/src/test/java/com/googlecode/cqengine/examples/introduction/).

---

## String-Based Queries: SQL and CQN Dialects

As an alternative to programmatic queries, CQEngine also has support for running string-based queries on the collection, in either SQL or CQN (CQEngine Native) format.

Example of running an SQL query on a collection (full source [here](https://github.com/MSaifAsif/cqengine-next/blob/master/code/src/test/java/com/googlecode/cqengine/examples/parser/SQLQueryDemo.java)):
```java
public static void main(String[] args) {
    SQLParser<Car> parser = SQLParser.forPojoWithAttributes(Car.class, createAttributes(Car.class));
    IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>();
    cars.addAll(CarFactory.createCollectionOfCars(10));

    ResultSet<Car> results = parser.retrieve(cars, "SELECT * FROM cars WHERE (" +
                                    "(manufacturer = 'Ford' OR manufacturer = 'Honda') " +
                                    "AND price <= 5000.0 " +
                                    "AND color NOT IN ('GREEN', 'WHITE')) " +
                                    "ORDER BY manufacturer DESC, price ASC");
                                    
    results.forEach(System.out::println); // Prints: Honda Accord, Ford Fusion, Ford Focus
}
```

Example of running a CQN query on a collection (full source [here](https://github.com/MSaifAsif/cqengine-next/blob/master/code/src/test/java/com/googlecode/cqengine/examples/parser/CQNQueryDemo.java)):
```java
public static void main(String[] args) {
    CQNParser<Car> parser = CQNParser.forPojoWithAttributes(Car.class, createAttributes(Car.class));
    IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>();
    cars.addAll(CarFactory.createCollectionOfCars(10));

    ResultSet<Car> results = parser.retrieve(cars,
                                    "and(" +
                                        "or(equal(\"manufacturer\", \"Ford\"), equal(\"manufacturer\", \"Honda\")), " +
                                        "lessThanOrEqualTo(\"price\", 5000.0), " +
                                        "not(in(\"color\", GREEN, WHITE))" +
                                    ")");
                                    
    results.forEach(System.out::println); // Prints: Ford Focus, Ford Fusion, Honda Accord
}
```

---

## Feature Matrix for Included Indexes

**Legend for the feature matrix**

| **Abbreviation** | **Meaning** | **Example** |
|:----------------:|:------------|:------------|
| **EQ** | _Equality_ | `equal(Car.DOORS, 4)` |
| **IN** | _Equality, multiple values_ | `in(Car.DOORS, 3, 4, 5)` |
| **LT** | _Less Than (numerical range / `Comparable`)_ | `lessThan(Car.PRICE, 5000.0)` |
| **GT** | _Greater Than (numerical range / `Comparable`)_ | `greaterThan(Car.PRICE, 2000.0)` |
| **BT** | _Between (numerical range / `Comparable`)_ | `between(Car.PRICE, 2000.0, 5000.0)` |
| **SW** | _String Starts With_ | `startsWith(Car.NAME, "For")` |
| **EW** | _String Ends With_ | `endsWith(Car.NAME, "ord")` |
| **SC** | _String Contains_ | `contains(Car.NAME, "or")` |
| **CI** | _String Is Contained In_ | `isContainedIn(Car.NAME, "I am shopping for a Ford Focus car")` |
| **RX** | _String Matches Regular Expression_ | `matchesRegex(Car.MODEL, "Ford.*")` |
| **HS** | _Has (aka `IS NOT NULL`)_ | `has(Car.DESCRIPTION)` / `not(has(Car.DESCRIPTION))` |
| **SQ** | _Standing Query_ | _Can the index accelerate a query (as opposed to an attribute) to provide constant time complexity for any simple query, complex query, or fragment_ |
| **QZ** | _Quantization_ | _Does the index accept a quantizer to control granularity_ |
| **LP** | _LongestPrefix_ | `longestPrefix(Car.NAME, "Ford")` |

Note: CQEngine also supports complex queries via **`and`**, **`or`**, **`not`**, and combinations thereof, across all indexes.

**Index Feature Matrix**

| <sub>**Index Type**</sub> | <sub>**EQ**</sub> | <sub>**IN**</sub> | <sub>**LT**</sub> | <sub>**GT**</sub> | <sub>**BT**</sub> | <sub>**SW**</sub> | <sub>**EW**</sub> | <sub>**SC**</sub> | <sub>**CI**</sub> | <sub>**HS**</sub> | <sub>**RX**</sub> | <sub>**SQ**</sub> | <sub>**QZ**</sub> | <sub>**LP**</sub> |
|:---------------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|
| [<sub>Hash</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/hash/HashIndex.html) | ✓ | ✓ | | | | | | | | ✓ | | | | |
| [<sub>Unique</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/unique/UniqueIndex.html) | ✓ | ✓ | | | | | | | | | | | | |
| [<sub>Compound</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/compound/CompoundIndex.html) | ✓ | ✓ | | | | | | | | ✓ | | | | |
| [<sub>Navigable</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/navigable/NavigableIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | | | | | ✓ | | | | |
| [<sub>PartialNavigable</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/navigable/PartialNavigableIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | | | | | ✓ | | | | |
| [<sub>RadixTree</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/radix/RadixTreeIndex.html) | ✓ | ✓ | | | | ✓ | | | | | | | | |
| [<sub>ReversedRadixTree</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/radixreversed/ReversedRadixTreeIndex.html) | ✓ | ✓ | | | | | ✓ | | | | | | | |
| [<sub>InvertedRadixTree</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/radixinverted/InvertedRadixTreeIndex.html) | ✓ | ✓ | | | | | | ✓ | | | | | | ✓ |
| [<sub>SuffixTree</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/suffix/SuffixTreeIndex.html) | ✓ | ✓ | | | | | ✓ | ✓ | | | | | | |
| [<sub>StandingQuery</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/standingquery/StandingQueryIndex.html) | | | | | | | | | | | | ✓ | | |
| [<sub>Fallback</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/fallback/FallbackIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | | | ✓ |
| [<sub>OffHeap</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/offheap/OffHeapIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | | | | ✓<sup>[1]</sup> | | | | |
| [<sub>PartialOffHeap</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/offheap/PartialOffHeapIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | | | | ✓ | | | | |
| [<sub>Disk</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/disk/DiskIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | | | | ✓<sup>[1]</sup> | | | | |
| [<sub>PartialDisk</sub>](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/index/disk/PartialDiskIndex.html) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | | | | ✓ | | | | |

<sup>[1]</sup> See: [forStandingQuery()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/query/QueryFactory.html#forStandingQuery-com.googlecode.cqengine.query.Query-)

The [Benchmark](documentation/Benchmark.md) page contains examples of how to add these indexes to a collection, and measures their impact on latency.

---

## Attributes

### Read Fields

CQEngine needs to access fields inside objects, so that it can build indexes on fields, and retrieve the value of a certain field from any given object.

CQEngine does not use reflection to do this; instead it uses **attributes**, which is a more powerful concept. An attribute is an accessor object which can read the value of a certain field in a POJO.

Here's how to define an attribute for a Car object (a POJO), which reads the `Car.carId` field:
```java
public static final Attribute<Car, Integer> CAR_ID = new SimpleAttribute<Car, Integer>("carId") {
    public Integer getValue(Car car, QueryOptions queryOptions) { return car.carId; }
};
```
...or alternatively, from a lambda expression or method reference:
```java
public static final Attribute<Car, Integer> Car_ID = attribute("carId", Car::getCarId);
```
(For some caveats on using lambdas, please read [LambdaAttributes](documentation/LambdaAttributes.md))

Usually attributes are defined as anonymous `static` `final` objects like this. Supplying the `"carId"` string parameter to the constructor is actually optional, but it is recommended as it will appear in query `toString`s.

Since this attribute reads a field from a `Car` object, the usual place to put the attribute is inside the `Car` class - and this makes queries more readable. However it could really be defined in any class, such as in a `CarAttributes` class or similar. The example above is for a **[SimpleAttribute](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/attribute/SimpleAttribute.html)**, which is designed for fields containing only one value.

CQEngine also supports **[MultiValueAttribute](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/attribute/MultiValueAttribute.html)** which can read the values of fields which themselves are collections. And so it supports building indexes on objects based on things like keywords associated with those objects.

Here's how to define a `MultiValueAttribute` for a `Car` object which reads the values from `Car.features` where that field is a `List<String>`:
```java
public static final Attribute<Car, String> FEATURES = new MultiValueAttribute<Car, String>("features") {
    public Iterable<String> getValues(Car car, QueryOptions queryOptions) { return car.features; }
};
```
...or alternatively, from a lambda expression or method reference:
```java
public static final Attribute<Car, String> FEATURES = attribute(String.class, "features", Car::getFeatures);
```

#### Null values
Note **if your data contains `null` values**, you should use **[SimpleNullableAttribute](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/attribute/SimpleNullableAttribute.html)** or **[MultiValueNullableAttribute](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/attribute/MultiValueNullableAttribute.html)** instead.

In particular, note that `SimpleAttribute` and `MultiValueAttribute` do not perform any null checking on your data, and so if your data inadvertently contains null values, you may get obscure `NullPointerException`s. This is because null checking does not come for free. Attributes are accessed heavily, and the non-nullable versions of these attributes are designed to minimize latency by skipping explicit null checks. They defer to the JVM to do the null checking implicitly. 

As a rule of thumb, if you get a `NullPointerException`, it's probably because you used the wrong type of attribute. The problem will usually go away if you switch your code to use a nullable attribute instead. If you don't know if your data may contain null values, just use the nullable attributes. They contain the logic to check for and handle null values automatically.

The nullable attributes also allow CQEngine to work with [object inheritance](https://github.com/MSaifAsif/cqengine-next/tree/master/code/src/test/java/com/googlecode/cqengine/examples/inheritance), where some objects in the collection have certain optional fields (e.g. in subclasses) while others might not.

#### Creating queries dynamically

Dynamic queries can be composed at runtime by instantiating and combining Query objects directly; see [this package](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/query/simple/package-summary.html) and [this package](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/query/logical/package-summary.html). For advanced cases, it is also possible to define attributes at runtime, using [ReflectiveAttribute](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/attribute/ReflectiveAttribute.html) or [AttributeBytecodeGenerator](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/codegen/AttributeBytecodeGenerator.html).

#### Generate attributes automatically

CQEngine also provides several ways to generate attributes automatically.

Note these are an alternative to using [ReflectiveAttribute](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/attribute/ReflectiveAttribute.html), which was discussed above. Whereas `ReflectiveAttribute` is a special type of attribute which reads values at runtime using reflection, `AttributeSourceGenerator` and `AttributeBytecodeGenerator` generate code for attributes which is compiled and so does not use reflection at runtime, which can be more efficient.

* [AttributeSourceGenerator](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/codegen/AttributeSourceGenerator.html) can automatically generate the source code for the simple and multi-value attributes discussed above.
* [AttributeBytecodeGenerator](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/codegen/AttributeBytecodeGenerator.html) can automatically generate the class bytecode for the simple and multi-value attributes discussed above, and load them into the application at runtime as if they had been compiled from source code.

See [AutoGenerateAttributes](documentation/AutoGenerateAttributes.md) for more details.

### Attributes as Functions

It can be noted that attributes are only required to return a value given an object. Although most will do so, there is no requirement that an attribute must provide a value by reading a field in the object. As such attributes can be _virtual_, implemented as _functions_.

**Calculated Attributes**

An attribute can **_calculate_** an appropriate value for an object, based on a function applied to data contained in other fields or from external data sources.

Here's how to define a calculated (or virtual) attribute by applying a function over the Car's other fields:
```java
public static final Attribute<Car, Boolean> IS_DIRTY = new SimpleAttribute<Car, Boolean>("is_dirty") {
    public Boolean getValue(Car car, QueryOptions queryOptions) { return car.description.contains("dirty"); }
};
```
...or, the same thing using a lambda:
```java
public static final Attribute<Car, Boolean> IS_DIRTY = attribute("is_dirty", car -> car.description.contains("dirty"));
```

A `HashIndex` could be built on the virtual attribute above, enabling fast retrievals of cars which are either dirty or not dirty, without needing to scan the collection.

**Associations with other `IndexedCollections` or External Data Sources**

Here is an example for a virtual attribute which **associates** with each `Car` a list of locations which can service it, from an external data source:
```java
public static final Attribute<Car, String> SERVICE_LOCATIONS = new MultiValueAttribute<Car, String>() {
    public List<String> getValues(Car car, QueryOptions queryOptions) {
        return CarServiceManager.getServiceLocationsForCar(car);
    }
};
```
The attribute above would allow the `IndexedCollection` of cars to be searched for cars which have _servicing options in a particular location_.

The locations which service a car, could alternatively be retrieved from another `IndexedCollection`, of `Garage`s, for example. **Care should be taken if building indexes on virtual attributes** however, if referenced data might change leaving obsolete information in indexes. A **strategy to accommodate this** is: if no index exists for a virtual attribute referenced in a query, and other attributes are also referenced in the query for which indexes exist, CQEngine will automatically reduce the candidate set of objects to the minimum using other indexes before querying the virtual attribute. In turn if virtual attributes perform retrievals from _other_ `IndexedCollection`s, then those collections could be indexed appropriately without a risk of stale data.

---

## Joins

The examples above define attributes on a primary `IndexedCollection` which read data from secondary collections or external data sources.

It is also possible to perform SQL EXISTS-type queries and JOINs between `IndexedCollection`s on the query side (as opposed to on the attribute side). See [Joins](documentation/Joins.md) for examples.

---

## Persistence: On-Heap, Off-Heap, Disk

CQEngine's `IndexedCollection`s can be configured to store objects added to them on-heap (the default), or off-heap, or on disk.

**On-heap**

Store the collection on the Java heap:
```java
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>();
```

**Off-heap**

Store the collection in native memory, within the JVM process but outside the Java heap:
```java
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>(OffHeapPersistence.onPrimaryKey(Car.CAR_ID));
```

Note that the off-heap persistence will automatically create an index on the specified primary key attribute, so there is no need to add an index on that attribute later.

**Disk**

Store the collection in a temp file on disk (then see `DiskPersistence.getFile()`):
```java
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>(DiskPersistence.onPrimaryKey(Car.CAR_ID));
```
Or, store the collection in a particular file on disk:
```java
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>(DiskPersistence.onPrimaryKeyInFile(Car.CAR_ID, new File("cars.dat")));
```

Note that the disk persistence will automatically create an index on the specified primary key attribute, so there is no need to add an index on that attribute later.

**Wrapping**

Wrap any Java collection, in a CQEngine IndexedCollection without any copying of objects.
* This can be a convenient way to run queries or build indexes on existing collections.
* However some caveats relating to concurrency support and the performance of the underlying collection apply, see [WrappingPersistence](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/persistence/wrapping/WrappingPersistence.html) for details.

```java
Collection<Car> collection = // obtain any Java collection

IndexedCollection<Car> indexedCollection = new ConcurrentIndexedCollection<Car>(
        WrappingPersistence.aroundCollection(collection)
);
```

**Composite**

`CompositePersistence` configures a combination of persistence types for use within the same collection.
The collection itself will be persisted in the first persistence provided (the _primary persistence_), and the additional persistences provided will be used by off-heap or disk indexes added to the collection subsequently.

Store the collection on-heap, and also configure DiskPersistence for use by DiskIndexes added to the collection subsequently:
```java
IndexedCollection<Car> cars = new ConcurrentIndexedCollection<Car>(CompositePersistence.of(
    OnHeapPersistence.onPrimaryKey(Car.CAR_ID),
    DiskPersistence.onPrimaryKeyInFile(Car.CAR_ID, new File("cars.dat"))
));
```

### Index Persistence

Indexes can similarly be stored on-heap, off-heap, or on disk. Each index requires a certain type of persistence. It is necessary to configure the collection in advance with an appropriate combination of persistences for use by whichever indexes are added.

It is possible to store the collection on-heap, but to store some indexes off-heap. Similarly it is possible to have a variety of index types on the same collection, each using a different type of persistence. On-heap persistence is by far the fastest, followed by off-heap persistence, and then by disk persistence.

If both the collection and all of its indexes are stored off-heap or on disk, then it is possible to have extremely large collections which don't use any heap memory or RAM at all.

CQEngine has been tested using off-heap persistence with collections of 10 million objects, and using disk persistence with collections of 100 million objects.

**On-heap**

Add an on-heap index on "manufacturer":
```java
cars.addIndex(NavigableIndex.onAttribute(Car.MANUFACTURER));
```

**Off-heap**

Add an off-heap index on "manufacturer":
```java
cars.addIndex(OffHeapIndex.onAttribute(Car.MANUFACTURER));
```

**Disk**

Add a disk index on "manufacturer":
```java
cars.addIndex(DiskIndex.onAttribute(Car.MANUFACTURER));
```

### Querying with Persistence

When either the `IndexedCollection`, or one or more indexes are located off-heap or on disk, take care to close the ResultSet when finished reading. You can use a _try-with-resources_ block to achieve this:
```java
try (ResultSet<Car> results = cars.retrieve(equal(Car.MANUFACTURER, "Ford"))) {
    results.forEach(System.out::println);
}
```

---

## Result Sets

CQEngine [ResultSet](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html)s provide the following methods:

* [iterator()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#iterator()) - Allows the `ResultSet` to be iterated, returning the next object matching the query in each iteration as determined via _lazy evaluation_
  * Result sets support **concurrent iteration** while the collection is being modified; the set of objects returned simply may or may not reflect changes made during iteration (depending on whether changes are made to areas of the collection or indexes already iterated or not)

* [uniqueResult()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#uniqueResult()) - Useful if the query is expected to only match one object, this method returns the first object which would be returned by the iterator, and it throws an exception if zero or more than one object is found

* [size()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#size()) - Returns the number of objects which _would be returned by the `ResultSet` if it was iterated_; CQEngine can often **accelerate** this calculation of size, based on the sizes of individual sets in indexes; see JavaDoc for details

* [contains()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#contains(O)) - Tests if a _given object_ would be contained in results matching a query; this is also an **accelerated** operation; when suitable indexes are available, CQEngine can avoid iterating results to test for containment; see JavaDoc for details

* [getRetrievalCost()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#getRetrievalCost()) - This is a metric used internally by CQEngine to allow it to _choose between multiple indexes_ which support the query. This could occasionally be used by applications to ascertain if suitable indexes are available for any particular query, this will be `Integer.MAX_VALUE` for queries for which no suitable indexes are available

* [getMergeCost()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#getMergeCost()) - This is a metric used internally by CQEngine to allow it to _re-order_ elements of the query to minimize time complexity; for example CQEngine will order intersections such that the smallest set drives the _merge_; this metric is _roughly_ based on the theoretical cost to iterate underlying result sets
  * For query fragments requiring _set union_ (`or`-based queries), this will be the _sum_ of merge costs from underlying result sets
  * For query fragments requiring _set intersection_ (`and`-based queries), this will be the _Math.min()_ of merge costs from underlying result sets, because intersections will be re-ordered to perform lowest-merge-cost intersections first
  * For query fragments requiring _set difference_ (`not`-based queries), this will be the merge cost from the first underlying result set

* [stream()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#stream()) - Returns a Java 8+ `Stream` allowing CQEngine results to be grouped, aggregated, and transformed in flexible ways using lambda expressions.

* [close()](http://htmlpreview.github.io/?http://raw.githubusercontent.com/MSaifAsif/cqengine-next/master/documentation/javadoc/apidocs/com/googlecode/cqengine/resultset/ResultSet.html#close()) - Releases any resources or closes the transaction which was opened for the query. Whether or not it is necessary to close the ResultSet depends on which implementation of IndexedCollection is in use and the types of indexes added to it.

---

## Deduplicating Results

It is possible that a query would result in the same object being returned more than once.

For example if an object matches several attribute values specified in an `or`-type query, then the object will be returned multiple times, one time for each attribute matched. Intersections (`and`-type queries) and negations (`not`-type queries) do not produce duplicates.

By default, CQEngine does _not_ perform de-duplication of results; however it can be _instructed_ to do so, using various strategies such as Logical Elimination and Materialize. Read more: [DeduplicationStrategies](documentation/DeduplicationStrategies.md)

---

## Ordering Results

By default, CQEngine does not order results; it simply returns objects in the order it finds them in the collection or in indexes.

CQEngine can be instructed to order results via query options as follows.

**Order by price descending**
```java
ResultSet<Car> results = cars.retrieve(query, queryOptions(orderBy(descending(Car.PRICE))));
```

**Order by price descending, then number of doors ascending**
```java
ResultSet<Car> results = cars.retrieve(query, queryOptions(orderBy(descending(Car.PRICE), ascending(Car.DOORS))));
```

Note that ordering results as above uses the default _materialize_ ordering strategy. This is relatively expensive, dependent on the number of objects matching the query, and can cause latency in accessing the first object. It requires all results to be materialized into a sorted set up-front _before iteration can begin_.

Read more: [OrderingStrategies](documentation/OrderingStrategies.md)

---

## Using CQEngine with Hibernate / JPA / ORM Frameworks

CQEngine has seamless integration with JPA/ORM frameworks such as Hibernate or EclipseLink.

Simply put, CQEngine can build indexes on, and query, any type of Java collection or arbitrary data source. ORM frameworks return entity objects loaded from database tables in Java collections, therefore CQEngine can act as a very fast in-memory query engine on top of such data.

---

## Using CQEngine in Scala, Kotlin, or Other JVM Languages

CQEngine should generally be compatible with other JVM languages besides Java too, however it can be necessary to apply a few tricks to make it work. See [OtherJVMLanguages.md](documentation/OtherJVMLanguages.md) for some tips.

---

## Related Projects

* CQEngine is somewhat similar to [Microsoft LINQ](http://en.wikipedia.org/wiki/Language_Integrated_Query), but a difference is LINQ queries on collections are evaluated via iteration/filtering whereas CQEngine uses set theory, thus CQEngine would outperform LINQ

* [Concurrent Trees](http://github.com/npgall/concurrent-trees/) provides Concurrent Radix Trees and Concurrent Suffix Trees, used by some indexes in CQEngine

---

## Project History

**Original Project**: CQEngine was created by Niall Gallagher and was initially hosted on Google Code (versions ≤2.1.0, prior to September 2015), then migrated to GitHub. The original repository can be found at [https://github.com/npgall/cqengine](https://github.com/npgall/cqengine).

**This Fork**: Created to modernize the codebase for Java 21, update all dependencies, and continue active development. See the [archive/](archive/) directory for information about historical releases.

**Project Status**:
- CQEngine 3.6.0 was the last release from the original repository (January 2021)
- This fork version 1.0.0 includes all features from 3.6.0 plus Java 21 support and modernized dependencies
- A [ReleaseNotes](documentation/ReleaseNotes.md) page documents changes between releases

---

## Documentation

### This Fork

- **Project Repository**: [https://github.com/MSaifAsif/cqengine-next](https://github.com/MSaifAsif/cqengine-next)
- **Changelog**: [CHANGELOG.md](CHANGELOG.md)
- **Notice & License**: [NOTICE](NOTICE) (Apache 2.0)
- **JavaDocs**: Available in `documentation/javadoc/apidocs/` directory

### Additional Documentation

- [Benchmark Results](documentation/Benchmark.md)
- [Transaction Isolation](documentation/TransactionIsolation.md)
- [Deduplication Strategies](documentation/DeduplicationStrategies.md)
- [Merge Strategies](documentation/MergeStrategies.md)
- [Ordering Strategies](documentation/OrderingStrategies.md)
- [Index Quantization](documentation/IndexQuantization.md)
- [Auto-Generate Attributes](documentation/AutoGenerateAttributes.md)
- [Lambda Attributes](documentation/LambdaAttributes.md)
- [Joins](documentation/Joins.md)
- [Other JVM Languages](documentation/OtherJVMLanguages.md)
- [Frequently Asked Questions](documentation/FrequentlyAskedQuestions.md)
- [Release Notes](documentation/ReleaseNotes.md)

### Original Project Reference

For historical reference, the original project documentation is available at [https://github.com/npgall/cqengine](https://github.com/npgall/cqengine). Note that this fork maintains 100% API compatibility, so most documentation from the original project applies here as well.

---

## Contributing

Contributions are welcome! This fork aims to maintain backward compatibility while adding modern features.

### Guidelines

1. **Preserve backward compatibility** - All existing APIs must continue to work
2. **Add @author attribution** - Add your name to modified files
3. **Follow Keep a Changelog** - Update `CHANGELOG.md` with your changes
4. **Run tests before committing**: `mvn clean test`
5. **Include Docker tests** if modifying persistence/index code
6. **Update documentation** as needed

### Development Setup

```bash
# Clone the repository
git clone https://github.com/MSaifAsif/cqengine-next.git
cd cqengine-next

# Build and test
mvn clean install

# Run Docker tests
mvn test -Dtest=Docker*IntegrationTest
```

---

## License

This project is licensed under the **Apache License 2.0**.

This fork maintains the same license as the original project and includes proper attribution to the original author in the [NOTICE](NOTICE) file.

For the full license text, see: [https://www.apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0)

---

## Acknowledgments

### Original Author

**Niall Gallagher** - Creator of CQEngine  
Original Project: [https://github.com/npgall/cqengine](https://github.com/npgall/cqengine)

This maintained fork builds upon the excellent foundation created by Niall Gallagher and the original contributors. All credit for the core architecture and design goes to them.

### This Fork

**Saif Asif** - Maintainer of CQEngine Next  
Repository: [https://github.com/MSaifAsif/cqengine-next](https://github.com/MSaifAsif/cqengine-next)

Contributions:
- Java 21 migration and modernization
- Docker-based integration testing with Testcontainers
- Dependency updates and security fixes
- Build system improvements
- Continued maintenance and support

---

## Support

- **Issues**: [https://github.com/MSaifAsif/cqengine-next/issues](https://github.com/MSaifAsif/cqengine-next/issues)
- **Discussions**: [https://github.com/MSaifAsif/cqengine-next/discussions](https://github.com/MSaifAsif/cqengine-next/discussions)

For questions about the original CQEngine project, please refer to the [original repository](https://github.com/npgall/cqengine).

---

**Made with ❤️ to keep CQEngine alive and thriving in the modern Java ecosystem**

