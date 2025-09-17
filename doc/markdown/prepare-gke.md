# Google Kubernetes Engine (GKE) running on Google Cloud Platform (GCP)

This section describes the requirements for using {{#include ./product_name.md}} with Google Kubernetes Engine (GKE) deployments.

## <a id="softwarereq"></a>Required Software

To deploy {{#include ./product_name.md}} on Google Kubernetes Engine, you require the following software:

* `kubectl` command-line utility. Install the version of `kubectl` that is distributed with VMware Tanzu Kubernetes Grid Integrated (TKGI) Edition, even if you are deploying Greenplum to Minikube. See [Installing the Kuberenetes CLI](https://docs.pivotal.io/runtimes/pks/1-3/installing-kubectl-cli.html) in the VMware Tanzu Kubernetes Grid Integrated (TKGI) Edition documentation for instructions.

* Docker. Install a recent version of [Docker](https://www.docker.com/community-edition) to your machine, and start Docker.

* Helm package manager utility version 3.3 or later. Follow the instructions at [Kubernetes Helm](https://github.com/helm/helm) to install `helm`.

* VMware Tanzu Greenplum for Kubernetes requires the ability to map the host system's `/sys/fs/cgroup` directory onto each container's `/sys/fs/cgroup`. Ensure that no kernel security module (for example, AppArmor) uses a profile that disallows mounting `/sys/fs/cgroup`.

* The `watch` command-line utility is used to monitor the process of new deployments. If necessary, use your operating system package management utility to install this utility (for example, `brew install watch` on MacOS platforms).

* Google Kubernetes Engine (GKE) Kubernetes 1.16.7

## <a id="clusterreq"></a>Cluster Requirements

When creating the GKE cluster, ensure that you make the following selections on the **Create a Kubernetes cluster** screen of the Google Cloud Platform console:

- For the **Cluster Version** option, select the most recent version of Kubernetes.
- Scale the **Machine Type** option to at least **2 vCPUs / 7.5 GB memory**.
- For the **Node Image** option, you must select **Ubuntu**.  You cannot deploy Greenplum with the **Container-Optimized OS (cos)** image.
- Set the **Size** to 4 or more nodes.
- Set **Automatic node repair** to **Disabled**.

In addition to the above, the {{#include ./product_name.md}} deployment process requires the ability to map the host system's `/sys/fs/cgroup` directory onto each container's `/sys/fs/cgroup`. Ensure that no kernel security module (for example, AppArmor) uses a profile that disallows mounting `/sys/fs/cgroup`.

## <a id="context"></a>Setting the Kubernetes Context

After creating your GKE cluster, use the `gcloud` utility to login to GCP, and to set your current project and cluster context:

1. Log into GCP:

    ``` bash
    $ gcloud auth login
    ```

2. Set the current project to the project where you will deploy Greenplum:

    ``` bash
    $ gcloud config set project <project-name>
    ```

3. Set the context to the Kubernetes cluster that you created for Greenplum:

    1. Access GCP Console.
    2. Select **Kubernetes Engine > Clusters**.
    3. Click **Connect** next to the cluster that you configured for Greenplum, and copy the connection command.
    4. On your local client machine, paste the command to set the context to your cluster.  For example:

        ``` bash
        $ gcloud container clusters get-credentials <cluster-name> --zone us-central1-a --project <my-project>
        ```

        ``` bash
        Fetching cluster endpoint and auth data.
        kubeconfig entry generated for <cluster-name>.
        ```

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