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
