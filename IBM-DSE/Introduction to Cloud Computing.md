## Overview of Cloud Computing
Cloud computing gives users on-demand network access to a shared pool of configurable resources, including servers, storage, networks, applications, and services. Providers can provision and release these resources quickly with little manual administration. Compared with locally hosted infrastructure, cloud services can reduce upfront capital expenditure, shorten deployment times, and support rapid changes in capacity. They do not automatically cost less or deliver stronger security, availability, or performance. Outcomes depend on architecture, configuration, contracts, workload patterns, and operational discipline.
### Essential characteristics
The National Institute of Standards and Technology defines five essential characteristics:
- On-demand self-service allows a customer to provision capabilities without requiring human interaction with each provider. It describes the provisioning process, not continuous availability.
- Broad network access makes capabilities available through standard network mechanisms across devices such as computers, tablets, and phones. Public cloud access does not always require the public internet because providers may support private connections.
- Resource pooling lets a provider serve multiple customers through shared physical or virtual resources that it assigns and reassigns according to demand. Customers usually control selected regions or availability zones rather than exact hardware locations.
- Rapid elasticity allows resources to scale out, scale in, scale up, or scale down as demand changes. Capacity can appear extensive, but it remains subject to quotas, regional supply, service limits, and cost.
- Measured service automatically monitors, controls, and reports resource use. Metering supports transparency and may support usage-based billing, but it does not require every service to charge by consumption.

Cloud computing includes three service models. Infrastructure as a service gives customers control over operating systems, applications, and selected network settings while the provider runs the underlying infrastructure. Platform as a service lets customers deploy applications without managing the underlying operating systems and hardware. Software as a service delivers provider-run applications through client interfaces or application programming interfaces.

The four NIST deployment models are private, community, public, and hybrid cloud. A private cloud serves one organisation. A community cloud serves organisations with shared requirements. A public cloud supports general use. A hybrid cloud connects two or more distinct cloud environments so that data and applications can move between them.
### Development of the cloud model
Cloud computing developed from earlier approaches to shared computing. Mainframe time-sharing allowed multiple users to share processing capacity. IBM's CP-40 research project introduced virtualisation in 1964, and IBM announced VM/370 in 1972. Virtualisation later allowed one physical server to run several isolated virtual machines. A hypervisor allocates processing, memory, and storage across those machines.

Virtualisation improved hardware use and helped providers build large resource pools. Widespread networking, automated provisioning, distributed data centres, and usage-based commercial models then supported modern cloud services. Virtual machines provide logical isolation, but they cannot guarantee that every workload will survive a hardware failure, software defect, or security incident. Resilience requires deliberate redundancy, backup, recovery, and testing.
### Business value and adoption
Cloud services can help organisations launch products faster, test ideas at lower initial cost, reach users across regions, and scale variable workloads. Managed databases, analytics, artificial intelligence, messaging, and development platforms can also reduce the specialist infrastructure work that product teams perform. High-performance computing becomes accessible for short, intensive workloads without requiring an organisation to own a permanent cluster.

The commercial model commonly shifts some spending from capital expenditure to operating expenditure. This shift can improve cash flow and flexibility, although idle resources, data transfer, licensing, support, and poorly governed consumption can produce unexpected costs. Organisations therefore need cost allocation, budgets, monitoring, and lifecycle controls.

Major providers offer overlapping infrastructure, platform, software, data, security, and artificial intelligence services. Examples include Alibaba Cloud, Amazon Web Services, Google Cloud, IBM Cloud, Microsoft Azure, Oracle Cloud, Salesforce, and SAP. Provider selection should follow workload and organisational requirements rather than brand breadth alone.
### Strategy, governance, and risk
A sound cloud strategy begins with business outcomes and workload assessment. Organisations should compare cloud, on-premises, edge, and hybrid options against performance, latency, portability, data location, regulatory, continuity, skills, integration, and total-cost requirements. Legacy applications may need redesign, staged migration, or continued local hosting.

