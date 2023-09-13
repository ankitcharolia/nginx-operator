# Nginx Operator
A Helm based Nginx Operator

Helm based operators are quite easy to build and are preferred to deploy a **stateless application** using operator pattern.

### How to generate boilerplate code for Helm based operator and build nginx operator
```bash
# initialize the project
operator-sdk init --domain charolia.io --plugins helm

# Create a simple nginx API using Helm’s built-in chart boilerplate
operator-sdk create api --group nginx --version v1alpha1 --kind Nginx
```


**Configure the operator’s image registry**
```bash
-IMG ?= controller:latest
+IMG ?= $(IMAGE_TAG_BASE):$(VERSION)
```

**Build and push your operator’s image**
```bash
make docker-build docker-push
```

**Make sure to login to redhat registry**
```bash
docker login registry.redhat.io
Username: ankitcharolia
Password: ******
Login Succeeded
```

**Create a kubernetes secret for docker config json**
```bash
$ kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/acharolia/.docker/config.json --type=kubernetes.io/dockerconfigjson (-n nginx-operator-system)
secret/regcred created
```

**NOTE:** Use docker config json as a secret to pull the image from redhat registry
```bash
spec:
  template:
    spec:
      imagePullSecrets:
      - name: regcred
```
### Run the Operator
```bash
make deploy
```
**NOTE:** This deployments the CRDs to the Kubernetes Cluster.

```bash
$ kubectl get crd | grep nginx
nginxes.nginx.charolia.io                            2023-09-13T19:50:54Z
```

**Create a Nginx CR**
```bash
kubectl apply -f config/samples/nginx_v1alpha1_nginx.yaml
```

### Clean up the resources:
```bash
kubectl delete -f config/samples/demo_v1alpha1_nginx.yaml
```

**Note:** Make sure the above custom resource has been deleted before proceeding to run make undeploy, as helm-operator’s controller adds finalizers to the custom resources. Otherwise your cluster may have dangling custom resource objects that cannot be deleted.

```bash
make undeploy
```