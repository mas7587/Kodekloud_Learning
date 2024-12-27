# Q. 1

Task
SECTION: SERVICE NETWORKING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



Create a pod with name tester-cka02-svcn in dev-cka02-svcn namespace with image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3. Make sure to use command sleep 3600 with restart policy set to Always .


Once the tester-cka02-svcn pod is running, store the output of the command nslookup kubernetes.default from tester pod into the file /root/dns_output on student-node.


Solution
Change to the cluster1 context before attempting the task:

kubectl config use-context cluster1

Since the dev-cka02-svcn namespace doesn't exist, let's create it first:

    kubectl create ns dev-cka02-svcn

Create the pod as per the requirements:

    kubectl apply -f - << EOF
    apiVersion: v1
    kind: Pod
    metadata:
      name: tester-cka02-svcn
      namespace: dev-cka02-svcn
    spec:
      containers:
      - name: tester-cka02-svcn
        image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
        command:
          - sleep
          - "3600"
      restartPolicy: Always
    EOF



Now let store the correct output into the /root/dns_output on student-node :

    kubectl exec -n dev-cka02-svcn -i -t tester-cka02-svcn -- nslookup kubernetes.default > /root/dns_output



We should have something similar to below output:

    student-node ~ ➜  cat /root/dns_output
    Server:         172.20.0.10
    Address:        172.20.0.10#53
    
    Name:   kubernetes.default.svc.cluster.local
    Address: 172.20.0.1

Details






# Q. 2

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



Run a pod called alpine-sleeper-cka15-arch using the alpine image in the default namespace that will sleep for 7200 seconds.


Solution
Create the pod definition:

student-node ~ ➜  vi alpine-sleeper-cka15-arch.yaml




##Content should be:

    apiVersion: v1
    kind: Pod
    metadata:
      name: alpine-sleeper-cka15-arch
    spec:
      containers:
      - name: alpine
        image: alpine
        command: ["/bin/sh", "-c", "sleep 7200"]
    



Create the pod:

student-node ~ ➜  kubectl apply -f alpine-sleeper-cka15-arch.yaml --context cluster3

Details

# Q. 3

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



We have created a service account called green-sa-cka22-arch, a cluster role called green-role-cka22-arch and a cluster role binding called green-role-binding-cka22-arch.


Update the permissions of this service account so that it can only get all the namespaces in cluster1.


# Q. 4

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



There is a Cronjob called orange-cron-cka10-trb which is supposed to run every two minutes (i.e 13:02, 13:04, 13:06…14:02, 14:04…and so on). This cron targets the application running inside the orange-app-cka10-trb pod to make sure the app is accessible. The application has been exposed internally as a ClusterIP service.


However, this cron is not running as per the expected schedule and is not running as intended.


Make the appropriate changes so that the cronjob runs as per the required schedule and it passes the accessibility checks every-time.

Solution
Check the cron schedule
    kubectl get cronjob

Make sure the schedule for orange-cron-cka10-trb crontjob is set to */2 * * * * if not then edit it.

Also before that look for the issues why this cron is failing

    kubectl logs orange-cron-cka10-trb-xxxx

You will see some error like

    curl: (6) Could not resolve host: orange-app-cka10-trb

You will notice that the curl is trying to hit orange-app-cka10-trb directly but it is supposed to hit the relevant service which is orange-svc-cka10-trb so we need to fix the curl command.

Edit the cronjob
    kubectl edit cronjob orange-cron-cka10-trb

Change schedule * * * * * to */2 * * * *
Change command curl orange-app-cka10-trb to curl orange-svc-cka10-trb
Wait for 2 minutes to run again this cron and it should complete now.

Details



# Q. 5

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster4 by running:


kubectl config use-context cluster4



The pink-depl-cka14-trb Deployment was scaled to 2 replicas however, the current replicas is still 1.


Troubleshoot and fix this issue. Make sure the CURRENT count is equal to the DESIRED count.


You can SSH into the cluster4 using ssh cluster4-controlplane command.

# Q. 6

Task
SECTION: STORAGE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



A persistent volume called papaya-pv-cka09-str is already created with a storage capacity of 150Mi. It's using the papaya-stc-cka09-str storage class with the path /opt/papaya-stc-cka09-str.


Also, a persistent volume claim named papaya-pvc-cka09-str has also been created on this cluster. This PVC has requested 50Mi of storage from papaya-pv-cka09-str volume.


Resize the PVC to 80Mi and make sure the PVC is in Bound state.


