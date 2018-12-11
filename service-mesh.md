## Service Mesh

In this lab you will enable tracing and monitoring of your backend services using Service Mesh.

#### What is OpenShift Service Mesh?

Red Hat OpenShift Service Mesh is a platform that provides behavioral insight and operational control over the service mesh, providing a uniform way to connect, secure, and monitor microservice applications.

The term *service mesh* describes the network of microservices that make up applications in a distributed microservice architecture and the interactions between those microservices. As a service mesh grows in size and complexity, it can become harder to understand and manage.

Based on the open source [Istio](https://istio.io/) project, Red Hat OpenShift Service Mesh adds a transparent layer on existing distributed applications without requiring any changes to the service code. You add Red Hat OpenShift Service Mesh support to services by deploying a special sidecar proxy throughout your environment that intercepts all network communication between microservices. You configure and manage the service mesh using the control plane features.

Red Hat OpenShift Service Mesh provides an easy way to create a network of deployed services that provides discovery, load balancing, service-to-service authentication, failure recovery, metrics, and monitoring. A service mesh also provides more complex operational functionality, including A/B testing, canary releases, rate limiting, access control, and end-to-end authentication.

#### OpenShift Service Mesh Architecture

Red Hat OpenShift Service Mesh is logically split into a data plane and a control plane:

* The **data plane** is composed of a set of intelligent proxies deployed as sidecars. These proxies intercept and control all inbound and outbound network communication between microservices in the service mesh; sidecar proxies also communicate with Mixer, the general-purpose policy and telemetry hub.

* The **control plane** is responsible for managing and configuring proxies to route traffic, and configuring Mixers to enforce policies and collect telemetry.

![Istio Architecture]({% image_path istio_architecture.svg %}){:width="400px"}

The components that make up the data plane and the control plane are:

* **Envoy proxy** - is the data plane component that intercepts all inbound and outbound traffic for all services in the service mesh. Envoy is deployed as a sidecar to the relevant service in the same pod.

* **Mixer** - is the control plane component responsible responsible for enforcing access control and usage policies (such as authorization, rate limits, quotas, authentication, request tracing) and collecting telemetry data from the Envoy proxy and other services.

* **Pilot** - is the control plane component responsible for configuring the proxies at runtime. Pilot provides service discovery for the Envoy sidecars, traffic management capabilities for intelligent routing (for example, A/B tests or canary deployments), and resiliency (timeouts, retries, and circuit breakers).

* **Citadel** - is the control plane component responsible for certificate issuance and rotation. Citadel provides strong service-to-service and end-user authentication with built-in identity and credential management. You can use Citadel to upgrade unencrypted traffic in the service mesh. Using Citadel, operators can enforce policies based on service identity rather than on network controls.

#### Allowing Service Mesh to your project

Istio needs to manipulate the iptables rules of pods in order to intercept connections to the application containers. In order to add pods within your project to the Openshift Service Mesh, your project needs to gain access to the *privileged* Security Context Constraints (SCC). 

> cluster-admin privileges is needed. Please ask to the instructor to run the following commands

~~~shell
$ oc adm policy add-scc-to-user privileged -z default -n {{COOLSTORE_PROJECT}}
$ oc delete limits --all -n {{COOLSTORE_PROJECT}} // A bit too hard
~~~

#### Injecting Sidecar Proxy to Catalog Service

Let's add the annotation `sidecar.istio.io/inject:true` in the Catalog DeploymentConfig to automatically inject to sidecar to your service.

~~~shell
$ oc rollout pause dc/catalog
$ oc patch dc/catalog -p '{"spec": {"template": {"spec": {"containers": [{"name": "spring-boot", "command" : ["/bin/bash"], "args": ["-c", "curl http://localhost:15000/server_info | grep live && sleep 10 && /usr/local/s2i/run"]}]}}}}'
$ oc set probe dc/catalog --readiness --liveness --initial-delay-seconds=30 --failure-threshold=3 -- curl -f http://localhost:8080/health
$ oc patch dc/catalog -p '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}'
$ oc rollout resume dc/catalog
~~~

To confirm that the application is successfully deployed, run this command:

~~~shell
$ oc get pods -lapp=catalog,deploymentconfig=catalog
NAME			READY	STATUS		RESTARTS	AGE
catalog-3-ppj47   	2/2	Running		1		8m
~~~

The status should be `Running` and there should be `2/2` pods in the `Ready` column.

#### Injecting Sidecar Proxy to Inventory Service

Let's add the annotation `sidecar.istio.io/inject:true` in the Inventory DeploymentConfig to automatically inject to sidecar to your service.

~~~shell
$ oc rollout pause dc/inventory
$ oc set probe dc/inventory --readiness --liveness --initial-delay-seconds=30 --failure-threshold=3 -- curl -f http://localhost:8080/node
$ oc patch dc/inventory -p '{"spec": {"template": {"spec": {"containers": [{"name": "wildfly-swarm", "command" : ["/bin/bash"], "args": ["-c", "curl http://localhost:15000/server_info | grep live && sleep 10 && /usr/local/s2i/run"]}]}}}}'
$ oc patch dc/inventory -p '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}'
$ oc rollout resume dc/inventory
~~~

To confirm that the application is successfully deployed, run this command:

~~~shell
$ oc get pods -lapp=inventory,deploymentconfig=inventory
NAME			READY	STATUS		RESTARTS	AGE
inventory-2-k6ftf	2/2	Running		1		3m
~~~

The status should be `Running` and there should be `2/2` pods in the `Ready` column.

#### Injecting Sidecar Proxy to Gateway Service

Let's add the annotation `sidecar.istio.io/inject:true` in the Gateway DeploymentConfig to automatically inject to sidecar to your service.

~~~shell
$ oc rollout pause dc/gateway
$ oc set probe dc/gateway --readiness --liveness --initial-delay-seconds=30 --failure-threshold=3 -- curl -f http://localhost:8080/health
$ oc patch dc/gateway -p '{"spec": {"template": {"spec": {"containers": [{"name": "vertx", "command" : ["/bin/bash"], "args": ["-c", "curl http://localhost:15000/server_info | grep live && sleep 10 && /usr/local/s2i/run"]}]}}}}'
$ oc patch dc/gateway -p '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}'
$ oc rollout resume dc/gateway
~~~

To confirm that the application is successfully deployed, run this command:

~~~shell
$ oc get pods -lapp=gateway
NAME			READY	STATUS		RESTARTS	AGE
gateway-2-zqsmn		2/2	Running		1		1m
~~~

The status should be `Running` and there should be `2/2` pods in the `Ready` column.

#### Testing the application

To confirm that the Service Mesh is configured, run several times the following curl command:

~~~shell
$ curl -o /dev/null -s -w "%{http_code}\n" http://gateway.{{COOLSTORE_PROJECT}}.svc:8080/api/products
200
~~~

#### Observing and Tracing with the Service Mesh

Please ask to the instructor

