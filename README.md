# Study Kubernetes

## VLAN

Create Cluster => a private and a public vlan were created automaticlly for each zone => 
if it is a multizone cluster => open a support ticket to enable VRF(vitual routing and forwarding), VRF enables all the private VLANs and subnets in your infrastructure account to communicate with each other. 

## ALB

Ingress application load balancer will be created automatically when your cluster is created.
You can also create new ALB or update the current ALB.
To expose TCP applications, create CM ibm-ingress-deploy-config to specify tcp-ports CM of each ALB.
Then list all your TCP ports in a new CM kube-system/custom-tcp-ports.
Edit each ALB service to open the TCP ports. and do a refresh

Create a YAML file for an ibm-ingress-deploy-config ConfigMap.
```
apiVersion: v1
kind: ConfigMap
metadata:
 name: ibm-ingress-deploy-config
 namespace: kube-system
data:
 private-cr5f6431dcf6a14abd88bb6fada18e226f-alb7: '{"tcpServicesConfig":"kube-system/custom-tcp-ports"}'
 private-cr5f6431dcf6a14abd88bb6fada18e226f-alb8: '{"tcpServicesConfig":"kube-system/custom-tcp-ports"}'
 private-cr5f6431dcf6a14abd88bb6fada18e226f-alb9: '{"tcpServicesConfig":"kube-system/custom-tcp-ports"}'
 public-cr5f6431dcf6a14abd88bb6fada18e226f-alb7: '{"tcpServicesConfig":"kube-system/custom-tcp-ports"}'
 public-cr5f6431dcf6a14abd88bb6fada18e226f-alb8: '{"tcpServicesConfig":"kube-system/custom-tcp-ports"}'
 public-cr5f6431dcf6a14abd88bb6fada18e226f-alb9: '{"tcpServicesConfig":"kube-system/custom-tcp-ports"}'
```

Create a YAML file for an custom-tcp-ports ConfigMap.
```
apiVersion: v1
kind: ConfigMap
metadata:zZ
  name: custom-tcp-ports
  namespace: kube-system
data:
  "3000": qa-datapower/qa-mqdatapower:3000
  "10002": qa-datapower/qa-mqdatapower:10002
  "10003": qa-sftpdsgpub/qa-octopus:10003
  "10004": qa-datapower/qa-sftpdatapower:10004
  "40002": prod-datapower/prod-sftpdatapower:40002
  "40003": prod-datapower/prod-datapower:40003
``` 

Edit ALB service to add ports part to open TCP ports, don't add nodePort!
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    razee.io/build-url: https://travis.ibm.com/alchemy-containers/armada-ingress-microservice/builds/43020212
    razee.io/source-url: https://github.ibm.com/alchemy-containers/armada-ingress-microservice/commit/33ff564362adda87e0db282d97abd907841cf7a0
    service.kubernetes.io/ibm-load-balancer-cloud-provider-ip-type: private
    service.kubernetes.io/ibm-load-balancer-cloud-provider-vlan: "2511385"
    service.kubernetes.io/ibm-load-balancer-cloud-provider-zone: wdc07
  creationTimestamp: "2021-01-07T14:41:51Z"
  finalizers:
  - service.kubernetes.io/load-balancer-cleanup
  labels:
    app: private-crb75ecd28fd3b408a94c9a0b724a28bc7-alb1
  name: private-crb75ecd28fd3b408a94c9a0b724a28bc7-alb1
  namespace: kube-system
  resourceVersion: "180703821"
  uid: 80064ac1-1768-4821-8271-09ad93ba8bf9
spec:
  clusterIP: 172.21.206.194
  clusterIPs:
  - 172.21.206.194
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  loadBalancerIP: 10.191.29.90
  ports:
  - name: port-80
    nodePort: 30835
    port: 80
    protocol: TCP
    targetPort: 80
  - name: port-443
    nodePort: 30129
    port: 443
    protocol: TCP
    targetPort: 443
  - name: port-8001
    nodePort: 30344
    port: 8001
    protocol: TCP
    targetPort: 8001
  - name: port-40018
    nodePort: 30280
    port: 40018
    protocol: TCP
    targetPort: 40018
  selector:
    app: private-crb75ecd28fd3b408a94c9a0b724a28bc7-alb1
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 10.191.29.90
```