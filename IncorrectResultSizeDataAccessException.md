# IncorrectResultSizeDataAccessException – Spring Data JPA

## Error

```text
org.springframework.dao.IncorrectResultSizeDataAccessException:
Query did not return a unique result:
2 results were returned
```

---

# What does this error mean?

This error occurs when the application expects **only one record**, but the database returns **multiple records**.

Simple understanding:

```text
Expected → 1 row

Actual → 2 or more rows

Result → Exception
```

---

# Why does this error happen?

Most commonly, the repository method is written to return a **single object**, but duplicate records exist in the database.

Example:

Repository:

```java
Department findByDepartmentCode(
String departmentCode
);
```

Database:

| id | department_code |
| -- | --------------- |
| 1  | IT001           |
| 2  | IT001           |

Query result:

```text
2 rows returned
```

But repository expects:

```java
Department
```

(single object)

So Spring throws:

```text
IncorrectResultSizeDataAccessException
```

---

# Internal Flow (What happens internally?)

```text
Controller

↓

Service

↓

Repository

↓

SimpleJpaRepository

↓

Hibernate

↓

SQL Execution

↓

Database returns multiple rows

↓

Spring expected one object

↓

Exception thrown
```

---

# Root Cause

Repository expects:

```java
Department
```

Database returns:

```text
Department1
Department2
```

Spring cannot convert multiple rows into one object.

---

# Solution 1 — Return List (Recommended if duplicates are allowed)

Repository:

```java
List<Department>
findByDepartmentCode(
String departmentCode
);
```

Now:

```text
Multiple rows

↓

Stored inside List
```

---

# Solution 2 — Enforce Unique Values

Entity:

```java
@Column(unique = true)
private String departmentCode;
```

This prevents future duplicate inserts.

Example:

```text
IT001 ✅

IT001 ❌
```

Note:

```text
unique=true does NOT remove old duplicate data
```

---

# Solution 3 — Remove Existing Duplicate Records

Check duplicates:

```sql
select *
from department;
```

Delete unwanted rows.

---

# Solution 4 — Fetch Only First Record

Repository:

```java
Department
findFirstByDepartmentCode(
String departmentCode
);
```

Returns only the first matching row.

---

# Interview Ready Answer

"IncorrectResultSizeDataAccessException occurs when Spring Data JPA expects a single result but receives multiple records from the database. This usually happens because repository methods return a single entity while duplicate data exists. The fix is either enforcing uniqueness or returning List."

---

# Key Learning

```text
Entity
→ Expect ONE row

List<Entity>
→ Expect MULTIPLE rows
```
