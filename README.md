# CWIC - CoreWeave Intelligent CLI

> **Note:** CWIC is a CoreWeave developed tool and is in-development. It is provided "as-is" without warranty. For official support, please refer to CoreWeave's main support channels.

CWIC (CoreWeave Intelligent CLI) is a powerful command-line interface for interacting with CoreWeave's high-performance AI infrastructure. Built for developers, researchers, and ML engineers who demand speed, scalability, and control over their cloud resources.

```
╔════════════════════════════════════════════════════════════╗
║                                                            ║
║               ██████╗ ██╗    ██╗ ██╗  ██████╗              ║
║              ██╔════╝ ██║    ██║ ██║ ██╔═══                ║
║              ██║      ██║ █╗ ██║ ██║ ██║                   ║
║              ██║      ██║███╗██║ ██║ ██║                   ║
║              ╚██████╗ ╚███╔███╔╝ ██║ ╚██████               ║
║               ╚═════╝  ╚══╝╚══╝  ╚═╝  ╚═════╝              ║
║                                                            ║
║            C W I C — CoreWeave Intelligent CLI             ║
╚════════════════════════════════════════════════════════════╝
```

## Features

- **Authentication Management**: Secure token-based authentication with CoreWeave services
- **Sandbox Runners**: [Create and manage runners and policies through the v1 API](#managed-sandbox-runners)
- **Cluster Operations**: List, manage, and generate kubeconfigs for CoreWeave Kubernetes clusters
- **Node Management**: Comprehensive node operations including drain, cordon, reboot, and monitoring
- **SUNK Cluster Interaction**: Seamlessly interact with SUNK (Slurm) clusters
- **Object Storage**: Complete CoreWeave Object Storage (`cwobject`) management capabilities
- **Container Registry**: Manage CWCR namespaces, manifests, tags, referrers, and lifecycle operations
- **DFS Verification**: Trigger and inspect fio-backed distributed file system verification runs
- **CoreWeave Dashboards**: Link straight into the relevant dashboard, pre-filtered, in CoreWeave's managed Grafana

## Table of Contents

- [CWIC - CoreWeave Intelligent CLI](#cwic---coreweave-intelligent-cli)
  - [Features](#features)
  - [Table of Contents](#table-of-contents)
  - [Installation](#installation)
    - [Pre-built Binaries](#pre-built-binaries)
      - [Linux](#linux)
      - [MacOS](#macos)
    - [From Source](#from-source)
    - [Auto-Update](#auto-update)
  - [Getting Started](#getting-started)
    - [1. Authentication](#1-authentication)
    - [2. Verify Authentication](#2-verify-authentication)
    - [3. Set Up Cluster Access](#3-set-up-cluster-access)
    - [4. Basic Usage](#4-basic-usage)
  - [Commands](#commands)
    - [Authentication](#authentication)
      - [Object Storage Credentials from OIDC](#object-storage-credentials-from-oidc)
      - [Object Storage Credentials from an API Token](#object-storage-credentials-from-an-api-token)
      - [Kubernetes Credentials from an API Token](#kubernetes-credentials-from-an-api-token)
    - [Cluster Management](#cluster-management)
    - [Managed Sandbox runners](#managed-sandbox-runners)
    - [Node Operations](#node-operations)
    - [SUNK (Slurm) Management](#sunk-slurm-management)
      - [Cluster Operations](#cluster-operations)
      - [Node Management](#node-management)
      - [Job Management](#job-management)
    - [Object Storage (cwobject)](#object-storage-cwobject)
    - [Container Registry](#container-registry)
    - [NodePool Management](#nodepool-management)
    - [DFS Verification](#dfs-verification)
  - [Configuration](#configuration)
  - [Development](#development)
    - [Prerequisites](#prerequisites)
    - [Testing](#testing)
    - [Code Style](#code-style)

## Installation

### Pre-built Binaries

Download the latest release for your platform from the [releases page](https://github.com/coreweave/cwic/releases). You will then need to add the binary to your PATH environment variable and/or `/usr/local/bin`.

#### Linux

```bash
gh release download -R coreweave/cwic -p "cwic_$(uname)_$(uname -m).tar.gz" -O - --clobber | tar zxf - cwic && mv cwic $HOME/.local/bin/
```

#### MacOS

```bash
gh release download -R coreweave/cwic -p "cwic_$(uname)_$(uname -m).tar.gz" -O - --clobber | tar zxf - cwic && mv cwic /usr/local/bin
```

### From Source

Requires Go 1.24.1 or later:

```bash
git clone https://github.com/coreweave/cwic.git
cd cwic
make build
```

### Auto-Update

CWIC can update itself to the latest version:

```bash
cwic update
```

## Getting Started

### 1. Authentication

Before using CWIC, you need to authenticate with CoreWeave. You can either provide a token directly or use the interactive browser-based authentication:

```bash
# Interactive authentication (opens browser)
cwic auth login

# Direct token authentication
cwic auth login YOUR_TOKEN_HERE
```

To get a token:
1. Visit https://console.coreweave.com/tokens
2. Generate a new API token
3. Use it with `cwic auth login YOUR_TOKEN`

This CW token will be used for different functionality throughout `cwic` including but not limited to:
 - kubeconfig generation
 - metrics querying
- `cwobject` interactions
 - CW API interactions (CKS cluster listing)

### 2. Verify Authentication

Check your authentication status:

```bash
# Check current user/organization
cwic auth whoami

# Include group and role details
cwic auth whoami -o wide

# Inspect the complete principal as structured data
cwic auth whoami -o json

# Verify access by listing clusters
cwic cluster get
```

The `whoami` command shows which organization you're authenticated as, and `cluster get` will list your current CKS clusters if authentication is successful.

### 3. Set Up Cluster Access
> [!NOTE]
> Only clusters with a Public endpoint are supported for `cwic cluster auth`

Generate a kubeconfig for your cluster:

```bash
# List available clusters
cwic cluster get

# Generate kubeconfig for a specific cluster
cwic cluster auth CLUSTER_NAME

# Generate kubeconfig for all clusters
cwic cluster auth all

# Use cwic's exec credential plugin instead of embedding the token
cwic cluster auth CLUSTER_NAME --cwic-auth
```

Generated kubeconfigs embed a static token by default. With `--cwic-auth`,
they instead authenticate through an exec block that runs `cwic auth k8s api-token`
at use time, so re-running `cwic auth login` re-credentials every kubeconfig
at once without storing the token in the kubeconfig.

> [!IMPORTANT]
> Kubernetes-based cwic commands (`node`, `sunk`, `nodepool`, `dfs`) require this kubeconfig.
> If you created a new kubeconfig file (not appended to default), set the KUBECONFIG environment variable:
> ```bash
> export KUBECONFIG=/path/to/your/kubeconfig
> ```

### 4. Basic Usage

Once authenticated, you can start managing your CoreWeave resources:

```bash
# List nodes in your cluster (requires kubeconfig from step 3)
cwic node get

# Check cluster status (requires kubeconfig from step 3)
cwic sunk cluster describe

# List storage buckets (requires only CW auth token, no kubeconfig needed)
cwic cwobject list
```

## Commands

### Authentication

Manage your CoreWeave authentication credentials.

The `whoami` command queries the CoreWeave WhoAmI service and translates its permissions into the role names used by CoreWeave access policies.
Its default table shows the principal UID and organization ID; `wide` adds groups and roles. 
JSON and YAML include the same identity details and identify permissions without a known role label under `unrecognized_permissions`; `name` outputs only the principal UID. 
Policies can contain arbitrary role bundles, so the flattened WhoAmI response cannot identify which named policies granted the roles.

```bash
# Interactive login (opens browser)
cwic auth login

# Login with token
cwic auth login <token>

# Login with a friendly name for the organization
cwic auth login <token> --name "My Org"
cwic auth login <token> -n "Production"

# Check current authentication status
cwic auth whoami

# Include group and role details
cwic auth whoami -o wide

# Script-friendly principal output
cwic auth whoami -o json
cwic auth whoami -o yaml
cwic auth whoami -o name

# Switch between authenticated accounts
cwic auth switch [organization]

# List all authenticated accounts
cwic auth switch

# Logout from current organization
cwic auth logout

# Print the current organization's token
cwic auth token

# Emit the token as a Kubernetes ExecCredential, for kubeconfig exec blocks
cwic auth k8s api-token --org-id cwXXXX

# Exchange an OIDC token for temporary object-storage credentials
cwic auth accesskey oidc -- <command that prints an OIDC token>

# Exchange a CoreWeave API token for temporary object-storage credentials
cwic auth accesskey api-token
```

**Examples:**

```bash
# Login to multiple organizations with friendly names
cwic auth login abc123... --name "Production"
cwic auth login xyz789... --name "Development"

# List all authenticated accounts
cwic auth switch
# Output:
#   Production (org-id-123) (active)
#   Development (org-id-456)

# Switch to different account
cwic auth switch Development
# Or use the organization ID directly
cwic auth switch org-id-456

# Check which account you're currently using
cwic auth whoami
# Output:
# UID           ORG ID
# principal-1   org-id-123

# Include groups and roles
cwic auth whoami -o wide
```

#### Object Storage Credentials from OIDC

`cwic auth accesskey oidc` exchanges an OIDC token for temporary CoreWeave
object-storage credentials and prints them as AWS
[process credentials](https://docs.aws.amazon.com/sdkref/latest/guide/feature-process-credentials.html),
so the AWS CLI and SDKs can authenticate to CoreWeave Object Storage with your
existing identity provider instead of a long-lived access key.

Point a profile at it in `~/.aws/config`:

```ini
[profile coreweave]
region = US-EAST-04A
endpoint_url = https://cwobject.com
s3 =
    addressing_style = virtual

credential_process = cwic auth accesskey oidc --org-id cwXXXX --storage=keyring -- kubelogin get-token \
    --oidc-issuer-url=https://example.okta.com/oauth2/xxx \
    --oidc-client-id=xxx --oidc-extra-scope=openid,email,profile \
    --oidc-pkce-method S256
```

Then use it like any other profile:

```bash
AWS_PROFILE=coreweave aws s3 ls
```

Everything after `--` is the command that produces the OIDC token — `kubelogin
get-token` above — and is run unmodified, so its own flags pass through. cwic's
flags must come before the `--`.

The resulting credentials are cached per organization, endpoint and command, and
reused until they near expiry, so repeated `aws` calls do not
re-run the OIDC flow each time. Caching the OIDC token itself is left to the
command (`kubelogin` does this).

```bash
# Use the active organization from 'cwic auth login'
cwic auth accesskey oidc -- kubelogin get-token --oidc-issuer-url=<issuer> --oidc-client-id=<id>

# Name the organization explicitly (required when several are logged in)
cwic auth accesskey oidc --org-id cwXXXX -- kubelogin get-token --oidc-issuer-url=<issuer> --oidc-client-id=<id>

# Cache the credentials in the OS keyring rather than on disk
cwic auth accesskey oidc --storage=keyring -- kubelogin get-token --oidc-issuer-url=<issuer> --oidc-client-id=<id>
```

Credentials are cached in the backend `--storage` names. Without it they go to
whichever backend that organization's token already uses (see
[Configuration](#configuration)), falling back to disk.

#### Object Storage Credentials from an API Token

`cwic auth accesskey api-token` does the same job as the `oidc` subcommand, but
exchanges a CoreWeave **API token** rather than an OIDC token. Use it where
there is no identity provider to talk to — CI jobs, servers, containers — since
it needs no browser and no external helper command.

The token is read from `COREWEAVE_API_TOKEN`, or, when that is unset, from the
token stored by `cwic auth login` for the active organization. So on a laptop it
generally needs no configuration at all:

```ini
[profile coreweave]
region = US-EAST-04A
endpoint_url = https://cwobject.com
s3 =
    addressing_style = virtual

credential_process = cwic auth accesskey api-token --storage=keyring
```

```bash
# Uses the token from 'cwic auth login'
cwic auth accesskey api-token

# Or supply one explicitly, e.g. in CI
COREWEAVE_API_TOKEN=CW-SECRET-... cwic auth accesskey api-token

# Cache the credentials in the OS keyring rather than on disk
cwic auth accesskey api-token --storage=keyring
```

Credentials are cached per token and organization, and reused until they near
expiry, so repeated `aws` calls do not mint a new
access key each time. The cache key is a digest, so the token itself is never
written to the cache directory or the keyring.

#### Kubernetes Credentials from an API Token

`cwic auth k8s api-token` emits the CoreWeave API token in the
Kubernetes [ExecCredential](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins)
format, so a kubeconfig can look the token up at use time through an `exec`
block instead of embedding a copy of it. Rotating the token with
`cwic auth login` then updates every kubeconfig at once, and the files carry no
secret.

```yaml
users:
- name: coreweave
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1
      command: cwic
      args: ["auth", "k8s", "api-token", "--org-id", "cwXXXX"]
      interactiveMode: Never
```

Pin `--org-id` in kubeconfigs to keep kubeconfig functional after `cwic auth switch`.
An explicit `--org-id` always reads that organization's stored login; without
it the token comes from `COREWEAVE_API_TOKEN` when set, or the login stored
by `cwic auth login` otherwise. 

### Cluster Management

**Features:**
- Automatic kubeconfig generation
- Multi-cluster support
- Secure cluster authentication

```bash
# List all available clusters
cwic cluster get

# Generate kubeconfig for specific cluster
cwic cluster auth <cluster-name>

# Generate kubeconfig for all clusters
cwic cluster auth all
```

### Managed Sandbox runners

`cwic sandbox runner` uses the v1 RunnerManagementService for create, get,
list, edit, delete, and upgrade. The API must implement full managed-runner
CRUD (Sandman v0.176.0 or later). These operations require `sandbox_admin`.

#### Create a runner

Start with a template, fill in the runner ID, zone, and CKS cluster UUID, and
review the policy:

```sh
cwic sandbox runner create --print-template > runner.yaml
$EDITOR runner.yaml
cwic sandbox runner create -f runner.yaml --request-id my-runner-provisioning-1
```

The file is a v1 ManagedRunner resource, without a request wrapper:

```yaml
identity:
  runner_id: my-runner
  zone: us-east-04a
  cluster_id: 00000000-0000-4000-8000-000000000001
  runner_group_id: default
display_name: Development runners
spec:
  release_channel: RELEASE_CHANNEL_STABLE
  enforce_resource_limits: true
  overrides:
    env:
      LOG_LEVEL: info
policy:
  constraints:
    lifecycle:
      default_lifetime_seconds: 86400
```

The runner and explicit policy are created together. `policy: {}` is accepted
when an empty policy is intended; omitting policy is an error. Profile bindings
are not required. The server defaults an omitted runner group to `default`.

Without `-f`, an interactive terminal offers cluster selection, then opens the
runner configuration and policy in `$EDITOR` for review before confirmation.
Non-interactive use requires `-f` and never prompts or creates a profile template.

The `--request-id` flag sets the create request's idempotency token, which the
server stores to recognize retries. It is separate from the HTTP request ID
used for tracing; middleware-generated tracing IDs do not replace this token.
A token is generated when the flag is omitted. To retry across CLI invocations
after an uncertain response, supply an explicit token on the first attempt and
reuse it with identical input. Use a new token for a different configuration or
after deleting the original runner. Tokens must be valid UTF-8 and at most 128
bytes.

#### Inspect and edit

```sh
cwic sandbox runner get
cwic sandbox runner get my-runner -o json
cwic sandbox runner describe my-runner
cwic sandbox runner edit my-runner
```

To filter a list, pass the returned `identity.cluster_id` to `--cluster` and
optionally add `--zone`. Older runners may return a cluster name in that field;
the list filter accepts it. Creating a new runner still requires a CKS UUID.

`edit` opens the current mutable settings and policy in `$EDITOR`. Saving
without changes skips the update. Removing an optional setting in the editor
clears that setting. Removing the policy or release channel is rejected; choose
`RELEASE_CHANNEL_STABLE` or `RELEASE_CHANNEL_RAPID` explicitly when changing channels.

For file-based edits, export a complete template first:

```sh
cwic sandbox runner edit my-runner --print-template > runner-edit.yaml
$EDITOR runner-edit.yaml
cwic sandbox runner edit my-runner -f runner-edit.yaml
```

The template retains all deployment override fields, including environment
variables, arguments, node selectors, resources, and scaling. Objects such as
`spec.overrides`, `spec.data_plane`, and `policy` are replacement boundaries:
keep their sibling settings when editing a file. Fields absent from a file patch
are not updated; use explicit `null` to clear an optional object or `false` to
turn off a boolean. To change runner environment configuration, edit
`spec.overrides.env` in the exported document.

Policy edits carry the `etag` from the exported document. A stale revision fails
without retrying against a new token. `--etag TOKEN` can pin a revision explicitly.
For a policy patch with no token, cwic reads the current revision before submitting.
The dedicated `cwic sandbox runner policy` commands continue to edit policy alone.

Supported spec fields include release channel, maintenance policy, deployment
overrides, resource-limit enforcement, volumes, tenant metrics, image pull policy,
and direct data-plane configuration. For example, use `spec.data_plane.disabled: {}`
to keep data operations on the Gateway.

#### Upgrade and delete

```sh
cwic sandbox runner upgrade my-runner
cwic sandbox runner delete my-runner --yes
```

Upgrade and deletion are asynchronous. The commands report acceptance; use
`get` or `describe` to inspect completion. Sandman enforces deletion guards,
including registered-volume checks. Batch deletion reports individual failures.

#### Migrating older command input

- Use the operator-assigned `runner_id` for lookups and mutations. v1 does not
  expose or resolve the old database UUID as a separate runner identifier.
- Create files use `identity.cluster_id` (a CKS UUID). `cluster_name` is now
  output-only; the interactive flow resolves the UUID from cluster selection.
- Rename `managed_spec` to `spec`, replace profile bindings with an explicit
  policy, and use the new template to check supported fields. The CLI rejects
  legacy fields instead of silently ignoring them.
- v1 does not expose legacy profile-binding or `endpoint_routes` edits. Existing
  beta-only settings remain stored on the server when v1 configuration is updated.
  Legacy `sandbox profile` commands retain their beta API and binding-reference
  checks; `sandbox template` commands continue using v1.
- JSON output is still an array, now containing v1 ManagedRunner objects:
  `identity.runner_id`, `spec`, `policy`, `etag`, and v1 timestamps such as
  `create_time`. Scripts that read `id`, `managed_spec`, or `created_at` must update.

### Node Operations

**Features:**
- Safe node operations with confirmation prompts
- Bulk operations support
- Interactive shell access
- Monitoring integration
- Hardware verification


CKS stores a lot of important information as metadata on kubernetes node objects including hardware information about the node, lifecycle actions, pending action, health check success, and more. When using `kubectl` directly, this information isn't easily surfaced. `cwic` surfaces the important information allowing you to move quickly.


```bash
# List all nodes
cwic node get

# Get detailed node information
cwic node describe <node-name>
```

`cwic`'s node commands also made it easy to trigger lifecycle actions against a node like HPC verification tests and reboots.

```bash
# Cordon a node (prevent new pods)
cwic node cordon <node-name>

# Uncordon a node
cwic node uncordon <node-name>

# Drain a node (safely evict pods)
cwic node drain <node-name>

# Remove drain from node
cwic node undrain <node-name>

# Reboot a node (immediate by default, prompts for active nodes)
cwic node reboot <node-name>

# Safe reboot (waits for workloads to terminate)
cwic node reboot --safe <node-name>

# Force immediate reboot without prompting
cwic node reboot --force <node-name>

# Run verification tests
cwic node verify <node-name>
```

CKS offers fully isolated k8s clusters that run on bare-metal nodes, which gives you the ability to take actions like running priviledged nodes. If you ever need to get into the underlying node, `cwic node shell` makes that easy. Don't forget that if you mess something up while in a node shell, you can run a `cwic node reboot` to get the node back to it's original state.

```bash
# Open interactive shell on node
cwic node shell <node-name>
```

The Node Details dashboard in CoreWeave's managed Grafana shows a wealth of information about any node in your cluster in easy to digest visuals, along with multiple log streams. `cwic node view` will take you directly to the dashboard for the given node.

```bash
# Open Grafana dashboard for node
cwic node view <node-name>
```

### SUNK (Slurm) Management

**Features:**
- **Multi-cluster SUNK Management**: Interact with SUNK (Slurm on Kubernetes) clusters across your CoreWeave fleet
- **Comprehensive Job Monitoring**: View job queues, resource allocation, and performance metrics with partition-level statistics
- **Node Status Tracking**: Monitor compute node health, utilization, and lifecycle states across nodesets
- **Integrated Grafana Dashboards**: Direct access to pre-configured monitoring dashboards for clusters, nodes, and jobs
- **Resource Analytics**: Detailed resource usage statistics including CPU, GPU, and node allocation tracking

SUNK is CoreWeave's implementation of Slurm on Kubernetes, providing HPC-grade workload management with cloud-native orchestration. The `cwic sunk` commands give you full visibility and control over your SUNK deployments without needing to connect to SUNK's login pods.

#### Cluster Operations

Get an overview of your SUNK clusters and their current state:

```bash
# List all SUNK clusters in the current CKS cluster
cwic sunk cluster get

# Get detailed cluster information with nodesets, login pods, partition statistics, and running jobs
cwic sunk cluster describe [CLUSTER_NAME]

# Open cluster monitoring dashboard in Grafana
cwic sunk cluster view [CLUSTER_NAME]
```

The `describe` command provides comprehensive cluster information including:
- **Cluster Details**: Name, version, namespace, and status
- **Nodesets Table**: Shows running/drained nodes, desired vs actual counts, and version information
- **Login Nodes**: External IP addresses for cluster access
- **Job Statistics**: Partition-level job counts and resource utilization
- **Top Jobs**: Largest jobs by node count (use `-v` for all jobs, `-vv` to include nodes table)

#### Node Management

Monitor and manage individual compute nodes within your SUNK clusters:

```bash
# List all SUNK nodes
cwic sunk node get

# Get detailed information about specific nodes
cwic sunk node describe <node-name>

# Open node monitoring dashboard
cwic sunk node view <node-name>
```

Node information includes:
- **K8s Integration**: Shows both Slurm node name and underlying Kubernetes node
- **Status Tracking**: Running state, drain status, and version information
- **Nodeset Assignment**: Which nodeset the node belongs to for organizational visibility
- **Hardware Details**: CPU, GPU, and memory specifications from Kubernetes metadata

#### Job Management

Track and analyze Slurm jobs across your clusters. Slurm jobs aren't available as kubernetes resources, so `cwic sunk job` uses your configured CoreWeave token to query SUNK-specific metrics from the CW Observe endpoint.

```bash
# List all jobs in the cluster
cwic sunk job get

# Get information about specific jobs
cwic sunk job get [job-id1] [job-id2]

# Get detailed job information with resource allocation
cwic sunk job describe <job-id>

# Open job metrics dashboard in Grafana
cwic sunk job view <job-id>
```

Job information includes:
- **Basic Details**: Job ID, name, partition, state, user, and runtime
- **Resource Allocation**: Allocated CPUs, GPUs, and node count
- **Node Assignment**: List of specific nodes allocated to the job

### Object Storage (cwobject)

Manage CoreWeave AI Object Storage resources using the S3-compatible API.

#### Configuration

`cwobject` needs an endpoint and a set of credentials. Credentials can come from
an s3cfg file, from environment variables, or — when neither carries an
access/secret pair — from your CoreWeave API token, which `cwobject` exchanges
for temporary ones automatically.

That last case usually means no credential configuration at all: log in once and
use `cwobject`, with only the endpoint set.

```bash
cwic auth login
export AWS_ENDPOINT_URL=https://cwobject.com
export AWS_REGION=EU-SOUTH-03B   # your CoreWeave zone

cwic cwobject list
```

The token is taken from `COREWEAVE_API_TOKEN`, or the login stored by
`cwic auth login`. The credentials it mints are refreshed automatically as they
near expiry, so long-running transfers do not fail partway through, and they are
cached between invocations exactly as
[`cwic auth accesskey api-token`](#object-storage-credentials-from-an-api-token)
caches them — `--storage` selects the backend for the credentials (disk or keyring). 
A static access/secret pair in your s3cfg or environment always takes precedence, 
so existing setups are unaffected.

The rest of this section covers configuring those static credentials explicitly.
You can use either an s3cfg file or environment variables.

When using an `s3cfg` file that follows the `s3cmd` format, you would create the file at `~/.s3cfg`, or alternatively use the `--config` flag to specify a different path. The content should look like this:

```bash
[default]
# Your credentials
access_key = <your-access-key-id>
secret_key = <your-secret-access-key>

# New CoreWeave Global Endpoint
host_base = cwobject.com
host_bucket = %(bucket)s.cwobject.com

# Connection settings
use_https = True
check_ssl_certificate = True
check_ssl_hostname = True

# Storage location default (using us-east-13a as an example, but should be set to your CoreWeave zone)
bucket_location = us-east-13a

# Interface settings
human_readable_sizes = True
website_endpoint = http://%(bucket)s.cwobject.com/

```bash
# Step 1: Create an access key (if you don't have one)
cwic cwobject token create --name my-access-key --duration 3600

# Step 2: Set environment variables with your credentials
export AWS_ACCESS_KEY_ID=<your-access-key-id>
export AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
export AWS_ENDPOINT_URL=https://cwobject.com  # or https://bucket.cwobject.com
export AWS_REGION=EU-SOUTH-03B  # Your CoreWeave zone - required for bucket creation
```

**Note:** 
- The endpoint URL can be either:
  - Bucket-specific: `https://your-bucket.cwobject.com` (automatically normalized to base endpoint)
  - Base endpoint: `https://cwobject.com` (recommended for listing all buckets)
  - Internal (from within CKS clusters): `http://cwlota.com` (HTTP-only)
- `AWS_REGION` should be set to your CoreWeave zone (e.g., `EU-SOUTH-03B`). It is **required** when creating new buckets (`mb` command) as the `LocationConstraint`. Other operations (list, get, move, delete) work without it.
- Newly created buckets may take up to 60 seconds to become available for operations due to zone routing propagation.

#### Usage

```bash
# List buckets
cwic cwobject list

# List objects in a bucket
cwic cwobject list s3://my-bucket/

# Create bucket
cwic cwobject mb <bucket-name>

# Remove bucket (empty buckets only)
cwic cwobject rb <bucket-name>

# Force remove bucket (deletes all objects first)
cwic cwobject rb --force <bucket-name>

# Bucket information
cwic cwobject bucket describe <bucket-name>
cwic cwobject bucket describe --all  # describe all buckets

# Enable archive for objects with 60 days since last access
cwic cwobject bucket update my-bucket \
  --archive-enabled=true \
  --archive-after-last-access-days=60

# Change the last access inactivity threshold without changing whether archive is enabled
cwic cwobject bucket update my-bucket \
  --archive-after-last-access-days=90

# Disable archive
cwic cwobject bucket update my-bucket \
  --archive-enabled=false

# Move objects between buckets
cwic cwobject move s3://source-bucket/object s3://dest-bucket/object

# Access token management
cwic cwobject token create --name <key-name> --duration <seconds>
cwic cwobject token get
cwic cwobject token get --name <key-name>
cwic cwobject token get --cwic-only

# Policy management
cwic cwobject policy create --file <policy-file>
cwic cwobject policy get
cwic cwobject policy delete --name <policy-name>
```

**Features:**
- S3-compatible object storage
- Access control management
- Policy-based permissions
- Token lifecycle management

### Container Registry

Manage CoreWeave Container Registry resources. Registry commands use the active CWIC authentication token and support `table`, `wide`, `json`, `yaml`, and `name` output.
List commands automatically retrieve every page, and commands that take targets support multiple arguments or newline-delimited stdin.

See the [registry command guide](cmd/registry/README.md) for the complete command surface, reference syntax, login behavior, pipelines, batch operations, idempotent retries, and deletion/reclamation semantics.

```bash
# Namespace lifecycle
cwic registry namespace create acme --zone US-LAB-01A --wait
cwic registry namespace get acme
cwic registry namespace list
cwic registry namespace list --zone US-LAB-01A --sort-by=-created
cwic registry namespace delete acme --yes --wait

# Inspect organization-zone and per-namespace quotas
cwic registry quota list
cwic registry namespace quota list
cwic registry namespace list | cwic registry namespace quota list

# Inspect, preview, and apply manifest retention policies
cwic registry namespace lifecycle get acme -o yaml
cwic registry namespace lifecycle preview acme --file lifecycle.yaml
cwic registry namespace lifecycle update acme --file lifecycle.yaml --wait

# Inspect and mutate namespace access policy
cwic registry namespace access get acme -o yaml
cwic registry namespace list | cwic registry namespace access get
cwic registry namespace access update acme --file access.yaml --wait
cwic registry namespace access policy-set update acme ci-pull --file policy-set.yaml --wait
cwic registry namespace access policy-set delete acme ci-pull --yes --wait
cwic registry namespace access revision list acme
cwic registry namespace list | cwic registry namespace access revision list
cwic registry namespace access revision list acme | cwic registry namespace access revision get
cwic registry namespace access revision rollback acme 2 --wait

# Configure Docker credentials for one or more namespaces
cwic registry login acme
cwic registry namespace list -o name | cwic registry login

# Indexed manifest metadata (selectors are always explicit)
cwic registry manifest list acme --include-untagged
cwic registry manifest list acme/team/app
cwic registry manifest list acme/team/app --sort-by=-reachable-size
cwic registry manifest get acme/team/app:latest
cwic registry manifest get acme/team/app:stable@sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
cwic registry manifest get acme/team/app@sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

# Resolve and list mutable tags
cwic registry tag get acme/team/app:latest
cwic registry tag get acme/team/app:stable@sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
cwic registry tag list acme/team/app
cwic registry tag list acme/team/app --sort-by=digest

# Safe mutations and OCI referrers
cwic registry tag delete acme/team/app:stale --yes
cwic registry tag delete acme/team/app:stale@sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef --yes
cwic registry manifest delete acme/team/app@sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef --yes
cwic registry referrer list acme/team/app@sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

# Namespace lifecycle operation records
cwic registry operation list acme
cwic registry operation get namespaces/acme/operations/4d75b57a-9ee4-4b4d-889b-35b7da3e1939

# Compose reads and mutations. The trailing "-" is optional for piped input.
cwic registry namespace list | cwic registry manifest list
cwic registry manifest list acme -o name | cwic registry manifest delete --yes
cwic registry tag list acme/team/app | cwic registry tag delete --yes
cwic registry operation list acme | cwic registry operation get
```

- Set `CWIC_REGISTRY_API_URL` or pass `--api-url` to use a non-production API endpoint.
- A `repository:tag@digest` reference selects the tag and uses the digest as a compare-and-match condition, so lookups and deletions fail safely if the tag has moved.
- Prefer `-o name` for scripts; CWIC also understands its own headed table output when commands are connected directly with a pipe.

### NodePool Management

Manage CoreWeave NodePool resources.

```bash
# Apply pending node profile to NodePool 
cwic nodepool upgrade <nodepool-name>

# Rollback to the immediate prior node profile from the current active profile
cwic nodepool rollback <nodepool-name>

# Rollback to the specified nodeprofile
cwic nodepool rollback <nodepool-name> <nodeprofile-name>

# View Nodes associated with nodepool
cwic nodepool node get <nodepool-name>

# View only nodes requiring a reconfigure reboot that are associated with the nodepool
cwic nodepool node get <nodepool-name> --requiring-reconfiguration

# View only supplied node names within the nodepool
cwic nodepool node get <nodepool-name> <list-of-space-separated-nodes>

# Trigger a rollout of a staged node configuration 
# For use with the RolloutOnCommand Reconfiguration Strategy
cwic nodepool rollout start <nodepool-name>

# Pauses an active node config rollout. Nodes will stop 
# reconfiguring until the start command is run again  
cwic nodepool rollout stop <nodepool-name> 

```

**Features:**
- Manage staging and rollback of Node configurations

### DFS Verification

Verify distributed file system health. `cwic dfs verify` launches an
asynchronous fio-backed verification run as a Kubernetes Job against a
StorageClass (or a node's local NVMe) and returns immediately. `cwic dfs
describe` reads run results back from metrics and cluster state, grading
each run PASS, FAIL, RUNNING, or PENDING (metrics not yet ingested).

```bash
# Trigger a verification run against a StorageClass
cwic dfs verify --storage-class shared-vast

# Verify local NVMe, pinned to a node for node-scoped triage
cwic dfs verify --storage-class local-nvme --node <node-name>

# Change the throwaway PVC size (default 20Gi)
cwic dfs verify --storage-class shared-vast --size 40Gi

# Run only the data or metadata fio profiles (default: both)
cwic dfs verify --storage-class shared-vast --test data

# List PersistentVolumes and the latest runs, newest first
cwic dfs describe

# Filter the run list by StorageClass
cwic dfs describe --storage-class shared-vast

# Full report for one run: IOPS, bandwidth, latency percentiles, I/O errors.
# A failed run exits with code 2.
cwic dfs describe --run-id <run-id>

# Machine-readable report
cwic dfs describe --run-id <run-id> -o json
```

## Configuration

CWIC stores configuration in your home directory:

- **Linux/macOS**: `~/.cwic/config.json`

## Development

### Prerequisites

- Go 1.24.1 or later
- Make

### Testing

```bash
# Run all tests
make test
```

### Code Style

This project follows standard Go conventions:

```bash
# Run linter
make lint

# Fix linting issues
make lint-fix
```
