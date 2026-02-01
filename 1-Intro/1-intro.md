- prerequisites
- what is envoy
- envoy building blocks

**What is Envoy?** \
The industry is moving toward microservice architectures and cloud-native solutions. With hundreds and thousands of microservices developed using different technologies, these systems can become complex and hard to debug.

As an application developer, you’re thinking about business logic -- purchasing a product or generating an invoice. However, any business logic like that will result in multiple service calls between different services. Each of the services probably has its timeouts, retry logic, and other network-specific code that might have to be tweaked or fine-tuned.

If the initial request fails at any point, tracing it through multiple services will be challenging, making it difficult to pinpoint the failure and understand the cause. Was the network unreliable? Do we need to adjust retries or timeouts? Or is it a business logic issue or a bug?

Adding to this complexity of debugging is the fact that services might use inconsistent tracing and logging mechanisms. These issues make it hard to identify the problem, where it happened, and how to fix it. This is especially true if you’re an application developer and debugging network issues falls outside of your core skills.

What would make debugging these network issues easier is to push networking concerns out of the application stack and have another component deal with the networking part. This is what Envoy can do.

In one of its deployment patterns, we have an Envoy instance running next to every service instance. This type of deployment is also called a sidecar deployment. The other pattern Envoy works well with is the edge proxy, which is used to build API gateways.

The Envoy and the application form an atomic entity, but are still separate processes. The application deals with the business logic, and Envoy deals with network concerns.

In case of a failure, separating concerns makes it easier to determine if the failure is coming from the application or the network.

To help with network debugging, Envoy provides the following high-level features:

**Out-of-process architecture** \
Envoy is a self-contained process designed to run alongside every application -- the sidecar deployment pattern we mentioned earlier. The collection of centrally configured Envoys forms a transparent mesh.

The responsibility of routing and other network features gets pushed to Envoy. Applications send requests to a virtual address (localhost) instead of a real one (like a public IP address or a hostname), unaware of the network topology.
Applications no longer bear the responsibility of routing as that task is delegated to an external process.

Instead of having applications manage their network configuration, the network configuration is managed independently of the application at the Envoy level.In an organization, this can free app developers to focus on the business logic of the application.

Envoy works with any programming language. You can write your applications in Go, Java, C++, or any other language, and Envoy can bridge the gap between them. Its behavior is identical, regardless of the application’s programming language or the operating system they are running on.

Envoy can also be deployed and upgraded transparently across the entire infrastructure. This is compared to deploying library upgrades for each separate app, which can be extremely painful and time-consuming.

The out-of-process architecture is beneficial as it gives us consistency across programming languages/application stacks, and we get an independent lifecycle and all of the Envoy networking features for free, without having to address them in every application individually.

**L3/L4 filter architecture** \
Envoy is an L3/L4 network proxy that makes decisions based on IP addresses and TCP or UDP ports. It features a plugable filter chain to write your filters to perform different TCP/UDP tasks.

A filter chain comes from a shell idea where the output of one operation is piped into another operation. For example:

ls -l | grep "Envoy*.cc" | wc -l
Envoy can construct logic and behavior by stacking desired filters that form a filter chain. Many filters exist and support tasks such as raw TCP proxy, UDP proxy, HTTP proxy, TLS client cert authentication, etc. Envoy is also extensible, and we can write our filters.

**L7 filter architecture** \
Envoy supports an additional HTTP L7 filter layer. We can insert HTTP filters into the HTTP connection management subsystem that performs different tasks such as buffering, rate limiting, routing/forwarding, etc.

**First-class HTTP/2 support** \
Envoy supports both HTTP/1.1 and HTTP/2 and can operate as a transparent HTTP/1.1 to HTTP/2 proxy in both directions. This means that any combination of HTTP/1.1 and HTTP/2 clients and target servers can be bridged. Even if your legacy applications aren't communicating via HTTP/2, if you deploy them alongside the Envoy proxy, they'll end up communicating via HTTP/2.

The recommended service-to-service configuration uses HTTP/2 between all Envoys to create a mesh of persistent connections in which requests and responses can be multiplexed.

**HTTP routing** \
When operating in HTTP mode and using REST, Envoy supports a routing subsystem capable of routing and redirecting requests based on path, authority, content type, and runtime values. This functionality is useful when using Envoy as a front/edge proxy for building API gateways and is leveraged when building a service mesh (sidecar deployment pattern).

**gRPC ready** \
Envoy supports all HTTP/2 features required to be used as the routing and load balancing substrate for gRPC requests and responses.

gRPC is an open-source remote procedure call (RPC) system that uses HTTP/2 for transport and protocol buffers as the interface description language (IDL), and that provides features such as authentication, bidirectional streaming, and flow control, blocking/nonblocking bindings, and cancellation and timeouts.

**Service discovery and dynamic configuration** \
We can configure Envoy using static configuration files that describe the services and how to communicate with them.

For advanced scenarios where statically configuring Envoy would be impractical, Envoy supports dynamic configuration and automatically reloads configuration at runtime. A set of discovery services called xDS can be used to dynamically configure Envoy through the network and provide Envoy information about hosts, clusters, HTTP routing, listening sockets, and cryptographic material. At that time, Envoy will attempt to gracefully drain all connections.

**Health checking** \
One feature associated with load balancers is routing traffic only to healthy and available upstream services. Envoy supports a health checking subsystem that performs active health checks of upstream service clusters. Envoy then uses the union of service discovery and health checking information to determine healthy load balancing targets. Envoy can also support passive health checking via an outlier detection subsystem.

**Advanced load balancing** \
Envoy supports automatic retries, circuit breaking, global rate limiting (using an external rate-limiting service), request shadowing (or traffic mirroring), outlier detection, and request hedging.

**Front/edge proxy support** \
Envoy has features that make it well-suited to run as an edge proxy. Such features include TLS termination, HTTP/1.1, HTTP/2, and HTTP/3 support, and HTTP L7 routing.

**TLS termination** \
The decoupling of the application and the proxy enables TLS termination (mutual TLS) between all services in the mesh deployment model.

**Best-in-class observability** \
For observability, Envoy generates logs, metrics, and traces. Envoy currently supports statsd (and compatible providers) as the statistics sink for all subsystems. Thanks to extensibility, we can also plug in different statistics providers if needed.

**HTTP/3 (Alpha)** \
Envoy 1.19.0 supports HTTP/3 upstream and downstream and translates between HTTP/1.1, HTTP/2, and HTTP/3 in either direction.