Solution
Edit papaya-pv-cka09-str PV:

    kubectl get pv papaya-pv-cka09-str -o yaml > /tmp/papaya-pv-cka09-str.yaml

Edit the template:

    vi /tmp/papaya-pv-cka09-str.yaml

Delete all entries for uid:, annotations, status:, claimRef: from the template.
Edit papaya-pvc-cka09-str PVC:

    kubectl get pvc papaya-pvc-cka09-str -o yaml > /tmp/papaya-pvc-cka09-str.yaml

Edit the template:

    vi /tmp/papaya-pvc-cka09-str.yaml

Under resources: -> requests: change storage: 50Mi to storage: 80Mi and save the template.
Delete the exsiting PVC:

    kubectl delete pvc papaya-pvc-cka09-str

Delete the exsiting PV and create using the template:

    kubectl delete pv papaya-pv-cka09-str
    kubectl apply -f /tmp/papaya-pv-cka09-str.yaml

Create the PVC using template:

    kubectl apply -f /tmp/papaya-pvc-cka09-str.yaml

Details

# Q. 7

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



A pod called logger-cka03-arch has been created in the default namespace. Inspect this pod and save ALL INFO and ERROR's to the file /root/logger-cka03-arch-all on the student-node.


Solution
Run the command kubectl logs logger-cka03-arch --context cluster3 > /root/logger-cka03-arch-all on the student-node.

Run the command :

    student-node ~ ➜ kubectl logs logger-cka03-arch --context cluster3 > /root/logger-cka03-arch-all

    student-node ~ ➜  head /root/logger-cka03-arch-all
    INFO: Wed Oct 19 10:38:57 UTC 2022 Logger is running
    ERROR: Wed Oct 19 10:38:57 UTC 2022 Logger encountered errors!
    INFO: Wed Oct 19 10:38:57 UTC 2022 Logger is running
    ERROR: Wed Oct 19 10:38:57 UTC 2022 Logger encountered errors!
    INFO: Wed Oct 19 10:38:57 UTC 2022 Logger is running
    ERROR: Wed Oct 19 10:38:57 UTC 2022 Logger encountered errors!
    INFO: Wed Oct 19 10:38:57 UTC 2022 Logger is running
    ERROR: Wed Oct 19 10:38:57 UTC 2022 Logger encountered errors!
    INFO: Wed Oct 19 10:38:57 UTC 2022 Logger is running
    ERROR: Wed Oct 19 10:38:57 UTC 2022 Logger encountered errors!
    
    student-node ~ ➜  

Details

# Q. 8

Task
SECTION: SERVICE NETWORKING


For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



Create a ReplicaSet with name checker-cka10-svcn in ns-12345-svcn namespace with image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3.


Make sure to specify the below specs as well:



command sleep 3600
replicas set to 2
container name: dns-image



Once the checker pods are up and running, store the output of the command nslookup kubernetes.default from any one of the checker pod into the file /root/dns-output-12345-cka10-svcn on student-node.

Solution
Change to the cluster4 context before attempting the task:

kubectl config use-context cluster3




Create the ReplicaSet as per the requirements:



    kubectl apply -f - << EOF
    ---
    apiVersion: v1
    kind: Namespace
    metadata:
      creationTimestamp: null
      name: ns-12345-svcn
    spec: {}
    status: {}
    
    ---
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: checker-cka10-svcn
      namespace: ns-12345-svcn
      labels:
        app: dns
        tier: testing
    spec:
      replicas: 2
      selector:
        matchLabels:
          tier: testing
      template:
        metadata:
          labels:
            tier: testing
        spec:
          containers:
          - name: dns-image
            image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
            command:
              - sleep
              - "3600"
    EOF




Now let's test if the nslookup command is working :


    student-node ~ ➜  k get pods -n ns-12345-svcn 
    NAME                       READY   STATUS    RESTARTS   AGE
    checker-cka10-svcn-d2cd2   1/1     Running   0          12s
    checker-cka10-svcn-qj8rc   1/1     Running   0          12s
    
    student-node ~ ➜  POD_NAME=`k get pods -n ns-12345-svcn --no-headers | head -1 | awk '{print $1}'`
    
    student-node ~ ➜  kubectl exec -n ns-12345-svcn -i -t $POD_NAME -- nslookup kubernetes.default
    ;; connection timed out; no servers could be reached
    
    command terminated with exit code 1




