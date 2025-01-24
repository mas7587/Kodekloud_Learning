# Q. 1

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



A template to create a Kubernetes pod is stored at /root/red-probe-cka12-trb.yaml on the student-node. However, using this template as-is is resulting in an error.


Fix the issue with this template and use it to create the pod. Once created, watch the pod for a minute or two to make sure its stable i.e, it's not crashing or restarting.


Make sure you do not update the args: section of the template.

Solution
Try to apply the template
kubectl apply -f red-probe-cka12-trb.yaml 

You will see error:

error: error validating "red-probe-cka12-trb.yaml": error validating data: [ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): unknown field "command" in io.k8s.api.core.v1.HTTPGetAction, ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): missing required field "port" in io.k8s.api.core.v1.HTTPGetAction]; if you choose to ignore these errors, turn validation off with --validate=false

From the error you can see that the error is for liveness probe, so let's open the template to find out:

    vi red-probe-cka12-trb.yaml

    Under livenessProbe: you will see the type is httpGet however the rest of the options are command based so this probe should be of exec type.

Change httpGet to exec
Try to apply the template now
    kubectl apply -f red-probe-cka12-trb.yaml 

Cool it worked, now let's watch the POD status, after few seconds you will notice that POD is restarting. So let's check the logs/events

kubectl get event --field-selector involvedObject.name=red-probe-cka12-trb

You will see an error like:

21s         Warning   Unhealthy   pod/red-probe-cka12-trb   Liveness probe failed: cat: can't open '/healthcheck': No such file or directory

So seems like Liveness probe is failing, lets look into it:

vi red-probe-cka12-trb.yaml

Notice the command - sleep 3 ; touch /healthcheck; sleep 30;sleep 30000 it starts with a delay of 3 seconds, but the liveness probe initialDelaySeconds is set to 1 and failureThreshold is also 1. Which means the POD will fail just after first attempt of liveness check which will happen just after 1 second of pod start. So to make it stable we must increase the initialDelaySeconds to at least 5

vi red-probe-cka12-trb.yaml

Change initialDelaySeconds from 1 to 5 and save apply the changes.
Delete old pod:

kubectl delete pod red-probe-cka12-trb

Apply changes:

kubectl apply -f red-probe-cka12-trb.yaml

Details



Q. 2

Task
SECTION: STORAGE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



Create a persistent volume called data-pv-cka02-str with the below properties:


- Its capacity should be 128Mi.

- The volume type should be hostpath and path should be /opt/data-pv-cka02-str.


Next, create a persistent volume claim called data-pvc-cka02-str as per below properties:


- Request 50Mi of storage from data-pv-cka02-str PV.


Q. 3

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



Decode the existing secret called beta-sec-cka14-arch created in the beta-ns-cka14-arch namespace and store the decoded content inside the file /opt/beta-sec-cka14-arch on the student-node.


Q. 4

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster4 by running:


kubectl config use-context cluster4



On cluster4 we are having some weird issue where we are intermittently getting below error while running kubectl commands.


The connection to the server cluster4-controlplane:6443 was refused - did you specify the right host or port?

Whenever you get this error, you can wait for 10-15 seconds to make kubectl command work again, but it will come again after few second


We also noticed that kube-controller-manager-cluster4-controlplane pod is restarting continuously. Look into the issue and troubleshoot the same.


You can SSH into the cluster4 using ssh cluster4-controlplane command.

Solution
Let's check the POD status
kubectl get pod --context=cluster4 -n kube-system

You will see that kube-controller-manager-cluster4-controlplane pod is crashing or restarting. So let's try to watch the logs.

kubectl logs -f kube-controller-manager-cluster4-controlplane --context=cluster4 -n kube-system

You will see some logs as below:

 leaderelection.go:330] error retrieving resource lock kube-system/kube-controller-manager: Get "https://10.10.129.21:6443/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=5s": dial tcp 10.10.129.21:6443: connect: connection refused

You will notice that somehow the connection to the kube api is breaking, let's check if kube api pod is healthy.

kubectl get pod --context=cluster4 -n kube-system

Now you might notice that kube-apiserver-cluster4-controlplane pod is also restarting, so we should dig into its logs or relevant events.

kubectl logs -f kube-apiserver-cluster4-controlplane -n kube-system
kubectl get event --field-selector involvedObject.name=kube-apiserver-cluster4-controlplane -n kube-system

In events you will see this error

Warning   Unhealthy      pod/kube-apiserver-cluster4-controlplane   Liveness probe failed: Get "https://10.10.132.25:6444/livez": dial tcp 10.10.132.25:6444: connect: connection refused

From this we can see that the Liveness probe is failing for the kube-apiserver-cluster4-controlplane pod, and we can see its trying to connect to port 6444 port but the default api port is 6443. So let's look into the kube api server manifest.

ssh cluster4-controlplane
vi /etc/kubernetes/manifests/kube-apiserver.yaml

Under livenessProbe: you will see the port: value is 6444, change it to 6443 and save. Now wait for few seconds let the kube api pod come up.

kubectl get pod -n kube-system

Watch the PODs status for some time and make sure these are not restarting now.

Details

