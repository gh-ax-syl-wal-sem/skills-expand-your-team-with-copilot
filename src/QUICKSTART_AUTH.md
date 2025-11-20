# Authentication Quick Start Guide

This guide helps you quickly test the authentication system.

## Quick Test with FastAPI Interactive Docs

1. **Start the server:**
   ```bash
   cd src
   uvicorn app:app --reload
   ```

2. **Open your browser:**
   Navigate to http://localhost:8000/docs

3. **Authenticate:**
   - Click the green "Authorize" button at the top right
   - Enter credentials:
     - Username: `mrodriguez`
     - Password: `art123`
   - Click "Authorize" then "Close"

4. **Try protected endpoints:**
   - Expand `POST /activities/{activity_name}/signup`
   - Click "Try it out"
   - Enter:
     - `activity_name`: `Chess Club`
     - `email`: `newstudent@mergington.edu`
   - Click "Execute"
   - You should see a successful response!

## Quick Test with curl

1. **Get a token:**
   ```bash
   curl -X POST "http://localhost:8000/auth/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "username=mrodriguez&password=art123"
   ```

   Response:
   ```json
   {
     "access_token": "eyJhbGc...",
     "token_type": "bearer"
   }
   ```

2. **Save the token:**
   ```bash
   TOKEN="paste-your-token-here"
   ```

3. **Get your user info:**
   ```bash
   curl -X GET "http://localhost:8000/auth/me" \
     -H "Authorization: Bearer $TOKEN"
   ```

4. **Sign up a student:**
   ```bash
   curl -X POST "http://localhost:8000/activities/Chess%20Club/signup?email=student@mergington.edu" \
     -H "Authorization: Bearer $TOKEN"
   ```

## Test Accounts

| Username   | Password | Role    |
|------------|----------|---------|
| mrodriguez | art123   | teacher |
| mchen      | chess456 | teacher |
| principal  | admin789 | admin   |

## Common Issues

### Issue: "Could not validate credentials"
- **Cause:** Token expired (30 minutes)
- **Solution:** Get a new token by logging in again

### Issue: "Not authenticated"
- **Cause:** Missing or invalid Authorization header
- **Solution:** Make sure to include `Authorization: Bearer <token>` header

### Issue: "Incorrect username or password"
- **Cause:** Wrong credentials or user doesn't exist
- **Solution:** Check username and password, use test accounts above

## What's Protected?

These endpoints require authentication:
- ✅ `POST /activities/{activity_name}/signup`
- ✅ `POST /activities/{activity_name}/unregister`
- ✅ `GET /auth/me`

These endpoints are public (no auth needed):
- ✅ `GET /activities/`
- ✅ `GET /activities/days`
- ✅ `POST /auth/token`

## Testing Token Expiration

To test that tokens expire:

1. Get a token
2. Wait 31 minutes
3. Try to use the token
4. Should get "Could not validate credentials" error

To test immediately, change `ACCESS_TOKEN_EXPIRE_MINUTES` in `backend/routers/auth.py` to `1` and restart the server.

## Python Test Script

Quick Python script to test authentication:

```python
import requests

# Base URL
BASE_URL = "http://localhost:8000"

# 1. Login
response = requests.post(
    f"{BASE_URL}/auth/token",
    data={"username": "mrodriguez", "password": "art123"}
)
token = response.json()["access_token"]
print(f"✓ Got token: {token[:20]}...")

# 2. Get user info
headers = {"Authorization": f"Bearer {token}"}
response = requests.get(f"{BASE_URL}/auth/me", headers=headers)
user = response.json()
print(f"✓ Logged in as: {user['display_name']}")

# 3. Sign up a student
response = requests.post(
    f"{BASE_URL}/activities/Chess Club/signup",
    params={"email": "test@mergington.edu"},
    headers=headers
)
print(f"✓ Signup response: {response.json()}")
```

Save as `test_auth.py` and run: `python test_auth.py`

## Next Steps

- Read [AUTHENTICATION.md](./AUTHENTICATION.md) for detailed documentation
- Check the [FastAPI docs](https://fastapi.tiangolo.com/tutorial/security/) for more info
- Explore the automatically generated API docs at http://localhost:8000/docs

## Security Reminder

⚠️ **For production:**
1. Change the `SECRET_KEY` in `backend/routers/auth.py`
2. Use HTTPS
3. Set secret key via environment variable
4. Add rate limiting
5. Monitor authentication logs
