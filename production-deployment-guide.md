# Datalab Production Deployment Guide

This guide provides instructions for properly configuring and deploying Datalab in production mode with all required environment settings.

## Configuration Overview

1. **Essential Settings**:
   - Identifier prefix
   - Authentication configuration
   - Deployment metadata
   
2. **Authentication Options**:
   - GitHub OAuth
   - Email authentication
   - ORCID OAuth (optional)

3. **Deployment Details**:
   - Security settings
   - Backup configuration
   - Email notification setup

## Step 1: Configure the Environment File

1. Copy the provided `.env.updated` file to `.env`:
   ```bash
   cp .env.updated .env
   ```

2. Edit the `.env` file to update values specific to your deployment:
   ```bash
   nano .env
   ```

## Step 2: Configure Authentication

### Identifier Prefix

Set a unique identifier prefix for your deployment (required):
```
PYDATALAB_IDENTIFIER_PREFIX=mydatalab
```

### GitHub Authentication
Follow the instructions in `github-oauth-setup.md` to:
1. Create a GitHub OAuth application
2. Configure the GitHub client ID and secret
3. Set up organization restrictions if needed

### Email Authentication
Follow the instructions in `email-auth-setup.md` to:
1. Configure SMTP settings
2. Set up email domain restrictions if needed
3. Configure auto-activation settings

### ORCID Authentication (Optional)
If needed, configure ORCID authentication:
```
PYDATALAB_ORCID_CLIENT_ID=your_orcid_client_id
PYDATALAB_ORCID_CLIENT_SECRET=your_orcid_client_secret
PYDATALAB_ORCID_AUTO_ACTIVATE_ACCOUNTS=false
```

## Step 3: Set Deployment Metadata

Configure information about your deployment:
```
PYDATALAB_DEPLOYMENT_METADATA__MAINTAINER__DISPLAY_NAME=Your Name
PYDATALAB_DEPLOYMENT_METADATA__MAINTAINER__CONTACT_EMAIL=your.email@example.com
PYDATALAB_DEPLOYMENT_METADATA__HOMEPAGE=https://yourdomain.com
```

## Step 4: Security Configuration

Generate a secure random string for the secret key:
```bash
# Generate a random string
openssl rand -hex 32
```

Add it to your `.env` file:
```
PYDATALAB_SECRET_KEY=your_generated_secure_string
```

## Step 5: Deploy Datalab

1. Stop any running containers:
   ```bash
   sudo docker compose --file docker-compose.prod.yml --profile prod down
   ```

2. Start the updated deployment:
   ```bash
   sudo docker compose --file docker-compose.prod.yml --profile prod --env-file .env up --build
   ```

3. Verify all services are running:
   ```bash
   sudo docker compose --file docker-compose.prod.yml --profile prod ps
   ```

## Step 6: Verify Configuration

1. Access the frontend at http://localhost:8081
2. Check that you can log in using the configured authentication methods
3. Verify that the API is accessible at http://localhost:5001
4. Check that database connections are working properly

## Troubleshooting

If you encounter issues:

1. Check container logs:
   ```bash
   sudo docker compose --file docker-compose.prod.yml --profile prod logs api
   ```

2. Verify environment variables are properly loaded:
   ```bash
   sudo docker compose --file docker-compose.prod.yml --profile prod exec api env | grep PYDATALAB
   ```

3. If authentication is not working, check the specific authentication method guides.

## Maintenance

- Regularly back up your data
- Monitor the logs for errors
- Update the application as new versions become available
