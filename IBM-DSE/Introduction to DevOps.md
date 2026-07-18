# Introduction to DevOps
> [!NOTE]
> These notes explain how DevOps unites culture, collaboration, automation, measurement and shared ownership to help cross-functional teams deliver reliable software rapidly through small changes, continuous feedback, and learning.
## Overview of DevOps
### Why DevOps developed
Digital disruption rewards organisations that can test ideas, learn from customers and improve services quickly without sacrificing reliability. Technology enables that work, but a strong business model and an effective operating system direct it. Ride-sharing, streaming and mobile services succeeded by combining existing technologies with new ways to create and deliver value.

Traditional software delivery often separated analysts, architects, developers, testers and operations staff into functional silos. A Waterfall process moved a large body of work through sequential phases, with extensive handovers and limited feedback. Teams integrated and tested the system late, when defects and mistaken assumptions cost more to correct. Operations staff then received unfamiliar software and carried responsibility for running it. Long queues, delayed releases and conflicting incentives weakened the entire delivery system.

DevOps developed in response to these constraints. It joins software development and technology operations across the service lifecycle. Teams apply Lean and Agile principles to shorten feedback loops, improve flow and deliver small changes safely and frequently. DevOps therefore combines culture, working methods and enabling technology. Tools support the approach, but tools alone cannot create it.
### From Agile development to shared delivery
Extreme Programming helped establish short feedback loops, iterative planning, automated testing, pair programming and responsiveness to changing requirements. In 2001, the Agile Manifesto placed greater value on individuals and interactions, working software, customer collaboration and response to change than on rigid processes, exhaustive documentation, contract negotiation and fixed plans.

Agile improved development, but many organisations left operations unchanged. Developers could produce increments every few weeks while infrastructure requests, approvals and deployments remained slow. This imbalance created queues and encouraged shadow IT, where staff acquired external services without organisational oversight.

DevOps extends fast feedback and shared responsibility into operations. Development, operations, security, testing and product specialists collaborate from design through production. They work towards common service outcomes instead of optimising separate departments. A team labelled "DevOps" may help enable change, but renaming a team or hiring automation specialists does not transform the organisation.
### Origins and influence
Patrick Debois began exploring closer cooperation between development and operations after confronting delivery problems in 2007. At the 2008 Agile conference, Debois and Andrew Clay Shafer discussed Agile infrastructure. John Allspaw and Paul Hammond then demonstrated the practical potential of frequent deployment in their 2009 presentation about cooperation between development and operations at Flickr.

Debois organised the first DevOpsDays conference in Ghent in 2009. The event helped popularise the shortened term "DevOps" and connected a practitioner-led community. Important books later consolidated its practices. Jez Humble and David Farley's Continuous Delivery described automated build, test and deployment pipelines in 2010. Gene Kim, Kevin Behr and George Spafford used a business novel to explain flow and constraint management in The Phoenix Project in 2013. The DevOps Handbook translated these ideas into practical guidance in 2016.

DevOps grew through practitioners rather than a single standard, product or job title. Its decentralised origins explain why definitions vary, but the central purpose remains consistent: improve the safety, speed and sustainability of software delivery through collaboration and continuous learning.

Early examples showed that this approach could scale beyond start-ups. Flickr reported more than ten deployments a day, while Etsy later described hundreds of production deployments in a month. Large enterprises also adopted DevOps practices to shorten lead times and recovery times. These results did not come from deploying entire monoliths repeatedly. Teams reduced batch sizes, automated delivery and changed approval processes so that routine, low-risk changes could move without long queues.
### Culture and organisation
A healthy DevOps culture builds trust, openness, psychological safety, accountability and respect. Leaders create the conditions for change, while teams adapt daily work from the ground up. Staff share code, knowledge and operational responsibility. They treat failures as opportunities to improve the system, not as reasons to blame individuals.

