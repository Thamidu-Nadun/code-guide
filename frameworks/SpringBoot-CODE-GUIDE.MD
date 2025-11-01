# 🌱 Spring Framework Code Style Guide

This document defines the **coding standards** for the **Spring Framework** source code.
It aligns closely with the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) and is intended for both Spring team members and contributors.

---

## 1. 📄 Source File Basics

### Encoding

- All source files **must use UTF-8** encoding.

### Indentation

- **Tabs**, not spaces.
- **Unix (LF)** line endings (not CRLF).
- **No trailing whitespace.**

To remove trailing whitespace:

```bash
find . -type f -name "*.java" -exec perl -p -i -e "s/[ \t]$//g" {} \;
```

---

## 2. 🧩 File Structure

Each source file must include, in order:

1. License header
2. Package statement
3. Import statements
4. One top-level class

Each section is separated by **exactly one blank line**.

---

### 2.1 License Header

Every file starts with:

```java
/*
 * Copyright 2002-present the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

---

### 2.2 Import Structure

Import groups should follow this order, separated by blank lines:

1. `import java.*`
2. `import javax.*`
3. `import jakarta.*`
4. All other imports
5. `import org.springframework.*`
6. Static imports (only when necessary)

**Rules:**

- No wildcard imports (`*`).
- **Static imports**:

  - Allowed in test code (e.g., `assertThat`).
  - Allowed for constants or DSL-style factory methods.
  - Forbidden in production code otherwise.

---

### 2.3 Class Organization

Order members as follows:

1. Static fields
2. Instance fields
3. Constructors
4. Private methods called by constructors
5. Static factory methods
6. Getters and setters
7. Interface method implementations
8. Protected/private helper methods
9. Other methods
10. `equals()`, `hashCode()`, and `toString()`

> ⚠️ Private helper methods should appear **immediately below** the methods that use them.

---

## 3. 🎨 Formatting

### 3.1 Braces

Use **K&R (Egyptian) style** for braces:

```java
if (condition) {
    doSomething();
}
else {
    handleElse();
}
```

**Rules:**

- Space before `{`
- Line break after `{` and before `}`
- Line break before `else`, `catch`, `finally`

---

### 3.2 Line Wrapping

- **Preferred max line length:** 90 characters
- **Acceptable range:** up to 105 characters
- **Absolute limit:** 120 characters

> Javadoc should wrap around **80 characters**.

**Wrap rules:**

- Break lines **after** separators (`,`, `+`, `&&`, etc.).
- Example:

  ```java
  if (isLongCondition(param1, param2) &&
          anotherCheck() && yetAnotherCheck()) {
      execute();
  }
  ```

---

### 3.3 Blank Lines

- **Two blank lines** before:

  - `static {}` block
  - Field declarations
  - Constructors
  - Inner classes

- **One blank line** after multiline method signatures.

Example:

```java
@Override
protected Object invoke(FooContext context,
        AnotherParameter name) {

    // ...
}
```

---

## 4. 🧱 Class & Naming Conventions

### 4.1 Class Declaration

- Keep `extends` and `implements` on the same line as the class name if possible.
- Order implemented interfaces logically (most important first).

---

### 4.2 Constants

- Use `CONSTANT_CASE` for true constants:

  ```java
  private static final int DEFAULT_PORT = -1;
  ```

- Not every `static final` field is a constant.

---

### 4.3 Variables

- Use descriptive names. Avoid single-letter names like `m` or `t`.

---

## 5. ⚙️ Programming Practices

### 5.1 File History

- Files should look like they were written by **one author**.
- Keep related elements **grouped together**.

---

### 5.2 Setter Methods

- Add new setters near **related fields or methods**.
- Maintain consistent ordering.

---

### 5.3 Ternary Operator

- Always wrap in parentheses.

  ```java
  return (value != null ? value : "default");
  ```

---

### 5.4 Null Checks

**For method arguments:**

```java
Assert.notNull(event, "Event must not be null");
```

**For state variables:**

```java
Assert.state(event != null, "Event must not be null");
```

---

### 5.5 Null Safety (with `@NonNullApi`)

Use:

```java
@NonNullApi
@NonNullFields
package org.springframework.mypackage;
```

For nullable values:

```java
@Nullable
private String description;
```

Use `@Contract` annotations to describe behavior for NullAway static analysis.

---

### 5.6 `@Override` and `@since`

- Always use `@Override` for implemented/overridden methods.
- Always document public and protected classes/methods with `@since`.

---

### 5.7 Utility Classes

Utility classes should:

- Be `abstract`
- Have a private constructor

Example:

```java
public abstract class MyUtils {

    private MyUtils() {
        // Prevent instantiation
    }
}
```

---

### 5.8 References

- Fields → use `this.field`
- Methods → **do not** prefix with `this.`

---

### 5.9 Local Variable Type Inference

- ❌ `var` is **not permitted** — always use explicit types.

---

## 6. 📝 Javadoc Standards

### 6.1 Method Javadoc Example

```java
/**
 * Parse the given {@link Element} and register resulting {@link BeanDefinition BeanDefinitions}.
 * <p>Implementations must return the primary {@link BeanDefinition}.
 * @param element the element to parse
 * @param context the parsing context
 * @return the primary BeanDefinition
 */
```

**Rules:**

- Use **imperative voice** (“Return” not “Returns”).
- Separate paragraphs with `<p>`.
- Align `@param` descriptions; no extra indentation.
- No blank line between description and tags.

---

### 6.2 Class Javadoc Example

```java
/*
 * Interface used by the {@link DefaultBeanDefinitionDocumentReader}
 * to handle custom top-level tags.
 *
 * <p>Implementations can define multiple {@link BeanDefinition}s.
 *
 * @author ...
 * @since 2.0
 * @see NamespaceHandler
 */
```

**Tag order:**

- Type-level: `@author`, `@since`, `@see`
- Method-level: `@param`, `@return`, `@throws`, `@since`, `@see`, `@deprecated`

---

## 7. 🧪 Test Code Standards

### 7.1 Framework

- Use **JUnit 5 (Jupiter)** for all tests.
- Only `spring-test` may use JUnit 4 or TestNG for compatibility tests.

### 7.2 Naming

- Test classes must end with `Tests`.

### 7.3 Assertions

- Use **AssertJ** for fluent assertions.

### 7.4 Mocking

- Use **Mockito** for mocks and spies.

---

## ✅ Summary Highlights

| Category          | Key Rule                         |
| ----------------- | -------------------------------- |
| Encoding          | UTF-8                            |
| Line endings      | Unix (LF)                        |
| Tabs or spaces    | Tabs                             |
| Line length       | Prefer ≤ 90, hard limit 120      |
| Imports           | No wildcards                     |
| `var` usage       | Not allowed                      |
| Null checks       | `Assert.notNull`, `Assert.state` |
| Test naming       | Ends with `Tests`                |
| Assertion library | AssertJ                          |
| Mocking framework | Mockito                          |

---
