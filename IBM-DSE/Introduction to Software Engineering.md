# Introduction to Software Engineering
## The Software Development Lifecycle
### Scope and origins
Software engineering applies a systematic, disciplined, and measurable approach to the development, operation, maintenance, and retirement of software. It combines technical work with requirements analysis, design, quality assurance, security, delivery, documentation, and collaboration. The discipline aims to produce software that satisfies stakeholder needs and remains reliable, secure, usable, maintainable, and efficient within its operating context.

Engineers balance competing qualities rather than maximise one in isolation. For example, stronger security can add friction, higher performance can increase cost, and rapid delivery can reduce time for assurance. Explicit trade-offs, recorded assumptions, measurable acceptance criteria, and feedback help stakeholders choose appropriate compromises and revisit them when conditions change.

The field gained formal recognition during the late 1960s as organisations struggled to build increasingly large and complex systems. Projects often exceeded budgets and schedules, failed to satisfy requirements, or became difficult to change. The 1968 and 1969 NATO conferences helped establish the term "software engineering" and promoted engineering approaches to these problems. No single event ended the software crisis, and modern projects still face cost, schedule, quality, and complexity risks. Better methods, tools, automation, computing resources, and professional practices help teams manage those risks.

Employers often use "software engineer", "software developer", and "programmer" interchangeably. In some organisations, an engineer takes wider responsibility for architecture, quality attributes, operations, and the full life cycle, while a developer concentrates on implementing software. These distinctions describe local roles rather than universal professional boundaries. Regulation, professional licensing, and immigration rules can also assign specific meanings to job titles in some jurisdictions.
### Software life cycles
A software development life cycle provides a framework for organising work from an initial need through development, operation, maintenance, and retirement. It does not prescribe one fixed sequence or guarantee a particular cost, completion date, or quality level. Teams select and tailor processes to the product, risks, regulatory setting, organisational context, and delivery method.

Many teams group the work into these activities:
1. Planning and requirements define the problem, stakeholders, constraints, risks, resources, and measures of success.
2. Architecture and design translate requirements into system structures, interfaces, data models, user interactions, and technical decisions.
3. Construction implements the design through code, configuration, data, and infrastructure.
4. Verification and validation assess whether work products satisfy specifications and stakeholder needs.
5. Deployment releases approved changes to target environments and makes them available to users or operators.
6. Operation, maintenance, and retirement monitor the system, resolve defects, improve capabilities, manage obsolescence, and withdraw the system safely.

These activities can overlap and recur. Iterative, incremental, continuous-delivery, and DevOps approaches integrate feedback, testing, security, deployment, and operational learning throughout development. A well-tailored life cycle clarifies responsibilities and decision points, improves communication and traceability, exposes risks earlier, and supports controlled change.
### Requirements and design
Requirements engineering identifies stakeholders and defines the outcomes, capabilities, constraints, interfaces, and quality attributes that a system must satisfy. Teams establish goals, elicit needs through interviews, workshops, observation, surveys, prototypes, and existing evidence, then analyse and prioritise the resulting requirements. Effective requirements remain necessary, clear, feasible, consistent, traceable, and verifiable. Teams confirm them with authorised stakeholders and manage changes throughout the life cycle.

A user requirements specification can record user and business needs. A system requirements specification defines requirements for the whole system, including people, processes, hardware, software, data, interfaces, policies, safety, security, and performance. A software requirements specification defines the requirements allocated to software. Organisations may combine, divide, or rename these information items, so a user specification does not automatically form a subset of a software specification.

Architecture and design turn requirements into implementable decisions. Architects and development teams define components, responsibilities, boundaries, interfaces, data structures, deployment topology, and quality strategies. Prototypes can clarify needs or test design ideas at any stage. Teams record decisions at the level needed for implementation, assurance, operation, and future change.
### Development approaches
| Approach | Structure | Strengths and constraints |
| --- | --- | --- |
| Waterfall | Work progresses mainly through planned, sequential stages and formal hand-offs. | Clear baselines and reviews support stable or tightly controlled work, but late changes can prove costly. |
| V-model | Definition and design activities correspond to later verification and validation activities. | Early test planning strengthens traceability, but a rigid implementation handles changing needs poorly. |
| Agile | Teams develop in short, iterative cycles, seek frequent feedback, and adapt plans as evidence changes. | Frequent learning supports change, but uncertain scope complicates long-range estimates and requires active stakeholder involvement. |