Organisational design shapes software architecture and communication. Cross-functional teams reduce handovers and own services throughout their lifecycle. They keep work in small batches, expose problems early and use fast feedback to guide decisions. Test-driven development, behaviour-driven development and peer collaboration can strengthen quality when teams apply them appropriately.

Measurements also shape behaviour. Local targets can push development to maximise change while operations tries to prevent it. Shared measures align both groups around delivery and service performance. Useful indicators include deployment frequency, lead time for changes, change failure rate and time to restore service. Customer and product outcomes remain essential because delivery speed has little value when changes do not improve the service.
### Technical capabilities
Software architecture has not followed one universal sequence from Waterfall monoliths to Agile service-oriented systems and then to DevOps microservices. Delivery methods, architectural styles and infrastructure models overlap. A monolith can support continuous delivery when teams design it for automated testing and safe deployment. Likewise, a microservices system can perform poorly when tight coupling prevents independent change. Deployability and testability are more important than labels.

Continuous integration combines small code changes frequently and checks them with automated tests. Continuous delivery keeps software in a deployable state. Deployment automation makes releases repeatable and reduces manual error, although organisations may still choose when to release. Infrastructure as code applies versioned, tested definitions to environments and supports rapid, consistent provisioning.

Loosely coupled architecture allows teams to test and deploy components independently. Microservices can support this goal, but they are not mandatory and do not guarantee resilience. They also introduce distributed-system complexity, including network failures, data consistency, observability and deployment coordination. Organisations should select an architecture that fits the product and enables testability and deployability rather than treating microservices as a universal destination.

Containers package applications and dependencies into portable runtime units with fast startup. Teams often replace failed or outdated container instances from known images instead of repairing them in place. Orchestration and automated platforms can provision, deploy, monitor and replace many such instances. Containers remain optional implementation tools, not a defining requirement of DevOps.

Cloud services can supply programmable, on-demand infrastructure, but cloud adoption alone does not produce effective delivery. The strongest systems combine automation, version control, testing, observability, security and recovery practices. They limit the impact of failures through small changes, staged releases, feature controls and rapid rollback or roll-forward procedures.

Immutable infrastructure strengthens consistency by replacing deployed instances from versioned definitions instead of modifying them manually. This pattern reduces configuration drift and makes recovery more repeatable. It does not remove the need to manage persistent data, secrets, dependencies and platform security. Teams should automate these concerns and retain clear audit trails.
### Agility with stability
Frequent delivery does not require reckless change. Small, independent releases reduce the scope of each failure and make diagnosis easier. Teams can test a feature with a limited audience, compare outcomes and expand or withdraw it according to evidence. This approach replaces large, high-risk releases with controlled experiments and rapid learning.

High-performing organisations improve throughput and stability together. They design systems for testing and independent deployment, automate routine work and provide fast production feedback. They also maintain discipline through security, compliance, change management and reliability engineering that fit the speed and risk of each change.

DevOps has no universal toolchain or organisational template. Each organisation must adapt its practices to its products, regulations, architecture and customers. Successful adoption changes how people think, work, organise and measure. Culture provides the foundation, methods guide the work and technology makes reliable flow possible.
## Thinking DevOps
### Social coding and collaborative workflow
Social coding applies open-source collaboration inside an organisation. Often called inner source, it makes internal code discoverable and invites contributions beyond the owning team while retaining appropriate access controls. This visibility reduces duplicated work. A team that finds an existing component can discuss its needs with the owner, record the proposed change in an issue, develop it on a branch, add tests, and submit a pull request. The owner maintains standards and decides whether to merge the contribution.

Pair programming strengthens this collaboration. One developer acts as the driver and writes the code while the navigator reviews the emerging solution, considers the wider design, and anticipates the next step. The pair changes roles regularly and may work at one workstation or through remote collaboration tools. Pairing can expose defects earlier, spread knowledge, improve design decisions, and reduce reliance on one developer. Its value depends on the task, the participants, and effective role rotation.

