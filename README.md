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
- [Maintenance Notice](#maintenance-notice)
- [Quick Start](#quick-start)
- [Migration from Original CQEngine](#migration-from-original-cqengine)
- [What's New in This Fork](#whats-new-in-this-fork)
- [Docker-Based Testing](#docker-based-testing)
- [Building from Source](#building-from-source)
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

### Troubleshooting Docker Tests

If tests fail with "Could not find valid Docker environment":

1. **Check Docker is running**: `docker ps`
2. **Check Docker socket**: `ls -la /var/run/docker.sock` (or `~/.docker/run/docker.sock` on macOS)
3. **Restart Docker Desktop** if needed
4. **Set DOCKER_HOST**: `export DOCKER_HOST=unix:///var/run/docker.sock`
5. **Check Testcontainers config**: `src/test/resources/.testcontainers.properties`

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

## Documentation

### Official Documentation (Original Project)

- **GitHub**: [https://github.com/npgall/cqengine](https://github.com/npgall/cqengine)
- **JavaDoc**: [https://www.javadoc.io/doc/com.googlecode.cqengine/cqengine](https://www.javadoc.io/doc/com.googlecode.cqengine/cqengine)
- **Wiki**: [https://github.com/npgall/cqengine/wiki](https://github.com/npgall/cqengine/wiki)

### This Fork

- **Changelog**: [CHANGELOG.md](CHANGELOG.md)
- **Notice & License**: [NOTICE](NOTICE) (Apache 2.0)
- **Project Repository**: [https://github.com/MSaifAsif/cqengine-next](https://github.com/MSaifAsif/cqengine-next)

### Examples

See the original project's examples and tutorials - they all work with this fork without modification:

- [Getting Started Guide](https://github.com/npgall/cqengine/wiki/Getting-Started)
- [Query Examples](https://github.com/npgall/cqengine/wiki/Query-Examples)
- [Index Types](https://github.com/npgall/cqengine/wiki/Index-Types)
- [Persistence Options](https://github.com/npgall/cqengine/wiki/Persistence)

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

