# Keycloak Development Environment

A simple setup for running Keycloak in GitHub Codespaces for development work.

## Quick Start

1. Increase your Codespace timeout limit for inactivity as follows:
  * Go to **GitHub > Settings > Codespaces**
  * Under `Idle timeout`, increase the value (max is **240 minutes**)
2. Fork repo
3. **Open in Codespaces**: Click the green "Code" button in GitHub and select "Create codespace on main"
4. **Wait for setup**: The environment will automatically set up Keycloak with PostgreSQL
5. **Access Keycloak**: Once ready, open `http://localhost:8080/admin` in your browser
6. On subsequent Codespace starts, run `docker-compose up -d`

## Getting Started

All references to http://localhost:8080 should use the Codespace forwarded port (e.g. `https://animated-space-succotash-wr545qpxc9qgx-8080.app.github.dev`).

* Change the port visibility from private to public
* Log into the Admin Console at `http://localhost:8080/admin` with Username: `admin`, Password: `admin`
* The first thing you should do is create a realm as it is not recommended to use the master realm for anything other than managing Keycloak itself (e.g., managing other realms and admin users)
* Your JWKS URL is: https://your-keycloak-url/realms/{realm-name}/protocol/openid-connect/certs
* Your token URL is: https://your-keycloak-url/realms/{realm-name}/protocol/openid-connect/token

## Default Credentials

- **Admin Console**: `http://localhost:8080/admin`
- **Username**: `admin`
- **Password**: `admin`

## What's Included

- **Keycloak 23.0**: Latest stable version running in development mode
- **PostgreSQL 15**: Database backend for Keycloak
- **Docker Compose**: Easy container orchestration
- **VS Code Extensions**: JSON, YAML, and Docker support

## Development Workflow

### Starting the Environment
The environment starts automatically when you open the Codespace. If you need to restart:

```bash
docker-compose down
docker-compose up -d
```

### Accessing Services
- **Keycloak Admin Console**: http://localhost:8080/admin
- **Keycloak Account Console**: http://localhost:8080/realms/master/account
- **PostgreSQL**: localhost:5432

### Custom Themes
Place custom themes in the `themes/` directory. They'll be automatically mounted into Keycloak.

## Data Persistence

**Important**: By default, PostgreSQL data persists between Codespace sessions using a bind mount to `./postgres-data/`. This directory is stored in the Codespace's persistent storage.

However, if the Codespace is deleted or rebuilt, all data will be lost. For important configurations:

### Export Your Realm Configuration
```bash
# Export a specific realm
docker-compose exec keycloak /opt/keycloak/bin/kc.sh export --dir /opt/keycloak/data/import --realm your-realm-name

# Export all realms
docker-compose exec keycloak /opt/keycloak/bin/kc.sh export --dir /opt/keycloak/data/import
```

### Import Realm on Startup
Place your realm JSON files in the `imports/` directory. They'll be available at `/opt/keycloak/data/import` inside the container.

### Database Access
Connect to PostgreSQL directly:
```bash
docker-compose exec keycloak-db psql -U keycloak -d keycloak
```

## Configuration

The setup uses development-friendly configurations:
- Hostname restrictions disabled
- HTTPS not enforced
- Detailed logging enabled
- Metrics and health endpoints enabled

## Troubleshooting

### Keycloak won't start
- Check if ports 8080 or 5432 are already in use
- Restart the containers: `docker-compose restart`

### Can't access admin console
- Ensure Keycloak is fully started (check logs: `docker-compose logs keycloak`)
- Verify port forwarding is working in Codespaces

### Database connection issues
- Check if PostgreSQL is running: `docker-compose ps`
- Restart database: `docker-compose restart keycloak-db`

## Useful Commands

```bash
# View logs
docker-compose logs -f keycloak
docker-compose logs -f keycloak-db

# Restart services
docker-compose restart

# Stop everything
docker-compose down

# Start fresh (removes data)
docker-compose down -v
docker-compose up -d
```

## Next Steps

1. Create your first realm
2. Configure authentication flows
3. Set up user federation
4. Develop custom themes
5. Integrate with your applications