Repository structure should follow product boundaries, coupling, release needs, tooling, and team ownership. Neither a monorepo nor one repository per service suits every system. Teams should document their choice and automate builds, tests, and ownership controls accordingly.

GitHub flow provides a lightweight branch-based workflow:
1. Create a short-lived branch from the default branch for a feature, fix, or other focused change.
2. Commit and push small changes, then open a pull request early enough to gather useful feedback.
3. Run automated checks and request review under the team's risk-based approval rules.
4. Resolve feedback, merge the approved change, and delete the branch.

Independent review often improves quality and shared ownership, but a universal ban on self-merging does not suit every team or risk level. Protected branches and required reviews can enforce the chosen policy.
### Small batches and minimum viable products
Small batches shorten feedback loops, reduce work in progress, and limit the cost of a wrong assumption. Teams can release a useful increment, observe its effect, and adjust before investing heavily. Features that span several iterations often need further decomposition, although the appropriate batch size depends on the product and deployment context.

A simple envelope exercise shows the timing advantage. If folding, inserting, sealing, and stamping each take six seconds, processing batches of 50 takes five minutes per stage. The first completed envelope appears after about 15 minutes, and the whole batch finishes after 20 minutes. Single-piece flow completes the first envelope in 24 seconds, so quality feedback arrives far earlier. Software delivery follows the same principle when teams integrate, test, and release small changes.

A minimum viable product, or MVP, is the smallest experiment that can test an important value hypothesis with real users or credible evidence. It is not automatically the first project phase, a low-quality release, or the cheapest product. Each MVP should generate evidence that helps the team persevere, change direction, or stop. Useful increments support learning, whereas disconnected components may delay feedback until the final assembly.
### Test-driven and behaviour-driven development
Test-driven development, or TDD, uses a short Red, Green, Refactor cycle:
1. Write a focused test that fails for the expected reason.
2. Write the simplest production code that makes the test pass.
3. Improve the design while keeping the tests green.
4. Repeat for the next behaviour.

Writing the test first makes the intended interface and behaviour explicit from the caller's perspective. A fast regression suite supports confident changes, refactoring, dependency upgrades, and continuous integration. TDD does not guarantee good design or defect-free software. Outcomes depend on relevant tests, clear assertions, maintainable code, and complementary forms of verification. Reliable continuous delivery needs fast automated checks, but those checks extend beyond unit tests and may include integration, security, performance, and acceptance testing.

Behaviour-driven development, or BDD, helps business specialists, testers, developers, and other stakeholders build a shared understanding through concrete examples. It complements TDD rather than approaching development from an opposing direction. Teams commonly use BDD for business-facing behaviour and TDD for focused implementation feedback, but the practices can overlap.

Gherkin expresses scenarios with `Given`, `When`, and `Then`:
- `Given` establishes relevant context.
- `When` describes an action or event.
- `Then` states an observable outcome tied to value.
- `And` or `But` adds steps where they improve readability.

Feature files can serve as readable, executable specifications. Tools such as Cucumber do not execute plain-language scenarios unaided. Developers must connect each step to automation code called a step definition. Clear scenarios and maintained step definitions can provide living documentation, precise acceptance criteria, and faster feedback about whether the system exhibits the agreed behaviour.
### Cloud-native architecture and microservices
Cloud-native design uses automation, elasticity, observability, and replaceable infrastructure to exploit cloud platforms. It does not require microservices. A well-structured monolith may offer simpler development and operations, especially for a new or modest system.

Microservice architecture divides an application into services organised around business capabilities. Each service should be independently deployable and communicate through explicit contracts, including HTTP APIs, remote procedure calls, or asynchronous messages. Independent data ownership can reduce coupling, but services still require coordination when contracts, workflows, or shared business rules change.