Cloud security follows a shared responsibility model. Providers usually protect facilities and underlying services, while customers retain responsibilities that vary by service model. These commonly include identity and access, data classification, configuration, application security, endpoint security, backup choices, monitoring, and legal compliance. Contracts should define availability, support, incident notification, data ownership, exit arrangements, and recovery objectives. No provider removes the customer's accountability for risk.

Strong adoption programs establish governance before uncontrolled use expands. They set architecture standards, security baselines, access controls, cost policies, observability, disaster recovery, and supplier management. They also test portability and exit plans to reduce concentration and lock-in risks.
### Operating cloud workloads
Teams should design each workload for its required service level instead of assuming that a provider makes every application highly available. Multiple availability zones can protect against some local failures, while multiple regions may support broader continuity. Both approaches add cost and complexity. Backups need defined retention, separation from production access, and regular restoration tests.

Cloud dashboards and automation can improve productivity, but rapid self-service can also spread inconsistent configurations. Infrastructure as code, policy checks, central logging, and automated patching help teams apply controls repeatedly. Identity design remains critical because excessive permissions and exposed credentials can undermine otherwise secure services.

Migration should proceed in measured stages. Teams can retire an application, retain it locally, rehost it with limited change, move it to a managed platform, or redesign it for cloud-native services. Pilot workloads can reveal latency, compatibility, cost, and skills gaps before broader migration. After deployment, teams should compare actual performance and expenditure with the business case, then optimise or reverse decisions when evidence supports a change.
### Cloud-enabled technologies
Cloud platforms accelerate technologies that need elastic processing, large-scale storage, global distribution, or managed services.

- Internet of Things systems use cloud services for device registration, identity, data storage, analytics, and fleet management. Edge or fog computing can process time-sensitive data near devices, reduce latency and bandwidth use, and continue selected operations during poor connectivity.
- Artificial intelligence systems use cloud computing to train models, process large datasets, and deliver inference at scale. Connected devices can supply data, and AI can turn that data into predictions or automated actions. Organisations still need data quality, privacy, security, human oversight, and model governance.
- Blockchain systems can provide shared, tamper-evident transaction records among participants. Their trust, privacy, immutability, and performance depend on governance, permissions, consensus design, and implementation. Blockchain does not automatically make an AI system explainable or trustworthy.
- Cloud analytics can combine data streams, identify patterns, and support predictive maintenance. Organisations may process some data at the edge and aggregate it in the cloud.

Reported applications include wildlife monitoring in South Africa, digital services for US Open tennis fans, food supply-chain traceability, and predictive maintenance for KONE equipment. Other cloud adoption examples include customer applications at American Airlines, development platforms at UBank, globally distributed link services at Bitly, and trading infrastructure at ActivTrades. These cases illustrate possible uses, not guaranteed results for every organisation.
## Cloud Computing Models
Cloud computing gives users on-demand network access to a shared pool of configurable resources, including servers, storage, networks, applications, and services. Providers rapidly allocate and release these resources, meter their use, and commonly charge by subscription or consumption. The three main service models divide responsibility across the technology stack. The four deployment models describe who can use the infrastructure, who controls it, and how separate environments connect.

Five characteristics distinguish a cloud service. On-demand self-service lets customers provision capacity without routine provider intervention. Broad network access makes services available through standard mechanisms. Resource pooling allows providers to serve multiple customers while assigning physical and virtual capacity dynamically. Rapid elasticity expands or contracts resources as demand changes. Measured service monitors consumption for control, reporting, and billing. These characteristics describe an operating model, not a location. A virtualised data centre without self-service, elasticity, and measured use does not gain the full capabilities of cloud computing.
### Service models
| Model | Provider manages | Customer manages | Typical users and needs |
| --- | --- | --- | --- |
| Infrastructure as a service (IaaS) | Data centres, physical servers, storage hardware, networking, and virtualisation | Guest operating systems, applications, data, identities, and many network and security settings | Administrators and teams that need detailed control |
| Platform as a service (PaaS) | IaaS components plus operating systems, runtimes, middleware, databases, and development services | Application code, data, access, and service configuration | Developers who want to build and deploy applications quickly |
| Software as a service (SaaS) | The application and its supporting platform and infrastructure | Data, users, access policies, configuration, and lawful use | Individuals and organisations that need ready-to-use software |