There seems to be a problem with the name resolution. Let's check if our coredns pods are up and if any service exists to reach them:



    student-node ~ ➜  k get pods -n kube-system | grep coredns
    coredns-6d4b75cb6d-cprjz                        1/1     Running   0             42m
    coredns-6d4b75cb6d-fdrhv                        1/1     Running   0             42m
    
    student-node ~ ➜  k get svc -n kube-system 
    NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   62m
    



Everything looks okay here but the name resolution problem exists, let's see if the kube-dns service have any active endpoints:

    student-node ~ ➜  kubectl get ep -n kube-system kube-dns 
    NAME       ENDPOINTS   AGE
    kube-dns   <none>      63m




Finally, we have our culprit.


If we dig a little deeper, we will it is using wrong labels and selector:



    student-node ~ ➜  kubectl describe svc -n kube-system kube-dns 
    Name:              kube-dns
    Namespace:         kube-system
    ....
    Selector:          k8s-app=core-dns
    Type:              ClusterIP
    ...
    
    student-node ~ ➜  kubectl get deploy -n kube-system --show-labels | grep coredns
    coredns   2/2     2            2           66m   k8s-app=kube-dns
    



Let's update the kube-dns service it to point to correct set of pods:



    student-node ~ ➜  kubectl patch service -n kube-system kube-dns -p '{"spec":{"selector":{"k8s-app": "kube-dns"}}}'
    service/kube-dns patched
    
    student-node ~ ➜  kubectl get ep -n kube-system kube-dns 
    NAME       ENDPOINTS                                              AGE
    kube-dns   10.50.0.2:53,10.50.192.1:53,10.50.0.2:53 + 3 more...   69m




NOTE: We can use any method to update kube-dns service. In our case, we have used kubectl patch command.




Now let's store the correct output to /root/dns-output-12345-cka10-svcn:



    student-node ~ ➜  kubectl exec -n ns-12345-svcn -i -t $POD_NAME -- nslookup kubernetes.default
    Server:         10.96.0.10
    Address:        10.96.0.10#53
    
    Name:   kubernetes.default.svc.cluster.local
    Address: 10.96.0.1
    
    
    student-node ~ ➜  kubectl exec -n ns-12345-svcn -i -t $POD_NAME -- nslookup kubernetes.default > /root/dns-output-12345-cka10-svcn

Details







# Q. 9

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1


Create a service account called deploy-cka20-arch. Further create a cluster role called deploy-role-cka20-arch with permissions to get the deployments in cluster1.


Finally create a cluster role binding called deploy-role-binding-cka20-arch to bind deploy-role-cka20-arch cluster role with deploy-cka20-arch service account.


Solution
Create the service account, cluster role and role binding:

    student-node ~ ➜  kubectl --context cluster1 create serviceaccount deploy-cka20-arch
    student-node ~ ➜  kubectl --context cluster1 create clusterrole deploy-role-cka20-arch --resource=deployments --verb=get
    student-node ~ ➜  kubectl --context cluster1 create clusterrolebinding deploy-role-binding-cka20-arch --clusterrole=deploy-role-cka20-arch --serviceaccount=default:deploy-cka20-arch
    



You can verify it as below:

    student-node ~ ➜  kubectl --context cluster1 auth can-i get deployments --as=system:serviceaccount:default:deploy-cka20-arch
    yes

Details

# Q. 10

Task
SECTION: SERVICE NETWORKING


For this question, please set the context to cluster1 by running:


    kubectl config use-context cluster1



Create a nginx pod called nginx-resolver-cka06-svcn using image nginx, expose it internally with a service called nginx-resolver-service-cka06-svcn.



Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc.cka06.svcn and /root/CKA/nginx.pod.cka06.svcn


Solution
Switching to cluster1:



    kubectl config use-context cluster1




To create a pod nginx-resolver-cka06-svcn and expose it internally:



    student-node ~ ➜ kubectl run nginx-resolver-cka06-svcn --image=nginx 
    student-node ~ ➜ kubectl expose pod/nginx-resolver-cka06-svcn --name=nginx-resolver-service-cka06-svcn --port=80 --target-port=80 --type=ClusterIP 




To create a pod test-nslookup. Test that you are able to look up the service and pod names from within the cluster:



    student-node ~ ➜  kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service-cka06-svcn
    student-node ~ ➜  kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service-cka06-svcn > /root/CKA/nginx.svc.cka06.svcn




