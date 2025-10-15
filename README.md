# Frappe CI/CD - Multi-Site Deployment

Automated build and deployment pipeline for Frappe/ERPNext with custom apps support. Deploy multiple isolated Frappe sites on a single server using GitHub Actions and Docker Compose.

## 🚀 Features

- ✅ Build custom Frappe images with your apps
- ✅ Deploy multiple independent Frappe sites
- ✅ Each site has isolated database, Redis, and volumes
- ✅ Easy site management with Docker Compose
- ✅ Automatic health checks and deployment verification

## 📋 Prerequisites

- GitHub account
- Docker and Docker Compose installed on your server
- Self-hosted GitHub Actions runner configured
- Docker Hub account (for storing images)

## 🛠️ Setup Instructions

### Step 1: Fork the Repository

1. Click the **Fork** button at the top right of this repository
2. Clone your forked repository to your local machine

### Step 2: Configure Custom Apps

Edit the `apps.json` file to include your custom Frappe apps. For example:

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/your-org/your-custom-app",
    "branch": "main"
  }
]
```

**Commit and push your changes:**
```bash
git add apps.json
git commit -m "Add custom apps configuration"
git push origin main
```

### Step 3: Configure GitHub Secrets

Add your Docker Hub credentials to GitHub Secrets:

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password/token

### Step 4: Setup Self-Hosted Runner

Configure a self-hosted GitHub Actions runner on your server:

1. Go to **Settings** → **Actions** → **Runners**
2. Click **New self-hosted runner**
3. Follow the instructions to install and configure the runner on your server:

  Note -> instead of ./run.sh. You have to run
  ```bash
  ./run.sh &
  ```

4. Verify the runner shows as **Active** in GitHub

## 🚢 Deploy a Frappe Site

### Step 1: Trigger the Workflow

1. Go to **Actions** tab in your repository
2. Select **Build and Deploy Frappe Custom App** workflow
3. Click **Run workflow**

### Step 2: Provide Deployment Parameters
Note -> If you are installing multiple apps then u have to provide argument using the syntax without any space-> 

```
  app1,app2,app3
```

### Step 3: Access Your Site

After deployment completes (5-10 minutes), access your site:

```
http://localhost:<port>
```

**Default credentials:**
- Username: `Administrator`
- Password: `admin` (or your custom `admin_password`)

## 🔄 Deploy Multiple Sites

You can deploy multiple independent Frappe sites on the same server:

**Example: Deploy 3 sites**

| Site Name | Port | Apps |
|-----------|------|------|
| `development.local` | `8080` | `frappe,erpnext` |
| `staging.local` | `8081` | `frappe,erpnext,hrms` |
| `production.local` | `8082` | `frappe,erpnext,hrms,insights` |

Each site runs in complete isolation with:
- ✅ Separate database
- ✅ Separate Redis instances
- ✅ Separate volumes and data
- ✅ Independent scaling and management