A stateless service instance does not retain durable session or business state in local memory between requests. The system stores necessary state in databases, object stores, caches, or other external services. Stateless instances are easier to replace and scale horizontally, subject to cost, quotas, data constraints, and downstream capacity. Teams can scale a busy capability without scaling the entire application, but they also inherit distributed-system complexity, including network failures, data consistency, tracing, deployment coordination, and operational overhead. Replacing an unhealthy instance restores capacity. Engineers must still investigate causes, patch defects, and prevent recurrence.
### Resilience and controlled failure
Distributed systems experience partial failures, latency, throttling, and unavailable dependencies. Developers and operators should design for rapid detection, graceful degradation, and recovery. Current DORA guidance tracks failed deployment recovery time as one software delivery performance measure.

Resilience patterns address different failure modes:
- Timeouts prevent calls from waiting indefinitely.
- Retries handle transient faults only when another attempt is safe. They should use limits, exponential backoff, jitter, and server guidance such as `Retry-After` to avoid retry storms.
- Circuit breakers stop calls to a persistently failing dependency, then probe for recovery before restoring normal traffic.
- Bulkheads partition resources such as connection pools, worker pools, queues, or service instances so one failure cannot exhaust the whole system.
- Caching, fallbacks, rate limits, and load shedding can preserve reduced but useful service under pressure.

Chaos engineering tests resilience through controlled experiments, not random destruction without safeguards. A team defines a steady state and a failure hypothesis, limits the blast radius, observes the system, and prepares a stop or rollback mechanism. Experiments may terminate instances, inject latency, or restrict dependencies. The results expose weaknesses in detection, recovery, and architecture before an uncontrolled incident does.
## Working DevOps
DevOps brings software development, operations, security, and other delivery functions together around a shared goal: delivering reliable value to users. It extends Agile's emphasis on people, working software, collaboration, and responsiveness across the entire service lifecycle. Teams support this culture through shared ownership, rapid feedback, extensive automation, and small, frequent changes.

Frequent releases reduce the size of each change, which usually makes faults easier to isolate and recovery easier to manage. Automation makes repeated builds, tests, deployments, and infrastructure changes more consistent. Neither speed nor automation defines DevOps alone. Effective delivery also requires sound engineering, observability, security controls, and cooperation.

Teams should replace avoidable ticket queues and manual fulfilment with safe self-service. They should also replace blame-driven alarms and escalations with actionable telemetry and clear incident responsibilities. Manual intervention can remain necessary for exceptional or high-consequence decisions. The goal is to automate repeatable work, expose fast feedback, and let people concentrate on judgement and improvement.
### From divided labour to product ownership
Frederick Winslow Taylor's scientific management, published in 1911, separated management's planning from workers' execution and sought standard methods for repeatable industrial tasks. Taylor did not devise the assembly line or establish every later form of functional hierarchy. However, applying a rigid planning-and-execution divide to software can create long handoffs, lost context, queues, and weak ownership.

Software development involves discovery as well as construction. Requirements, dependencies, threats, and user expectations continue to change after release. A completed building offers an imperfect analogy because a live software service requires continuing development and operation. Stable, cross-functional teams should therefore own products or services throughout their lifecycles. Product ownership preserves technical knowledge, strengthens accountability, and gives teams direct feedback from production.

Traditional organisations often judge development by change and operations by stability, encouraging conflict. DevOps aligns both groups around service outcomes. Shared measures should cover delivery throughput, failed changes, recovery, availability, and user value. Change approval remains appropriate in regulated or high-risk settings, but organisations can make routine, low-risk changes through automated, evidence-based controls instead of slow, uniform processes.
### Infrastructure as code and replaceable environments
Infrastructure as code defines and manages computing resources through machine-readable, version-controlled configuration. Tools can then provision, change, and reproduce networks, hosts, cloud services, identities, and software settings. Reviews, automated tests, execution plans, and audit histories make infrastructure changes safer than undocumented manual alterations.

Version control identifies the approved configuration and records who changed it. Teams can review proposed changes, test them in representative environments, and apply the same process repeatedly. They should keep credentials and other secrets out of source files, validate configuration before deployment, and protect destructive operations with suitable approvals.