Get the IP of the nginx-resolver-cka06-svcn pod and replace the dots(.) with hyphon(-) which will be used below.



    student-node ~ ➜  kubectl get pod nginx-resolver-cka06-svcn -o wide
    student-node ~ ➜  IP=`kubectl get pod nginx-resolver-cka06-svcn -o wide --no-headers | awk '{print $6}' | tr '.' '-'`
    student-node ~ ➜  kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup $IP.default.pod > /root/CKA/nginx.pod.cka06.svcn

Details



# Q. 11

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster4 by running:


kubectl config use-context cluster4



We tried to schedule grey-cka21-trb pod on cluster4 which was supposed to be deployed by the kubernetes scheduler so far but somehow its stuck in Pending state. Look into the issue and fix the same, make sure the pod is in Running state.


You can SSH into the cluster4 using ssh cluster4-controlplane command.

Solution

---
Follow below given steps
Let's check the POD status
        kubectl get pod --context=cluster4

You will see that grey-cka21-trb pod is stuck in Pending state. So let's try to look into the logs and events

        kubectl logs grey-cka21-trb --context=cluster4
        kubectl get event --context=cluster4 --field-selector involvedObject.name=grey-cka21-trb

You might not find any relevant info in the logs/events. Let's check the status of the kube-scheduler pod

        kubectl get pod --context=cluster4 -n kube-system

You will notice that kube-scheduler-cluster4-controlplane pod us crashing, let's look into its logs

        kubectl logs kube-scheduler-cluster4-controlplane --context=cluster4 -n kube-system
    
You will see an error as below:

run.go:74] "command failed" err="failed to get delegated authentication kubeconfig: failed to get delegated authentication kubeconfig: stat /etc/kubernetes/scheduler.config: no such file or directory"

From the logs we can see that its looking for a file called /etc/kubernetes/scheduler.config which seems incorrect, let's look into the kube-scheduler manifest on cluster4.

        ssh cluster4-controlplane

First let's find out if /etc/kubernetes/scheduler.config

        ls /etc/kubernetes/scheduler.config

You won't find it, instead the correct file is /etc/kubernetes/scheduler.conf so let's modify the manifest.

        vi /etc/kubernetes/manifests/kube-scheduler.yaml 

Search for config in the file, you will find some typos, change every occurence of /etc/kubernetes/scheduler.config to /etc/kubernetes/scheduler.conf.

Let's see if kube-scheduler-cluster4-controlplane is running now

        kubectl get pod -A

It should be good now and grey-cka21-trb should be good as well.

# Q. 12

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster2 by running:


kubectl config use-context cluster2



The yello-cka20-trb pod is stuck in a Pending state. Fix this issue and get it to a running state. Recreate the pod if necessary.

Do not remove any of the existing taints that are set on the cluster nodes.


Solution
Let's check the POD status
kubectl get pod --context=cluster2

So you will see that yello-cka20-trb pod is in Pending state. Let's check out the relevant events.

    kubectl get event --field-selector involvedObject.name=yello-cka20-trb --context=cluster2

You will see some errors like:

    Warning   FailedScheduling   pod/yello-cka20-trb   0/2 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/master: }, 1 node(s) had untolerated taint {node: node01}. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.

Notice this error 1 node(s) had untolerated taint {node: node01} so we can see that one of nodes have taints applied. We don't want to remove the node taints and we are not going to re-create the POD so let's look into the POD config if its using any other toleration settings.

    kubectl get pod yello-cka20-trb --context=cluster2 -o yaml

You will notice this in the output

    tolerations:
      - effect: NoSchedule
        key: node
        operator: Equal
        value: cluster2-node01

Here notice that the value for key node is cluster2-node01 but the node has different value applied i.e node01 so let's update the taints values for the node as needed.

    kubectl --context=cluster2 taint nodes cluster2-node01 node=cluster2-node01:NoSchedule --overwrite=true

Let's check the POD status again

    kubectl get pod --context=cluster2

It should be in Running state now.

Details


# Q. 13

Task
SECTION: SCHEDULING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



In the dev-wl07 namespace, one of the developers has performed a rolling update and upgraded the application to a newer version. But somehow, application pods are not being created.


To get back the working state, rollback the application to the previous version .


After rolling the deployment back, on the controlplane node, save the image currently in use to the /root/rolling-back-record.txt file and increase the replica count to the 5.


You can SSH into the cluster1 using ssh cluster1-controlplane command.

Solution
Run the command to change the context: -

kubectl config use-context cluster1



Check the status of the pod: -

    kubectl get pods -n dev-wl07



One of the pods is in an error state. As a quick fix, we need to rollback to the previous revision as follows: -

    kubectl rollout undo -n dev-wl07 deploy webapp-wl07



