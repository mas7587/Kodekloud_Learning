Q. 1

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
