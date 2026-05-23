# ArgoCD Complete Guide

## Table of Contents

1. [Introduction to ArgoCD](#introduction-to-argocd)
2. [GitOps Fundamentals](#gitops-fundamentals)
3. [ArgoCD Features](#argocd-features)
4. [ArgoCD Architecture](#argocd-architecture)
5. [Installation](#installation)
6. [Application Management](#application-management)
7. [Sync Policies and Options](#sync-policies-and-options)
8. [Projects and Access Control](#projects-and-access-control)
9. [Best Practices](#best-practices)
10. [User Management](#user-management)

---

## Introduction to ArgoCD

ArgoCD is a declarative, GitOps continuous delivery tool specifically designed for Kubernetes environments. It helps organizations automate and manage their application deployments with several powerful capabilities.

At its core, ArgoCD synchronizes your applications with their desired state as defined in Git repositories. This means any changes to your application configuration in Git are automatically reflected in your Kubernetes clusters.

### Key Features

ArgoCD offers four key features that make it particularly valuable:

1. **Automated Deployment Capabilities**: Ensures your applications stay synchronized with the configurations stored in your Git repositories.
2. **Easy Rollback Functionality**: Allows quick recovery if something goes wrong during deployment.
3. **Multi-Cluster Deployments**: Enables you to manage applications across multiple Kubernetes clusters from a single interface.
4. **Comprehensive Visualization**: Provides a rich UI that makes it easier to monitor and manage your applications.

---

## GitOps Fundamentals

GitOps represents a modern approach to managing and automating IT infrastructure and application delivery. The core concept revolves around using Git as the single source of truth for both infrastructure and application code.

### Core Principles

The methodology is built on three core principles:

1. **Declarative Configuration**: Defines the desired state of the system
2. **Version Control**: Tracks all changes and enables rollbacks when needed
3. **Reconciliation**: Ensures the actual state matches the desired state

### Key Benefits

GitOps offers three key benefits:

- **Consistency**: In deployment and management across environments
- **Traceability**: Of all changes through Git's version control system
- **Enhanced Collaboration**: Among team members working on infrastructure and applications

### GitOps Workflow

The GitOps workflow consists of two main models:

#### Push-based Model

In this model, developers push changes directly to the Git repository, which then triggers corresponding changes in the infrastructure or application. This approach is designed to simplify automation and maintain consistency across different environments.

#### Pull and Reconciliation

Automation tools continuously monitor the Git repository, pulling updates and reconciling the actual system state with the declared state in the repository. This automated synchronization process:

- Reduces the need for manual intervention
- Helps prevent configuration drift
- Maintains system consistency

Together, these two models create a streamlined workflow that maintains system consistency and reduces manual overhead.

---

## ArgoCD Features

ArgoCD offers a comprehensive set of features built around GitOps principles and Kubernetes management.

### Core Capabilities

- **GitOps-centric Operations**: Declarative configuration management and first-class Kubernetes support
- **Monitoring**: Health status analysis and self-healing mechanisms
- **Multiple Interfaces**: Rich UI dashboard, CLI tools, and API access
- **Flexible Synchronization**: Supports both automated and manual sync with pre-sync and post-sync hooks
- **Enterprise Features**: Multi-tenancy support and role-based access control (RBAC)
- **Integration Capabilities**: Custom resource definitions (CRDs) and support for multiple configuration management tools
- **Reliability**: High availability and scalability features with notifications and alerts
- **Extensibility**: Custom plugins and flexible integration options through API

### Key Differentiators

ArgoCD distinguishes itself through four main areas:

1. **Argo CD Interactions**: Unique deployment workflow enhancements
2. **Scalability and High Availability**: Reliable operation at enterprise scale
3. **Security**: Trustworthy choice for organizations with strict security requirements
4. **Customization and Extensibility**: Adaptable to specific needs and workflows

---

## ArgoCD Architecture

### Component Overview

ArgoCD consists of several core components that work together to provide a complete GitOps solution:

### 1. API Server

The API Server is the core interaction layer for ArgoCD, responsible for exposing its capabilities to users and other tools.

#### Key Responsibilities

- **User Interaction**: Serves as the entry point for the ArgoCD Web UI, CLI, and API requests
- **Interface Management**: Provides an interface for managing applications, clusters, and settings
- **Authentication & Authorization**: Manages access control through RBAC policies and authentication mechanisms like OAuth2 or SSO
- **Command-Line Interface (CLI)**: Facilitates automation and scripting by providing access to ArgoCD features via CLI commands

The API Server acts as a bridge between the user (or external tools) and the internal components of ArgoCD.

### 2. Application Controller

The Application Controller is a key component responsible for the continuous state reconciliation of applications.

#### Key Responsibilities

- **Lifecycle Management**: Continuously monitors the application state as defined in the Git repository and deploys, updates, or rolls back applications in the Kubernetes cluster
- **State Reconciliation**: Checks whether the actual state in the Kubernetes cluster matches the desired state defined in Git and triggers actions when discrepancies are found
- **Health Monitoring**: Tracks the health of applications and Kubernetes resources to ensure deployments succeed and the system remains stable

The Application Controller automates much of the heavy lifting, making ArgoCD a true implementation of GitOps.

### 3. Repo Server

The Repo Server is responsible for interfacing with Git repositories to retrieve and process application configurations.

#### Key Responsibilities

- **Git Operations**: Clones Git repositories securely using SSH or HTTPS and monitors changes in specified branches or tags
- **Manifest Generation**: Processes Kubernetes manifests and templates (e.g., Helm, Kustomize, or plain YAML) and resolves configurations for deployment
- **Caching**: Caches Git data locally to reduce network latency and improve performance for large repositories
- **Security**: Handles sensitive information such as repository credentials, ensuring secure access

The Repo Server is vital for maintaining the connection between ArgoCD and the Git repositories that act as the single source of truth for your Kubernetes applications.

### 4. Dex (Optional)

Dex is an open-source identity provider often used for authentication in Kubernetes environments, including ArgoCD.

#### Key Features

- **External Identity Provider**: Acts as a bridge to external identity providers:
  - LDAP
  - Active Directory (AD)
  - Google OAuth
  - GitHub OAuth
- **Security Integration**: Provides seamless integration with identity and access management (IAM) systems
- **Enhanced User Experience**: Simplifies user management by delegating authentication to an external provider

Dex enables single sign-on (SSO) capabilities for ArgoCD and reduces the need to manually create and manage users.

### 5. Redis (Optional)

Redis is an in-memory data store that is used to enhance performance in systems like ArgoCD.

#### Key Features

- **Performance Boost**: Provides caching for frequently accessed data, reducing the load on other components
- **Scalability**: Helps handle high workloads in large-scale deployments
- **Reduces Latency**: Speeds up operations like fetching application status or logs during synchronization

While Redis is not required for smaller setups, it is highly recommended for larger, enterprise-grade deployments.

### Component Interaction Flow

1. **GitOps Workflow**:
   - Developers push changes to the Git repository
   - The Repo Server pulls the updated manifests or configurations

2. **Application Controller**:
   - Compares the desired state (from Git) with the actual state (in Kubernetes)
   - Ensures synchronization by applying changes as needed

3. **API Server**:
   - Exposes these processes to users via the Web UI, CLI, or APIs
   - Provides status information and control options (e.g., sync, rollback)

### When to Use Optional Components

- **Dex**: Used when there is a need for SSO or integration with enterprise-grade identity providers
- **Redis**: Improves responsiveness and ensures ArgoCD remains efficient and scalable as the number of applications and users increases

For smaller deployments, these components can be skipped without impacting core functionality.

---

## Installation

### Prerequisites

- Kubernetes cluster (version 1.18 or higher recommended)
- `kubectl` configured to access your cluster
- Cluster admin permissions

### ArgoCD Installation

1. **Create ArgoCD Namespace**:
   ```bash
   kubectl create namespace argocd
   ```

2. **Install ArgoCD**:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Verify Installation**:
   ```bash
   kubectl get all --namespace=argocd
   ```

4. **Expose ArgoCD Server (NodePort)**:
   ```bash
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
   ```

   Alternatively, for LoadBalancer:
   ```bash
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
   ```

### ArgoCD CLI Installation

#### Linux

```bash
# Download the latest ArgoCD CLI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

# Install
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

# Cleanup
rm argocd-linux-amd64
```

#### Windows (PowerShell)

```powershell
# Get latest version
$version = (Invoke-RestMethod https://api.github.com/repos/argoproj/argo-cd/releases/latest).tag_name

# Download
$url = "https://github.com/argoproj/argo-cd/releases/download/" + $version + "/argocd-windows-amd64.exe"
$output = "argocd.exe"
Invoke-WebRequest -Uri $url -OutFile $output

# Add to PATH (optional)
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\Path\To\ArgoCD-CLI", "User")
```

#### macOS

```bash
# Using Homebrew
brew install argocd

# Or manual installation
curl -sSL -o argocd-darwin-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-darwin-amd64
sudo install -m 555 argocd-darwin-amd64 /usr/local/bin/argocd
rm argocd-darwin-amd64
```

### Initial Setup and Login

1. **Get Initial Admin Password**:
   ```bash
   # Method 1: Using kubectl
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
   
   # Method 2: Using ArgoCD CLI
   argocd admin initial-password -n argocd
   ```

2. **Login to ArgoCD**:
   ```bash
   # Get the NodePort or LoadBalancer address
   kubectl get svc argocd-server -n argocd
   
   # Login (default username: admin)
   argocd login <IP>:<NodePort>
   # Or if using LoadBalancer
   argocd login <LoadBalancer-IP>
   ```

3. **Verify Access**:
   ```bash
   argocd cluster list
   argocd app list
   ```

4. **Logout**:
   ```bash
   argocd logout <IP>:<NodePort>
   ```

### Accessing the Web UI

After exposing the ArgoCD server, you can access the Web UI:

- **NodePort**: `http://<Node-IP>:<NodePort>`
- **LoadBalancer**: `http://<LoadBalancer-IP>`

Use the username `admin` and the password retrieved from the initial admin secret.

---

## Application Management

### Understanding Applications in ArgoCD

ArgoCD Applications are declarative definitions that describe the desired state of applications in a Git repository, including:

- **Source Repository**: Where your Kubernetes manifests are stored (e.g., Git)
- **Path**: The directory within the repository where manifests are located
- **Destination Cluster**: The Kubernetes cluster where the application will be deployed
- **Destination Namespace**: The namespace in the cluster where resources will reside

### Application Features

- **Management**: Managed through the ArgoCD CLI or UI, allowing operations such as creating, listing, syncing, and deleting applications
- **Sync Policy**: Applications can be configured with a sync policy to automatically apply changes from the Git repository to the cluster
- **Health Status**: ArgoCD continually monitors applications, providing a health status that reflects the operational state of the application resources
- **Rollback and History**: ArgoCD stores a history of deployment states, enabling easy rollback to previous versions if needed
- **Multi-cluster Support**: Applications can be deployed across multiple clusters, allowing for centralized management

### Creating Applications

#### Method 1: Using ArgoCD CLI

```bash
# Login to ArgoCD
argocd login <IP>:<NodePort>

# Create an application
argocd app create <app-name> \
  --repo <repository-url> \
  --path <path-in-repo> \
  --dest-server <cluster-url> \
  --dest-namespace <namespace> \
  --revision <branch-or-tag>

# List applications
argocd app list

# Get application details
argocd app get <app-name>

# Sync application
argocd app sync <app-name>

# Delete application
argocd app delete <app-name>
```

#### Method 2: Using YAML Manifest

Create an `application.yaml` file:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-repo.git
    targetRevision: main
    path: apps/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

Apply the manifest:

```bash
kubectl apply -f application.yaml
```

### Deployment Strategies

ArgoCD supports various deployment strategies:

- **Automated Sync**: Automatically apply changes from the repository
- **Manual Sync**: Manually trigger the synchronization of the application

---

## Sync Policies and Options

### Sync Policy Configuration

ArgoCD provides various sync policies and options to control how applications are synchronized with the Git repository.

### CreateNamespace

The `CreateNamespace=true` option allows ArgoCD to create the specified namespace if it does not already exist.

**Example**:
```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
```

### Self-Healing (selfHeal)

Self-healing ensures that any manual changes made to the resources in the cluster are brought back to the state defined in Git.

**Example Configuration**:
```yaml
syncPolicy:
  automated:
    selfHeal: true
```

**Behavior**:
- With `selfHeal: true`: Manual changes are automatically reverted to match Git state
- With `selfHeal: false`: Manual changes remain, but the application shows as "OutOfSync"

### Prune

Prune deletes resources in the cluster that are no longer present in the Git repository.

**Example Configuration**:
```yaml
syncPolicy:
  automated:
    prune: true
```

**Behavior**:
- With `prune: true`: Resources removed from Git are deleted from the cluster
- With `prune: false`: Resources removed from Git remain in the cluster, but the application shows as "OutOfSync"

### Prune Propagation Policies

Prune Propagation Policies determine how resources should be pruned (deleted) when they are no longer part of the desired state.

#### 1. Foreground

Ensures that child resources are deleted before the parent resource is deleted.

**Use Case**: When you want to ensure that child resources are cleaned up first before the parent resource is deleted, preventing orphaned resources.

**Example**:
```yaml
syncPolicy:
  syncOptions:
    - PrunePropagationPolicy=foreground
```

#### 2. Background

Allows the parent resource to be deleted first, with child resources pruned afterward.

**Use Case**: When you want to ensure primary resources are deleted immediately, with associated resources cleaned up afterward.

**Example**:
```yaml
syncPolicy:
  syncOptions:
    - PrunePropagationPolicy=background
```

#### 3. Orphaned

Only orphaned resources (those not controlled by any other resource) are pruned.

**Use Case**: When you want to preserve resources that are still in use by other resources, removing only truly orphaned resources.

**Example**:
```yaml
syncPolicy:
  syncOptions:
    - PrunePropagationPolicy=orphan
```

### Prune Last

Prune Last ensures that resources are deleted last in the synchronization process, after all other resources have been handled.

**Example**:
```yaml
syncPolicy:
  syncOptions:
    - PruneLast=true
```

**Benefits**:
- Prevents pruning from interfering with synchronization
- Ensures necessary resources are updated before deletions occur
- Minimizes risk of resource inconsistencies during sync

### Dry Run

Dry run allows you to preview changes before applying them without making actual modifications to the cluster.

```bash
# Preview differences
argocd app diff <app-name>

# Preview sync without applying
argocd app sync <app-name> --dry-run
```

### Apply Only

Apply Only applies Kubernetes manifests without deleting unmanaged resources in the cluster. Useful when you don't want ArgoCD to prune resources not present in Git.

```bash
argocd app sync <app-name> --apply-only
```

**Use Case**: When Kubernetes resources (e.g., Secrets, ConfigMaps, CRDs) are created manually but should not be removed by ArgoCD.

**Example**: If a ConfigMap is commented out in Git and prune is disabled with apply-only enabled, the resource will remain in the cluster but show as "OutOfSync".

### Force Sync

Force Sync bypasses usual safeguards when syncing a resource, making ArgoCD apply changes even if they conflict with the current state.

```bash
argocd app sync <app-name> --force
```

**Use Cases**:
- Overriding manual modifications made outside ArgoCD
- Forcing reconciliation when resources are out of sync
- Preventing configuration drift in environments where immediate correction is needed

### Refresh and Hard Refresh

#### Refresh

A Refresh operation updates the application's state from the cluster without reapplying any changes.

```bash
argocd app refresh <app-name>
```

**Use Cases**:
- Syncing the ArgoCD UI with the current state of resources
- Refreshing status after manual interventions outside ArgoCD

#### Hard Refresh

A Hard Refresh fetches the current state and reconciles resources with the desired state, forcing a resync even if no changes are detected.

```bash
argocd app sync <app-name> --hard-refresh
```

**Use Cases**:
- Forcing ArgoCD to check and resync resources when synchronization state is in doubt
- Reconciling when application state has diverged from Git repository

### Apply Out of Sync

Apply Out of Sync applies changes to resources that are out of sync with the Git repository state.

**Use Cases**:
- When resources have been manually modified and need to be overwritten with Git version
- When resources have drifted due to external tools or Kubernetes events

### Server-Side Apply

Server-side apply (available by default in ArgoCD 2.1+) allows Kubernetes to handle resource updates in a more declarative way.

**Benefits**:
- Better conflict resolution when multiple clients modify the same resource
- More efficient resource management
- Declarative merging handled by Kubernetes API server

**Requirements**:
- Kubernetes cluster version 1.18 or higher
- ArgoCD version 2.1 or higher (enabled by default)

### Skip Schema Validation

Skip schema validation bypasses schema validation during resource synchronization. Useful when working with custom resources that might not have complete schemas.

**Use Case**: Working with certain custom resources that might not have a complete schema or when the schema is not fully compatible with validation rules.

### Respect Ignore Differences

Respect Ignore Differences determines whether ArgoCD should respect the `ignoreDifferences` configuration during synchronization.

**Example Configuration**:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: https://github.com/my-repo
    targetRevision: HEAD
    path: apps/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  ignoreDifferences:
    - kind: ConfigMap
      name: my-configmap
      namespace: default
      jsonPointers:
        - /data
    - kind: Secret
      name: my-secret
      namespace: default
      jsonPointers:
        - /data
```

**Use Cases**:
- Ignoring automatically generated timestamps
- Ignoring status fields that are dynamically updated by Kubernetes controllers
- Ignoring fields updated by external systems that shouldn't trigger resync

### Replace Sync Option

Replace sync option forces ArgoCD to delete and recreate resources during sync, rather than updating them in place.

```bash
argocd app sync <app-name> --replace
```

**Use Cases**:
- Corrupted or inconsistent resources that cannot be fixed by simple updates
- Resource modifications that require recreation (e.g., certain Pod or Deployment fields)
- Ensuring clean creation from scratch

**Considerations**:
- May cause downtime or service interruptions
- Can result in state loss for certain resource types
- Should be used carefully, especially for stateful applications

### Retry

Retry allows you to retry a failed sync operation without manual intervention.

**Using UI**:
1. Navigate to the Applications page
2. Find the application with failed sync
3. Click the Sync button
4. Click Retry or Sync again

**Using CLI**:
```bash
argocd app sync <app-name>
```

**Use Case**: When an operation fails due to temporary issues (network problems, API server unavailability, unavailable container images).

### Rollback

ArgoCD stores a history of deployment states, enabling easy rollback to previous versions.

```bash
# View application history
argocd app history <app-name>

# Rollback to a specific revision
argocd app rollback <app-name> <revision>
```

The UI also provides rollback functionality with commit IDs for easy version identification.

---

## Projects and Access Control

### Projects

Projects in ArgoCD provide a way to organize applications and enforce restrictions on source repositories, destination clusters, and namespaces.

#### Creating a Project

Projects can be created through the UI or using YAML manifests.

**Example Project Manifest**:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: My Project Description
  sourceRepos:
    - 'https://github.com/my-org/my-repo.git'
  destinations:
    - namespace: 'my-namespace'
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
```

#### Project Restrictions

**Repository Restriction**: Limit projects to specific source repositories

```yaml
spec:
  sourceRepos:
    - 'https://github.com/my-org/allowed-repo.git'
```

**Namespace Restriction**: Restrict projects to specific destination namespaces

```yaml
spec:
  destinations:
    - namespace: 'allowed-namespace'
      server: https://kubernetes.default.svc
```

**Behavior**: If an application tries to use a repository or namespace not allowed by the project, ArgoCD will reject it with an appropriate error message.

### Adding Clusters

ArgoCD can manage applications across multiple Kubernetes clusters.

```bash
# Login to ArgoCD
argocd login <IP>:<NodePort>

# List available contexts
kubectl config get-contexts

# Add a cluster (EKS example)
argocd cluster add <context-name>

# Or add using cluster ARN (EKS)
argocd cluster add arn:aws:eks:ap-south-1:891543987898:cluster/eks

# List registered clusters
argocd cluster list
```

### Sync Windows

Sync Windows provide controlled and scheduled deployment capabilities.

#### Benefits

1. **Controlled Deployments**: Plan and schedule deployments during specific times (off-peak hours, maintenance windows)
2. **Enhanced Compliance**: Ensure system changes occur only during approved timeframes
3. **Better Resource Management**: Avoid system overload during high-traffic periods
4. **Automated Deployments with Oversight**: Prevent deployments at inappropriate times

#### Configuration Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncWindows:
    - kind: allow
      schedule: '10 2 * * *'  # Allow sync at 2:10 AM daily
      duration: 1h
      applications:
        - '*'
      manualSync: true
    - kind: deny
      schedule: '0 9 * * 1-5'  # Deny sync 9 AM-5 PM on weekdays
      duration: 8h
```

### Roles and RBAC

Roles provide granular access control in ArgoCD. They can be configured for:

- Integrating with CI/CD tools (Jenkins, GitLab CI, etc.)
- API access using tokens
- CLI access
- Third-party tool integration

**Configuration**:
- Roles are configured in the ArgoCD ConfigMap and RBAC ConfigMap
- Policies follow a CSV format for defining permissions

---

## Best Practices

### 1. Repository Management

#### Use a Dedicated Git Repository for Manifests

Separate application code from Kubernetes manifests:

- **App Code Repo**: `github.com/my-org/my-app`
- **Manifests Repo**: `github.com/my-org/my-app-config`

#### Use Branches and Tags for Version Control

**Branching Strategy**:
- `main` → Stable releases
- `develop` → Pre-release testing
- `feature/*` → Feature branches

**Tagging**:
- Use semantic versioning (e.g., `v1.0.0`) for easier rollbacks

#### Organize Repository Structure

For multi-app repositories, structure them as follows:

```
my-gitops-repo/
├── apps/
│   ├── app1/
│   │   ├── base/
│   │   └── kustomization.yaml
│   ├── app2/
│   │   ├── base/
│   │   └── kustomization.yaml
│   └── app3/
├── clusters/
│   ├── production/
│   ├── staging/
│   └── development/
└── README.md
```

#### Secure Access to Repositories

- Use SSH keys or Personal Access Tokens (PATs) instead of HTTP(S) passwords
- Limit permissions (use read-only access for ArgoCD)
- Enable branch protection to prevent direct commits to main

### 2. Application Definition

#### Define Applications as Code

Manage applications using YAML manifests (Declarative Approach):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: tetris
  namespace: argocd
spec:
  destination:
    namespace: default
    server: 'https://kubernetes.default.svc'
  source:
    path: manifests
    repoURL: 'https://github.com/OpqTech/tetris.git'
    targetRevision: argo-cd
  project: default
  syncPolicy:
    automated:
      prune: false
      selfHeal: false
    syncOptions:
      - CreateNamespace=true   
```

### 3. Use Helm or Kustomize for Customization

#### Kustomize Example

```yaml
# kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml

patches:
  - path: patch-deployment.yaml
```

#### Helm Chart Reference

```yaml
source:
  chart: my-helm-chart
  repoURL: https://charts.my-org.com
  targetRevision: 1.2.3
```

### 4. Implement Automated Sync Policies Carefully

```yaml
syncPolicy:
  automated:
    prune: true     # Auto-remove deleted resources
    selfHeal: true  # Auto-fix drift
```

**Recommendation**: Use manual sync for production to avoid accidental rollouts.

### 5. Use Multiple Repositories for Multi-Tenant Architectures

Separate repositories per team, environment, or tenant:

- `github.com/my-org/tenant-a-config`
- `github.com/my-org/tenant-b-config`

### 6. Monitor and Audit Changes

- Enable ArgoCD notifications (`argocd-notifications`)
- Use GitHub Actions or GitLab CI to validate YAMLs before merging
- Set up alerts for sync failures and health issues

### 7. Regularly Clean Up Old Resources

- Remove unused applications and repositories from ArgoCD
- Archive old Helm charts or Kubernetes manifests
- Review and clean up old sync history

### 8. Security Best Practices

- Use RBAC to enforce least-privilege access
- Enable audit logging
- Use projects to isolate applications
- Regularly rotate credentials and tokens
- Enable OIDC/SSO for authentication in production

### 9. Performance Optimization

- Use Redis for caching in large deployments
- Enable horizontal scaling of API Server and Application Controller
- Implement sync windows to reduce cluster load
- Use appropriate sync intervals

### 10. Disaster Recovery

- Backup ArgoCD configuration and applications
- Document rollback procedures
- Test disaster recovery scenarios regularly
- Maintain multiple cluster registrations for redundancy

---

## User Management

### Adding Users

Users can be added to ArgoCD through configuration files.

#### Step 1: Edit ConfigMap to Add User Account

```bash
kubectl edit configmap argocd-cm -n argocd
```

Add the user account in the `data` section:

```yaml
data:
  accounts.kumar: apiKey,login
```

Where:
- `kumar` is the username
- `apiKey,login` grants API key generation and login capabilities

#### Step 2: Configure RBAC Policy

```bash
kubectl edit configmap argocd-rbac-cm -n argocd
```

Add RBAC policy:

```yaml
data:
  policy.csv: |
    g, kumar, role:admin
  scopes: '[groups]'
```

**Policy Format**:
- `g, <user>, <role>`: Grant role to user
- Available roles: `role:admin`, `role:readonly`, custom roles

#### Step 3: Set Initial Password

For new users, set an initial password:

```bash
# Get admin password as reference (if needed)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

# Or set password for specific user via CLI (after login as admin)
argocd account update-password --account kumar
```

### User Roles

#### Built-in Roles

- **admin**: Full access to all resources
- **readonly**: Read-only access to view applications and resources

#### Custom Roles

Custom roles can be defined in the RBAC ConfigMap:

```yaml
data:
  policy.csv: |
    p, role:developer, applications, get, */*, allow
    p, role:developer, applications, sync, */*, allow
    g, developer-user, role:developer
```

### API Keys

Generate API keys for programmatic access:

```bash
# Generate API key for a user
argocd account generate-token --account kumar

# List tokens
argocd account list

# Delete a token
argocd account delete-token --account kumar <token-id>
```

### Authentication Methods

ArgoCD supports various authentication methods:

1. **Local Accounts**: Username/password stored in ArgoCD
2. **OIDC**: OpenID Connect integration
3. **Dex Integration**: External identity providers (LDAP, AD, SAML, GitHub, etc.)
4. **GitHub/GitLab**: OAuth integration

---

## Additional Resources

### Official Documentation

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [GitOps Principles](https://www.gitops.tech/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

### Community

- [ArgoCD GitHub](https://github.com/argoproj/argo-cd)
- [ArgoCD Slack](https://argoproj.github.io/community/join-slack)
- [ArgoCD Discussions](https://github.com/argoproj/argo-cd/discussions)

### Learning Resources

- ArgoCD tutorials and examples
- GitOps best practices guides
- Kubernetes deployment patterns

---

## Conclusion

ArgoCD provides a powerful, declarative GitOps solution for Kubernetes deployments. By following best practices and understanding its architecture and features, you can effectively manage your applications across multiple clusters with confidence, security, and reliability.

Remember to:
- Keep Git as the single source of truth
- Use appropriate sync policies for your environment
- Implement proper access controls and security measures
- Monitor and audit your deployments regularly
- Follow the principle of least privilege for user access

---

*Last Updated: [Current Date]*
*ArgoCD Version: Latest Stable*