After successful rolling back, inspect the updated image: -

    kubectl describe deploy -n dev-wl07 webapp-wl07 | grep -i image



On the Controlplane node, save the image name to the given path /root/rolling-back-record.txt: -

    ssh cluster1-controlplane

    echo "kodekloud/webapp-color" > /root/rolling-back-record.txt



And increase the replica count to the 5 with help of kubectl scale command: -

    kubectl scale deploy -n dev-wl07 webapp-wl07 --replicas=5



Verify it by running the command: kubectl get deploy -n dev-wl07

Details



# Q. 14

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1


The db-deployment-cka05-trb deployment is having 0 out of 1 PODs ready.


Figure out the issues and fix the same but make sure that you do not remove any DB related environment variables from the deployment/pod.

Solution
Find out the name of the DB POD:
    kubectl get pod

Check the DB POD logs:
    kubectl logs <pod-name>

You might see something like as below which is not that helpful:

Error from server (BadRequest): container "db" in pod "db-deployment-cka05-trb-7457c469b7-zbvx6" is waiting to start: CreateContainerConfigError

So let's look into the kubernetes events for this pod:

    kubectl get event --field-selector involvedObject.name=<pod-name>

You will see some errors as below:

Error: couldn't find key db in Secret default/db-cka05-trb

Now let's look into all secrets:

    kubectl get secrets db-root-pass-cka05-trb -o yaml
    kubectl get secrets db-user-pass-cka05-trb -o yaml
    kubectl get secrets db-cka05-trb -o yaml

Now let's look into the deployment.

Edit the deployment
    kubectl edit deployment db-deployment-cka05-trb -o yaml

You will notice that some of the keys are different what are reffered in the deployment.

Change some env keys: db to database , db-user to username and db-password to password
Change a secret reference: db-user-cka05-trb to db-user-pass-cka05-trb
Finally save the changes.
Details

# Q. 15

Task
SECTION: SCHEDULING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



Create a new deployment called ocean-tv-wl09 in the default namespace using the image kodekloud/webapp-color:v1.
Use the following specs for the deployment:


1. Replica count should be 3.

2. Set the Max Unavailable to 40% and Max Surge to 55%.

3. Create the deployment and ensure all the pods are ready.

4. After successful deployment, upgrade the deployment image to kodekloud/webapp-color:v2 and inspect the deployment rollout status.

5. Check the rolling history of the deployment and on the student-node, save the current revision count number to the /opt/revision-count.txt file.

6. Finally, perform a rollback and revert back the deployment image to the older version.




Solution
Set the correct context: -

kubectl config use-context cluster1



Use the following template to create a deployment called ocean-tv-wl09: -

    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: ocean-tv-wl09
      name: ocean-tv-wl09
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: ocean-tv-wl09
      strategy: 
       type: RollingUpdate
       rollingUpdate:
         maxUnavailable: 40%
         maxSurge: 55%
      template:
        metadata:
          labels:
            app: ocean-tv-wl09
        spec:
          containers:
          - image: kodekloud/webapp-color:v1
            name: webapp-color



Now, create the deployment by using the kubectl create -f command in the default namespace: -

    kubectl create -f <FILE-NAME>.yaml



After sometime, upgrade the deployment image to kodekloud/webapp-color:v2: -

    kubectl set image deploy ocean-tv-wl09 webapp-color=kodekloud/webapp-color:v2



And check out the rollout history of the deployment ocean-tv-wl09: -

    kubectl rollout history deploy ocean-tv-wl09
    deployment.apps/ocean-tv-wl09 
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>



NOTE: - Revision count is 2. In your lab, it could be different.



On the student-node, store the revision count to the given file: -

    echo "2" > /opt/revision-count.txt



In final task, rollback the deployment image to an old version: -

    kubectl rollout undo deployment ocean-tv-wl09



Verify the image name by using the following command: -

    kubectl describe deploy ocean-tv-wl09



It should be kodekloud/webapp-color:v1 image.

Details





# Q. 16

Task
SECTION: ARCHITECTURE, INSTALL AND MAINTENANCE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



There is a script located at /root/pod-cka26-arch.sh on the student-node. Update this script to add a command to filter/display the label with value component of the pod called kube-apiserver-cluster1-controlplane (on cluster1) using jsonpath.


Solution
Update pod-cka26-arch.sh script:

    student-node ~ ➜ vi pod-cka26-arch.sh




Add below command in it:

    kubectl --context cluster1 get pod -n kube-system kube-apiserver-cluster1-controlplane  -o jsonpath='{.metadata.labels.component}'

