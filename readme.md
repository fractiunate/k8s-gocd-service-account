
# Create a Kubernetes Service Account

## Step 1: Create service account in a namespace
kubectl create namespace devops-tools

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service-account
  namespace: devops-tools
EOF
```

## Step 2: Create a Cluster Role
kubectl create serviceaccount api-service-account -n devops-tools

```
cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-cluster-role
  namespace: devops-tools
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

To get the list of available API resources execute the following command.
```
kubectl api-resources
```


## Step 3: Create a CluserRole Binding
Now that we have the ClusterRole and service account, it needs to be mapped together.

Bind the cluster-api-role to api-service-account using a RoleBinding
```
cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: api-cluster-role-binding
subjects:
- namespace: devops-tools 
  kind: ServiceAccount
  name: api-service-account 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-cluster-role 
EOF
```


## Get the secret name associated with the api-service-account

```
kubectl get serviceaccount api-service-account  -o=jsonpath='{.secrets[0].name}' -n devops-tools

    kubectl get secrets  <service-account-token-name>  -o=jsonpath='{.data.token}' -n devops-tools | base64 -d

kubectl get endpoints | grep kubernetes

```

Now that you have the cluster endpoint and the service account token, you can test the API connectivity using the CURL or the postman app.

For example, list all the namespaces in the cluster using curl. Use the token after Authorization: Bearer section.

```
curl -k  https://192.168.49.2/api/v1/namespaces -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkhOSEpEZWpjNHFMVTdGTVg0bzZmckVFQVNfTDdmbERtMkhTQWZvUkhMME0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXZvcHMtdG9vbHMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiYXBpLXNlcnZpY2UtYWNjb3VudC10b2tlbi00ZndwNSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhcGktc2VydmljZS1hY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiN2I3ZTNhMWQtYmZjYy00YWQ5LWI2ZWMtZWRhOGNiOGZjYTgwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRldm9wcy10b29sczphcGktc2VydmljZS1hY2NvdW50In0.jlAHljdityoJng3b-qIfyXPV2Nb63VT1hxqBA2yGVlGp5S_wsmqujzpnhQY2UE9BOe4fO-dlfM9E8i-MJW9ZVRH33e8NOM7hWJr3RKCE_NW7EnCihya6qDDp_IV9vSICeZBUZyt_w2lGc6ZdGkaUGwxvhAWkkYTSpA5uid8ACQkz9hNfSFqwiKpAdy1-MiFKHx4o513j_f2p9qc_epESfQGPD5oDHOBWpo5YV8ORTd7UYPFF2CDlwdKixAEkmClFkiUSBMtfEJrYlFVUt6JHmHthqBKxeuEjyqNkJBlkBGrwhSQ4rwtkOeSaBQEgl95nutYmwoYP8zpVgVhLp_nmBQ"
```

### Minikube
The easiest way to access the Kubernetes API with when running minikube is to use
```
kubectl proxy --port=8084
You can then access the API with

curl http://localhost:8084/api/
```