
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


# Q. 12

## Task
### SECTION: TROUBLESHOOTING
For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

A service account called `deploy-cka19-trb` is created in cluster1 along with a cluster role called `deploy-cka19-trb-role`. This role should have the permissions to get all the deployments under the default namespace. However, at the moment, it is not able to.

Find out what is wrong and correct it so that the `deploy-cka19-trb` service account is able to get deployments under the default namespace.

## Solution
1. Let's see if `deploy-cka19-trb` service account is able to get the deployments.
    ```bash
    kubectl auth can-i get deployments --as=system:serviceaccount:default:deploy-cka19-trb
    ```
    We can see it's not since we are getting `no` in the output.

2. Let's look into the cluster role:
    ```bash
    kubectl get clusterrole deploy-cka19-trb-role -o yaml
    ```
    The rules would be fine but we can see that there is no cluster role binding and service account associated with this. So let's create a cluster role binding.

3. Create the cluster role binding:
    ```bash
    kubectl create clusterrolebinding deploy-cka19-trb-role-binding --clusterrole=deploy-cka19-trb-role --serviceaccount=default:deploy-cka19-trb
    ```

4. Let's see if `deploy-cka19-trb` service account is able to get the deployments now:
    ```bash
    kubectl auth can-i get deployments --as=system:serviceaccount:default:deploy-cka19-trb
    ```

---

# Q. 13

## Task
### SECTION: TROUBLESHOOTING

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

`demo-pod-cka29-trb` pod is stuck in a `Pending` state, look into the issue to fix the same. Make sure the pod is in the `Running` state and stable.

## Solution
1. Look into the POD events:
    ```bash
    kubectl get event --field-selector involvedObject.name=demo-pod-cka29-trb
    ```

    You will see some warnings like:
    ```
    Warning   FailedScheduling   pod/demo-pod-cka29-trb   0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
    ```

2. This seems to be something related to PersistentVolumeClaims, let's check that:
    ```bash
    kubectl get pvc
    ```

    You will notice that `demo-pvc-cka29-trb` is stuck in `Pending` state. Let's dig into it:
    ```bash
    kubectl get event --field-selector involvedObject.name=demo-pvc-cka29-trb
    ```

    You will notice this error:
    ```
    Warning   VolumeMismatch   persistentvolumeclaim/demo-pvc-cka29-trb   Cannot bind to requested volume "demo-pv-cka29-trb": incompatible accessMode
    ```

3. Let's check the PVC and PV details:
    ```bash
    kubectl get pvc demo-pvc-cka29-trb -o yaml
    kubectl get pv demo-pv-cka29-trb -o yaml
    ```

4. Let's re-create the PVC with correct access mode (i.e., `ReadWriteMany`):
    ```bash
    kubectl get pvc demo-pvc-cka29-trb -o yaml > /tmp/pvc.yaml
    vi /tmp/pvc.yaml
    ```

    Under `spec:`, change `accessModes` from `ReadWriteOnce` to `ReadWriteMany`.

5. Delete the old PVC and create a new one:
    ```bash
    kubectl delete pvc demo-pvc-cka29-trb
    kubectl apply -f /tmp/pvc.yaml
    ```

6. Check the pod now:
    ```bash
    kubectl get pod demo-pod-cka29-trb
    ```

    It should be good now.

---

# Q. 14

## Task
### SECTION: STORAGE
For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

We want to deploy a python-based application on the cluster using a template located at `/root/olive-app-cka10-str.yaml` on student-node. However, before you proceed we need to make some modifications to the YAML file as per details given below:

- The YAML should also contain a persistent volume claim with name `olive-pvc-cka10-str` to claim a 100Mi of storage from `olive-pv-cka10-str` PV.
- Update the deployment to add a sidecar container named `busybox`, which can use the `busybox` image (you might need to add a sleep command for this container to keep it running.)
- Share the `python-data` volume with this container and mount the same at path `/usr/src`. Make sure this container only has read permissions on this volume.
- Finally, create a pod using this YAML and make sure the POD is in Running state.

**Note:** Additionally, you can expose a NodePort service for the application. The service should be named `olive-svc-cka10-str` and expose port 5000 with a nodePort value of `32006`. However, inclusion/exclusion of this service won't affect the validation for this task.

## Solution
Update the `olive-app-cka10-str.yaml` template as below:

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: olive-stc-cka10-str
  volumeName: olive-pv-cka10-str
  resources:
    requests:
      storage: 100Mi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      - name: busybox
        image: busybox
        command:
          - "bin/sh"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: python-data
            mountPath: "/usr/src"
            readOnly: true
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str

---
apiVersion: v1
kind: Service
metadata:
  name: olive-svc-cka10-str
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 32006
  selector:
    app: olive-app-cka10-str
