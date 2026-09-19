# Argo CD Components

Argo CD is a declarative, GitOps continuous delivery platform for Kubernetes. It continuously compares the desired state defined in a source repository with the live state in a Kubernetes cluster, reports differences, and—when configured—reconciles the cluster automatically.

This guide explains the major Argo CD components, how they communicate, and how they work together during application deployment and reconciliation.

## 1. High-Level Architecture

```text
                         Git / Helm / OCI
                         Desired State
                                |
                                v
                       +------------------+
                       | Repository Server|
                       | Render manifests |
                       +--------+---------+
                                |
                                v
Users / Automation -----> +-----+------------+       +--------+
                          |   API Server     |<----->| Redis  |
                          | UI, CLI, auth    |       | Cache  |
                          +-----+------------+       +--------+
                                |
                                v
                       +------------------+
                       | Application      |
                       | Controller       |
                       | Compare and sync |
                       +--------+---------+
                                |
                                v
                       Kubernetes API Server
                                |
                                v
                    Deployments, Services, Pods,
                    ConfigMaps, and other resources
```

The core reconciliation path is:

1. A source repository stores the desired configuration.
2. The Repository Server fetches the source and renders Kubernetes manifests.
3. The Application Controller compares rendered manifests with live resources.
4. The controller reports whether the application is `Synced` or `OutOfSync`.
5. A manual or automated sync applies the desired resources through the Kubernetes API.

## 2. Main Argo CD Components

| Component | Typical Kubernetes name | Primary responsibility |
| --- | --- | --- |
| API Server | `argocd-server` | API, UI backend, authentication, authorization, and orchestration |
| Application Controller | `argocd-application-controller` | Reconciliation, health evaluation, and synchronization |
| Repository Server | `argocd-repo-server` | Source access and manifest generation |
| Redis | `argocd-redis` | Caching and temporary application data |
| Dex | `argocd-dex-server` | Optional identity-provider integration |
| ApplicationSet Controller | `argocd-applicationset-controller` | Generates Applications from templates |
| Notifications Controller | `argocd-notifications-controller` | Sends application event notifications |
| Web UI | Served by the API Server | Graphical application management |
| CLI | `argocd` executable | Command-line access to the API |
| Application | `Application` custom resource | Describes one desired deployment |
| AppProject | `AppProject` custom resource | Defines project boundaries and policies |

The exact pods and deployment topology depend on the Argo CD version, installation method, and whether optional features are enabled.

## 3. API Server

The API Server is the central entry point for users, automation, the Argo CD CLI, and the Web UI. It is usually deployed as `argocd-server`.

### Responsibilities

The API Server:

- Exposes the Argo CD REST and gRPC APIs.
- Serves the Web UI.
- Authenticates users and issues sessions or tokens.
- Applies Argo CD RBAC rules.
- Manages Applications, AppProjects, repositories, and clusters.
- Accepts sync, rollback, and refresh requests.
- Streams application and resource updates to clients.
- Coordinates access to the Application Controller and Repository Server.

The API Server does not normally perform reconciliation itself. Reconciliation belongs to the Application Controller.

### Typical CLI flow

```text
argocd app sync my-app
          |
          v
    API Server
          |
          v
Application Controller
          |
          v
 Kubernetes API
```

The CLI generally communicates with Argo CD rather than directly modifying Kubernetes resources.

## 4. Application Controller

The Application Controller is the main reconciliation engine. It is usually deployed as `argocd-application-controller`.

For each `Application`, it repeatedly:

1. Reads the desired source and target revision.
2. Requests rendered manifests from the Repository Server.
3. Reads the corresponding live resources from the Kubernetes API.
4. Compares desired and live state.
5. Calculates sync status and resource health.
6. Updates the `Application` status.
7. Performs synchronization when requested or enabled.

### Desired state versus live state

If Git contains:

```yaml
spec:
  replicas: 5
```

but the live Deployment has three replicas, Argo CD reports:

```text
Desired replicas: 5
Live replicas:    3
Sync status:      OutOfSync
```

With automated synchronization enabled, the controller can change the Deployment back to five replicas.

### Controller responsibilities

- Detecting drift.
- Applying creates, updates, and deletes.
- Pruning resources removed from the source.
- Evaluating built-in and custom health checks.
- Respecting sync waves and resource hooks.
- Tracking resource ownership.
- Maintaining operation history and status.
- Reconciling applications in local or external clusters.

## 5. Repository Server

The Repository Server, usually `argocd-repo-server`, handles source retrieval and manifest generation. It isolates Git and configuration-management operations from the Application Controller.

### Supported source patterns

Depending on configuration and installed plugins, it can process:

- Plain Kubernetes YAML and JSON.
- Helm charts.
- Kustomize applications.
- Jsonnet.
- Git repositories.
- OCI-based sources.
- Config management plugins.

### Example source