Responsibility shifts from the customer to the provider as abstraction increases from IaaS to PaaS and SaaS. Greater abstraction usually improves convenience and reduces operational work, while lower abstraction gives customers more control. The provider never assumes every responsibility. Customers still protect accounts and data, configure services safely, meet legal obligations, and plan for continuity according to the service contract.

Individual offerings may combine categories, so organisations should use each provider's service documentation and contract to confirm the exact division of responsibilities.
### Infrastructure as a service
IaaS supplies compute, networking, and storage over a network on a pay-as-you-go basis. Customers provision virtual machines in available regions and zones, select operating systems, attach storage, and configure networks. Providers often add monitoring, load balancing, automatic scaling, backup, and disaster recovery services.

The main infrastructure components include:
- Compute resources, such as virtual machines, processors, memory, and accelerators
- Network resources, such as virtual networks, addresses, routing, firewalls, and load balancers
- Object, file, and block storage for applications, datasets, and backups
- Physical data centres, power, cooling, hardware, and hypervisors operated by the provider

IaaS suits development and test environments, web hosting, high-performance computing, large-scale analytics, and recovery environments. Organisations can acquire capacity quickly and avoid purchasing physical infrastructure. They must still patch guest operating systems, secure configurations, monitor workloads, and design for availability. IaaS does not automatically make a workload cheaper, safer, or more reliable than other options.
### Platform as a service
PaaS provides a managed environment for developing, testing, deploying, running, and scaling applications. It can include runtimes, databases, application servers, API management, messaging, caching, identity services, analytics, and deployment pipelines. Developers push code while the provider operates the underlying platform.

PaaS reduces infrastructure work and the amount of integration code teams must maintain. It supports rapid prototyping, elastic applications, microservices, APIs, Internet of Things systems, business analytics, and process automation. Its constraints may include supported languages and runtimes, limited low-level control, portability challenges, provider changes, and service outages. Teams should assess exit options, data protection, interoperability, and operational dependencies before adoption.
### Software as a service
SaaS delivers complete applications through a browser, mobile app, or application programming interface. Providers centrally maintain the software, infrastructure, upgrades, availability mechanisms, and platform security. Customers typically pay a subscription and scale licences or usage as demand changes.

Common SaaS categories include email, collaboration, customer relationship management, human resources, finance, billing, commerce, and productivity. SaaS can shorten procurement and deployment, reduce internal maintenance, support remote access, and spread costs over time. Multitenant services commonly share application infrastructure while isolating each customer's data and access.

SaaS often limits deep customisation, although many products support branding, configurable fields, workflows, and feature controls. Organisations remain responsible for identity management, permissions, data governance, retention, regulatory compliance, integration, and contingency planning. Network dependence, provider concentration, data portability, and contract terms require careful review.
### Deployment models
| Model | Core arrangement | Appropriate conditions |
| --- | --- | --- |
| Public cloud | A provider offers pooled services for open use by many customers | Variable demand, rapid provisioning, broad service choice, and economies of scale |
| Private cloud | One organisation receives exclusive use of cloud infrastructure, on premises or off premises | Strong control, specialised requirements, sensitive workloads, or existing investment |
| Community cloud | Organisations with shared mission, security, policy, or compliance needs use exclusive infrastructure | Common governance, assurance, residency, or sector requirements |
| Hybrid cloud | Two or more distinct private, community, or public clouds connect to enable data and application portability | Workload placement across environments, cloud bursting, staged modernisation, and resilience |

A public cloud provider owns and operates the underlying facilities and rents services to customers. Customers gain rapid access to large resource pools, but must evaluate service availability, security controls, data location, sovereignty, performance, and cost. Shared infrastructure does not remove logical isolation or customer accountability.