## Resources

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Keycloak Admin Guide](https://www.keycloak.org/docs/latest/server_admin/)
- [Keycloak Developer Guide](https://www.keycloak.org/docs/latest/server_development/)

## JWKS Endpoint URL
https://your-keycloak-url/realms/{realm-name}/protocol/openid-connect/certs

e.g. https://animated-space-succotash-wr545qpxc9qgx-8080.app.github.dev/realms/{realm-name}/protocol/openid-connect/certs

## Token URL
https://your-keycloak-url/realms/{realm-name}/protocol/openid-connect/token

e.g. https://animated-space-succotash-wr545qpxc9qgx-8080.app.github.dev/realms/lbrenman/protocol/openid-connect/token

## REST API Calls

https://www.keycloak.org/docs-api/latest/rest-api/index.html

Create a Dedicated Service Account Client
This is more secure for programmatic access:

Create a client in the Keycloak admin console:

Go to Clients → Create Client
Client ID: api-client
Client authentication: ON
Service accounts roles: ON


Get client credentials from the Credentials tab
Use client credentials grant:

```bash
# Replace with your actual client secret
CLIENT_SECRET="your-client-secret-here"

TOKEN_RESPONSE=$(curl -s -X POST \
  'https://your-codespace-url.app.github.dev/realms/master/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials' \
  -d 'client_id=api-client' \
  -d 'client_secret='$CLIENT_SECRET)

ACCESS_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.access_token')
```

You will need to assign roles to your `api-client` service account to access the Admin REST API. Here's how to set it up:

### Assign Admin Roles to Your Service Account

1. **Go to your realm** (`lbrenman`) in Keycloak admin console
2. **Navigate to**: Clients → `api-client` → **Service Account Roles** tab
3. **Assign roles** depending on what level of access you need:

#### Option A: Full Admin Access (Easiest)
- **Client roles** → Select `realm-management` from dropdown
- Assign: `realm-admin`

This gives full administrative access to the realm.

#### Option B: Specific Permissions (More Secure)
Instead of `realm-admin`, assign only what you need:

**For user management:**
- `view-users`
- `manage-users` 
- `create-user`

**For client management:**
- `view-clients`
- `manage-clients`

**For realm configuration:**
- `view-realm`
- `manage-realm`

**For identity providers:**
- `view-identity-providers`
- `manage-identity-providers`

## Test Your API Access

After assigning roles, test with your token:

```bash
# Get token
TOKEN_RESPONSE=$(curl -s --location 'https://animated-space-succotash-wr545qpxc9qgx-8080.app.github.dev/realms/lbrenman/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data 'grant_type=client_credentials' \
--data 'client_id=api-client' \
--data 'client_secret=NsFVZCo0Zf8TB4DfwN6cRzSgobgUBbs6')

ACCESS_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.access_token')

# Test API calls
# List users
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
'https://animated-space-succotash-wr545qpxc9qgx-8080.app.github.dev/admin/realms/lbrenman/users'

# Get realm info
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
'https://animated-space-succotash-wr545qpxc9qgx-8080.app.github.dev/admin/realms/lbrenman'
```

## Role Reference

Here are the main realm-management roles and what they allow:

| Role | Permissions |
|------|-------------|
| `realm-admin` | Full administrative access |
| `manage-users` | Create, update, delete users |
| `view-users` | Read user information |
| `manage-clients` | Create, update, delete clients |
| `view-clients` | Read client information |
| `manage-realm` | Configure realm settings |
| `view-realm` | Read realm configuration |
| `manage-identity-providers` | Configure SSO providers |
| `view-identity-providers` | Read SSO provider configs |

## Quick Verification

You can also check what roles your service account has by decoding the JWT token:

```bash
# Decode the token payload (requires base64 and jq)
echo $ACCESS_TOKEN | cut -d'.' -f2 | base64 -d 2>/dev/null | jq '.realm_access.roles'
```

Start with `realm-admin` for full access, then narrow it down to specific roles based on your actual needs for better security.

## Common API Examples

Once you have an access token:

```bash
# List all realms
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  'https://your-codespace-url.app.github.dev/admin/realms'

# Get users in master realm
curl -H "Authorization: Bearer $ACCESS_TOKEN" \
  'https://your-codespace-url.app.github.dev/admin/realms/master/users'

# Create a new user
curl -X POST \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "enabled": true,
    "firstName": "Test",
    "lastName": "User",
    "email": "test@example.com"
  }' \
  'https://your-codespace-url.app.github.dev/admin/realms/master/users'
```

## Quick Test Script
Here's a simple script to test the API access:

```bash
#!/bin/bash
KEYCLOAK_URL="https://your-codespace-url.app.github.dev"

# Get token using admin credentials
TOKEN_RESPONSE=$(curl -s -X POST \
  "$KEYCLOAK_URL/realms/master/protocol/openid-connect/token" \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password' \
  -d 'client_id=admin-cli' \
  -d 'username=admin' \
  -d 'password=admin')

if [ $? -eq 0 ]; then
  ACCESS_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.access_token')
  
  if [ "$ACCESS_TOKEN" != "null" ] && [ "$ACCESS_TOKEN" != "" ]; then
    echo "✅ Successfully obtained access token"
    
    # Test API call
    curl -s -H "Authorization: Bearer $ACCESS_TOKEN" \
      "$KEYCLOAK_URL/admin/realms" | jq '.[].realm'
  else
    echo "❌ Failed to get access token"
    echo $TOKEN_RESPONSE | jq '.'
  fi
else
  echo "❌ Failed to connect to Keycloak"
fi
```