```yaml
spec:
  source:
    repoURL: https://github.com/example/platform-config.git
    targetRevision: main
    path: apps/nginx
```

The flow is:

```text
Application Controller
          |
          | Request manifests
          v
    Repository Server
          |
          v
  Git / Helm / OCI source
          |
          v
 Rendered Kubernetes manifests
```

The Repository Server may cache repositories and generated manifests to reduce repeated network and rendering work. Cached data is not the source of truth; the configured source remains authoritative.

## 6. Redis

Redis is commonly deployed as `argocd-redis`. It improves performance by caching frequently accessed information.

Typical uses include:

- Repository and manifest caching.
- Application state caching.
- Session-related data.
- Temporary coordination data.
- Reducing repeated expensive rendering and API operations.

Redis is not the source of truth for GitOps configuration. If Redis is lost, Argo CD should be able to reconstruct its important information from Git, Kubernetes, and configuration.

For production installations, follow the Argo CD version's guidance for Redis high availability and persistence.

## 7. Dex

Dex is an optional identity service, usually deployed as `argocd-dex-server`. It allows Argo CD to use external identity providers through OAuth2 or OpenID Connect.

Possible identity sources include:

- LDAP.
- GitHub or GitLab.
- Microsoft identity services.
- SAML or OIDC-compatible providers.
- Other OAuth-compatible identity systems.

```text
User -> Argo CD UI -> Dex -> Identity Provider
                         |
                         v
                    Authenticated user
```

Dex primarily handles authentication. Authorization is controlled separately through Argo CD RBAC policies and AppProjects.

## 8. ApplicationSet Controller

The ApplicationSet Controller, usually `argocd-applicationset-controller`, creates and manages multiple `Application` resources from a reusable template.

It is useful when many applications share a deployment pattern across:

- Environments such as development, staging, and production.
- Multiple clusters.
- Teams or tenants.
- Directories in a repository.
- Pull requests or branches.

### Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: environments
spec:
  generators:
    - list:
        elements:
          - environment: dev
          - environment: staging
          - environment: production
  template:
    metadata:
      name: '{{environment}}-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/example/apps.git
        path: '{{environment}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{environment}}'
```

Common generators include List, Git, Cluster, Matrix, Merge, Pull Request, and SCM Provider generators.

## 9. Web UI

The Web UI provides graphical access to Argo CD. It communicates with the API Server and does not normally talk directly to the Kubernetes API.

Users can:

- View application health and sync status.
- Inspect resource trees and manifests.
- Review events and operation history.
- View logs, where permissions allow.
- Trigger synchronization and rollback operations.
- Inspect differences between desired and live state.
- Manage repositories, clusters, projects, and applications.

```text
Browser
   |
   v
API Server
   |
   +--> Application Controller
   +--> Repository Server
   +--> Kubernetes API
```

## 10. Argo CD CLI

The `argocd` CLI provides scriptable access to the same Argo CD APIs used by the UI.

```bash
argocd login argocd.example.com
argocd app list
argocd app get my-app
argocd app diff my-app
argocd app sync my-app
argocd app history my-app
```

The CLI is useful for CI/CD workflows, operational automation, troubleshooting, and controlled manual actions.

## 11. Application Resource

An `Application` is an Argo CD custom resource that describes one desired deployment. It specifies:

- The source repository, chart, or OCI source.
- The revision to deploy.
- The path or chart name.
- The destination cluster and namespace.
- The owning AppProject.
- Synchronization policy and options.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/nginx-config.git
    targetRevision: main
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

An Application is the link between source configuration and a Kubernetes destination:

```text
Source repository -> Application -> Kubernetes cluster
```

## 12. AppProject

An `AppProject` provides organizational and security boundaries for Applications. Projects are especially important in multi-team environments.

Projects can restrict:

- Which source repositories may be used.
- Which clusters and namespaces are valid destinations.
- Which resource kinds may be created.
- Which users or groups can manage Applications.
- Which deployment policies are allowed.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: platform
  namespace: argocd
spec:
  sourceRepos:
    - https://github.com/example/*
  destinations:
    - namespace: platform-*
      server: https://kubernetes.default.svc
```

AppProjects complement, but do not replace, Kubernetes RBAC and Argo CD RBAC.

## 13. Kubernetes API Server

Argo CD uses the Kubernetes API Server to inspect and modify live resources. The Application Controller uses it to:

- Read resources.
- Watch changes.
- Create and update resources.
- Delete resources during pruning.
- Evaluate resource health.
- Manage resources in registered external clusters.

```text
Application Controller
          |
          v
 Kubernetes API Server
          |
          +--> Deployment
          +--> Service
          +--> ConfigMap
          +--> Ingress
          +--> Pod
```

Argo CD must have suitable credentials and permissions in every destination cluster it manages.

## 14. Notifications Controller

The Notifications Controller, usually `argocd-notifications-controller`, sends messages when application events occur.

