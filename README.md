# my-debug-diary
Collection of real project errors, debugging steps, root causes, and solutions during backend development practice.
---One issue I encountered during Spring Boot API development was:

"HttpMessageNotReadableException: Cannot map null into type int."

This happened when I was sending request data from Postman to my REST API. One field in JSON either had a null value or was missing completely, but in my entity that field was declared as primitive int.

For example:

private int trainNumber;

If the request body sends:

{
"trainNumber": null
}

Spring uses Jackson internally to convert JSON into Java objects. Since primitive int cannot store null values, deserialization failed and the request was rejected before reaching the service layer.

To resolve this issue, I used one of these approaches:

First, I validated and corrected the request payload so mandatory fields were always sent.

Second, where null values were acceptable, I changed primitive types to wrapper classes.

Example:

private Integer trainNumber;

Wrapper classes can handle null safely.

Additionally, I used validation annotations such as @NotNull and handled exceptions globally using @ControllerAdvice so users receive proper error messages instead of internal server errors.

From this issue I learned that request validation and correct datatype selection are important while designing APIs.

---

# Error Log Repository

## 1. Error Name

### Error:

org.springframework.http.converter.HttpMessageNotReadableException:

JSON parse error:

Cannot map `null` into type `int`
(set `DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES` to false to allow)

---

### Why it happened:

This error occurred because Spring Boot was trying to convert incoming JSON data into a Java object (deserialization process).

One field in the request body was either:

* missing, or
* sent as null

But inside the entity the field datatype was declared as primitive `int`.

Example:

private int trainNumber;

Primitive datatypes cannot store null values, so Spring failed before entering the service layer.

Example request causing error:

{
"trainNumber": null,
"trainName": "Rajdhani"
}

---

### How I debugged:

Step 1:
Checked the complete exception stack trace in console logs.

Step 2:
Identified that exception occurred during JSON parsing before controller execution.

Step 3:
Verified request payload in Postman.

Step 4:
Compared JSON fields with entity datatypes.

Step 5:
Found that trainNumber was missing or null while entity expected primitive int.

Step 6:
Updated request and datatype accordingly.

---

### Solution:

Option 1 — Send valid request

{
"trainNumber": 101,
"trainName": "Rajdhani Express"
}

OR

Option 2 — Change datatype

Before:

private int trainNumber;

After:

private Integer trainNumber;

Optional:
Add validation.

@NotNull
private Integer trainNumber;

---

### Interview Answer:

"While building an IRCTC train booking project using Spring Boot, I encountered HttpMessageNotReadableException. The issue happened because a request field was coming as null, but my entity used primitive int which cannot accept null values. I debugged the issue by checking logs, validating the Postman payload, and tracing request mapping. I resolved it by correcting the JSON request and replacing primitive types with wrapper classes where null values were expected. I also added validation to prevent similar issues."
---
