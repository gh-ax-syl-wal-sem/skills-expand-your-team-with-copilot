# FastAPI Authentication Documentation

This document explains the authentication system implemented for the High School Management System API.

## Overview

The API uses **OAuth2 with JWT (JSON Web Tokens)** for authentication, following FastAPI best practices. This provides:
- Secure password storage using Argon2 hashing
- Token-based authentication for stateless API access
- Built-in support for FastAPI's automatic interactive documentation

## Authentication Flow

1. **Login**: Client sends username and password to `/auth/token`
2. **Token Generation**: Server validates credentials and returns a JWT access token
3. **Token Usage**: Client includes token in `Authorization: Bearer <token>` header
4. **Token Validation**: Protected endpoints validate the token and extract user information
5. **Expiration**: Tokens expire after 30 minutes

## Endpoints

### POST /auth/token
OAuth2-compliant login endpoint that returns a JWT access token.

**Request** (form data):
```
username=mrodriguez
password=art123
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

### GET /auth/me
Returns information about the currently authenticated user.

**Request Headers**:
```
Authorization: Bearer <your_access_token>
```

**Response**:
```json
{
  "username": "mrodriguez",
  "display_name": "Ms. Rodriguez",
  "role": "teacher",
  "disabled": false
}
```

### Legacy Endpoints (Deprecated)
- `POST /auth/login` - Use `/auth/token` instead
- `GET /auth/check-session` - Use `/auth/me` instead

## Protected Endpoints

The following endpoints require authentication:
- `POST /activities/{activity_name}/signup` - Sign up a student for an activity
- `POST /activities/{activity_name}/unregister` - Remove a student from an activity

These endpoints automatically validate the JWT token and extract user information.

## Security Features

### Password Hashing
- Uses **Argon2** for password hashing (industry best practice)
- Argon2 is memory-hard and resistant to GPU cracking attacks
- Replaced insecure SHA-256 hashing from previous implementation

### JWT Tokens
- Tokens are signed using HS256 algorithm
- Include expiration timestamp (30 minutes)
- Contain minimal user information (username only)
- Cannot be forged without the secret key

### Token Validation
- Automatic validation on protected endpoints
- Checks signature, expiration, and payload structure
- Returns 401 Unauthorized for invalid tokens
- Includes proper `WWW-Authenticate` headers

## Testing Authentication

### Using curl

1. **Get a token**:
```bash
curl -X POST "http://localhost:8000/auth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=mrodriguez&password=art123"
```

2. **Use the token**:
```bash
curl -X GET "http://localhost:8000/auth/me" \
  -H "Authorization: Bearer <your_token>"
```

3. **Access protected endpoint**:
```bash
curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=student@example.com" \
  -H "Authorization: Bearer <your_token>"
```

### Using FastAPI Docs

1. Navigate to `http://localhost:8000/docs`
2. Click "Authorize" button
3. Enter username and password
4. Click "Authorize"
5. Try protected endpoints - authentication is automatic!

## Default Users

The system comes with three pre-configured teacher accounts:

| Username   | Password | Display Name        | Role    |
|------------|----------|---------------------|---------|
| mrodriguez | art123   | Ms. Rodriguez       | teacher |
| mchen      | chess456 | Mr. Chen           | teacher |
| principal  | admin789 | Principal Martinez | admin   |

## Configuration

### Secret Key
⚠️ **Important**: Change the `SECRET_KEY` in production!

The secret key is currently hardcoded in `backend/routers/auth.py`:
```python
SECRET_KEY = "your-secret-key-here-change-in-production"
```

For production, use an environment variable:
```python
import os
SECRET_KEY = os.getenv("SECRET_KEY", "fallback-key-for-dev")
```

### Token Expiration
Default: 30 minutes

To change, modify `ACCESS_TOKEN_EXPIRE_MINUTES` in `backend/routers/auth.py`.

## Implementation Details

### Dependencies
- `pyjwt` - JWT token creation and validation
- `python-multipart` - Required for OAuth2 form data
- `argon2-cffi` - Argon2 password hashing

### Key Functions
- `verify_password()` - Verify password against Argon2 hash
- `create_access_token()` - Generate JWT with expiration
- `get_current_user()` - Dependency to validate token and extract user
- `get_current_active_user()` - Dependency to ensure user is not disabled

### Models
- `Token` - Response model for login endpoint
- `TokenData` - Internal model for JWT payload validation
- `User` - User information model

## Best Practices Followed

1. ✅ OAuth2 with Password Flow (recommended for first-party apps)
2. ✅ JWT tokens with expiration
3. ✅ Argon2 password hashing (current best practice)
4. ✅ Dependency injection for authentication
5. ✅ Proper HTTP status codes and headers
6. ✅ Automatic OpenAPI/Swagger documentation
7. ✅ Type hints with Pydantic models

## Migration from Old System

The old authentication system has been updated:
- ❌ SHA-256 password hashing → ✅ Argon2
- ❌ Manual username checking → ✅ JWT tokens with dependencies
- ❌ No token expiration → ✅ 30-minute expiration
- ❌ Query parameters for auth → ✅ Authorization headers

Legacy endpoints are maintained for backward compatibility but marked as deprecated.

## References

- [FastAPI Security Documentation](https://fastapi.tiangolo.com/tutorial/security/)
- [OAuth2 Password Flow](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/)
- [Argon2 Password Hashing](https://github.com/P-H-C/phc-winner-argon2)
- [JWT Specification](https://jwt.io/)
