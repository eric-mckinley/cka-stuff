# Youtube Cluster

[...Back](../../)

### Deploy Cluster from Scratch

Automate the steps outlined in the youtube video tutorial below.

Youtube: [How to Deploy a Kubernetes Cluster from Scratch](https://www.youtube.com/watch?v=t4H6hdvB9iQ)

Nice simplified tutorial based on deploying the individual components of kubernetes cluster and 
explaining what each component does and how they interact with each.

Tutorial based on older *v1.9.11* version of kubernetes for simplicity.

### Vagrant + Ansible

Repo provides a Vagrant setup for creating and provisioning with ansible the staps of the tutorial.

The vagrant file will create the master node which also acts as a worker node, and adds two extra worker nodes to the cluster
that is not in the tutorial.

### Execute

Install Vagrant + Ansible, suggested versions below:

```shell script
vagrant version                                                                                                                    ✔  youtube ⎈  23:33:28  
Installed Version: 2.2.9
Latest Version: 2.2.14
```

```shell script
ansible --version                                                                                                                5 ✘  youtube ⎈  10:32:25  
ansible 2.9.7
```

Navigate to folder and run vagrant:
```shell script
cd clusters/youtube-video/
vagrant up
```

Once vagrant finishes test if cluster is up using local kube context
```shell script
kubectl get nodes --kubeconfig=youtube-kube.config                                                          ✔  youtube ⎈  10:51:45  
NAME                 AGE
youtube-k8s-worker-1               8m32s
youtube-k8s-worker-2               5m36s
youtube-k8s-master   11m
```

Create a test nginx deployment and NodePort Service (port 30050) 
```shell script
kubectl create -f test-setup/example-deployment.yaml --kubeconfig=youtube-kube.config

kubectl create -f test-setup/example-service.yaml --kubeconfig=youtube-kube.config
```

Verify service up by curling any node in the cluster to see default nginx 
```shell script
curl 192.168.50.11:30050

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Verify pods running on each node

ssh into a node or master, switch to root and check running docker containers.
Should find the running nginx container along with the pod pause container.

```shell script
vagrant ssh youtube-k8s-worker-1

sudo -i
docker ps

CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS               NAMES
d0fe97fe014a        nginx                                      "/docker-entrypoint.…"   24 minutes ago      Up 24 minutes                           k8s_nginx_example-deployment-6bb65997d8-z5pnn_default_7db5290c-6220-11eb-9593-0295c5cf8a88_0
21a3490c40d9        gcr.io/google_containers/pause-amd64:3.0   "/pause"                 24 minutes ago      Up 24 minutes                           k8s_POD_example-deployment-6bb65997d8-z5pnn_default_7db5290c-6220-11eb-9593-0295c5cf8a88_0
```