Agile values individuals and interactions, working software, customer collaboration, and responsiveness to change while recognising value in processes, tools, documentation, contracts, and plans. Agile does not replace the software life cycle. It guides how teams conduct life-cycle work.

Scrum implements an agile approach through Sprints of one month or less. Each Sprint creates a valuable, usable Increment that meets the Definition of Done, although the organisation need not release that Increment immediately. Scrum defines three accountabilities: Developers, Product Owner, and Scrum Master. It does not treat a Scrum Master as an agile project manager. The Product Owner maximises product value and manages the Product Backlog, while the Scrum Master establishes Scrum and improves team effectiveness.
### Quality, testing, release, and versioning
Quality depends on both product attributes and development practices. Maintainable code uses coherent structure, meaningful names, appropriate comments, consistent conventions, and controlled dependencies. Teams strengthen quality through peer review, automated builds, linters, static analysis, secure coding, automated tests, configuration management, and continuous integration. Security work includes defining security requirements, protecting development environments and artefacts, reviewing code, checking dependencies, and responding to vulnerabilities.

Testing supplies evidence about quality and risk. It can reveal failures and defects, but it cannot guarantee error-free software. Teams design tests as requirements and designs emerge rather than waiting for all requirements to become final. A test case commonly records its objective, preconditions, inputs, steps, and expected results.

Common test types include:
- Functional testing checks specified behaviour and outputs.
- Non-functional testing evaluates attributes such as performance, security, usability, compatibility, resilience, scalability, and accessibility.
- Regression testing checks whether changes have damaged existing behaviour.

Teams apply these types at several test levels. Component tests examine units in isolation. Component integration tests examine interactions between components. System integration tests examine interfaces with other systems or services. System tests assess the complete system. Acceptance tests assess readiness against user, business, operational, contractual, or regulatory criteria. User acceptance, operational acceptance, alpha, and beta testing represent distinct forms or contexts, not interchangeable labels. Teams often combine manual, automated, static, dynamic, scripted, and exploratory techniques.

Release labels such as alpha, beta, release candidate, and general availability express maturity or audience according to an organisation's policy. They do not carry universal feature or quality guarantees. Deployment may use staged environments, pilot groups, feature flags, canary releases, or progressive rollouts to limit risk.

Version schemes also vary. Semantic Versioning uses `MAJOR.MINOR.PATCH`: incompatible public API changes increase MAJOR, backward-compatible functionality increases MINOR, and backward-compatible fixes increase PATCH. Pre-release identifiers can follow a hyphen, and build metadata can follow a plus sign. A fourth numeric field does not form part of the Semantic Versioning core. Date-based schemes and product-specific conventions remain valid alternatives. Backward compatibility means a newer version continues to support defined behaviours, interfaces, or data from earlier versions.
### Engineering control and maintenance
Modern engineering environments extend the computer-aided software engineering tools that grew during the 1980s. Toolchains can support business analysis, modelling, source control, issue tracking, code generation, debugging, verification, validation, dependency analysis, build automation, deployment, observability, measurement, and project coordination. Tools improve consistency and feedback when teams configure and govern them well, but they do not replace technical judgement or stakeholder decisions.

Configuration management identifies controlled items such as source code, infrastructure definitions, data schemas, documentation, tests, third-party components, and release packages. Version control records authorised changes, while build and release records connect deployed artefacts to their sources and approvals. Traceability links needs and requirements to designs, implementation, tests, defects, and releases. These controls help teams reproduce builds, assess proposed changes, investigate incidents, satisfy audits, and recover from failures.

Teams also distinguish delivery, deployment, and release. Delivery makes a change available for deployment. Deployment installs it in an environment. Release exposes it to intended users. Automation can perform these actions together, but separating them enables staged testing, controlled activation, and rapid rollback.