A private cloud serves one organisation exclusively. The organisation, a third party, or both may own and operate it, and it may run on or off premises. Private clouds can provide self-service, virtualisation, elastic capacity, and tailored controls, but they require sufficient investment and operational capability.

A virtual private cloud (VPC) is not automatically a private cloud deployment. A VPC usually provides a logically isolated virtual network within a public cloud. It helps customers control addressing, routing, connectivity, and network security while the provider continues to operate shared public cloud infrastructure.

A community cloud supports several organisations that share requirements. Physical separation can provide exclusivity, while software-defined isolation and policy controls can enforce boundaries in some architectures. The participating organisations must define governance, eligible users, assurance controls, data location, support access, and accountability.

A hybrid cloud keeps its component clouds distinct but connects them through technology that supports portability. Organisations can retain regulated or stable workloads in private environments and place dynamic or less sensitive workloads in public clouds. Common uses include cloud bursting, disaster recovery, SaaS integration, analytics, legacy application modernisation, and virtual machine migration. Hybrid designs can improve placement flexibility, but they also add complexity across identity, networking, observability, security, data consistency, latency, policy, and cost management.
### Choosing a model
Organisations should select the highest practical level of managed service that satisfies business and technical requirements. SaaS fits standard business capabilities, PaaS fits custom application development, and IaaS fits workloads that require operating-system or network control. Decisions should account for data sensitivity, regulation, portability, skills, time, performance, resilience, total cost, and provider dependence. Many organisations combine service and deployment models because different workloads impose different constraints.
## Components of Cloud Computing
Cloud computing provides on-demand access to shared computing resources that providers can provision, scale, meter, and release with limited manual effort. Infrastructure architects select locations, compute models, networks, and storage according to workload performance, availability, security, compliance, and cost requirements.
### Regions, zones, and data centres
A cloud region is a provider-defined geographic area containing multiple isolated locations, commonly called availability zones. An availability zone can contain one or more data centres with independent power, cooling, networking, and connectivity. Provider-specific labels distinguish regions from zones. For example, `us-east-1` identifies an AWS region, while `us-east-1a` identifies an availability zone within it.

Physical separation limits the effect of local failures, but isolation alone does not make an application highly available. Architects must distribute redundant resources and data across zones or regions, then test failover. Deploying close to customers can reduce latency, while multi-region designs can support disaster recovery and data-residency obligations. These benefits bring additional cost, operational complexity, and data-consistency considerations.

Data centres contain servers, storage systems, network equipment, and supporting facilities. Providers organise equipment into racks and other standard physical units, then expose capacity through software-controlled services and application programming interfaces (APIs).
### Compute models
Cloud platforms offer virtual machines, dedicated physical servers, containers, and serverless services. Each model shifts a different share of control and management between provider and customer.

| Model | Main characteristics | Suitable workloads | Principal trade-offs |
| --- | --- | --- | --- |
| Virtual machine (VM) | A hypervisor allocates host resources to an isolated software-defined computer with a guest operating system | General applications, legacy systems, mixed operating systems, and workloads needing OS control | More overhead and slower start-up than containers, but stronger isolation and broader OS flexibility |
| Bare-metal server | One customer receives a physical server without a provider-controlled virtualisation layer | High-performance computing, specialised hardware, restrictive licensing, and some compliance workloads | Predictable access to hardware, but slower scaling, higher cost, and greater administration |
| Container | An image packages application code, runtimes, libraries, and configuration while containers share a host kernel | Microservices, repeatable delivery, rapid scaling, and portable application deployment | Requires a compatible operating system, processor architecture, runtime, and security model |
| Serverless service | The provider runs code or managed application components in response to requests or events | Intermittent, event-driven, and rapidly varying workloads | Reduces infrastructure administration, but introduces service limits, platform dependencies, and potentially variable latency |
#### Virtualisation and VMs
Virtualisation abstracts physical compute, storage, and network resources. A hypervisor creates and manages VMs, allocating processor time, memory, storage, and network access to each guest.