Q. 5

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



There is a deployment called nginx-dp-cka04-trb which has been used to deploy a static website. The access to this website can be tested by running: curl http://cluster1-controlplane:30002. However, it is not working at the moment.


Troubleshoot and fix it.

Solution
First list the available pods:

kubectl get pod

Look into the nginx-dp-xxxx POD logs

kubectl logs -f <pod-name>

You may not see any logs so look into the kubernetes events for <pod-name> POD

Look into the POD events

kubectl get event --field-selector involvedObject.name=<pod-name>

You will see an error something like:

70s         Warning   FailedMount   pod/nginx-dp-cka04-trb-767b767dc-6c5wk   Unable to attach or mount volumes: unmounted volumes=[nginx-config-volume-cka04-trb], unattached volumes=[index-volume-cka04-trb kube-api-access-4fbrb nginx-config-volume-cka04-trb]: timed out waiting for the condition

From the error we can see that its not able to mount nginx-config-volume-cka04-trb volume
Check the nginx-dp-cka04-trb deployment

kubectl get deploy nginx-dp-cka04-trb -o=yaml

Under volumes: look for the configMap: name which is nginx-configuration-cka04-trb. Now lets look into this configmap.

kubectl get configmap nginx-configuration-cka04-trb

The above command will fail as there is no configmap with this name, so now list the all configmaps.

kubectl get configmap

You will see an configmap named nginx-config-cka04-trb which seems to be the correct one.
Edit the nginx-dp-cka04-trb deployment now

kubectl edit deploy nginx-dp-cka04-trb

Under configMap: change nginx-configuration-cka04-trb to nginx-config-cka04-trb. Once done, wait for the POD to come up.

Try to access the website now:

curl http://cluster1-controlplane:30002

Details

Q. 6

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



The blue-dp-cka09-trb deployment is having 0 out of 1 pods running. Fix the issue to make sure that pod is up and running.

Solution
List the pods

kubectl get pod

Most probably you see Init:Error or Init:CrashLoopBackOff for the corresponding pod.

Look into the logs

kubectl logs blue-dp-cka09-trb-xxxx -c init-container

You will see an error something like

sh: can't open 'echo 'Welcome!'': No such file or directory

Edit the deployment

kubectl edit deploy blue-dp-cka09-trb

Under initContainers: -> - command: add -c to the next line of - sh, so final command should look like this
   initContainers:
   - command:
     - sh
     - -c
     - echo 'Welcome!'

If you will check pod then it must be failing again but with different error this time, let's find that out

kubectl get event --field-selector involvedObject.name=blue-dp-cka09-trb-xxxxx

You will see an error something like

Warning   Failed      pod/blue-dp-cka09-trb-69dd844f76-rv9z8   Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting "/var/lib/kubelet/pods/98182a41-6d6d-406a-a3e2-37c33036acac/volumes/kubernetes.io~configmap/nginx-config" to rootfs at "/etc/nginx/nginx.conf": mount /var/lib/kubelet/pods/98182a41-6d6d-406a-a3e2-37c33036acac/volumes/kubernetes.io~configmap/nginx-config:/etc/nginx/nginx.conf (via /proc/self/fd/6), flags: 0x5001: not a directory: unknown

Edit the deployment again

kubectl edit deploy blue-dp-cka09-trb

Under volumeMounts: -> - mountPath: /etc/nginx/nginx.conf -> name: nginx-config add subPath: nginx.conf and save the changes.
Finally, the pod should be in running state.

Details


Q. 7

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1


There is a sample script located at /root/service-cka25-arch.sh on the student-node.
Update this script to add a command to filter/display the targetPort only for service service-cka25-arch using jsonpath. The service has been created under the default namespace on cluster1.


Solution
Update service-cka25-arch.sh script:

student-node ~ ➜ vi service-cka25-arch.sh

Add below command in it:

kubectl --context cluster1 get service service-cka25-arch -o jsonpath='{.spec.ports[0].targetPort}'

Details

Q. 8

Task
SECTION: STORAGE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



There is a requirement to share a volume between two containers that are running within the same pod. Use the following instructions to create the pod and related objects:


- Create a pod named grape-pod-cka06-str.

- The main container should use the nginx image and mount a volume called grape-vol-cka06-str at path /var/log/nginx.

- The sidecar container can use busybox image, you might need to add a sleep command to this container to keep it running. Next, mount the same volume called grape-vol-cka06-str at the path /usr/src.

- The volume should be of type emptyDir.


Solution
Create a YAML file as below:
apiVersion: v1
kind: Pod
metadata:
  name: grape-pod-cka06-str
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
      - name: grape-vol-cka06-str
        mountPath: "/var/log/nginx"
  - name: busybox
    image: busybox
    command:
      - "bin/sh"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: grape-vol-cka06-str
        mountPath: "/usr/src"
  volumes:
  - name: grape-vol-cka06-str
    emptyDir: {}

Apply the template:
kubectl apply -f <template-file-name>.yaml

Details





Q. 9

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



There is an existing persistent volume called orange-pv-cka13-trb. A persistent volume claim called orange-pvc-cka13-trb is created to claim storage from orange-pv-cka13-trb.