It can notify external systems about:

- Successful or failed synchronization.
- Health changes.
- Deployment failures.
- Persistent `OutOfSync` status.
- Sync operations and their results.

Possible destinations include Slack, email, Microsoft Teams, webhooks, and other supported integrations.

```text
Application status event
          |
          v
Notifications Controller
          |
          +--> Slack
          +--> Email
          +--> Webhook
          +--> Teams
```

## 15. Synchronization and Reconciliation

Consider this state:

```text
Git:         replicas = 5
Kubernetes:  replicas = 3
```

The reconciliation sequence is:

1. The controller reads the Application source and revision.
2. The Repository Server renders the manifests.
3. The controller reads live resources from Kubernetes.
4. It compares desired and live state.
5. The Application becomes `OutOfSync`.
6. A user runs `argocd app sync`, or automated sync starts.
7. The controller applies the required change.
8. Argo CD evaluates health.
9. The Application eventually reports `Synced` and `Healthy`.

Sync status and health are related but different:

- **Synced** means desired and live configuration match.
- **Healthy** means the resources are operating as expected.

An application can be synced but unhealthy, or healthy but out of sync.

## 16. Automated Sync, Self-Healing, and Pruning

Automated synchronization can be configured as follows:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### Self-healing

If an administrator manually changes a resource with `kubectl`, Argo CD detects the drift. With `selfHeal: true`, the controller restores the Git-defined state.

### Pruning

Pruning removes resources that were previously managed by Argo CD but no longer exist in the desired source. Without pruning, deleted Git manifests may leave resources behind in the cluster.

Pruning should be enabled deliberately and tested carefully, especially for shared resources.

## 17. Sync Waves and Resource Hooks

### Sync waves

Sync waves control the order in which resources are applied:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Lower wave numbers are generally processed before higher numbers. A common order is:

```text
Wave 0: Namespace
Wave 1: ConfigMap and Secret
Wave 2: Deployment
Wave 3: Ingress
```

### Resource hooks

Hooks run at specific points in a synchronization operation:

- `PreSync`
- `Sync`
- `PostSync`
- `SyncFail`
- `Skip`

For example, a database migration Job can run before an application Deployment:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/hook: PreSync
```

Hooks should be designed to be repeatable and should have an appropriate deletion policy.

## 18. High Availability and Scaling

Production installations may run multiple replicas of components that support horizontal scaling:

- API Server replicas behind a load balancer.
- Repository Server replicas for manifest-generation capacity.
- Application Controller sharding or replicas according to the Argo CD version.
- Highly available Redis where required.
- Multiple ApplicationSet and Notifications Controller replicas where supported.

High availability also requires:

- Reliable storage and backups for configuration.
- Redundant cluster connectivity.
- Proper resource requests and limits.
- Monitoring of queue depth, reconciliation latency, and errors.
- Tested recovery procedures.

The exact HA topology should follow the documentation for the installed Argo CD release.

## 19. Typical Installation

After installing Argo CD in the `argocd` namespace, a standard installation may contain:

```bash
kubectl get pods -n argocd
```

Example components:

```text
argocd-application-controller
argocd-applicationset-controller
argocd-dex-server
argocd-notifications-controller
argocd-redis
argocd-repo-server
argocd-server
```

Some installations omit optional components or add additional services depending on enabled features.

## 20. End-to-End Example

Suppose a developer changes a Deployment from three replicas to five in Git.

```text
Developer commit
       |
       v
Git repository
       |
       v
Repository Server renders manifests
       |
       v
Application Controller compares state
       |
       +--> OutOfSync
       |
       v
Automated sync or CLI request
       |
       v
Kubernetes API Server updates Deployment
       |
       v
Controller evaluates health
       |
       v
Application becomes Synced / Healthy
       |
       v
Notifications Controller sends event
```

The API Server provides the user-facing interface throughout the process, while Redis may cache repeated operations and Dex may authenticate the user.

## 21. Practical Mental Model

The most important relationship is:

```text
Git / source
     |
     | desired configuration
     v
Repository Server
     |
     | rendered manifests
     v
Application Controller
     |
     | compare and reconcile
     v
Kubernetes API
     |
     v
Live resources
```

Users and automation interact through:

```text
Web UI / CLI
       |
       v
API Server
       |
       +--> Repository Server
       +--> Application Controller
       +--> Redis
       +--> Kubernetes API
```

In short:

- The **Repository Server** generates desired manifests.
- The **Application Controller** reconciles desired state against live state.
- The **Kubernetes API Server** is where live resources are read and changed.
- The **API Server**, **Web UI**, and **CLI** provide management access.
- **AppProjects**, RBAC, and cluster permissions enforce boundaries.
- **Redis**, **Dex**, **ApplicationSet**, and **Notifications** provide supporting capabilities.

This separation of responsibilities is the foundation of Argo CD's scalable GitOps workflow.
