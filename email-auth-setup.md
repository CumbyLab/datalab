# Setting up Email Authentication for Datalab

To properly configure email-based authentication for your Datalab deployment, follow these steps:

## 1. Configure SMTP Settings

You'll need access to an SMTP server that can send emails. This could be:
- Your organization's email server
- A third-party email service like SendGrid, Mailgun, or Amazon SES
- A Gmail account (with "Less secure app access" enabled for testing)

## 2. Update your Datalab .env file

Add or update the following variables in your `.env` file:

```
# Email Authentication
PYDATALAB_EMAIL_AUTH_SMTP_SETTINGS__MAIL_SERVER=smtp.example.com
PYDATALAB_EMAIL_AUTH_SMTP_SETTINGS__MAIL_PORT=587
PYDATALAB_EMAIL_AUTH_SMTP_SETTINGS__MAIL_USERNAME=your_username
PYDATALAB_MAIL_PASSWORD=your_password
PYDATALAB_EMAIL_AUTH_SMTP_SETTINGS__MAIL_USE_TLS=true
PYDATALAB_EMAIL_AUTH_SMTP_SETTINGS__MAIL_DEFAULT_SENDER=no-reply@example.com
PYDATALAB_EMAIL_AUTO_ACTIVATE_ACCOUNTS=false
```

If you want to restrict registration to specific email domains:

```
PYDATALAB_EMAIL_DOMAIN_ALLOW_LIST=["yourdomain.com"]
```

## 3. Auto-activation Options

You have a few options for how accounts are activated:

1. **Manual activation by admin** (default):
   - `PYDATALAB_EMAIL_AUTO_ACTIVATE_ACCOUNTS=false`
   - New users will need to be activated by an administrator

2. **Auto-activation for all users**:
   - `PYDATALAB_EMAIL_AUTO_ACTIVATE_ACCOUNTS=true`
   - All users who verify their email address will be automatically activated

3. **Domain-based auto-activation**:
   - Configure `PYDATALAB_EMAIL_DOMAIN_ALLOW_LIST=["yourdomain.com"]`
   - Users with email addresses in the allowed domains will be auto-activated upon verification

## 4. Restart your Datalab application

After updating the configuration, restart your Datalab containers:

```bash
sudo docker compose --file docker-compose.prod.yml --profile prod --env-file .env down
sudo docker compose --file docker-compose.prod.yml --profile prod --env-file .env up --build
```

## 5. Test Email Authentication

1. Go to your Datalab instance (http://localhost:8081)
2. Click on the login/sign-up option
3. Select "Sign up with Email"
4. Enter your email address and follow the verification process
5. Depending on your activation settings, either wait for admin approval or log in immediately

## Troubleshooting

- If verification emails are not being received, check:
  - Spam/junk folders
  - SMTP server logs
  - Datalab API logs for email sending errors
- Ensure your SMTP credentials are correct
- If using a corporate email server, ensure it allows sending from the configured sender address
