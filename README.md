# Notes CRUD API – Assignment Submission

## 1. Framework Choice
- **Spring Boot**   
- **Reason:**  
  - Enterprise-ready framework with built-in **Spring Security** and **Spring Data JPA**.  
  - Strong ecosystem for authentication and CRUD APIs.  
  - Production-friendly and widely used in the industry.  

---

## 2. Database Schema

```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Notes table
CREATE TABLE notes (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    title VARCHAR(100) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    version BIGINT, -- used for optimistic locking
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```
## 3. Authentication Choice

- *JWT (JSON Web Token) authentication*

- **Why JWT?**

  - Stateless → no need to store sessions on the server

  - Works well with REST APIs and mobile/web clients

  - Supported by Spring Security

## 4. Routes (with Method, Path, Request/Response JSON)
- Auth

- POST /auth/signup
  Request:
```
{ "username": "alice", "password": "secret123" }
```

Response:
```
{ "id": 1, "username": "alice" }
```

POST /auth/login
Request:
```
{ "username": "alice", "password": "secret123" }
```

Response:
```
{ "access_token": "<jwt_token>", "token_type": "bearer" }
```
Notes (JWT Protected)

POST /notes
Request:
```
{ "title": "Shopping List", "content": "Milk, Eggs, Bread" }
```

Response:
```
{ "id": 101, "title": "Shopping List", "content": "Milk, Eggs, Bread", "user_id": 1 }
```

GET /notes
Response:
```
[
  { "id": 101, "title": "Shopping List", "content": "Milk, Eggs, Bread", "user_id": 1 }
]
```

GET /notes/{id}
Response:
```
{ "id": 101, "title": "Shopping List", "content": "Milk, Eggs, Bread", "user_id": 1 }
```

PUT /notes/{id}
Request:
```
{ "title": "Updated List", "content": "Milk, Eggs, Bread, Butter" }
```

Response:
```
{ "id": 101, "title": "Updated List", "content": "Milk, Eggs, Bread, Butter", "user_id": 1 }
```

DELETE /notes/{id}
Response:
```
{ "message": "Note deleted" }
```
## 5. Failure Mode & Mitigation

- Failure Mode: Race condition during concurrent updates (two clients editing the same note at once).

- Example Problem:

- User A fetches note, edits, and sends update at 10:00:01.

- User B fetches the same note, edits, and sends update at 10:00:02.

- User B’s update overwrites User A’s changes.

- *Mitigation:*

- Use Optimistic Locking via a @Version field in the Note entity.

- When a client updates a note, it must include the last known version.

- If the database detects a mismatch (another update already happened), Spring throws an OptimisticLockException.

- API returns 409 Conflict with message:
```
{ "error": "Note was updated by another user, please refresh before saving." }
```
