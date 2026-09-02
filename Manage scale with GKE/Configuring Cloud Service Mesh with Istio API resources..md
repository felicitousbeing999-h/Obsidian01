In this section, you learn to configure Cloud Service Mesh with Istio API resources. Cloud Service Mesh is configured in YAML files called Kubernetes Custom Resource Definitions, CRDs. The CSM control plane component reads the CRDs and shares them with the Envoy proxies via the Envoy XDS API. 
![Pasted image 20260806155655](../Pasted%20image%2020260806155655.png)The Envoy XDS API is a set of protocols used by Envoy proxies or Proxless gRPC to dynamically fetch configurations from the control plane.
![Pasted image 20260806155720](../Pasted%20image%2020260806155720.png)Let's explore the Istio API resources and their CRDs. There are five primary Istio CRDs. Virtual Service, Destination Rule, Gateway, Service Entry, and Sidecar. This diagram displays where each of these API resources fits into the traffic management architecture.
![Pasted image 20260806155810](../Pasted%20image%2020260806155810.png)You can think of Virtual Service as the where and Destination Rule as the how. Virtual Service defines how requests for a service are routed within an Istio Service Mesh. Can use routing rules to determine the destination of a request.
![Pasted image 20260806155841](../Pasted%20image%2020260806155841.png)Can use rules to route traffic based on elements like request headers, target hostname, and URL path. And can use target destinations as a service in the mesh registry using proxies, a subset of the service, specific group of pods using labels to select, or a non-proxied service registered via a service entry.

![Pasted image 20260806155908](../Pasted%20image%2020260806155908.png)Destination Rule defines how traffic aimed at a particular destination gets handled. Can break a single service into multiple subsets, sub-collections of pods, using labels to select. The Virtual Service can then route to the subset. Can specify load balancing behavior within the destination. And can specify TLS security mode, circuit breaker settings, and other service level properties. A request coming from a client will be routed using both the Virtual Service settings to pick a destination and Destination Rule settings to select how to connect to that destination. 



![Pasted image 20260806155957](../Pasted%20image%2020260806155957.png)Next, let's define gateways. There are two types of gateways which are used to manage inbound and outbound traffic for the mesh. Ingress gateways for managing incoming traffic. And egress gateways for managing outgoing traffic. Gateway configuration settings are applied to a deployment on Envoy proxy pods running on the edge of the mesh, not as sidecars to particular workloads. 


![Pasted image 20260806160027](../Pasted%20image%2020260806160027.png)The next API resource is the service entry, which is commonly used to enable requests to services outside of in-cloud service mesh. Service entry is useful for accessing external APIs or integrating with legacy services that are not part of the mesh. 

![Pasted image 20260806160054](../Pasted%20image%2020260806160054.png)Finally, sidecars are used to fine-tune Envoy proxy settings. By default, sidecar proxies are configured to accept traffic on all ports used by a workload and can reach every workload in the mesh. You can use sidecar configurations to limit protocols, ports, or accessible services in the mesh. Two other common API resources are workload entry and workload group. \

![Pasted image 20260806160128](../Pasted%20image%2020260806160128.png)
Workload entry configurations are used to onboard non-Kubernetes workloads. Examples of non-Kubernetes workloads include virtual machines and bare-metal servers. A workload entry uses a service entry to select workloads and provide the service definition.

![Pasted image 20260806160155](../Pasted%20image%2020260806160155.png)
Workload group describes the collection of workload instances. Workload group is designed for non-Kubernetes workloads, replicating the sidecar injection and deployment model used in Kubernetes to bootstrap Istio proxies.