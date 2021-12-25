# Service Account for jenkins

## Creating Service Account in Kubernetes

```yaml
# service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <service-account-name>
  namespace: <namespace>
```

then apply

```bash
kubectl apply -f service-account.yaml
```

you can check the yaml of your created service account by

```bash
kubectl get serviceaccount <service-account-name> --namespace <namespace> -o yaml
```

then you will see an secret fields

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2021-06-02T04:22:31Z"
  name: <service-account-name>
  namespace: <namespace>
  resourceVersion: "290"
  selfLink: /api/v1/namespaces/<namespace>/serviceaccounts/<service-account-name>
  uid: 3d2f915b-3b82-457a-ba88-29fb2b3d1f89
secrets:
- name: <secret-name>
```

survey what's inside the secret

```bash
kubectl get secret <secret-name> --namespace <namespace> -o yaml
```

the output will look like

```yaml
apiVersion: v1
data:
  ca.crt: [cert..]
  namespace: [base64 encoded namespace]
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsWkdSMGR2TTBJdGFXc3pOMFZuYWpKSWIwMXpTR2hrTmtvemJsSjZOVmM1TTNacWVsTkJkazk2VWpRaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUowWlhOMElpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WldOeVpYUXVibUZ0WlNJNkluUmxjM1F0YzJFdGRHOXJaVzR0YW1OeE4yZ2lMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2lkR1Z6ZEMxellTSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqaG1ZakZrWmpBeUxUWmlZV0l0TkRObU1TMWhabVU0TFRObFpHVTFaalJoTVRjeE1DSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHAwWlhOME9uUmxjM1F0YzJFaWZRLlJxeXFaQUx2M19qeUNhZnMtbF9pa0ctQXA4RFNPZjRoT2psaXRkODYxRzBDcXpmY0VYLW1GbkxlbFNHSDg1Z1MyeHR2SmxBci0zcG9sZjdnMEJCNlRKRHl0Ny1rN2V2a2dyTzVGbG85b2R3NVVqeWpLdFd6czdLbjBsci1DUDU4OXV1ekFPRTlIdHpnZTMwYlYxQWVmdEtZUmNsdUlxb1hVRk1ZWko5a3hXZk1hZkh4UlVOX2M1TkppRVNibktCeXhyelFqNV9xUVNoMFFKbkdxVjdCTWJJcksyZW02ME5hLWFJNWpCMDNxYnJsM2ZBUk91ZEphTW5BWEwwZDQxSkZJU0o2TW5qSlpKZlFXUWpTOEFNWlNMVXF4TlpXZlcwc2N0OGdhX3dLT0c4cUVMd2YxLWVJdFlZa1pRckRONVk5SEllc2U5QjZvRjFWZUtlcVdKdlY2QQ==
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: <service-account-name>
    kubernetes.io/service-account.uid: 83d2f915b-3b82-457a-ba88-29fb2b3d1f89
  creationTimestamp: "2021-06-02T04:22:31Z"
  name: <secret-name>
  namespace: <namespace>
  resourceVersion: "1652"
  selfLink: /api/v1/namespaces/<namespace>/secrets/<secret-name>
  uid: 45956fb0-2449-48c8-a0d3-d1a088a1f4ea
type: kubernetes.io/service-account-token
```

### or if you want only token

```bash
kubectl get secret $(kubectl get serviceaccount <service-account-name> \
 --namespace <namespace> -o jsonpath={.secrets[0].name}) \
 --namespace <namespace> -o jsonpath={.data.token} | base64 --decode
```

### receiving ca.crt

```bash
kubectl get secret $(kubectl get serviceaccount <service-account-name> \
 --namespace <namespace> -o jsonpath={.secrets[0].name}) \
 --namespace <namespace> -o jsonpath={.data.'ca\.crt'} | base64 --decode
```

## Creating Role or ClusterRole for Service Account

we will bind the role to the user or service account

after we create service account, we can check SA's permission by running the command

```bash
kubectl auth can-i list pods \
 --as=system:serviceaccount:<namespace>:<service-account-name> 
 --namespace <namespace>
>> no
```

Now We have 2 way to make it done

### 1. with clusterrole

```bash
kubectl create clusterrolebinding <clsuter-role-binding-name> \
 --clusterrole=cluster-admin --namespace <namespaec> \
 --serviceaccount=<namespace>:<service-account-name>
```

re-check permission

```bash
kubectl auth can-i list pods \
 --as=system:serviceaccount:<namespace>:<service-account-name> 
 --namespace <namespace>
>> yes
```

### 2. with rolebinding

Then we will create role.yaml

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: <role-name>
	namespace: <namespace>
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
```

```yaml
kubectl apply -f role.yaml
```

Then we create rolebinding.yaml

```yaml
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: <role-binding-name>
    namespace: <namespace>
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: <role-name>
  subjects:
  - kind: ServiceAccount
    name: <service-account-name>
    namespace: <service-account-namespace>
```

```yaml
kubectl apply -f rolebinding.yaml
```

re-check permission

```bash
kubectl auth can-i list pods \
 --as=system:serviceaccount:<namespace>:<service-account-name> 
 --namespace <namespace>
>> yes
```

## Create credential (secret file) to gain access for jenkins

receiving token

```bash
kubectl get secret $(kubectl get serviceaccount <service-account-name> \
 --namespace <namespace> -o jsonpath={.secrets[0].name}) \
 --namespace <namespace> -o jsonpath={.data.token} | base64 --decode
```

receiving certificated

```bash
kubectl get secret $(kubectl get serviceaccount <service-account-name> \
 --namespace <namespace> -o jsonpath={.secrets[0].name}) \
 --namespace <namespace> -o jsonpath={.data.'ca\.crt'}
```

secret file call config.yaml

```yaml
apiVersion: v1
kind: Config
users:
- name: <CONFIG NAME(up to you) matching:1>
  user:
    token: <BASE64 DECODED TOKEN of SA>*
clusters:
- cluster:
    certificate-authority-data: <BASE64 ENCODEED CERTIFICATED>*
    server: <CLUSTER URL>*
  name: <CLUSTER NAME>
contexts:
- context:
    cluster: <CLUSTER NAME>
    namespace: <NAMESPACE WHERE SA IS SATIT>*
    user: <CONFIG NAME(up to you) matching:1> 
  name: <CONTEXT CONFIG NAME(up to you) matching:2>
current-context: <CONTEXT CONFIG NAME(up to you) matching:2>
```

```yaml
export KUBECONFIG=config.yaml
kubectl get pods -A
```

then add this file to jenkins credentail, type secret file