Replaceable infrastructure reduces configuration drift. Teams can create environments when needed, rebuild unhealthy instances from a known definition, and remove unused resources. This approach suits many cloud workloads, but not every component should be ephemeral. Persistent data, specialised hardware, cost, recovery objectives, and regulatory controls can require different treatment.

Container images support repeatable application packaging. A Dockerfile describes how to build an image, and teams should update that definition, rebuild the image, test it, and replace running containers instead of patching them interactively. Containers package many application dependencies, but they share a host kernel and do not guarantee identical behaviour across every platform. Image provenance, vulnerability management, secrets, data migrations, runtime configuration, and rollback design remain essential.
### Continuous integration and delivery
Continuous integration requires developers to merge small changes into a shared codebase at least daily. An automated build and test suite checks each integration and returns fast feedback. Long-lived branches allow codebases to diverge and increase integration effort. Short-lived branches or trunk-based development keep the mainline healthy, while code review spreads knowledge and detects defects. A pull request is useful, but it is not a requirement for continuous integration.

Continuous delivery keeps software releasable throughout its lifecycle. A deployment pipeline moves a versioned change through progressively stronger checks, which may include compilation, unit and integration tests, policy checks, vulnerability scans, artefact creation, and deployment to representative environments. The same tested artefact should progress between environments. Continuous delivery permits an authorised person to release on demand. Continuous deployment goes further by automatically releasing every qualifying change to production.

A typical pipeline connects a source repository, automated build workers, test and policy stages, an artefact repository, and deployment automation. Each stage consumes a controlled output from the preceding stage and increases confidence in the release candidate. Fast checks should run early, while slower or more specialised checks can run later. Failed checks must stop promotion and return useful evidence to the team.

Five principles guide continuous delivery:
- Build quality into the pipeline.
- Work in small batches.
- Automate repetitive work and leave judgement to people.
- Measure outcomes and improve continuously.
- Share responsibility for delivery and operation.

Deployment and activation need not occur together. Feature flags can hide or expose functionality without another deployment, although unmanaged flags add complexity. Canary releases expose a new version to a small group before wider rollout. Blue-green deployment shifts traffic between separate environments. These techniques reduce risk only when teams combine them with monitoring, tested recovery procedures, and controls for databases and other stateful components. No technique guarantees zero downtime or instant rollback.
### Knight Capital
Knight Capital's trading incident on 1 August 2012 demonstrates the danger of inconsistent deployment and inadequate safeguards. Knight deployed new Retail Liquidity Program code to seven of eight SMARS servers. The eighth retained obsolete Power Peg code, which responded to a repurposed flag and sent millions of erroneous orders. In about 45 minutes, the system produced more than 4 million executions in 154 stocks for over 397 million shares. Knight lost more than US$460 million, not US$640 million. The loss caused significant net capital problems and severely damaged the business, but the firm did not become bankrupt the next day.

Automation could have prevented the inconsistent deployment, but deployment automation alone would not have addressed every failure. Knight also lacked adequate pre-deployment testing, post-deployment verification, automated rollback, effective monitoring, and risk controls capable of stopping abnormal order flow. Reliable DevOps practice combines repeatability with independent safeguards and rapid feedback.
## Organising for DevOps
DevOps combines culture, practices, and tools to help organisations deliver reliable software quickly. Its purpose is to shorten feedback loops and improve both delivery speed and operational stability. Development, operations, testing, security, and business specialists collaborate across the software lifecycle, share goals, and use automation and feedback to improve delivery. DevOps is not an operations function or a job title alone. Agile methods complement DevOps, but an organisation does not need to adopt Scrum before it can practise DevOps.
### Align teams and architecture
Conway's law links system design to organisational communication. Teams divided into front-end, application, and database functions tend to produce matching technical boundaries and repeated handoffs. Organisations can reduce these delays by assigning long-lived, cross-functional teams to products, services, or business domains. Each team should have the skills and authority to design, build, test, deploy, operate, and improve its service.