Type 1 hypervisors run directly on hardware and commonly support servers and data centres. Type 2 hypervisors run on a host operating system and commonly support desktop development, testing, and end-user virtualisation. This distinction does not by itself determine security or performance. Configuration, patching, workload isolation, hardware support, and management controls also shape both outcomes.

Each VM contains a guest operating system and applications. Multiple VMs can run different supported operating systems on one host and remain logically isolated. Live migration can move a running VM between compatible hosts, often with little interruption, but it is not instantaneous or universally available. Migration depends on platform features, compatible processors, storage and network design, available capacity, and workload behaviour.

Providers sell VMs through several purchasing and tenancy models:
- On-demand VMs suit variable use without a long commitment.
- Spot or transient VMs use spare capacity at lower prices but can be interrupted, so fault-tolerant batch processing, development, and other recoverable workloads suit them.
- Commitment discounts reduce eligible usage costs over a term. They do not always reserve capacity.
- Capacity reservations hold capacity in a specified location, subject to provider terms.
- Dedicated instances or hosts place one customer's VMs on dedicated hardware. Dedicated hosts can also provide placement visibility and support some licence models.

These categories differ across providers. Architects must distinguish a billing discount from a capacity guarantee.

VM families combine virtual processors, memory, local storage, and network capacity in ratios suited to general-purpose, compute-intensive, memory-intensive, storage-intensive, or accelerated workloads. Custom sizes can reduce waste when standard shapes fit poorly. Autoscaling can add or remove instances as demand changes, but applications must support horizontal scaling and preserve state outside disposable instances. Rightsizing requires observed utilisation, response time, throughput, and cost data rather than processor counts alone.
#### Bare-metal servers
Bare-metal services allocate a physical server to one customer. The customer can select or install an operating system, and may run a hypervisor to create private VMs. Some services offer graphics processing units or other accelerators for scientific computing, artificial intelligence, analytics, and rendering.

Bare metal avoids shared-host contention and can provide direct hardware access, predictable performance, and licensing control. It does not automatically provide better security or compliance. Organisations still need hardening, patching, access control, monitoring, backup, and recovery. Provisioning time, pricing, available configurations, and management responsibility vary by provider.
#### Containers and orchestration
A container image packages an application with its runtime dependencies and declared settings. A container runtime launches the image as isolated processes. Containers usually consume fewer resources and start faster than VMs because they share the host kernel instead of including a full guest operating system.

The usual workflow defines an image, builds and tests it, stores it in a registry, and deploys containers from that immutable image. This approach improves consistency across development, test, and production environments. Portability remains conditional on compatible operating systems, processor architectures, runtimes, external services, and configuration.

Container orchestration platforms such as Kubernetes schedule containers, maintain desired capacity, manage service discovery, and replace failed instances. Orchestration complements a container runtime. It does not replace one. Containers suit independently scalable services, but VMs or bare metal may remain preferable for stronger tenancy boundaries, unusual kernel requirements, specialised devices, or established applications.
### Cloud networking
Software-defined networking exposes logical network functions through APIs. A virtual private cloud (VPC) supplies an isolated address space that architects divide into subnets. Workloads receive virtual network interfaces and private IP addresses. Routing, gateways, and address translation determine connectivity to other subnets, the internet, on-premises networks, and other clouds.

Public connectivity does not arise simply from attaching a public interface. Route configuration, gateways, public addresses, and security controls must all permit the traffic. Private connectivity can use encrypted virtual private networks or dedicated links. A dedicated link can improve consistency and avoid the public internet, but encryption, authentication, redundancy, and provider controls still determine security.

Security groups commonly filter traffic for associated interfaces or resources. Network access control lists commonly apply broader subnet-level rules. Exact behaviour, including whether controls are stateful or stateless, varies by provider. Identity controls, least-privilege rules, segmentation, logging, encryption, and continuous review remain essential.

Managed load balancers distribute connections across healthy targets to improve scalability and availability. Virtual firewalls, domain name services, gateways, and traffic analysis services perform other functions formerly delivered mainly by physical appliances.

