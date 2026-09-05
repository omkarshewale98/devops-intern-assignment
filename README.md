DevOps Intern Assignment — Azure VM + GitHub Actions
Project Overview

This project deploys a static HTML web page to an Ubuntu Azure Virtual Machine using Nginx and GitHub Actions CI/CD.

The goal is that changes merged into the main branch are automatically built and deployed to the Azure VM without manually copying files to the server.

Architecture
Developer
    |
    | Pull Request
    v
GitHub Repository
    |
    v
GitHub Actions - CI
    |
    | Build / Validate
    v
Pull Request
    |
    | Merge to main
    v
GitHub Actions - CD
    |
    | SSH / SCP
    v
Azure Ubuntu VM
    |
    v
/var/www/app
    |
    v
Nginx
    |
    v
Public URL

Application

The application is a simple static HTML page.

The main application file is:

index.html


Nginx serves the application from:

/var/www/app

Azure VM

The VM runs:

Ubuntu 22.04
Azure B1s VM size
Nginx
SSH key authentication
HTTP on port 80

The Azure Network Security Group allows:

TCP port 22 — SSH
TCP port 80 — HTTP

Password-based SSH authentication is disabled.

Nginx Configuration

Nginx serves the application from:

/var/www/app


The website is available at:

http://20.198.0.6

Repository Structure
.
├── index.html
├── README.md
└── .github/
    └── workflows/
        ├── ci.yml
        └── cd.yml

CI Pipeline

The CI workflow runs on pull requests targeting main.

It validates the application before the pull request can be merged.

For the static HTML application, the CI workflow performs HTML validation/linting.

If the validation fails, the GitHub Actions job fails and the pull request should not be merged.

CD Pipeline

The CD workflow runs when changes are pushed to main.

The workflow:

Checks out the repository.
Builds/validates the application.
Connects to the Azure VM using SSH.
Copies the website files to /var/www/app.
Reloads Nginx.
Performs a post-deployment health check.

Sensitive deployment information is stored in GitHub Secrets and is not committed to the repository.

Expected secrets:

VM_HOST
VM_USER
VM_SSH_KEY

Branch Protection

The main branch is protected.

Pull requests are required before changes can be merged, and the CI workflow must pass before merging.

Deployment Process

The deployment process is:

Create branch
    ↓
Make change
    ↓
Push branch
    ↓
Open Pull Request
    ↓
CI runs
    ↓
CI passes
    ↓
Merge Pull Request
    ↓
CD runs automatically
    ↓
Files copied to Azure VM
    ↓
Nginx reloaded
    ↓
Website updated

Rollback

The deployment can be rolled back by restoring the previous application version to:

/var/www/app


If the bonus release-management implementation is enabled, the previous release can be selected through the release symlink and Nginx can be reloaded.

What Broke Along the Way

During the setup, Nginx initially displayed its default welcome page instead of the custom application page.

The issue was that the default Nginx site was still enabled alongside the custom app site.

Removing the default site and reloading Nginx fixed the issue:

sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx


After the change, Nginx correctly served:

/var/www/app/index.html

What I Would Do Differently

For a production deployment, I would use a dedicated low-privilege deployment user instead of deploying directly with the normal VM user.

I would also keep multiple releases on the VM and use a symbolic link for atomic deployments and easy rollback.

HTTPS with Let's Encrypt and an automated post-deployment health check would also improve the deployment.