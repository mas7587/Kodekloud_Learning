Q. 14

Task
SECTION: SCHEDULING


For this question, please set the context to cluster1 by running:

kubectl config use-context cluster1

Deploy a VPA named backend-scale-optimizer for a deployment named multi-container-deployment which is already present in the default namespace.

The deployment consists of two containers: frontend and backend.

Configure the VPA to manage resource requests for the backend container only, excluding the frontend container from automatic resource adjustments.

Use the following specifications:

VPA update mode: "Off" (recommendation-only mode)
Resource limits for backend container:
Minimum CPU: 10m
Minimum Memory: 10Mi
Maximum CPU: 300m
Maximum Memory: 256Mi
Solution
First set the context to cluster1:

kubectl config use-context cluster1

Create the VPA resource:

# backend-vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-scale-optimizer
  namespace: default
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: multi-container-deployment
  updatePolicy:
    updateMode: "Off"
  resourcePolicy:
    containerPolicies:
    - containerName: backend
      minAllowed:
        cpu: 10m
        memory: 10Mi
      maxAllowed:
        cpu: 300m
        memory: 256Mi
      controlledResources: ["cpu", "memory"]

# Apply the VPA
kubectl apply -f backend-vpa.yaml

Over here we face this issue which means we need to install CRDs for VerticalPodAutoscaler:

error: resource mapping not found for name: "backend-scale-optimizer" namespace: "default" from "a": no matches for kind "VerticalPodAutoscaler" in version "autoscaling.k8s.io/v1"
ensure CRDs are installed first

Let's install the CRDs

# Ref: https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md

# Clone the Kubernetes autoscaler repository
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/

# Install the VPA components
./hack/vpa-up.sh
cd -

Retry applying VerticalPodAutoscaler:

kubectl apply -f backend-vpa.yaml

Verify that the VPA resource was created:

kubectl get vpa backend-scale-optimizer -n default

Check the VPA configuration to ensure it's only targeting the backend container:

kubectl describe vpa backend-scale-optimizer -n default

Details









Q. 15

Task
SECTION: SCHEDULING


For this task you have to use cluster3. Make sure you are using the correct context.

kubectl config use-context cluster3

Create HPA api-hpa for a deployment named api-deployment in the api namespace.

The HPA should scale the deployment based on a custom metric named requests_per_second, targeting an average value of 1000 requests per second across all pods.

Set the minimum number of replicas to 1 and the maximum to 5.

Note: You will see FailedComputeMetricsReplicas error in the description of the HPA. This is expected, so ignore the errors with collecting the custom metrics as the adapter isn't installed.

Solution
Step 1: Create the HPA using the custom metric
Create a file named api-hpa.yaml:

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: 1000

Apply the HPA:

kubectl apply -f api-hpa.yaml

Verify the HPA by describing it:

kubectl describe hpa api-hpa -n api

The will show some errors, but that is okay as we have not configured the custom metric yet.

Details






Q. 16

Task
SECTION: SCHEDULING


For this question, please set the context to cluster3 by running:

kubectl config use-context cluster3

Create an HPA named webapp-hpa
for a deployment named webapp-deployment in the default namespace with service webapp-service associated to it.

The HPA should scale the deployment based on CPU utilization, maintaining an average CPU usage of 50% across all pods with a minimum of 1 and a maximum of 3 replicas.

Configure the HPA to scale down pods cautiously by setting a stabilization window of 300 seconds to prevent rapid fluctuations in pod count.

Solution
First, set the context to cluster3:

kubectl config use-context cluster3

Verify that the webapp-deployment exists:

kubectl get deployment webapp-deployment

Create a file named webapp-hpa.yaml with the following content:

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300

Apply the HPA configuration:

kubectl apply -f webapp-hpa.yaml

Verify that the HPA has been created with the correct configuration:

kubectl get hpa webapp-hpa
kubectl describe hpa webapp-hpa

The output should show the target CPU utilization of 50% and the scale-down stabilization window of 300 seconds.

Understanding Stabilization Windows in HPA
Stabilization windows help prevent rapid fluctuations in the number of replicas during scaling events, particularly when metrics fluctuate around the threshold.

When you set a stabilizationWindowSeconds value for scaling down:

The HPA controller keeps track of the desired number of replicas over a period of time (the stabilization window)
When deciding whether to scale down, it uses the highest recommendation within that window
This prevents "flapping" where pods would be repeatedly created and deleted in response to brief metric changes
In this case, the 300-second (5-minute) stabilization window means that the HPA will only scale down if the recommendation to use fewer replicas has been consistent for at least 5 minutes. This helps maintain system stability and prevents unnecessary pod terminations.

You can also set a separate stabilization window for scaling up by configuring the scaleUp.stabilizationWindowSeconds parameter. When not specified, Kubernetes uses default values of 300 seconds for scale-down and 0 seconds for scale-up.

Details





Q. 17

Task
















Solution














Details


Q. 18

Task

















Solution





























Details


Q. 19

Task












Solution






















Details






Q. 20

Task
SECTION: SERVICE NETWORKING


This task has to be performed on cluster1 context. Make sure you are using the correct context.

kubectl config use-context cluster1

We have a deployment named external-app and a service associated with it named external-service in the external namespace already created.

We have to create an HTTPRoute that routes incoming traffic on path / from gateway to the external-service in the external namespace.

Part 1

Create an HTTP Gateway named web-gateway in the nginx-gateway namespace, which uses the nginx gateway class, and listens on port 80, and allows routes from all namespaces.
Part 2

Create an HTTPRoute named external-route in the external namespace that binds to the web-gateway in the default namespace and routes traffic to the external-service in the external namespace.
In order to test accessing the service, use curl switch to host cluster1-controlplane.

ssh cluster1-controlplane

Hit curl on the gateway to test the service.

# This is the node port where gatewayclass is attached to
curl localhost:30080

Solution
Step 1: Create the GatewayClass named nginx and create a Gateway in the default namespace
Create a file named nginx-gateway.yaml:

# Create the Gateway
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
    allowedRoutes:
      namespaces:
        from: All

Note: The allowedRoutes.namespaces.from: All setting explicitly allows routes from all namespaces to bind to this Gateway, which is required for our cross-namespace routing scenario.

Apply it:

kubectl apply -f nginx-gateway.yaml

Step 2: Create the HTTPRoute in the external namespace
Create a file named external-route.yaml:

apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: external-route
  namespace: external
spec:
  parentRefs:
  - name: web-gateway
    namespace: nginx-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: external-service
      port: 80

Apply the HTTPRoute:

kubectl apply -f external-route.yaml

Step 3: Verify that the HTTPRoute is created and configured correctly
Describe the HTTPRoute to see its configuration:

kubectl describe httproute external-route -n external

Step 4: Test the routing functionality
Get the external IP of the Gateway:

kubectl get gateway web-gateway -n nginx-gateway

Test accessing the service using curl:

So switch to host cluster1-controlplane

ssh cluster1-controlplane

curl localhost:30080

You should see traffic being routed to the external-service in the external namespace.

Details
