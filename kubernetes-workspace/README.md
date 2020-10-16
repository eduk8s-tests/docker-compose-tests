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

Mounting local directory
------------------------

By default the home directory for the terminal of the workshop environment
resides in the ephemeral filesystem of the container. This means any work
done in the filesystem from the environment will be lost when the
environment is shutdown.

If you want to ensure work is saved back into the persistent storage of
the local host, you will need to mount a directory onto the home directory
for the user running in the environment. This can be done using:

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
    - .:/home/eduk8s
    extra_hosts:
    - workshop-console:127.0.0.1
    - workshop-editor:127.0.0.1
```

Note that unless running this in a situation such as a VM just for this
purpose, you should avoid mounting the home directory of the local user
into the container. This is because any configuration in hidden directories
of the home directory may be overridden by applications running inside of
the container. This could interfere with your ability to then run the
same applications on the local host if they are incompatible systems.

In the example above we still mounted the `kubeconfig.yaml` in explicitly.
If you were mounting in the home directory from a local user, and that user
had an existing configuration for Kubernetes in `~/.kube/config`, it would
be enough to mount the home directory, and you wouldn't need to explicitly
mount the Kubernetes config file.

Changing the hostname
---------------------

To access the environment you must use the `127-0-0-1.nip.io` domain. By
default the subdomain within that must be called `workshop`. If you want
to use a different subdomain name, you must change the `docker-compose`
configuration.

To use `jumpbox` instead of `workshop` as the sub domain, you would need
to use:

```
version: '2.0'
services:
  workshop:
    image: quay.io/eduk8s/base-environment:201007.062234.403f0f1
    environment:
    - SESSION_NAMESPACE=jumpbox
    - INGRESS_PORT_SUFFIX=:10080
    ports:
    - "10080:10080"
    volumes:
    - ./workshop.yaml:/opt/eduk8s/config/workshop.yaml
    - ./kubeconfig.yaml:/opt/eduk8s/config/kubeconfig.yaml
    extra_hosts:
    - jumpbox-console:127.0.0.1
    - jumpbox-editor:127.0.0.1
```

That is, set `SESION_NAMESPACE` to the name you want to use, and change
the names of the `extra_hosts` settings so they use the new name.

The environment would then be accessed as:

```
http://jumpbox.127-0-0-1.nip.io:10080
```

If you don't make both changes, then although you will be able to access
the terminals, you will not be able to access the console or editor.
