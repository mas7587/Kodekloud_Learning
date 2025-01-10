```markdown
# Q. 1
## Task
### SECTION: SCHEDULING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

One of our Junior DevOps engineers has deployed a pod `nginx-wl06` on the `cluster3-controlplane` node. However, while specifying the resource limits, instead of using Mebibyte as the unit, Gebibyte was used.

As a result, the node doesn't have sufficient resources to deploy this pod and it is stuck in a pending state.

**Fix the units and re-deploy the pod with about 100Mi** (Delete and recreate the pod if needed).

### Solution

1. Set the correct context:
    ```bash
    kubectl config use-context cluster3
    ```

2. Run the following command to check the pending pods on all the namespaces:
    ```bash
    kubectl get pods -A
    ```

3. After that, inspect the pod Events:
    ```bash
    kubectl get pods -A | grep -i pending
    kubectl describe po nginx-wl06
    ```

4. Use the `kubectl edit` command to update the values from Gi to Mi:
    ```bash
    kubectl edit po nginx-wl06
    ```

5. It will save the temporary file under the `/tmp/` directory. Use the `kubectl replace` command as follows:
    ```bash
    kubectl replace -f /tmp/kubectl-edit-xxxx.yaml --force
    ```

It will delete the existing pod and will re-create it again with new changes.

---

# Q. 2
## Task
### SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE

Find the pod that consumes the most memory and store the result to the file `/opt/high_memory_pod` in the following format: `cluster_name,namespace,pod_name`.

The pod could be in any namespace in any of the clusters that are currently configured on the student-node.

**NOTE:** It's recommended to wait for a few minutes to allow deployed objects to become fully operational and start consuming resources.

### Solution

Check out the metrics for all pods across all clusters:

```bash
student-node ~ ➜ kubectl top pods -A --context cluster1 --no-headers | sort -nr -k4 | head -1
kube-system       kube-apiserver-cluster1-controlplane            48m   262Mi   

student-node ~ ➜ kubectl top pods -A --context cluster2 --no-headers | sort -nr -k4 | head -1
kube-system   kube-apiserver-cluster2-controlplane            44m   258Mi   

student-node ~ ➜ kubectl top pods -A --context cluster3 --no-headers | sort -nr -k4 | head -1
default       backend-cka06-arch                        205m   596Mi   

student-node ~ ➜ kubectl top pods -A --context cluster4 --no-headers | sort -nr -k4 | head -1
kube-system   kube-apiserver-cluster4-controlplane            43m   266Mi   
```

Using this, find the pod that uses the most memory. In this case, it is `backend-cka06-arch` on `cluster3`.

Save the result in the correct format to the file:

```bash
student-node ~ ➜ echo cluster3,default,backend-cka06-arch > /opt/high_memory_pod
```

---

# Q. 3
## Task
### SECTION: SERVICE NETWORKING

For this question, please set the context to `cluster3` by running:

```bash
kubectl config use-context cluster3
```

**Part I:**
Create a `ClusterIP` service, i.e., `service-3421-svcn` in the `spectra-1267` namespace which should expose the pods namely `pod-23` and `pod-21` with port set to `8080` and `targetport` to `80`.

**Part II:**
Store the pod names and their IP addresses from the `spectra-1267` namespace at `/root/pod_ips_cka05_svcn`, where the output is sorted by their IPs.

Please ensure the format as shown below:

```
POD_NAME        IP_ADDR
pod-1           ip-1
pod-3           ip-2
pod-2           ip-3
...
```

### Solution

Switching to cluster3:

```bash
kubectl config use-context cluster3
```

The easiest way to route traffic to a specific pod is by using labels and selectors. List the pods along with their labels:

```bash
student-node ~ ➜ kubectl get pods --show-labels -n spectra-1267
```

Looks like there are a lot of pods created to confuse us, but we are only concerned with the labels of `pod-23` and `pod-21`.

```bash
student-node ~ ➜ kubectl get pod -l mode=exam,type=external -n spectra-1267
```

Now that we have identified the pods `pod-23` and `pod-21`, we can proceed further with the creation of the service:

```bash
student-node ~ ➜ kubectl create service clusterip service-3421-svcn -n spectra-1267 --tcp=8080:80 --dry-run=client -o yaml > service-3421-svcn.yaml
```

Modify the service definition with selectors as required before applying to the K8s cluster:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: service-3421-svcn
  name: service-3421-svcn
  namespace: spectra-1267
spec:
  ports:
  - name: 8080-80
    port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    mode: exam
    type: external
  type: ClusterIP
status:
  loadBalancer: {}
```

Finally, apply the service definition:

```bash
student-node ~ ➜ kubectl apply -f service-3421-svcn.yaml
```

To store all the pod names along with their IPs:

```bash
student-node ~ ➜ kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_cka05_svcn
```

---

# Q. 4
## Task
### SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE

For this question, please set the context to `cluster3` by running:

```bash
kubectl config use-context cluster3
```

A pod called `logger-complete-cka04-arch` has been created in the `default` namespace. Inspect this pod and save **ALL** the logs to the file `/root/logger-complete-cka04-arch` on the student-node.

### Solution

Run the command:

```bash
student-node ~ ➜ kubectl logs logger-complete-cka04-arch --context cluster3 > /root/logger-complete-cka04-arch
```

View the logs:

```bash
student-node ~ ➜ head /root/logger-complete-cka04-arch
```

---

# Q. 5
## Task
### SECTION: SCHEDULING

For this question, please set the context to `cluster1` by running:

```bash
kubectl config use-context cluster1
```

