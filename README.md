# Spring PetClinic Sample Application

# Ansible Docker Deployment

Automated deployment of a Spring Boot application with PostgreSQL database using Ansible and Docker.

## Overview

This project provides Ansible playbooks and roles to:
- Build a Docker image from a Git repository
- Deploy a PostgreSQL database container
- Deploy a Spring Boot application container
- Configure networking and health checks

## Architecture

- **Spring Boot Application**: Runs on port 8080, connects to PostgreSQL
- **PostgreSQL Database**: Runs on port 5432 with persistent storage
- **Docker Network**: Both containers communicate via `app_network`

## Project Structure

```
├── ansible/
│   ├── inventory/
│   │   ├── hosts.yml            # Inventory configuration
│   │   └── vault.yml            # Encrypted credentials
│   ├── roles/
│   │   ├── build/
│   │   │   ├── defaults/main.yml    # Build configuration
│   │   │   └── tasks/main.yml       # Build tasks
│   │   ├── db/
│   │   │   ├── defaults/main.yml    # Database configuration
│   │   │   ├── tasks/main.yml       # Database deployment
│   │   │   └── handlers/main.yml    # Database verification
│   │   └── app/
│   │       ├── defaults/main.yml    # App configuration
│   │       ├── tasks/main.yml       # App deployment
│   │       └── handlers/main.yml    # App health checks
│   ├── .vault_pass              # Vault password file
│   └── playbook.yml
├── app/                         # Spring Boot application code
└── .gitignore
```

## Prerequisites

- Ansible 2.9+
- Target server with:
  - Docker installed
  - Python 3
  - SSH access

## Configuration

### Ansible Vault

Create an Ansible vault file to store sensitive credentials:

```bash
ansible-vault create vault.yml
```

Add the following variables:

```yaml
vault_docker_username: "your_dockerhub_username"
vault_docker_password: "your_dockerhub_password"
vault_db_user: "postgres_user"
vault_db_password: "secure_password"
vault_db_name: "petclinic"
```

### Inventory

Update `inventory/hosts.yaml` with your target server:

```yaml
all:
  hosts:
    devserver:
      ansible_host: "{{ host_ip }}"
      ansible_user: "{{ host_user_name }}"
      ansible_password: "{{ host_user_password }}"
```

### Variables

Key variables in `roles/*/defaults/main.yml`:

**Build Role:**
- `docker_image_name`: Docker Hub repository name
- `git_repo_url`: Source code repository URL

**Database Role:**
- `db_image`: PostgreSQL Docker image (default: postgres:17)
- `db_port`: Database port (default: 5432)

**App Role:**
- `app_port`: Application port (default: 8080)
- `app_image`: Built Docker image from build role

## Usage

### Run Complete Deployment

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass -kK
```

### Run Specific Roles

Build image only:

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass -kK --tags build
```

Deploy database only:

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass -kK --tags db
```

Deploy application only:

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass -kK --tags app
```

## Roles Description

### Build Role
- Clones Git repository to a timestamped directory
- Builds Docker image with date-time tag
- Stores built image name for app deployment

### Database Role
- Creates Docker network
- Deploys PostgreSQL container with persistent storage
- Configures database credentials from vault
- Verifies container is running

### App Role
- Pulls built Docker image
- Deploys Spring Boot application container
- Configures environment variables for database connection
- Performs health checks with retries

## Health Checks

The deployment includes automatic health verification:
- **Database**: Checks container is running via `docker ps`
- **Application**:
  - Verifies container is running
  - HTTP health check on port 8080 with 3 retries

## Troubleshooting

**Container not starting:**

```bash
docker logs <container_name>
```

**Network issues:**

```bash
docker network inspect app_network
```

**Check running containers:**

```bash
docker ps -a
```

**View Ansible output:**

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml -vvv --ask-vault-pass -kK
```

## Security Notes

- All sensitive credentials stored in Ansible vault
- Docker credentials used for private registry access
- Database passwords encrypted at rest

## License

MIT

## Author

Fidan Karimova