Maintenance begins as soon as a system enters use and continues until retirement. Corrective maintenance fixes faults. Adaptive maintenance responds to changes in platforms, dependencies, regulations, or operating conditions. Perfective maintenance improves capability, performance, usability, or maintainability. Preventive maintenance reduces future risk through work such as refactoring, dependency updates, and test improvement. Operational telemetry, support requests, security reports, incident reviews, and user research feed new requirements and design decisions. Teams prioritise this work against risk, value, urgency, and available resources rather than treating maintenance as a final, isolated phase.
### Documentation and collaboration
Documentation supports both the product and the processes used to create and operate it. Product documentation can include requirements, architecture decisions, designs, API references, code comments, test plans, traceability records, release notes, runbooks, maintenance guides, tutorials, and user help. Process documentation describes workflows, controls, and responsibilities. A standard operating procedure gives detailed, organisation-specific instructions for a recurring task.

Teams choose written, graphical, interactive, or video formats for their audiences and risks. They create, review, version, and update documentation throughout the life cycle, especially when requirements, interfaces, behaviour, controls, or procedures change.

Software delivery relies on complementary roles and close communication:
- Stakeholders define needs, constraints, priorities, and acceptance expectations.
- Project and program managers coordinate scope, schedules, budgets, dependencies, risks, and communication where the delivery model uses those roles.
- Product managers shape product strategy and market outcomes, while Product Owners hold the specific accountability that Scrum defines.
- System, software, and solution architects guide structural and cross-cutting technical decisions.
- User experience designers and researchers study users, design interactions, and evaluate usability and accessibility.
- Developers design, build, review, test, deploy, and maintain software according to their team's responsibilities.
- Test and quality specialists design assurance strategies, assess risks, execute tests, and communicate findings.
- Site reliability engineers apply software engineering to operations, reliability, automation, monitoring, incident response, capacity, and performance.
- Technical writers and information developers create accurate content for users, operators, maintainers, and other technical audiences.

Titles and boundaries vary by organisation. Cross-functional teams reduce damaging hand-offs by involving relevant specialists early, sharing evidence, reviewing work together, and maintaining clear responsibility for decisions and outcomes.
## Introduction to Software Development
### Web architecture
Web applications divide work among clients, servers, data stores, and supporting services. When a user enters a URL, the browser normally resolves the domain name through the Domain Name System, establishes a connection, and sends an HTTP request. The server returns an HTTP response containing a status code, headers, and content. The browser parses the HTML, requests linked resources such as style sheets, scripts, images, and fonts, then renders the page.

HTML gives web content meaning and structure, CSS controls presentation, and JavaScript adds behaviour and application logic. A server can return stored static files or generate a response from templates, application code, databases, and external services. Modern pages often combine server-side rendering, client-side rendering, static assets, and API calls.

Cloud applications use remotely managed compute, storage, networking, databases, messaging, identity, and other services. Cloud hosting does not automatically make an application scalable or resilient. Architects must design for demand, failure, security, data consistency, recovery, monitoring, and cost. Front-end, back-end, platform, security, and operations specialists often share that responsibility.

Application design should also separate configuration from code, automate infrastructure changes, and define service-level objectives for availability, latency, and recovery. Observability should combine logs, metrics, and traces so operators can diagnose failures and verify that releases behave as intended.
### Front-end development
Front-end development covers the interface and behaviour that a browser presents to the user. Semantic HTML defines headings, forms, links, media, controls, and document regions. CSS controls colour, typography, spacing, layout, animation, and adaptation to different environments. JavaScript, a multi-paradigm programming language, handles events, updates the document, communicates with services, and manages application state.

Standards-compliant code alone cannot ensure identical rendering across every browser and device. Teams use progressive enhancement, accessibility practices, feature detection, automated checks, and testing on supported environments. Responsive design uses flexible layouts, images, media queries, and related CSS features to adapt one design to available space and user preferences. Adaptive design can select distinct layouts or experiences for defined conditions. Neither approach requires removing useful information from smaller screens.

Sass, which stands for Syntactically Awesome Style Sheets, compiles a stylesheet language to CSS. Its SCSS syntax supports CSS-compatible code alongside variables, nesting, mixins, and functions. Less, which stands for Leaner Style Sheets, also extends CSS and compiles to CSS. These tools can organise large style systems, but native CSS now supplies some features that once required preprocessors.

