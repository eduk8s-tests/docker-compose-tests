Kubernetes Workspace
====================

This test is for demonstrating how one could use the workshop environment
for creating a web based workspace for interacting with a Kubernetes
cluster. The workspace provides terminals, a web console for interacting
with the Kubernetes cluster, and the VS Code editor.

This could be run locally on your own machine to access a cluster visible
from your local machine, or it could be run on a virtual machine in the
data center where the cluster is running, with access to the workspace web
interface achieved by using SSH tunneling to the virtual machine in the
data center.

Workshop Definition
-------------------

The abbreviated workshop definition used to configure the environment can
be found in the file called `workshop.yaml`. It should contain:

```
apiVersion: training.eduk8s.io/v1alpha2
kind: Workshop
metadata:
  name: kubernetes-workbench
spec:
  session:
    applications:
      workshop:
        enabled: false
      terminal:
        enabled: true
        layout: split
      console:
        enabled: true
        vendor: octant
      editor:
        enabled: true
```

The configuration disables the displaying of any workshop content as it is
not required. It then enables two terminals, the Octant web console for
accessing the Kubernetes cluster, and the VS Code editor.

Nothing else of what would normally be added into the workshop definition
file when running a workshop is needed.

Docker Compose
--------------

The file for configuring `docker-compose` is called `docker-compose.yaml`.
It should contain:

```
version: '2.0'
services:
  workshop:
    image: quay.io/eduk8s/base-environment:201007.062234.403f0f1
    environment:
    - INGRESS_PORT_SUFFIX=:10080
    ports:
    - "10080:10080"
    volumes:
    - ./workshop.yaml:/opt/eduk8s/config/workshop.yaml
    - ~/.kube/config:/opt/eduk8s/config/kubeconfig.yaml
    - ~/.minikube:/Users/graham/.minikube
    extra_hosts:
    - workshop-console:127.0.0.1
    - workshop-editor:127.0.0.1
```

This specific example is set up to work against a local `minikube` instance.

When `minikube` is being used, the `~/.kube/config` file includes paths to
certificates for accessing the Kubernetes cluster under the `~/.minikube`
directory. Those paths include the full path of the users home directory,
so the `~/.minikube` directory needs to be mounted at the same path for the
user inside of the container.

If testing this on `minikube` you will therefore need to modify the mount
point for the `~/.minikube` directory inside of the container so it matches
the path to the home directory of your user.

If you aren't using `minikube` and have a self contained `kubeconfig.yaml`
file in the current directory, you could instead change it to:

```
version: '2.0'
services:
  workshop:
    image: quay.io/eduk8s/base-environment:201007.062234.403f0f1
    environment:
    - INGRESS_PORT_SUFFIX=:10080
    ports:
    - "10080:10080"
    volumes:
    - ./workshop.yaml:/opt/eduk8s/config/workshop.yaml
    - ./kubeconfig.yaml:/opt/eduk8s/config/kubeconfig.yaml
    extra_hosts:
    - workshop-console:127.0.0.1
    - workshop-editor:127.0.0.1
```

Starting Environment
--------------------

After changing the `docker-compose` configuration to match your setup, to
start the environment run:

```
docker-compose up --renew-anon-volumes
```

To access the environment, open your local web browser on the URL:

```
http://workshop.127-0-0-1.nip.io:10080
```

You must use a `nip.io` style hostname as shown, you cannot use `127.0.0.1`
or `localhost`.

Using an SSH Tunnel
-------------------

If you are running the environment in a VM in a data center and only have
SSH access to that VM, with only that VM having visibility of the Kubernetes
cluster, you can use an SSH tunnel to access the VM and proxy to the
environment.

To do this, on your local machine which has SSH access to the VM in the data
center, run:

```
ssh -N -L 10080:localhost:10080 username@remotehost
```

Replace `username@remotehost` with the login details of the VM. If necessary,
enter any password for the user when prompted.

To access the environment, you can then open your local web browser on the
URL:

```
http://workshop.127-0-0-1.nip.io:10080
```