Details

# Q. 17

Task
SECTION: SERVICE NETWORKING


For this question, please set the context to cluster3 by running:


kubectl config use-context cluster3



There is a deployment nginx-deployment-cka04-svcn in cluster3 which is exposed using service nginx-service-cka04-svcn.



Create an ingress resource nginx-ingress-cka04-svcn to load balance the incoming traffic with the following specifications:



    pathType: Prefix and path: /

    Backend Service Name: nginx-service-cka04-svcn

    Backend Service Port: 80

    ssl-redirect is set to false
Solution
First change the context to "cluster3":



    student-node ~ ➜  kubectl config use-context cluster3
Switched to context "cluster3".




Now apply the ingress resource with the given requirements:



    kubectl apply -f - << EOF
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: nginx-ingress-cka04-svcn
      annotations:
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
    spec:
      rules:
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service-cka04-svcn
                port:
                  number: 80
    EOF




Check if the ingress resource was successfully created:



    student-node ~ ➜  kubectl get ingress
    NAME                       CLASS    HOSTS   ADDRESS       PORTS   AGE
    nginx-ingress-cka04-svcn   <none>   *       172.25.0.10   80      13s
    



As the ingress controller is exposed on cluster3-controlplane using traefik service, we need to ssh to cluster3-controlplane first to check if the ingress resource works properly:



    student-node ~ ➜  ssh cluster3-controlplane
    
    cluster3-controlplane:~# curl -I 172.25.0.11
    HTTP/1.1 200 OK
    ...

Details







# Q. 18

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1


It appears that the black-cka25-trb deployment in cluster1 isn't up to date. While listing the deployments, we are currently seeing 0 under the UP-TO-DATE section for this deployment. Troubleshoot, fix and make sure that this deployment is up to date.

Solution
Check current status of the deployment

    kubectl get deploy 

Let's check deployment status

    kubectl get deploy black-cka25-trb -o yaml

Under status: you will see message: Deployment is paused so seems like deployment was paused, let check the rollout status

    kubectl rollout status deployment black-cka25-trb

You will see this message

Waiting for deployment "black-cka25-trb" rollout to finish: 0 out of 1 new replicas have been updated...

So, let's resume

    kubectl rollout resume deployment black-cka25-trb

Check again the status of the deployment

    kubectl get deploy 

It should be good now.

Details


# Q. 19

Task
SECTION: TROUBLESHOOTING


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



The purple-app-cka27-trb pod is an nginx based app on the container port 80. This app is exposed within the cluster using a ClusterIP type service called purple-svc-cka27-trb.


There is another pod called purple-curl-cka27-trb which continuously monitors the status of the app running within purple-app-cka27-trb pod by accessing the purple-svc-cka27-trb service using curl.


Recently we started seeing some errors in the logs of the purple-curl-cka27-trb pod.


Dig into the logs to identify the issue and make sure it is resolved.


Note: You will not be able to access this app directly from the student-node but you can exec into the purple-app-cka27-trb pod to check.

Solution
Check the purple-curl-cka27-trb pod logs

    kubectl logs purple-curl-cka27-trb

You will see some logs as below

Not able to connect to the nginx app on http://purple-svc-cka27-trb

Now to debug let's try to access this app from within the purple-app-cka27-trb pod

    kubectl exec -it purple-app-cka27-trb -- bash
    curl http://purple-svc-cka27-trb
    exit

You will notice its stuck, so app is not reachable. Let's look into the service to see its configured correctly.

    kubectl edit svc purple-svc-cka27-trb

Under ports: -> port: and targetPort: is set to 8080 but nginx default port is 80 so change 8080 to 80 and save the changes
Let's check the logs now

    kubectl logs purple-curl-cka27-trb

You will see Thank you for using nginx. in the output now.

Details

# Q. 20

Task
SECTION: STORAGE


For this question, please set the context to cluster1 by running:


kubectl config use-context cluster1



Create a storage class with the name banana-sc-cka08-str as per the properties given below:


- Provisioner should be kubernetes.io/no-provisioner,

- Volume binding mode should be WaitForFirstConsumer.

- Volume expansion should be enabled.


Solution
Create a yaml template as below:
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: banana-sc-cka08-str
    provisioner: kubernetes.io/no-provisioner
    allowVolumeExpansion: true
    volumeBindingMode: WaitForFirstConsumer

Apply the template:
    kubectl apply -f <template-file-name>.yaml

Details
