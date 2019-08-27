# COUCHBASE AUTONOMOUS OPERATOR

This guide walks through the recommended procedure for installing the Couchbase Autonomous Operator on an open source Kubernetes cluster that has RBAC enabled.

[prerequisites](https://docs.couchbase.com/operator/current/prerequisite-and-setup.html)

**Start Minikube**
```
minikube start --cpus 4 --memory 8192
```

**Install the Admission Controller**
Create the [admission controller](https://docs.couchbase.com/operator/current/install-admission-controller.html)
```
kubectl apply -f admission.yaml
```

**Install the CRD**
Install the [custom resource definition (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions)
that describes the **CouchbaseCluster** resource type. 
```
kubectl apply -f crd.yaml
```

**Create a Role**
To create the role for the Operator, run the following command:
```
kubectl apply -f operator-role.yaml
```

**Create a Service Account**
After the cluster role is created, you need to create a service account in the namespace where you are installing the Operator, and then assign the cluster role to that service account using a [cluster role binding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#api-overview). (In this example, the service account is created in the namespace called default.)
```
kubectl create serviceaccount couchbase-operator --namespace default
```

To assign the role to the service account:
```
kubectl create rolebinding couchbase-operator --role couchbase-operator --serviceaccount default:couchbase-operator
```

**Create the Operator**
```
kubectl apply -f operator-deployment.yaml
```

Running this command downloads the Operator Docker image (specified in the operator-deployment.yaml file) and creates a [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), which manages a single instance of the Operator. The Operator uses a deployment so that it can restart if the pod it’s running in dies.

**Create the Secret**
```
kubectl apply -f secret.yaml
```

**Deploying the Couchbase Cluster**
```
kubectl apply -f couchbase-cluster.yaml
```

**Check the Status of the Deployment**
```
kubectl get deployments
```

**Expose Couchbase Admin Console**
```
kubectl port-forward cb-example-0000 8091:8091
```
Login: Administrator
Password: password

1) Go to localhost:8091
2) Click on the Security tab on the left menu
3) Click on Add User on the top right
4) Create a local admin account with the login: admin and password: password

**Sync Gateway**
```
cd sync-gateway-config/ && kubectl create secret generic sgw-config --from-file sgw-config.json && cd ..
```

```
kubectl apply -f sgw-deployment.yaml
```