Create a deployment called `app-wl01` using the `nginx` image and scale the application pods to 2.

### Solution

Run the following commands:

```bash
kubectl create deployment app-wl01 --image=nginx --replicas=2
```

Cross-verify the deployed resources:

```bash
kubectl get pods,deployments
```

---

# Q. 6
## Task
### SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE

Find the node across all clusters that consumes the most memory and store the result to the file `/opt/high_memory_node` in the following format: `cluster_name,node_name`.

The node could be in any clusters that are currently configured on the student-node.

### Solution

Check out the metrics for all nodes across all clusters:

```bash
student-node ~ ➜ kubectl top node --context cluster1 --no-headers | sort -nr -k4 | head -1
cluster1-controlplane   124m   1%    768Mi   1%
```

Repeat for other clusters, and save the result:

```bash
student-node ~ ➜ echo cluster3,cluster3-controlplane > /opt/high_memory_node
```

---

# Q. 7
## Task
### SECTION: TROUBLESHOOTING

For this question, please set the context to `cluster1` by running:

```bash
kubectl config use-context cluster1
```

The `green-deployment-cka15-trb` deployment is having some issues since the corresponding POD is crashing and restarting multiple times continuously.

Investigate the issue and fix it, making sure the POD is in a running state and its stable (i.e., NO RESTARTS!).

### Solution

1. List the pods to check its status:
    ```bash
    kubectl get pod
    ```

2. Check the logs:
    ```bash
    kubectl logs -f green-deployment-cka15-trb-xxxx
    ```

3. You may see logs related to memory issues. Try deleting the pod:
    ```bash
    kubectl delete pod green-deployment-cka15-trb-xxxx
    ```

4. Investigate memory resource settings:
    ```bash
    kubectl edit deploy green-deployment-cka15-trb
    ```

5. Increase the memory limit and apply the changes.

---

# Q. 8
## Task
### SECTION: SCHEDULING

For this question, please set the context to `cluster3` by running:

```bash
kubectl config use-context cluster3
```

A manifest file is available at `/root/app-wl03/` on the student-node. There are some issues with the file; hence couldn't deploy a pod on the `cluster3-controlplane` node.

After fixing the issues, deploy the pod, and it should be in a running state.

**NOTE:** Ensure that the existing limits are unchanged.

### Solution

Set the correct context:

```bash
kubectl config use-context cluster3
```

Move to the given directory:

```bash
cd /root/app-wl03/
```

Open the manifest file and fix the memory request issue.

```yaml
resources:
  requests:
    memory: 100Mi
  limits:
    memory: 100Mi
```

Apply the fixed manifest:

```bash
kubectl create -f app-wl03.yaml
```

---

# Q. 9
## Task
### SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE

For this question, please set the context to `cluster3` by running:

```bash
kubectl config use-context cluster3
```

Run a pod called `looper-cka16-arch` using the `busybox` image that runs the `while` loop: `while true; do echo hello; sleep 10;done`. This pod should be created in the `default` namespace.

### Solution

Create the pod definition:

```bash
student-node ~ ➜ vi looper-cka16-arch.yaml
```

Content of the YAML file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: looper-cka16-arch
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo hello; sleep 10;done"]
```

Create the pod:

```bash
student-node ~ ➜ kubectl apply -f looper-cka16-arch.yaml --context cluster3
```

---

# Q. 10
## Task
### SECTION: STORAGE

For this question, please set the context to `cluster1` by running:

```bash
kubectl config use-context cluster1
```

A pod definition file is created at `/root/peach-pod-cka05-str.yaml` on the student-node. Update this manifest file to create a `persistent volume claim` called `peach-pvc-cka05-str` to claim 100Mi of storage from `peach-pv-cka05-str` PV (this is already created). Use the access mode `ReadWriteOnce`.

Further, add `peach-pvc-cka05-str` PVC to `peach-pod-cka05-str` POD and mount the volume at `/var/www/html` location. Ensure that the pod is running and the PV is bound.

### Solution

Update the manifest file:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: peach-pvc-cka05-str
spec:
  volumeName: peach-pv-cka05-str
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: peach-pod-cka05-str
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
      - mountPath: "/var/www/html"
        name: nginx-volume
  volumes:
    - name: nginx-volume
      persistentVolumeClaim:
        claimName: peach-pvc-cka05-str
```

Apply the updated template:

```bash
kubectl apply -f /root/peach-pod-cka05-str.yaml
```

---

# Q. 11
## Task
### SECTION: SERVICE NETWORKING

For this question, please set the context to `cluster3` by running:

```bash
kubectl config use-context cluster3
```

Create a deployment named `hr-web-app-cka08-svcn` using the image `kodekloud/webapp-color` with 2 replicas.

Expose the `hr-web-app-cka08-svcn` as service `hr-web-app-service-cka08-svcn` application on port 30082 on the nodes of the cluster. The web application listens on port 8080.

### Solution

Switch to `cluster3`:

```bash
kubectl config use-context cluster3
```

Create the deployment:

```bash
kubectl create deployment hr-web-app-cka08-svcn --image=kodekloud/webapp-color --replicas=2
```

Generate the service definition file:

```bash
kubectl expose deployment hr-web-app-cka08-svcn --type=NodePort --port=8080 --name=hr-web-app-service-cka08-svcn --dry-run=client -o yaml > hr-web-app-service-cka08-svcn.yaml
```

Modify the generated file to add the `nodePort` field with the given port number and apply the service:

```bash
kubectl apply -f hr-web-app-service-cka08-svcn.yaml
```
```