Frameworks and libraries add reusable structure. Angular provides a broad web framework maintained by Google. React provides a component-based user-interface library governed by the React Foundation under the Linux Foundation. Vue provides an incrementally adoptable framework that can scale from focused interface enhancement to a full application. Project requirements, team skills, ecosystem support, accessibility, performance evidence, and maintenance costs should guide selection. Blanket rankings of these tools obscure workload-specific trade-offs.
### Back-end development
Back-end services receive requests, enforce rules, coordinate data, and return responses. They commonly handle routing, input validation, business logic, authentication, authorisation, database access, file processing, messaging, and integration with external systems. Authentication verifies an identity, while authorisation decides what that identity may do.

An application programming interface defines how software components interact. HTTP APIs often exchange JSON, but an API does not require JSON or XML. A route associates an HTTP method and path with handling logic, while an endpoint identifies an exposed operation at an address. A server may return `404 Not Found` when it cannot locate a requested resource, but other failures require other status codes.

Back-end developers can use many languages and frameworks. Node.js provides an asynchronous, event-driven JavaScript runtime, while Express supplies a web framework for Node.js. Python frameworks include Django and Flask. Database work may involve relational systems, document stores, caches, search engines, and queues. Object-relational mapping tools reduce repetitive mapping code, but developers still need to understand queries, transactions, indexes, constraints, and performance.

Security spans the whole system. Teams protect traffic with HTTPS, hash passwords with suitable password-hashing algorithms, validate untrusted input, apply least privilege, manage secrets, patch dependencies, log security events, and protect sensitive data at rest. Front-end controls cannot replace server-side enforcement.
### Teamwork and pair programming
Effective teams align on goals, responsibilities, interfaces, quality standards, communication channels, and decision rights. Cross-functional collaboration gives developers access to product, design, data, security, testing, and operational perspectives. Planning sessions, design reviews, code reviews, demonstrations, retrospectives, mentoring, and shared documentation help teams detect gaps and spread knowledge. These practices support quality, but they do not guarantee fewer defects or lower stress in every context.

Some organisations call a small, cross-functional, self-managing team a squad. In software delivery, the term gained prominence through Spotify's organisational model and does not define a general Agile role or universal team size. Scrum instead defines a Scrum Team with a Product Owner, Scrum Master, and Developers, typically totalling 10 or fewer people.

Pair programming places two developers at one workstation or in a shared remote environment. In driver-navigator pairing, the driver operates the keyboard while the navigator reviews, considers the wider design, and guides the work. The pair swaps roles regularly. Ping-pong pairing combines pairing with test-driven development as one developer writes a failing test and the other writes code to pass it. Strong-style pairing routes an idea through the other person's hands. All styles require continuous discussion and respectful challenge.

Pairing can accelerate onboarding, expose assumptions, share context, and provide immediate review. It also consumes two people's attention, requires compatible schedules, and can cause fatigue or imbalance. Teams should use it where the learning, risk, or complexity justifies the cost, then assess results rather than assume universal gains.
### Development and delivery tools
Code editors provide text-oriented development features, while integrated development environments add combinations of navigation, refactoring, building, testing, debugging, profiling, and source-control integration. The boundary remains flexible, especially when extensions add capabilities.

Version control records changes and supports recovery, comparison, branching, and collaboration. Git provides distributed version control. GitHub hosts Git repositories and adds pull requests, reviews, issue tracking, automation, and access controls. Git can merge unambiguous changes, but people must resolve conflicting edits when automation cannot reconcile them.

Libraries provide reusable code that an application calls. Frameworks usually supply more application structure and often call project code through inversion of control. The distinction forms a spectrum, and teams can sometimes introduce a framework incrementally into an existing system. Compatibility, licensing, security, maintenance, and dependency health deserve review before adoption.

Continuous integration merges changes frequently and runs automated builds and tests to produce assessed artefacts. Continuous delivery keeps approved changes releasable and can deploy them to non-production environments. Continuous deployment automatically promotes qualifying changes to production. Teams may automate compilation, transpilation, bundling, testing, packaging, signing, and deployment within these pipelines.

Webpack bundles modules, and Babel compiles modern JavaScript syntax for selected environments. WebAssembly is a portable binary instruction format and compilation target, not a build tool. Build utilities transform sources and coordinate tasks, while automation servers or hosted workflows trigger those utilities.

