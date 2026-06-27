# Common Spring Boot Errors and Solutions

## 1. StackOverflowError

### Error

```java
java.lang.StackOverflowError
at com.securityImpl.entity.MyUser.toString(MyUser.java:10)
```

---

### Meaning

This error occurs when a method keeps calling itself again and again in an infinite loop until the JVM stack memory becomes full.

In simple words:

> The program goes into an endless cycle of method calls and runs out of memory for storing those calls.

---

### Why Does It Happen?

One common reason in Spring Boot is a bidirectional relationship between entities.

Example:

```java
// MyUser Entity
@ManyToMany
private List<Role> roles;
```

```java
// Role Entity
@ManyToMany(mappedBy = "roles")
private List<MyUser> users;
```

If `toString()` tries to print all fields:

```text
User
 ↓
Role
 ↓
User
 ↓
Role
 ↓
...
```

This loop continues forever and causes `StackOverflowError`.

---

### Other Possible Reasons

* Recursive method calls without a stopping condition.
* Using Lombok `@Data` on JPA entities.
* Printing complete entities using `System.out.println(entity)`.
* Circular references between objects.

---

### Solution

#### 1. Exclude relationship fields from `toString()`

```java
@ToString.Exclude
@ManyToMany
private List<Role> roles;
```

```java
@ToString.Exclude
@ManyToMany(mappedBy = "roles")
private List<MyUser> users;
```

---

#### 2. Avoid using `@Data` on JPA Entities

Instead of:

```java
@Data
```

Prefer:

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
```

---

#### 3. Create a Custom `toString()` Method

```java
@Override
public String toString() {
    return "MyUser{id=" + id + ", email='" + email + "'}";
}
```

---

### Quick Summary

| Problem            | Reason                                       | Solution                                                                       |
| ------------------ | -------------------------------------------- | ------------------------------------------------------------------------------ |
| StackOverflowError | Infinite method calls or circular references | Exclude relations from `toString()`, avoid `@Data`, create custom `toString()` |

---

## 2. IncorrectResultSizeDataAccessException

### Error

```java
org.springframework.dao.IncorrectResultSizeDataAccessException:
Query did not return a unique result: 2 results were returned
```

---

### Meaning

This error occurs when Spring expects only one record from the database, but the query returns multiple records.

In simple words:

> Spring asked for one result, but the database returned more than one matching record.

---

### Example

Repository method:

```java
public interface RoleRepo extends JpaRepository<Role, Integer> {

    Role findByName(String name);

}
```

This method expects exactly one `Role`.

But if the database contains:

| id | name  |
| -- | ----- |
| 1  | admin |
| 2  | admin |

Then:

```java
roleRepo.findByName("admin");
```

returns two records, causing the exception.

---

### Why Does It Happen?

#### 1. Duplicate Data in Database

```text
admin
admin
```

Multiple rows have the same value.

---

#### 2. Repository Method Expects a Single Object

```java
Role findByName(String name);
```

This means only one result is allowed.

---

#### 3. Missing Unique Constraint

Without:

```java
@Column(unique = true)
private String name;
```

duplicate values can be inserted into the database.

---

### Solution

#### 1. Remove Duplicate Records

Example:

```sql
DELETE FROM role WHERE id = 2;
```

Keep only one record for each role.

---

#### 2. Add a Unique Constraint

```java
@Column(unique = true)
private String name;
```

This prevents duplicate entries in the future.

---

#### 3. Return a List if Multiple Results Are Expected

```java
List<Role> findByName(String name);
```

Use this only when multiple records are valid.

---

### Quick Summary

| Problem                                | Reason                                         | Solution                                                   |
| -------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| IncorrectResultSizeDataAccessException | Query returned multiple records instead of one | Remove duplicates, add unique constraint, or return a List |

---

## Final Notes

* `StackOverflowError` is related to infinite recursion and memory usage.
* `IncorrectResultSizeDataAccessException` is related to database queries returning unexpected multiple results.
* Always use proper entity mappings and database constraints to avoid these issues.

Happy Coding! 🚀