Team design should follow context, not a fixed formula. Scrum describes teams of 10 or fewer people as typical, but this is Scrum guidance rather than a universal DevOps rule. Teams should remain small enough to communicate effectively and large enough to own meaningful outcomes. Domain alignment can support microservices, but it neither requires microservices nor removes every dependency between teams.
### Share responsibility for outcomes
Functional silos can separate people from the operational effects of their decisions. Shared ownership restores rapid feedback and encourages empathy. Teams should monitor production, respond to incidents through sustainable on-call arrangements, learn without blame, and improve the services they operate. Leaders should communicate common goals while giving teams local authority over implementation.

Quality also belongs to the whole delivery team. A separate quality assurance group does not automatically reduce quality, and removing specialists does not automatically improve it. Quality assurance specialists can work within or alongside cross-functional teams, while developers remain accountable for testable code. Continuous automated testing, exploratory testing, monitoring, and fast feedback should operate throughout delivery.

A standalone DevOps team that accepts work through handoffs risks creating another silo. Specialist platform or enablement teams can still help by providing self-service capabilities, coaching, and shared standards without taking product ownership from delivery teams.
## Measuring DevOps
### Measure outcomes, not activity
Measurement shapes behaviour. Steven Kerr's analysis of rewarding one outcome while expecting another explains why teams optimise for the incentives they receive. Counting lines of code encourages verbosity, while individual rankings can discourage colleagues from sharing knowledge. Useful social measures instead examine whether others reuse a developer's code and whether the developer reuses existing code. Together, these measures reward contribution, collaboration, and efficient reuse.

Improvement begins with a baseline, a specific goal, and repeated measurement. A team might reduce a ten-hour deployment to two hours, lower staffing requirements, or prevent more defects from reaching production. It should test changes, compare results with the baseline, retain effective practices, and then address the next priority.

Vanity metrics show activity without guiding a decision. Website hits, for example, reveal little about unique visitors, behaviour, or value. Actionable metrics connect an intervention with an outcome. An A/B test that exposes half of customers to a feature and records higher revenue per customer can support a rollout decision, provided the experiment is sound.

DORA now tracks five software delivery performance metrics:
- Change lead time
- Deployment frequency
- Failed deployment recovery time
- Change fail rate
- Deployment rework rate

These measures balance throughput with instability. Teams should apply them to a particular service and use trends to guide improvement, not turn arbitrary targets into incentives that invite gaming.
### Measure team culture
Teams can assess culture through the six-item Westrum survey, scored from 1 for strong disagreement to 7 for strong agreement. The statements test whether teams seek information, protect messengers who report bad news, share responsibilities, reward cross-functional collaboration, treat failures as opportunities to improve systems, and welcome new ideas. Teams may average the responses into one culture score. A blameless approach supports honest reporting and learning while still requiring accountability for improving systems and practices.
### Combine DevOps and SRE
DevOps promotes collaboration across the software lifecycle. SRE applies software engineering to operations through automation, service-level objectives, error budgets, monitoring, and blameless post-incident learning. SRE does not require organisations to hire only software engineers or preserve rigid silos. Google employs both software engineers and specialists with strong systems or networking expertise, and other organisations can adapt the structure to their scale.

Google caps operational work for SRE teams at 50 per cent so engineers retain time to reduce toil and improve services. An error budget equals one minus the service-level objective. A 99.9 per cent monthly availability target therefore allows about 43 minutes of unavailability in a 30-day month, not 44 seconds. A documented policy determines what happens when a team exhausts the budget. It may pause routine releases while allowing security and urgent reliability changes.

DevOps and SRE share the goals of fast, safe delivery, visible operational work, automation, collaboration, and learning from failure. SRE can provide specialised reliability practices or platforms while product teams retain responsibility for their services. Organisational design may vary, but development and reliability specialists must share evidence, objectives, and responsibility for production outcomes.