A multi-tier application commonly separates internet-facing services, application services, and databases into different subnets or security groups. Routes and rules should permit only the flows each tier requires. Public gateways expose intended endpoints, while private subnets can reach approved external services through controlled egress. Hybrid environments connect cloud networks to on-premises networks through VPNs or dedicated links. Load balancers perform health checks and route requests to healthy application instances, but they cannot correct failures in shared dependencies such as a single database or misconfigured identity service.
### Cloud storage
Storage selection depends on access pattern, latency, throughput, input/output operations per second (IOPS), durability, availability, sharing, security, lifecycle, recovery objectives, and total cost. IOPS counts operations, while throughput measures data transferred over time and latency measures response delay. None alone represents storage performance.

| Storage type | Access model | Typical uses | Key considerations |
| --- | --- | --- | --- |
| Local or direct-attached | Presented from the host or nearby hardware to one compute instance | Boot disks, caches, scratch data, and temporary high-speed processing | Often fast, but commonly tied to the instance and less resilient unless the service adds protection |
| File | Provides a shared hierarchical file system through protocols such as NFS or SMB | Shared directories, content repositories, home directories, and collaborative workloads | Supports familiar file operations and concurrent clients, with performance governed by the service and network |
| Block | Presents raw volumes that an operating system formats and mounts | Boot volumes, transactional databases, and latency-sensitive applications | Offers predictable low-level storage, but attachment and multi-writer support depend on the service and file system |
| Object | Stores objects and metadata in buckets or containers through HTTP APIs | Media, logs, backups, archives, data lakes, and static application assets | Scales to very large datasets, but applications must use an object API rather than treat it as a normal disk |

File and block services often use remote, provider-managed storage rather than a particular network medium. Neither fibre nor Ethernet alone guarantees performance. Some file services deliver high throughput and low latency, while some block services support attachment to multiple instances under strict coordination.

Object storage uses a flat key space, although key names can imitate folders. Buckets have quotas and service limits, so they are highly scalable rather than literally infinite. Applications normally upload, retrieve, or replace whole objects instead of modifying blocks in place. Many providers support APIs compatible with Amazon S3, which functions as a widely adopted interface rather than a universal formal standard.

Storage classes align cost with access frequency and resilience. Frequently accessed data may use a standard class, while infrequent or archival data may use cheaper classes with retrieval fees, minimum retention periods, or delayed access. Object storage is not always slow. Some classes provide millisecond access, while deep archive retrieval can take hours.

Persistent storage outlives its compute instance, while ephemeral storage is deleted or becomes inaccessible when the associated resource ends. Snapshots capture point-in-time storage state. Many services store subsequent snapshots incrementally, but implementation, application consistency, restoration speed, and file-level recovery vary. Snapshots support recovery but do not replace independent backups, tested restores, retention policies, or cross-location protection.

Storage costs can include provisioned capacity, operations, performance levels, retrieval, data transfer, replication, snapshots, and early-deletion charges. The least expensive capacity can therefore produce a higher total cost for an access-heavy workload. Lifecycle rules can move ageing objects to cheaper classes or delete them after an approved retention period. Encryption should protect data at rest and in transit, with access governed by identities, roles, network controls, and auditable policies. Highly available storage protects service continuity, while durable storage protects data against loss. Architects must evaluate both qualities because one does not guarantee the other.
### Content delivery networks
A content delivery network (CDN) caches eligible content at distributed edge locations and serves it closer to customers. Shorter network paths can reduce latency, while caching can lower origin traffic and cost. CDNs work best for cacheable static assets, media, downloads, and selected dynamic content.

A CDN does not secure an origin through obscurity. Effective protection combines restricted origin access, transport encryption, web application filtering, distributed denial-of-service protection, authentication, logging, and sound cache rules. Cache misses still reach the origin, so origin capacity and resilience remain important.
### Architecture decisions
No compute or storage model dominates every workload. Architects should begin with workload requirements, then test realistic performance, failure, recovery, and cost scenarios. A resilient design combines appropriate locations, redundant services, least-privilege networking, compatible compute models, fit-for-purpose storage, observability, automation, and tested recovery procedures.