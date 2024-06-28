Introduction to Kubernetes Network Policies
Welcome to the first lab of our Learn-By-Doing Kubernetes Network Policies course. Over the next nine labs, we'll explore the fundamental concepts of Kubernetes networking, engage in hands-on installation of components on a real Kubernetes cluster, and craft custom network policies.

Here are the topics that will be covered in this course:

Introduction to Kubernetes Network Policies
Understanding and Implementing CNIs in Kubernetes
Flannel vs Weave: A Detailed Comparison
Understanding the Kubernetes Network Policy YAML
Establishing a Baseline Security Posture with Default Deny Network Policies
Fine-Grained Network Policies in Kubernetes: Building on Top of a Default Deny Policy
Namespace-Based Isolation with Kubernetes Network Policies
Exploring Egress based policies
Final Challenge
Getting Started
We'll kick-start the course by familiarizing ourselves with our lab environment and understanding the necessity of Network Policies in Kubernetes.

In a Kubernetes cluster, pods can communicate with each other without any restrictions by default. While this unrestricted communication is advantageous for developers, it could potentially pose security risks.

Our Lab Environment
Our lab environment mirrors this setup. You'll find several pre-deployed pods and services, all capable of unrestricted intercommunication.

When you're ready, click on the Instructions tab to gain a deeper understanding of the environment and the hands-on tasks that will help us comprehend why Kubernetes network policies are necessary in the first place.

Instructions Tab


In this lab, we will get familiarized with the lab interface and explore a couple of pre-deployed Kubernetes objects. In doing so, we will begin to understand why we need Kubernetes Policies to begin with and how it can be used to secure traffic within our cluster.

This lab is quite straight-forward. You will have access to a 2-node Kubernetes Cluster with a few deployments as shown below:

lab1-diagram

When you are ready, click on the Tasks tab and you will be presented with a sequence of multiple-choice questions based on the environment.

These questions will have you explore the cluster, deployments, services and have you test connectivity between the pods.

To test if one pod can connect to another via a Kubernetes service, you can use the following options:

netcat and telnet command-line utilities
UI tabs (wherever provided)
Remember, you can toggle between the Terminal and this page at any time by clicking on the toggle button:
Toggle Button

  ![image](https://github.com/Althaf-official/Kodekloud_Learning/assets/105126131/c68e77e4-2457-46c6-82f3-fc6ca4c4ed21)
      
      controlplane ~ ➜  k get all
      NAME                            READY   STATUS    RESTARTS   AGE
      pod/connection-tester           1/1     Running   0          5m26s
      pod/deploy-a-5c56568dff-v9cvm   1/1     Running   0          5m26s
      pod/mysql                       1/1     Running   0          5m25s
      pod/pod-b                       1/1     Running   0          5m26s
      
      NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
      service/conntest-service   NodePort    10.97.100.146    <none>        8080:30082/TCP   5m26s
      service/db-service         ClusterIP   10.103.38.105    <none>        3306/TCP         5m25s
      service/kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP          76m
      service/service-a          ClusterIP   10.109.149.177   <none>        80/TCP           5m25s
      service/service-b          ClusterIP   10.97.165.133    <none>        80/TCP           5m25s
      
      NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/deploy-a   1/1     1            1           5m26s
      
      NAME                                  DESIRED   CURRENT   READY   AGE
      replicaset.apps/deploy-a-5c56568dff   1         1         1       5m26s
      
      controlplane ~ ➜  k get deployments.apps 
      NAME       READY   UP-TO-DATE   AVAILABLE   AGE
      deploy-a   1/1     1            1           7m21s
      
      controlplane ~ ➜  


How is the deployment deploy-a exposed in this cluster?

Run the below command:

controlplane ~ ➜  kubectl get deployments.apps deploy-a --show-labels 
NAME       READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deploy-a   1/1     1            1           13m   app=deploy-a
The above command shows that the deployment deploy-a has a label with key called app with a value of deploy-a.
Next, lets check for any services which uses this key-value pair as a selector:

controlplane ~ ➜  kubectl get svc --selector app=deploy-a
NAME        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service-a   ClusterIP   10.99.96.196   <none>        80/TCP    13m

controlplane ~ ➜  
As you can see the result of the above command there is indeed a service called service-a that exposes the deployment.


The type of this service is: ClusterIP
