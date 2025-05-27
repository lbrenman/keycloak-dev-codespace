# Keycloak Development Environment

A simple setup for running Keycloak in GitHub Codespaces for development work.

## Quick Start

1. **Open in Codespaces**: Click the green "Code" button in GitHub and select "Create codespace on main"
2. **Wait for setup**: The environment will automatically set up Keycloak with PostgreSQL
3. **Access Keycloak**: Once ready, open `http://localhost:8080` in your browser

## Default Credentials

- **Admin Console**: `http://localhost:8080/admin`
- **Username**: `admin`
- **Password**: `admin`

## What's Included

- **Keycloak 23.0**: Latest stable version running in development mode
- **PostgreSQL 15**: Database backend for Keycloak
- **Docker Compose**: Easy container orchestration
- **VS Code Extensions**: JSON, YAML, and Docker support

## File Structure

```
.
├── .devcontainer/
│   └── devcontainer.json     # Codespaces configuration
├── docker-compose.yml        # Container definitions
├── themes/                   # Custom Keycloak themes (optional)
├── imports/                  # Realm import files (optional)
└── README.md
```

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

For production deployment, you'll need to:
- Enable HTTPS
- Set proper hostname
- Use stronger passwords
- Configure proper database settings

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