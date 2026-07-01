# Mistakes & Corrections

- JWT validation happens on the server, not the client.
- prefix-suffix is not eb example of divide and conquer.

## Day 5

### Backend

- Avoid mixing JWT and OAuth too early. Complete one abstraction before introducing another.
- Remember: ID Tokens are for clients; Access Tokens are for APIs.
- JWT is not always better than sessions; revocation requirements matter.

### Coding

- Compact pointer movement inside comparisons is correct but less readable.
- Prefer clear pointer movement after comparison in production code.