Packages combine distributable artefacts with metadata such as names, versions, and dependencies. Package managers locate, install, update, verify, and remove packages, subject to each ecosystem's design. Examples include npm for JavaScript, pip for Python, and RubyGems for Ruby. Maven and Gradle primarily provide Java ecosystem build and dependency-management capabilities. Lockfiles improve repeatability, while signature checks, provenance, vulnerability review, and controlled registries reduce supply-chain risk.
### Software stacks
A software stack combines technologies that support an application. A simple three-tier stack separates presentation, business logic, and data, while larger systems add gateways, caches, queues, identity services, observability, containers, orchestration, and cloud infrastructure. "Technology stack" often includes this broader set, although usage varies and no universal boundary separates the terms.

Common web stacks include LAMP with Linux, Apache, MySQL, and PHP, and JavaScript-centred stacks such as MEAN, MERN, and MEVN. These pair MongoDB, Express, and Node.js with Angular, React, or Vue. Other combinations use Django and Python, Ruby on Rails, or ASP.NET. Teams can replace components when interfaces and constraints permit.

No stack remains best for every workload. Teams should compare data models, consistency needs, scale, latency, security, hosting, portability, ecosystem maturity, operational skills, and long-term support. MongoDB can support large systems and structured documents, while relational databases can also store semi-structured data. Framework size, age, or language uniformity alone cannot establish performance, flexibility, or suitability.
## Basics of Programming
### Compilers, interpreters, and execution
Programming languages let developers express instructions in human-readable forms. Toolchains and runtimes ultimately cause processors to execute binary machine instructions.

"Compiled" and "interpreted" describe implementation strategies, not exclusive language classes. A compiler translates source before or during execution into native code, object files, bytecode, or another intermediate form. An interpreter executes source or an intermediate representation through a runtime. Linkers and packagers may produce an executable, several files, or an application bundle.

Modern implementations often combine strategies. CPython compiles Python to bytecode and executes it in a virtual machine. Java normally compiles to Java Virtual Machine bytecode, which a runtime can interpret or compile just in time. JavaScript engines use just-in-time compilation in browsers and servers. C and C++ commonly compile ahead of time to native code. C# is a distinct language that normally targets .NET.

Developers consider speed, memory use, deployment, portability, libraries, safety tools, and team expertise. Compilation can detect some errors early but cannot ensure correctness. Interpretation can shorten feedback cycles without confining a language to small scripts. Android combines a Linux-based kernel, native daemons and libraries, a runtime, framework code, system services, and applications rather than using Java alone.
### High-level, query, and assembly languages
High-level languages hide hardware details and provide abstractions for data, control flow, and program structure. Low-level languages expose a processor's instruction set and storage model. ARM, MIPS, and x86 name instruction-set architectures with associated assembly syntaxes.

An assembler translates assembly instructions for a target architecture. Source lines can contain a label, mnemonic, operands, and a comment. Directives guide the assembler without necessarily producing machine instructions. Pseudo-instructions and macros may expand into several instructions, so assembly source does not always map one-to-one to machine instructions.

Query languages retrieve or change data and administer databases. SQL supports `SELECT`, data manipulation with `INSERT`, `UPDATE`, and `DELETE`, schema definition with `CREATE` and `ALTER`, and access control. In CRUD terminology, creating a record usually maps to `INSERT`, not SQL's schema-level `CREATE`.

Relational databases organise data around relations and commonly use SQL. "NoSQL" covers several alternatives, including document, key-value, wide-column, and graph models. These systems may use flexible schemas, explicit schemas, or validation rules. Relational systems can also store and query JSON, so structured and unstructured data do not form a strict SQL-NoSQL boundary.
### Algorithms, pseudocode, and flowcharts
An algorithm defines an ordered method for solving a problem or completing a task. Pseudocode expresses that logic in concise, language-independent statements. It lets participants review decisions before developers choose a language. Pseudocode has no universal syntax or page limit.

A flowchart represents control flow with standardised symbols and connecting lines. Common symbols include a terminal for a start or end, a rectangle for a process, a diamond for a decision, and a parallelogram for data. Colour remains optional. Flowcharts clarify branching, while pseudocode often handles detailed algorithms compactly. Complexity and audience should determine the format.