However, this PVC is stuck in a Pending state. As of now, there is no data in the volume.


Troubleshoot and fix this issue, making sure that orange-pvc-cka13-trb PVC is in Bound state.

Solution
List the PVC to check its status
kubectl  get pvc

So we can see orange-pvc-cka13-trb PVC is in Pending state and its requesting a storage of 150Mi. Let's look into the events

kubectl get events --sort-by='.metadata.creationTimestamp' -A

You will see some errors as below:

Warning   VolumeMismatch            persistentvolumeclaim/orange-pvc-cka13-trb          Cannot bind to requested volume "orange-pv-cka13-trb": requested PV is too small

Let's look into orange-pv-cka13-trb volume

kubectl get pv

We can see that orange-pv-cka13-trb volume is of 100Mi capacity which means its too small to request 150Mi of storage.
Let's edit orange-pvc-cka13-trb PVC to adjust the storage requested.

kubectl get pvc orange-pvc-cka13-trb -o yaml > /tmp/orange-pvc-cka13-trb.yaml
vi /tmp/orange-pvc-cka13-trb.yaml

Under resources: -> requests: -> storage: change 150Mi to 100Mi and save.
Delete old PVC and apply the change:

kubectl delete pvc orange-pvc-cka13-trb
kubectl apply -f /tmp/orange-pvc-cka13-trb.yaml 

Details

Q. 10

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



We deployed an app using a deployment called web-dp-cka06-trb. it's using the httpd:latest image. There is a corresponding service called web-service-cka06-trb that exposes this app on the node port 30005. However, the app is not accessible!


Troubleshoot and fix this issue. Make sure you are able to access the app using curl http://cluster1-controlplane:30005 command.

Solution
List the deployments to see if all PODs under web-dp-cka06-trb deployment are up and running.
kubectl get deploy

You will notice that 0 out of 1 PODs are up, so let's look into the POD now.

kubectl get pod

You will notice that web-dp-cka06-trb-xxx pod is in Pending state, so let's checkout the relevant events.

kubectl get event --field-selector involvedObject.name=web-dp-cka06-trb-xxx

You should see some error/warning like this:

Warning   FailedScheduling   pod/web-dp-cka06-trb-76b697c6df-h78x4   0/1 nodes are available: 1 persistentvolumeclaim "web-cka06-trb" not found. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.

Let's look into the PVCs

kubectl get pvc

You should see web-pvc-cka06-trb in the output but as per logs the POD was looking for web-cka06-trb PVC. Let's update the deployment to fix this.

kubectl edit deploy web-dp-cka06-trb

Under volumes: -> name: web-str-cka06-trb -> persistentVolumeClaim: -> claimName change web-cka06-trb to web-pvc-cka06-trb and save the changes.

Look into the POD again to make sure its running now

kubectl get pod

You will find that its still failing, most probably with ErrImagePull or ImagePullBackOff error. Now lets update the deployment again to make sure its using the correct image.

kubectl edit deploy web-dp-cka06-trb

Under spec: -> containers: -> change image from httpd:letest to httpd:latest and save the changes.
Look into the POD again to make sure its running now

kubectl get pod

You will notice that POD is still crashing, let's look into the POD logs.

kubectl logs web-dp-cka06-trb-xxxx

If there are no useful logs then look into the events

kubectl get event --field-selector involvedObject.name=web-dp-cka06-trb-xxxx --sort-by='.lastTimestamp'

You should see some errors/warnings as below

Warning   FailedPostStartHook   pod/web-dp-cka06-trb-67dccb7487-2bjgf   Exec lifecycle hook ([/bin -c echo 'Test Page' > /usr/local/apache2/htdocs/index.html]) for Container "web-container" in Pod "web-dp-cka06-trb-67dccb7487-2bjgf_default(4dd6565e-7f1a-4407-b3d9-ca595e6d4e95)" failed - error: rpc error: code = Unknown desc = failed to exec in container: failed to start exec "c980799567c8176db5931daa2fd56de09e84977ecd527a1d1f723a862604bd7c": OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin": permission denied: unknown, message: ""

Let's look into the lifecycle hook of the pod

kubectl edit deploy web-dp-cka06-trb

Under containers: -> lifecycle: -> postStart: -> exec: -> command: change /bin to /bin/sh
Look into the POD again to make sure its running now

kubectl get pod

Finally pod should be in running state. Let's try to access the webapp now.

curl http://cluster1-controlplane:30005

You will see error curl: (7) Failed to connect to cluster1-controlplane port 30005: Connection refused
Let's look into the service

kubectl edit svc web-service-cka06-trb

Let's verify if the selector labels and ports are correct as needed. You will note that service is using selector: -> app: web-cka06-trb
Now, let's verify the app labels:

kubectl get deploy web-dp-cka06-trb -o yaml

Under labels you will see labels: -> deploy: web-app-cka06-trb
So we can see that service is using wrong selector label, let's edit the service to fix the same

kubectl edit svc web-service-cka06-trb

Let's try to access the webapp now.

curl http://cluster1-controlplane:30005

Boom! app should be accessible now.

Details