```

Apply the template:

```bash
kubectl apply -f olive-app-cka10-str.yaml
```

---

# Q. 15

## Task
### SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

A pod called `elastic-app-cka02-arch` is running in the default namespace. The YAML file for this pod is available at `/root/elastic-app-cka02-arch.yaml` on the student-node. The single application container in this pod writes logs to the file `/var/log/elastic-app.log`.

One of our logging mechanisms needs to read these logs to send them to an upstream logging server but we don't want to increase the read overhead for our main application container so recreate this POD with an additional sidecar container that will run along with the application container and print to the STDOUT by running the command `tail -f /var/log/elastic-app.log`. You can use the `busybox` image for this sidecar container.

## Solution
Recreate the pod with a new container called `sidecar`. Update the `/root/elastic-app-cka02-arch.yaml` YAML file as shown below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: elastic-app-cka02-arch
spec:
  containers:
  - name: elastic-app
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      mkdir /var/log; 
      i=0;
      while true;
      do
        echo "$(date) INFO $i" >> /var/log/elastic-app.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: sidecar
    image: busybox:1.28
    args: [/bin/sh, -c, 'tail -f  /var/log/elastic-app.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

Next, recreate the pod:

```bash
kubectl replace -f /root/elastic-app-cka02-arch.yaml --force --context cluster3
```

---

# Q. 16

## Task
### SECTION: TROUBLESHOOTING

For this question, please set the context to cluster4 by running:

```bash
kubectl config use-context cluster4
```

There is some issue on the student-node preventing it from accessing the `cluster4` Kubernetes Cluster.

Troubleshoot and fix this issue. Make sure that you are able to run the kubectl commands (For example: `kubectl get node --context=cluster4`) from the student-node.

## Solution
1. Check if the `kubectl` context is correct:
    ```bash
    kubectl config use-context cluster4
    ```

2. If it is incorrect, reconfigure the `kubectl` context using the correct kubeconfig file.

---

# Q. 17

## Task
### SECTION: SERVICE NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

We have an external webserver running on student-node which is exposed at port 9999. We have created a service called `external-webserver-cka03-svcn` that can connect to our local webserver from within the Kubernetes cluster3 but at the moment it is not working as expected.

Fix the issue so that other pods within cluster3 can use `external-webserver-cka03-svcn` service to access the webserver.

## Solution
1. Check if the webserver is working or not:
    ```bash
    curl student-node:9999
    ```

2. Describe the service:
    ```bash
    kubectl describe svc external-webserver-cka03-svcn
    ```

    The output will indicate there are no endpoints.

3. Create the endpoint manually:
    ```bash
    export IP_ADDR=$(ifconfig eth0 | grep inet | awk '{print $2}')
    kubectl --context cluster3 apply -f - <<EOF
    apiVersion: v1
    kind: Endpoints
    metadata:
      name: external-webserver-cka03-svcn
    subsets:
      - addresses:
          - ip: $IP_ADDR
        ports:
          - port: 9999
    EOF
    ```

4. Test if it works:
    ```bash
    kubectl --context cluster3 run --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-cka03-svcn
    ```

---

# Q. 18

## Task
### SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a generic secret called `db-user-pass-cka17-arch` in the default namespace on cluster1 using the contents of the file `/opt/db-user-pass` on the student-node.

## Solution
Create the required secret:

```bash
kubectl create secret generic db-user-pass-cka17-arch --from-file=/opt/db-user-pass
```

---

# Q. 19

## Task
### SECTION: TROUBLESHOOTING

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

One of the nginx-based pods called `cyan-pod-cka28-trb` is running under the `cyan-ns-cka28-trb` namespace and it is exposed within the cluster using `cyan-svc-cka28-trb` service.

This is a restricted pod, so a network policy called `cyan-np-cka28-trb` has been created in the same namespace to apply some restrictions on this pod.

Two other pods called `cyan-white-cka28-trb` and `cyan-black-cka28-trb` are also running in the default namespace.

The nginx-based app running on the `cyan-pod-cka28-trb` pod is exposed internally on the default nginx port (80).

### Expectation: 
This app should only be accessible from the `cyan-white-cka28-trb` pod.

### Problem: 
This app is not accessible from anywhere.

Troubleshoot this issue and fix the connectivity as per the requirement listed above.

## Solution
1. Edit the network policy:
    ```bash
    kubectl edit networkpolicy cyan-np-cka28-trb -n cyan-ns-cka28-trb
    ```

    Under `spec: -> egress:`, you will notice there is no `cidr` block, and under `ports`, you will notice it is using port `8080`, but the app is running on port `80`.

2. Update the policy:
    ```yaml
    - ports:
      - port: 80
        protocol: TCP
    ```

3. Also, add a pod selector under ingress:
    ```yaml
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: default
      podSelector:
        matchLabels:
          app: cyan-white-cka28-trb
    ```

4. Verify connectivity:
    - From `cyan-white-pod-cka28-trb`:
      ```bash
      kubectl exec -it cyan-white-cka28-trb -- sh
      curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local
      ```

    - From `cyan-black-cka28-trb` (should not work):
      ```bash
      kubectl exec -it cyan-black-cka28-trb -- sh
      curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local
      ```

---

# Q. 20

## Task
### SECTION: TROUBLESHOOTING
For this question, please set the context to cluster4 by running:

```bash
kubectl config use-context cluster4
```

We tried to schedule `grey-cka21-trb` pod on cluster4, which was supposed to be deployed by the Kubernetes scheduler so far, but somehow it's stuck in a Pending state. Look into the issue and fix the same, make sure the pod is in Running state.

You can SSH into the cluster4 using `ssh cluster4-controlplane` command.

## Solution
1. Check the pod status:
    ```bash
    kubectl get pod --context=cluster4
    ```

2. Check the logs and events:
    ```bash
    kubectl logs grey-cka21-trb --context=cluster4
    kubectl get event --context=cluster4 --field-selector involvedObject.name=grey-cka21-trb
    ```

3. Check the status of the kube-scheduler pod:
    ```bash
    kubectl get pod --context=cluster4 -n kube-system
    ```

4. If the kube-scheduler pod is crashing, check the logs:
    ```bash
    kubectl logs kube-scheduler-cluster4-controlplane --context=cluster4 -n kube-system
    ```

5. Fix the kube-scheduler manifest:
    ```bash
    ssh cluster4-controlplane
    ls /etc/kubernetes/scheduler.config
    vi /etc/kubernetes/manifests/kube-scheduler.yaml
    ```

6. Replace `/etc/kubernetes/scheduler.config` with `/etc/kubernetes/scheduler.conf` in the manifest file.

7. Restart the kube-scheduler pod:
    ```bash
    kubectl get pod -A
    ```

8. `grey-cka21-trb` pod should now be running.

```