These techniques support design and documentation, but they do not constitute the only forms of code organisation. Developers also use modules, functions, types, naming conventions, tests, and source directories to keep software readable, maintainable, and scalable.
### Control flow
Programs commonly combine sequence, selection, and iteration. A Boolean expression evaluates to `true` or `false`, although some languages also convert other values to Boolean conditions. A variable names or refers to a value that can change. It need not serve as a function parameter.

Selection chooses a path. An `if` runs its body when its condition holds, and an optional `else` provides an alternative. A `switch` or `match` selects among cases under language-specific rules. A `goto` transfers control directly, but structured constructs usually express intent more clearly.

Iteration repeats work. A `while` loop tests before its body, while a `do-while` loop tests afterwards and therefore runs its body at least once. A `for` loop may use an initialiser, condition, and update, or it may iterate over a range or collection. `break`, `return`, and exceptions can end iteration early. Language rules, not one universal three-loop model, determine the available constructs.
### Identifiers, values, and collections
An identifier names a program element such as a variable, constant, function, method, class, or interface. Constants and variables are named elements rather than types of identifier. A constant prevents or discourages reassignment according to the language, while a variable can receive another value during execution. Meaningful names improve readability and let developers update a shared value in one place. Declaration, initialisation, scope, mutability, and type rules vary by language.

Collections hold multiple values. Languages provide arrays, dynamic arrays, lists, maps, sets, tuples, and other structures. Many languages use zero-based array indexing, but neither zero-based indexing nor fixed size applies universally. In C++, `std::array` has a fixed extent and `std::vector` manages a contiguous, dynamically sized sequence. A vector may reserve spare capacity and reallocate as it grows, but indexed access remains constant-time.
### Functions, objects, and programming styles
A function packages callable behaviour for reuse, testing, and composition. It may accept arguments, return a value, or cause side effects, but none is required. Languages distinguish functions, methods, procedures, subroutines, and modules in different ways. C and C++ require a declaration before a call when the definition appears later.

Procedural programming organises computation around operations and control flow. Object-oriented programming groups related state and behaviour in objects. Classes or prototypes often define them. Fields or attributes store state, and methods provide behaviour. Encapsulation, composition, and inheritance can model a domain effectively, but excessive hierarchies and boilerplate can reduce flexibility. Many systems combine procedural, object-oriented, and functional techniques. The problem, ecosystem, and maintainability needs should guide the choice.
## Software Architecture, Design, and Patterns
### Architecture and design decisions
Software architecture describes a system's fundamental structures, interactions, operating environment, and guiding principles. It records significant decisions that shape implementation and often become costly to reverse. Architecture addresses functional responsibilities and quality attributes such as performance, scalability, availability, security, interoperability, maintainability, and operability.

Effective architecture balances stakeholder concerns, exposes trade-offs, and guides technology choices and production topology. Requirements should determine the stack. Architecture evolves during iterative development as teams test assumptions and learn from operation.

Useful artefacts include architecture descriptions, decision records, interface contracts, architectural and deployment diagrams, and selected UML models. A software design document may consolidate requirements, assumptions, constraints, dependencies, interfaces, and implementation guidance. Teams should maintain only artefacts that support decisions, communication, construction, or operation.
### Structure, behaviour, and UML
Structured design decomposes a problem into cohesive modules with clear responsibilities and explicit, limited dependencies. This combination reduces the effect of change.

Behavioural models represent states, events, activities, and interactions. UML provides a standard graphical language for modelling structure and behaviour independently of a programming language. Common diagrams include:
- A class diagram shows classes, attributes, operations, and relationships.
- A state machine diagram shows states, transitions, triggers, and effects.
- A sequence diagram, which is an interaction diagram, shows messages between participants over time.

Diagrams clarify scope and support onboarding, but they do not automatically save time or money. Obsolete diagrams mislead, so teams should align important models with implementation.

