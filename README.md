# CWIC - CoreWeave Intelligent CLI

CWIC (CoreWeave Intelligent CLI) is a powerful command-line interface for interacting with CoreWeave's high-performance AI infrastructure. Built for developers, researchers, and ML engineers who demand speed, scalability, and control over their cloud resources.

```
╔════════════════════════════════════════════════════════════╗
║                                                            ║
║               ██████╗██╗    ██╗██╗  ██████╗                ║
║              ██╔════╝██║    ██║██║ ██╔═══                  ║
║              ██║     ██║ █╗ ██║██║ ██║                     ║
║              ██║     ██║███╗██║██║ ██║                     ║
║              ╚██████╗╚███╔███╔╝██║ ╚██████                 ║
║               ╚═════╝ ╚══╝╚══╝ ╚═╝  ╚═════╝                ║
║                                                            ║
║            C W I C — CoreWeave Intelligent CLI             ║
╚════════════════════════════════════════════════════════════╝
```

## Features

- **Authentication Management**: Secure token-based authentication with CoreWeave services
- **Cluster Operations**: List, manage, and generate kubeconfigs for CoreWeave Kubernetes clusters
- **Node Management**: Comprehensive node operations including drain, cordon, reboot, and monitoring
- **SUNK Cluster Interaction**: Seamlessly interact with SUNK (Slurm) clusters
- **Object Storage**: Complete CoreWeave Object Storage (`cwobject`) management capabilities
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
    - [Cluster Management](#cluster-management)
    - [Node Operations](#node-operations)
    - [SUNK (Slurm) Management](#sunk-slurm-management)
      - [Cluster Operations](#cluster-operations)
      - [Node Management](#node-management)
      - [Job Management](#job-management)
    - [Object Storage (cwobject)](#object-storage-cwobject)
    - [NodePool Management](#nodepool-management)
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
```

> [!IMPORTANT]
> Kubernetes-based cwic commands (`node`, `sunk`, `nodepool`) require this kubeconfig.
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

# Switch between authenticated accounts
cwic auth switch [organization]

# List all authenticated accounts
cwic auth switch

# Logout from current organization
cwic auth logout
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
# Output: Currently authenticated as: Production (org-id-123)
```

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

Manage CoreWeave AI Object Storage resources.

```bash
# List buckets
cwic cwobject list

# Create bucket
cwic cwobject mb <bucket-name>

# Remove bucket
cwic cwobject rb <bucket-name>

# Bucket information
cwic cwobject bucket describe <bucket-name>

# Move objects
cwic cwobject move <source> <destination>

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
