# VMware Tanzu Kubernetes Grid Integrated (TKGI) Edition running on Google Cloud Platform (GCP)

Follow this procedure to deploy {{#include ./product_name.md}} to VMware Tanzu Kubernetes Grid Integrated (TKGI) Edition.

## <a id="softwarereq"></a>Required Software

To deploy {{#include ./product_name.md}} on VMware Tanzu Kubernetes Grid Integrated Edition, you require the following software:

* `kubectl` command-line utility. Install the version of `kubectl` that is distributed with VMware Tanzu Kubernetes Gri\
d Integrated (TKGI) Edition, even if you are deploying Greenplum to Minikube. See [Installing the Kuberenetes CLI](http\
s://docs.pivotal.io/runtimes/pks/1-3/installing-kubectl-cli.html) in the VMware Tanzu Kubernetes Grid Integrated (TKGI)\
 Edition documentation for instructions.

* Docker. Install a recent version of [Docker](https://www.docker.com/community-edition) to your machine, and start Doc\
ker.

* Helm package manager utility version 3.3 or later. Follow the instructions at [Kubernetes Helm](https://github.com/he\
lm/helm) to install `helm`.

* VMware Tanzu Greenplum for Kubernetes requires the ability to map the host system's `/sys/fs/cgroup` directory onto e\
ach container's `/sys/fs/cgroup`. Ensure that no kernel security module (for example, AppArmor) uses a profile that dis\
allows mounting `/sys/fs/cgroup`.

* The `watch` command-line utility is used to monitor the process of new deployments. If necessary, use your operating \
system package management utility to install this utility (for example, `brew install watch` on MacOS platforms).

* VMware Tanzu Kubernetes Grid Integrated (TKGI) Edition 1.7.0 (contains Kubernetes 1.16.7)


## <a id="softwarereq"></a>Cluster Requirements

This procedure requires that VMware Tanzu Kubernetes Grid Integrated (TKGI) Edition is installed and running, along with all prerequisite
software and configuration. See [Installing VMware Tanzu Kubernetes Grid Integrated Edition for {{#include ./product_name.md}}, Using Google Cloud Platform (GCP)](prerequisites.html) for information.

In the TKGI tile under **Kubernetes Cloud Provider**, ensure that the service accounts that are listed under **GCP Master Service Account ID** and **GCP Worker Service Account ID** have permission to pull images from the GCS bucket named `artifacts.<project-name>.appspot.com`.

## <a id="getkey"></a>Obtaining the Service Account Key

Obtain a Kubernetes service account key (a `key.json` file) for an account that has read access (`storage.objectViewer` role) to the Google Cloud Registry. You will need to identify this file in your configuration to pull VMware Tanzu Greenplum for Kubernetes docker images from the remote registry. For example:

1. If necessary, create a new service account to use for VMware Tanzu Greenplum for Kubernetes. These example commands create a new account named `greenplum-image-pull` in your current GCP project:

    ``` bash
    $ export GCP_PROJECT=$(gcloud config get-value core/project)
    
    $ gcloud iam service-accounts create greenplum-image-pull
    ```

2. Assign the required `storage.objectViewer` role to the new account:

    ``` bash
    $ gcloud projects add-iam-policy-binding $GCP_PROJECT \
        --member serviceAccount:greenplum-image-pull@$GCP_PROJECT.iam.gserviceaccount.com \
        --role roles/storage.objectViewer
    ```

3. Create the key for the account:

    ``` bash
    $ gcloud iam service-accounts keys create \
        --iam-account "greenplum-image-pull@$GCP_PROJECT.iam.gserviceaccount.com" \
        ~/key.json
    ```
    
Before you attempt to deploy Greenplum, ensure that the target cluster is available. Execute the following command to make sure that the target cluster displays in the output:

```bash
pks list-clusters
```

**Note:** The `pks` login cookie typically expires after a day or two.

The {{#include ./product_name.md}} deployment process requires the ability to map the host system's `/sys/fs/cgroup` directory onto each container's `/sys/fs/cgroup`. Ensure that no kernel security module (for example, AppArmor) uses a profile that disallows mounting `/sys/fs/cgroup`.

To use pre-created disks with TKGI instead of (default) automatically-managed persistent volumes, follow the instructions in [(Optional) Preparing Pre-Created Disks](#pre_created_disks) before continuing with the procedure.

**Note:** If any problems occur during deployment, retry deploying Greenplum by first removing the previous deployment.

{{#include ./product_name.md}} requires the following recommended settings:

1. Ability to increase the open file limit on the TKGI cluster nodes

In order to accomplish this, do the following on each node using bosh:

```bash

#!/usr/bin/env bash

# fail fast if bosh isn't working
which bosh

deployment=$1
bosh deployment -d ${deployment}

# change limits file
bosh ssh master -d ${deployment} -c "sudo sed -i 's/#DefaultLimitNOFILE=/DefaultLimitNOFILE=65535/' /etc/systemd/system.conf"

# change running processes by getting pid for both monit and the kube-apiserver processes
bosh ssh master -d ${deployment} -c 'sudo bash -c "prlimit --pid $(pgrep monit) --nofile=65535:65535"'
bosh ssh master -d ${deployment} -c 'sudo bash -c "prlimit --pid $(pgrep kube-apiserver) --nofile=65535:65535"'
# repeat above for other nodes besides master in the deployment
```