Object-oriented analysis and design models a domain through collaborating objects. A class defines a type. An object is an instance with state in attributes and behaviour in methods. Class diagrams show attributes, operations, and relationships. Inheritance can reuse behaviour but cannot guarantee safe parent substitution. Analysts can use object models independently of implementation language, although they align naturally with object-oriented languages.
### Components, services, and distributed systems
A software component encapsulates a capability behind defined interfaces. A sound contract can support reuse, replacement, extension, and testing. Components usually depend on other components, so independence means controlling dependencies rather than removing them. An API defines an interface or contract. It does not, by itself, constitute a component. A data access component can hide persistence details from the rest of an application.

A service exposes a capability through a network-accessible contract and may support independent deployment and scaling. A platform may run zero, one, or many service instances according to demand and availability goals. Service-oriented architecture organises systems around interoperable services, while microservices divide a domain into smaller, independently deployable services aligned with business capabilities. Neither style requires every service to contain components made from objects.

Distributed systems coordinate processes across networked computers. Distribution adds network latency, partial failures, concurrency, data consistency challenges, security boundaries, and operational complexity. Replication, redundancy, health checks, timeouts, retries with backoff, idempotency, and failover can improve resilience. Architects must design for these properties because distribution does not automatically provide fault tolerance, scalability, or low latency.
### Common architecture styles
| Style | Organisation | Principal considerations |
| --- | --- | --- |
| Two-tier client-server | Clients request data or functions from a server | Simple separation can become a scaling or coupling constraint |
| N-tier | Logical layers divide responsibilities, and physical tiers host them | Strict designs use adjacent tiers, while relaxed designs allow selected tier skipping |
| Peer-to-peer | Peers can both provide and consume resources | Decentralisation improves resource sharing but complicates coordination, trust, and consistency |
| Event-driven | Producers publish events, and consumers subscribe and react | Asynchronous coupling requires decisions about ordering, duplication, delivery, and recovery |
| Microservices | Independently deployable services own bounded capabilities | Independent scaling adds network, observability, deployment, and data consistency costs |

Patterns can coexist at different scopes. A peer-to-peer system may still use coordination services, and a three-tier application may use events or microservices. An API gateway routes client requests to back-end services and can centralise authentication, throttling, caching, and observability. Services may coordinate workflows through central orchestration or decentralised choreography.
### Environments and hosting models
An application environment combines code, runtimes, libraries, configuration, networks, compute, memory, and storage. Development supports active coding, test or quality assurance supports verification, staging approximates important production characteristics, and production serves real users. Strong environment parity reduces deployment surprises. Production must satisfy workload, security, reliability, scalability, privacy, and recovery requirements, but pre-production environments should reproduce the characteristics needed for valid testing.

Organisations can operate infrastructure on-premises, use cloud services, or combine them. On-premises hosting gives an organisation direct operational responsibility for facilities, hardware, software, maintenance, and controls. Cloud providers and customers divide responsibilities according to the service model.

The NIST cloud definition identifies private, community, public, and hybrid deployment models. A private cloud serves one organisation, a community cloud serves organisations with shared concerns, a public cloud serves multiple customers through a provider, and a hybrid cloud connects two or more distinct cloud infrastructures. No model guarantees lower cost, stronger security, or greater flexibility. Workload, architecture, controls, skills, regulation, utilisation, and pricing determine the outcome.
### Production infrastructure and release practices
Production systems may use firewalls, load balancers, web servers, application services, proxies, database servers, caches, queues, and replicas. A firewall enforces traffic policy between networks or hosts but cannot replace layered security. A load balancer distributes requests across available back ends, often using health information. It cannot prevent every overload or application failure.

A web server handles HTTP or HTTPS content and requests. An application service runs domain or business logic. A forward or reverse proxy can route requests, terminate encryption, cache responses, or enforce policy. A database server runs a database management system that stores, queries, protects, and manages data. Replication can support availability or read scaling, while tested backups and recovery procedures protect against corruption, deletion, and wider failures. Each system should include only the components its requirements justify.

Deployment planning should define expected load, regions, latency, data flows, retention, privacy, identity, authorisation, availability targets, service level objectives, logging, monitoring, and rollback. Network calls add delay and failure modes, so distributed designs require explicit budgets and controls. Automated tests support releases, but test-driven development remains one option rather than a universal requirement. A canary release sends limited production traffic to a candidate version, compares it with the current version, and expands or reverses the rollout according to measured results.