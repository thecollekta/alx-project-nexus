# Deployment Guide: Render.com

This guide walks you through deploying the Project Nexus ALX E-Commerce backend to Render.com, a modern cloud platform that makes it easy to deploy and scale applications.

## Tech Stack and Tools

This section outlines the primary technologies and tools I worked with during the program.

- **Backend**: Python 3.10+, Django 5.0+, Django REST Framework
- **Database**: PostgreSQL 17+
- **Cache & Message Broker**: Redis
- **Async Tasks**: Celery
- **Containerization**: Docker, Docker Compose
- **CI/CD**: Scripts (for deployment on [render.com](https://render.com) and GitHub Actions (Did not use it in the project)
- **API Documentation**: OpenAPI/Swagger, GraphiQL (had to remove GraphQL due to time constraints and technical challenges to focus on REST API.)

## Prerequisites

- A [Render.com](https://render.com/) account
- GitHub account with repository access
- [Render CLI](https://render.com/docs/cli) (optional)
- [Docker](https://www.docker.com/) (for local testing)

## Prepare Your Application

### Update Configuration

Ensure your `.env` file is properly configured for production:

```bash
# .env.production
DEBUG=False
DJANGO_ALLOWED_HOSTS=your-render-app.onrender.com
SECRET_KEY=your-secure-secret-key
DATABASE_URL=postgres://username:password@host:port/dbname  # Will be provided by Render
REDIS_URL=redis://red-xxxxxxxxxxxx:6379  # If using Redis
```

### Update Requirements

Either use `requirements/production.txt` file included in this repo and edit to suite your application or make a `requirements.txt` file with all necessary packages. Generate it using `pip freeze > requirements.txt` or manually ensure it includes:

```bash
# requirements.txt
Django>=5.0
djangorestframework>=3.14
psycopg2-binary>=2.9.5
gunicorn>=20.1.0
whitenoise>=6.4.0
django-cors-headers>=4.1.0
python-dotenv>=0.21.0
```

### Create a `render.yaml` File

```yml
# render.yaml
services:

- type: web
    name: ecommerce-backend # a name for your service
    env: python
    buildCommand: "chmod +x render-build.sh && ./render-build.sh"
    startCommand: "gunicorn ecommerce_backend.wsgi:application --bind 0.0.0.0:$PORT --workers 2 --timeout 120"
    envVars:
        - key: DJANGO_SETTINGS_MODULE
            value: ecommerce_backend.settings.production
        - key: SUPERUSER_USERNAME
            value: admin
        - key: SUPERUSER_EMAIL
            value: admin@ecommerce.com
        - key: SUPERUSER_PASSWORD
            generateValue: true
        - key: PYTHON_VERSION
                value: "3.13.5"
        - key: DATABASE_URL
                fromDatabase:
                name: project-nexus-db
                property:connectionString
        - key: SECRET_KEY
                generateValue: true
        - key: WEB_CONCURRENCY
                value: "4"
            plan: free  # or 'starter' or 'standard' for production

databases:

- name: project-nexus-db
    databaseName: project_nexus
    user: project_nexus_user
    plan: free  # or 'starter'/'standard' for production
```

### Create a build.sh Script

```bash
#!/usr/bin/env bash
# Exit on error
set -o errexit

# Install Python dependencies
pip install -r requirements.txt

# Collect static files
python manage.py collectstatic --no-input

# Apply database migrations
python manage.py migrate
```

## Deploy to Render

### Connect Your GitHub Repository

1. Log in to your Render.com account.
2. Click on "New" and select "Web Service".
3. Connect your GitHub account and select the repository containing your Project Nexus code.
4. Choose the branch you want to deploy (e.g., `main` or `master`).
5. Render will automatically detect the `render.yaml` file and configure the service.
6. Click "Create Web Service".
7. Render will start building and deploying your application.
8. Once the deployment is complete, you will receive a URL where your application is hosted.
9. You can access the Django admin interface using the superuser credentials you set in the environment variables.
10. Monitor logs and performance from the Render dashboard.

Alternatively, you can use manually setup without `render.yaml`:

1. Log in to your Render Dashboard
2. Click "New" → "Web Service"
3. Connect your GitHub repository
4. Select the repository and branch to deploy

#### Configure Your Service

1. Name: `project-nexus` (or your preferred name)
2. Region: Select the closest region to your users
3. Branch: `main` (or your production branch)
4. Build Command: `./build.sh`
5. Start Command: `gunicorn ecommerce_backend.wsgi:application`

### Set Environment Variables

Add the following environment variables in the Render dashboard:

| Key | Value | Notes|
|-----|-------|------|
| PYTHON_VERSION | `3.13.5` | Match your local Python version |
| DJANGO_SETTINGS_MODULE | `ecommerce_backend.settings.production` | Your production settings module |
| SECRET_KEY | `your-secure-secret-key` | Generate a new one for production |
| DEBUG | `False` | Always False in production |
| ALLOWED_HOSTS | `your-render-app.onrender.com` | Your Render app URL |
| DISABLE_COLLECTSTATIC | `1` | If you're not using static files |
| DATABASE_URL | `postgres://username:password@host:port/dbname` | Provided by Render after creating a database |
| REDIS_URL | `redis://red-xxxxxxxxxxxx:6379` | If using Redis, provided by Render |
| SUPERUSER_USERNAME | `admin` | Your desired superuser username |
| SUPERUSER_EMAIL | `name-of--your-choice` | Your desired superuser email |
| SUPERUSER_PASSWORD | `your-secure-password` | Your desired superuser password |
| WEB_CONCURRENCY | `4` | Number of Gunicorn workers |
| CORS_ALLOWED_ORIGINS | `https://your-frontend-domain.com` | Your frontend domain for CORS |

### Set Up the Database

1. In the Render dashboard, go to "New" → "PostgreSQL"
2. Configure the database:
    - Name: `project-nexus-db`
    - Database: `project_nexus`
    - User: `project_nexus_user`
    - Plan: Free (or select a paid plan for production)
3. Copy the `DATABASE_URL` and add it to your environment variables

### Set Up Redis

1. In the Render dashboard, go to "New" → "Redis"
2. Configure Redis:
    - Name: `project-nexus-redis`
    - Plan: Free (or select a paid plan for production)
    - Copy the `REDIS_URL` and add it to your environment variables
    - Update your Django settings to use Redis for caching and Celery if applicable
    - Example:

      ```python
      CACHES = {
          'default': {
              'BACKEND': 'django_redis.cache.RedisCache',
              'LOCATION': os.getenv('REDIS_URL'),
              'OPTIONS': {
                  'CLIENT_CLASS': 'django_redis.client.DefaultClient',
              }
          }
      }
      ```

### Deploy

1. Click "Create Web Service"
2. Render will automatically build and deploy your application
3. Monitor the build logs for any issues

### Post-Deployment Steps

#### Run Migrations

If you didn't include migrations in your build script, run them manually:

```bash
render shell <your-service-name>
render run -- python manage.py migrate
```

#### Create a Superuser

If you didn't set up a superuser via environment variables, create one manually:

```bash
render shell <your-service-name>
render run -- python manage.py createsuperuser
```

#### Set Up a Custom Domain (Optional)

1. Go to your service in the Render dashboard
2. Click "Settings" → "Custom Domains"
3. Add your domain and follow the verification steps

### Continuous Deployment

Render automatically redeploys your application on every push to the connected GitHub branch. Ensure your tests pass before pushing changes to avoid deployment failures. To set up a production/staging workflow:

1. **Production**: Connect your `main` or `production` branch
2. **Staging**: Create a separate service connected to your `develop` branch

## Monitoring and Logs

- `Logs`: View real-time logs in the Render dashboard
- `Metrics`: Monitor CPU, memory, and request metrics
- `Alerts`: Set up email notifications for downtime

## Scaling

To scale your application:

1. Go to your service in the Render dashboard
2. Click "Manual Scaling"
3. Adjust the number of instances as needed

## Backups

For production databases:

1. Go to your database in the Render dashboard
2. Click "Backups"
3. Enable automatic backups and set a schedule

## Troubleshooting

### Common Issues

1. Build Failures

    - Check the build logs for specific error messages
    - Ensure all dependencies are in `requirements.txt` or `requirements/production.txt`

2. Database Connection Issues

    - Verify `DATABASE_URL` is correctly set
    - Check if the database is running and accessible

3. Static Files Not Loading:
    - Ensure `whitenoise` is properly configured
    - Check `STATIC_URL` and `STATIC_ROOT` settings

4. Environment Variables:
    - Double-check all required environment variables are set
    - Ensure no trailing spaces in variable values

## Next Steps

- Set up a CI/CD pipeline with GitHub Actions
- Configure monitoring and alerting
- Set up a staging environment
- Implement automated backups

## Need Help?

- [Render Documentation](https://render.com/docs)
- [Django on Render](https://render.com/docs/deploy-django)
- [Contact Render Support](https://render.com/contact)

This deployment guide provides a comprehensive walkthrough for deploying your Django application to Render.com, covering everything from initial setup to troubleshooting common issues.

---
