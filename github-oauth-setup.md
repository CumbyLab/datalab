# Setting up GitHub OAuth for Datalab

To properly configure GitHub authentication for your Datalab deployment, follow these steps:

## 1. Create a GitHub OAuth Application

1. Go to your GitHub account settings
2. Navigate to "Developer settings" → "OAuth Apps" → "New OAuth App"
3. Fill in the application details:
   - **Application name**: Your Datalab Instance Name
   - **Homepage URL**: http://localhost:8081 (or your production URL)
   - **Application description**: (Optional) Description of your Datalab instance
   - **Authorization callback URL**: http://localhost:5001/api/auth/github/callback

4. Click "Register application"
5. After creation, you'll see your Client ID
6. Generate a Client Secret by clicking "Generate a new client secret"
7. Copy both the Client ID and Client Secret

## 2. Update your Datalab .env file

Update the following variables in your `.env` file:

```
PYDATALAB_GITHUB_CLIENT_ID=your_client_id_from_github
PYDATALAB_GITHUB_CLIENT_SECRET=your_client_secret_from_github
PYDATALAB_GITHUB_AUTO_ACTIVATE_ACCOUNTS=true
```

If you want to restrict access to members of specific GitHub organizations:

```
PYDATALAB_GITHUB_ORG_ALLOW_LIST=["your-org-name"]
```

## 3. Restart your Datalab application

After updating the configuration, restart your Datalab containers:

```bash
sudo docker compose --file docker-compose.prod.yml --profile prod --env-file .env down
sudo docker compose --file docker-compose.prod.yml --profile prod --env-file .env up --build
```

## 4. Test GitHub Authentication

1. Go to your Datalab instance (http://localhost:8081)
2. Click on the login/sign-up option
3. Select "Sign in with GitHub"
4. You should be redirected to GitHub for authorization
5. After authorizing, you should be redirected back to Datalab and be logged in

## Troubleshooting

- If the redirect fails, double-check that the callback URL in GitHub exactly matches the URL your API is using
- Ensure your containers can reach GitHub's servers
- Check the logs of the API container for any authentication errors
