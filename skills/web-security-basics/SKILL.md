---
name: web-security-basics
description: "Use when writing login, forms, database queries, or file uploads in any web stack."
---

# web-security-basics Guidelines

Use prepared statements or parameterized queries for all SQL. Hash passwords with a strong algorithm such as bcrypt or password_hash and never store plain text. Validate input on the server, not only in the browser. Escape output to prevent XSS. Regenerate the session ID on login. Restrict upload types and sizes. Keep secrets in environment variables.
