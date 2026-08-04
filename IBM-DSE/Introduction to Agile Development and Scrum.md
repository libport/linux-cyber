# Introduction to Agile Development and Scrum
> [!NOTE]
> This guide explains how Agile and Scrum help cross-functional teams deliver value amid uncertainty through rapid feedback, adaptive planning, disciplined engineering, transparent workflows, and continuous improvement.
## Introduction to Agile and Scrum
### Agile philosophy
Agile software development uses short learning cycles to deliver value, gather evidence, and adapt. Teams plan enough work to pursue a useful outcome, create a working increment, test their assumptions with stakeholders and users, and revise the product or plan. Iteration alone does not make work agile. Frequent feedback, valuable delivery, technical quality, and responsiveness to change distinguish Agile from repeated execution of a fixed plan.

The Agile Manifesto values:
- Individuals and interactions over processes and tools
- Working software over comprehensive documentation
- Customer collaboration over contract negotiation
- Responding to change over following a plan

The items on the right retain value, but the items on the left receive greater emphasis. Agile teams still use tools, documentation, contracts, and plans when these help people create and operate the product. They avoid treating those aids as substitutes for collaboration, usable software, customer learning, or adaptation.

Agile practices favour early and continuous delivery, close cooperation, sustainable work, technical excellence, simplicity, and regular reflection. Teams make work and decisions transparent, but Agile does not require co-location or prescribe one team structure. Effective teams usually combine the skills needed to turn an idea into a safe, usable outcome.

Adaptive planning operates at several horizons. A product may retain a long-term vision and broad roadmap while the team commits less detail to distant work. Evidence from each delivery can change priorities, forecasts, and solution design. This approach manages uncertainty without abandoning direction or accountability.
### Sequential development, Extreme Programming, and Kanban
A traditional Waterfall model moves through requirements, design, implementation, integration, testing, deployment, and operation in largely sequential stages. Clear phase boundaries can assist governance when requirements and technology remain stable. However, late integration and delivery delay evidence about feasibility and user needs. Discovering an incorrect assumption late can increase rework, lead time, and cost. Waterfall does not make change impossible, but a rigid, single-pass implementation makes change difficult. Winston Royce's influential 1970 paper called that simple model risky and recommended feedback, prototyping, testing, and iteration.

Sequential delivery also creates hand-offs between specialist groups. Each hand-off can add queues, obscure context, and separate design decisions from operational consequences. Cross-functional collaboration, early integration, and incremental validation reduce those risks even when an organisation retains formal stages for compliance or governance.

Kent Beck and others developed Extreme Programming in the 1990s to combine rapid feedback with disciplined engineering. Its five values are communication, simplicity, feedback, courage, and respect. Practices such as pair programming, test-first development, continuous integration, simple design, collective ownership, and frequent releases reinforce one another. XP helped shape the wider Agile movement, but Agile did not arise from XP alone.

Kanban optimises the flow of value through a system. A Kanban system defines and visualises its workflow, actively manages work items, and improves the workflow. The Kanban Method expands this foundation through six general practices:
- Visualise work, workflow, and risks
- Limit work in progress to available capacity
- Manage flow by identifying and removing delays
- Make process policies explicit
- Establish feedback loops
- Improve collaboratively through experiments

Limiting work in progress encourages teams to finish work before starting more. Common flow measures include work in progress, throughput, work-item age, and cycle time. A board can support these practices, but a board alone does not constitute Kanban.
### Product learning and engineering practices
Small batches shorten the interval between action and evidence. They can expose defects, integration problems, and mistaken assumptions before a team invests heavily. Teams should choose batch sizes according to risk, transaction cost, dependencies, and the speed at which useful feedback becomes available. Single-piece flow illustrates the principle, but it is not a universal requirement.

A minimum viable product is the least-effort version of a new product that lets a team maximise validated learning about customers. It is not necessarily the first incomplete slice of a planned solution, and it may test demand before a team builds automated functionality. A team states a hypothesis, selects an ethical and useful experiment, defines evidence in advance, observes results, and then perseveres, changes direction, or stops. Learning has little value unless it informs a decision.

Behaviour-driven development aligns developers, testers, product specialists, and other stakeholders through conversation, concrete examples, and shared descriptions of behaviour. Gherkin scenarios often express an initial context with Given, an action with When, and an observable outcome with Then. BDD can guide acceptance at several system levels. It is not confined to user-interface or integration testing.

Test-driven development uses a short Red-Green-Refactor cycle. A developer writes a test for the next behaviour, sees it fail for the expected reason, writes enough production code to pass, and improves the design while keeping the tests passing. TDD often operates at unit level, but the technique is not limited to unit tests. It supports design and regression safety, while BDD strengthens shared understanding of valuable behaviour.

Pair programming places two practitioners at one workstation or shared session. A driver works directly with the code while a navigator reviews, anticipates risks, and considers the wider design. They exchange roles frequently. Pairing can support continuous review, knowledge sharing, and collective ownership, although teams should evaluate its effects in context rather than assume a guaranteed productivity gain.
### Scrum framework
Scrum is a lightweight framework for generating value through adaptive solutions to complex problems. It uses empiricism - transparency, inspection, and adaptation - and the values of commitment, focus, openness, respect, and courage. Scrum does not prescribe a complete engineering method, so teams often combine it with XP, Kanban, DevOps, and other practices.

A Sprint is a fixed-length event of one month or less that contains all other Scrum events. Each Sprint pursues a Sprint Goal and produces at least one usable Increment that meets the Definition of Done. A team may release an Increment before the Sprint ends. A two-week Sprint is common, but Scrum does not require that duration.

Scrum defines three accountabilities within one cohesive, self-managing, cross-functional Scrum Team:

| Accountability | Primary responsibility |
| --- | --- |
| Product Owner | Maximises product value, communicates the Product Goal, and manages the ordered, transparent Product Backlog |
| Developers | Create the Sprint plan, build a usable Increment, maintain quality, adapt the plan, and hold one another accountable |
| Scrum Master | Establishes Scrum, improves team effectiveness, coaches self-management, helps remove impediments, and supports productive events and stakeholder collaboration |

The Scrum Team usually has 10 or fewer people. Current Scrum guidance prescribes neither a development-team sub-group, a seven-plus-or-minus-two size, nor co-location. The Product Owner does not simply accept or reject work at a Sprint Review, and the Scrum Master does not direct the team.

Each Scrum artefact has a commitment that strengthens focus and transparency:

| Artefact | Content | Commitment |
| --- | --- | --- |
| Product Backlog | An emergent, ordered list of product improvements | Product Goal |
| Sprint Backlog | The Sprint Goal, selected Product Backlog items, and the delivery plan | Sprint Goal |
| Increment | Integrated, usable work that meets the Definition of Done | Definition of Done |

Product Backlog refinement is an ongoing activity, not one of Scrum's formal events. The Product Owner remains accountable for backlog management, while Developers remain responsible for sizing and contribute technical insight and delivery knowledge. Developers own and update the Sprint Backlog throughout the Sprint.

Scrum organises work through five events:
- The Sprint contains the work and the other events.
- Sprint Planning establishes why the Sprint is valuable, what it can deliver, and how Developers will perform the work.
- The 15-minute Daily Scrum lets Developers inspect progress towards the Sprint Goal and adapt their plan. Scrum requires no three-question script.
- The Sprint Review lets the Scrum Team and stakeholders inspect the outcome, discuss changed conditions, and adapt future work. It is not a release gate.
- The Sprint Retrospective plans improvements to quality and effectiveness.

Developers may adjust scope with the Product Owner as they learn, provided changes do not endanger the Sprint Goal or reduce quality. Scrum and Kanban differ in structure, but teams can use them together:

| Dimension | Scrum | Kanban |
| --- | --- | --- |
| Cadence | Fixed-length Sprints of one month or less | Continuous flow with no required iteration |
| People | Product Owner, Developers, and Scrum Master accountabilities | No mandatory roles or titles |
| Change | Protects the Sprint Goal while allowing scope to evolve | Pulls work and changes workflow through explicit policies |
| Measures | Prescribes no productivity measure or velocity target | Requires work in progress, throughput, work-item age, and cycle time |
| Delivery | Creates a usable Increment each Sprint and can release at any time | Delivers according to workflow, capacity, and service policies |

Scrum does not prescribe a Kanban board, end-of-Sprint releases, or fixed story commitments. Applying Kanban practices within Scrum can help a team see queues, control work in progress, and improve flow without replacing Scrum's accountabilities, artefacts, or events.

Well-used Scrum creates regular opportunities to expose risk, integrate work, inspect outcomes with stakeholders, and improve the team's process. These mechanisms can shorten time to value, strengthen quality, and improve collaboration. Scrum does not guarantee productivity, satisfaction, or commercial success. Results also depend on sound product choices, capable engineering, organisational support, and evidence-based adaptation.
### Teams, organisations, and delivery
Conway's Law observes that organisations tend to produce system designs resembling their communication structures. Organisations can respond by forming long-lived, cross-functional teams around products or business capabilities, reducing dependencies, and giving teams end-to-end responsibility for building, operating, and improving services. Autonomy works best within a clear mission, technical standards, safety constraints, and shared organisational goals.

Distributed teams can succeed when they protect overlap time, provide equitable access to decisions, document essential context, and invest in communication. Co-location may help some teams, but Scrum and Agile do not require it. Stable membership often preserves knowledge and ownership, although organisations may still need specialists to work across team boundaries.

DevOps extends collaboration across development, delivery, operations, security, and business functions. Automation, continuous integration, continuous delivery, observability, and shared operational ownership reduce hand-offs and shorten feedback loops. Every organisational unit need not use Scrum or the same Agile practices. The end-to-end delivery system must still support frequent, safe change, or downstream queues will erase the speed gained during development.

Agile does not forbid project managers, distributed teams, documentation, specialist expertise, or long-term direction. It does reject command-and-control allocation within a self-managing team, large speculative plans treated as fixed commitments, delayed integration, and iterations that produce no usable evidence. An organisation practising Water-Scrum-Fall retains extensive sequential planning and release controls around short development cycles. Sprints in the middle cannot compensate for slow approval, integration, deployment, or learning across the wider system.
## Agile Planning
### Adaptive planning and credible forecasts
Software product work contains uncertainty in user needs, technology, dependencies, and operating conditions. Detailed plans created at the start can therefore become obsolete as teams learn. Upfront planning does not automatically cause missed deadlines, and iterative planning does not guarantee accurate estimates. Excessive early detail creates false confidence when evidence remains scarce.

Rolling-wave planning keeps long-term direction while limiting detailed commitments to the near term. A team can maintain a Product Goal, roadmap, budget, and broad delivery forecast while planning the next increment in greater detail. After each delivery, the team reviews outcomes, updates assumptions, reorders work, and revises forecasts. Near-term plans usually rest on stronger evidence than distant plans, but teams should still express uncertainty through ranges, probabilities, assumptions, and confidence levels rather than unsupported precision.

Frequent delivery strengthens planning because working results reveal technical constraints and stakeholder responses. Teams should compare forecasts with actual outcomes, improve their forecasting method, and avoid treating any estimate as a promise. Leaders can ask what evidence supports a forecast, what could change it, and when the team will update it.

Some deadlines remain fixed because of regulation, market events, contracts, or safety obligations. Adaptive planning does not remove those constraints. Teams can protect a date by ordering essential outcomes, delivering the highest-value slices early, exposing dependencies, testing critical risks, and negotiating scope as evidence changes. Leaders should make trade-offs explicit rather than demand an unchanged scope, date, cost, and quality under changing conditions.
### Scrum accountabilities and organisational change
Changing job titles does not change behaviour, skills, authority, or incentives. Organisations should train people for new accountabilities, clarify decision rights, and redesign governance that conflicts with self-management. A product manager or project manager can serve within Scrum, but the person must understand the accountability and have enough capacity to fulfil it. Previous experience does not automatically qualify or disqualify anyone.

Scrum defines three accountabilities within one Scrum Team:

| Accountability | Core responsibility |
| --- | --- |
| Product Owner | Maximises product value, communicates the Product Goal, orders the Product Backlog, and ensures that it remains transparent and understood |
| Developers | Create every aspect of a usable Increment, plan the Sprint, maintain quality, adapt their plan, and hold one another accountable |
| Scrum Master | Establishes Scrum, improves team effectiveness, coaches self-management, supports the Product Owner, and helps the organisation remove impediments |

The Product Owner does more than manage a budget or relay stakeholder requests. The Product Owner integrates stakeholder interests, evidence, strategy, and delivery constraints into product decisions. A product manager may also hold this accountability when the organisation grants suitable authority and focus.

The Scrum Master does not manage tasks or assign work. The Scrum Master helps the team and organisation understand Scrum, develop effective practices, and remove barriers. This accountability does not require the Scrum Master to solve every impediment personally or shield the team from all contact. A former project manager can succeed after replacing command-and-control habits with coaching, facilitation, and service to the team.

Developers include everyone committed to creating an Increment, whatever their professional title. A software Scrum Team may need engineering, testing, security, operations, design, data, and domain skills. Scrum creates no internal development-team sub-group or hierarchy. Leaders support the team by aligning funding, policies, performance measures, and access to expertise with product delivery. They need not adopt one framework everywhere, but they should remove organisational controls that undermine frequent learning and self-management.
### Visual workflow and Zenhub
A work board makes invisible work visible. Cards represent work items, and columns represent meaningful workflow states. A simple board may show options, ready work, work in progress, review, and finished work. The team should define entry and exit policies for each state, identify when work starts and finishes, and limit work in progress according to capacity. Pulling new work only when capacity becomes available reduces overload and exposes queues.

A board alone does not create a Kanban system. Kanban combines three practices:
- Define and visualise the workflow
- Actively manage work items in the workflow
- Improve the workflow

Kanban also requires teams to track work in progress, throughput, work-item age, and cycle time. These measures reveal flow conditions more reliably than card counts in broad columns. Work commonly moves left to right on digital boards, but Kanban does not require that layout.

Zenhub provides a web application and a browser extension that integrate planning with GitHub. Its Work Tracker can display GitHub Issues and Zenhub Issues as cards in configurable pipelines. The current default pipelines include New Issues, Icebox, Product Backlog, Sprint Backlog, In Progress, Review/QA, Done, and Closed. Teams can change these names, add policies and work-in-progress limits, and automate updates to assignees, labels, sprints, and epics.

Tool defaults do not define the team's process. An Icebox can separate low-priority options from active planning, while New Issues can support intake triage. Review/QA should still count as work in progress. Done should represent the team's agreed completion policy, not one developer's finished coding. In Scrum, work contributes to an Increment only when it meets the Definition of Done. The Sprint Review inspects the outcome with stakeholders, but the Product Owner does not use that event as a final acceptance gate.

Integrating a board with GitHub can reduce duplicate updates, but no tool guarantees a single accurate record. Teams should name the authoritative fields, automate synchronisation where useful, and review stale or conflicting data.
### Product Backlog items and user stories
The Product Backlog is an emergent, ordered list of what the product needs for improvement and the single source of work for the Scrum Team. It is not a permanent inventory of every idea, every unimplemented requirement, or only user stories. It can contain features, experiments, defects, risks, research, technical improvements, and other work that advances the Product Goal. The Product Owner orders items by considering value, risk, learning, dependencies, cost, and urgency. Higher items usually need more detail than distant options.

A user story offers one optional way to describe a Product Backlog item from a user's perspective. It should prompt collaboration rather than replace it with a miniature specification. Ron Jeffries' three elements capture this purpose:
- Card - a concise reminder of the need
- Conversation - discussion that develops shared understanding
- Confirmation - concrete examples or checks that demonstrate the intended outcome

A third-person template can state: `A [user or stakeholder] needs [capability] to achieve [benefit].` Teams may use other formats when they communicate the need more clearly. The story should describe the outcome and relevant constraints without forcing an implementation choice unless a genuine architectural, legal, security, or operational constraint requires one.

The INVEST checklist helps teams assess stories:

| Letter | Quality | Practical meaning |
| --- | --- | --- |
| I | Independent | The team can order and deliver the story with minimal coupling where feasible |
| N | Negotiable | The story invites discussion instead of fixing every detail too early |
| V | Valuable | The outcome benefits a user, stakeholder, product, or delivery system |
| E | Estimable | The team understands enough to assess its size when estimation adds value |
| S | Small | The team can complete the item within a useful planning horizon, often one Sprint |
| T | Testable | Observable evidence can confirm the intended outcome |

Acceptance criteria describe item-specific conditions and examples. The Definition of Done instead establishes the quality state that every Increment must satisfy. Teams should not use these terms interchangeably. Gherkin can express acceptance examples in a shared Given-When-Then structure:

```text
Given a counter contains the value 2
When the service restarts
Then the counter still returns 2
```

This example confirms persistence behaviour, but the Definition of Done may also require testing, integration, security checks, documentation, and deployability across the Increment.

An epic is a planning convention for a large body of work, not an official Scrum artefact or a universally fixed size. Teams can divide an epic into thin, valuable slices that produce evidence or capability. Splitting by user outcome, workflow step, business rule, data variation, or operational scenario usually preserves value better than dividing work into isolated technical layers.
### Relative estimation and flow-based forecasting
Scrum does not require user stories, story points, planning poker, Fibonacci numbers, t-shirt sizes, or velocity. Teams may use these techniques when they improve decisions.

Story points express a team's relative assessment of work. A team may consider effort, complexity, uncertainty, and risk, then compare an item with agreed reference items. A Fibonacci-like scale can discourage false precision, but no sequence holds special authority. Group discussion can expose different assumptions, and an unusually large estimate often signals a need for more discovery or a smaller item.

Teams should not:
- Convert points into fixed hours or days
- Compare points or velocity across teams
- Reward higher point totals
- Inflate estimates to satisfy a target
- Treat a Sprint forecast as guaranteed scope

Historical velocity commonly means the points associated with items completed to the Definition of Done during a Sprint. It can provide one local planning signal, but it does not measure productivity, value, quality, or team maturity. A rising velocity may reflect changed estimation rather than improved delivery. Teams should never expect velocity to increase continually.

Sprint forecasting should consider capacity, absences, dependencies, recent outcomes, item clarity, risk, and the Definition of Done. Teams can also forecast with throughput and cycle-time data, especially when work items have reasonably consistent sizing. Probabilistic forecasts and ranges communicate uncertainty more honestly than a single date.
### Product Backlog refinement and technical work
Product Backlog refinement breaks down and clarifies items, adds useful details, and revises their order and size. Scrum treats refinement as an ongoing activity, not a required meeting. The Scrum Team decides when and how to refine. The Product Owner remains accountable for effective backlog management, but Developers, stakeholders, specialists, and the Scrum Master contribute as needed. Developers who will perform the work remain responsible for sizing.

Refinement should create enough shared understanding for Developers to select and complete an item within a Sprint. It need not eliminate every question before Sprint Planning or freeze the solution. Scrum does not require a separate Definition of Ready, an empty intake column, two Sprints of refined work, or weekly refinement. Those practices can help in some contexts, but teams should test them against their actual needs.

Intake triage can classify a new item for near-term ordering, later consideration, further discovery, or rejection. Product Owners should remove obsolete options rather than preserve an ever-growing backlog. Refinement meetings, when useful, should include the people needed for the decisions at hand. Excluding most Developers to save meeting time can create later confusion, while requiring everyone at every session can waste capacity.

One practical product workflow follows a short sequence:
1. Capture an option, problem, or opportunity without treating it as ready work.
2. Test its relevance to the Product Goal and decide whether to order, explore, defer, or discard it.
3. Discuss the need with users, stakeholders, and the people who may deliver it.
4. Split large ideas into small, coherent outcomes that can generate evidence.
5. Add useful constraints, acceptance examples, dependencies, and risks.
6. Size the item only when an estimate supports a decision.
7. Select work during Sprint Planning against the Sprint Goal, capacity, and quality obligations.
8. Build, integrate, verify, deliver, and use the outcome as evidence for the next decision.

Technical debt does not include all work that customers cannot see. It describes internal design or implementation deficiencies that make future change harder or more costly, much like financial debt incurs interest. A shortcut may create debt intentionally, or a team may discover debt later. Refactoring can reduce it.

Routine environment work, cloud deployment, library upgrades, vulnerability remediation, architecture changes, and operational maintenance are not automatically technical debt. They may represent enablement, risk reduction, compliance, resilience, or direct product value. Teams should classify work by its purpose, record the consequences of delay, and order it alongside other Product Backlog items. Labels can improve visibility, but arbitrary quotas for debt work can displace higher-risk or higher-value decisions.
### Sprint Planning and the Sprint Backlog
Sprint Planning starts the Sprint through the collaborative work of the entire Scrum Team. The team may invite other people to provide technical or domain advice. A Sprint lasts one month or less. Two weeks is a common choice, not a rule.

Sprint Planning addresses three topics:
1. Why is the Sprint valuable? The Product Owner proposes how the product could increase in value, and the Scrum Team collaborates to define a Sprint Goal before planning ends.
2. What can the Sprint deliver? Developers select Product Backlog items through discussion with the Product Owner and consider their past performance, current capacity, and the Definition of Done.
3. How will the work get done? Developers create an actionable delivery plan and decide how to turn the selected items into a usable Increment.

The resulting Sprint Backlog contains the Sprint Goal, selected Product Backlog items, and the delivery plan. The Sprint Goal creates one objective and allows flexibility in the exact work. Developers own and update the Sprint Backlog as they learn. If work differs from expectations, they collaborate with the Product Owner to adjust scope without endangering the Sprint Goal or reducing quality.

Velocity may inform selection when a team finds it useful, but the team should not stop mechanically at a point total. Nor should Developers commit to a fixed list of stories. They forecast work and commit to the Sprint Goal. A milestone, sprint field, label, or board pipeline can record the plan in Zenhub, but Scrum requires none of those tool features.

Successful planning produces a coherent goal, a realistic forecast, and enough initial structure for Developers to begin. It does not attempt to answer every future question. The team continues planning throughout the Sprint while evidence replaces assumptions.
## Daily Execution
## Scrum execution and continuous improvement
Scrum helps a small, cross-functional, self-managing team create value through repeated inspection and adaptation. A Scrum Team consists of one Product Owner, one Scrum Master, and Developers, typically totalling no more than 10 people. The Product Owner orders the Product Backlog, the Developers create and adapt the Sprint plan, and the Scrum Master establishes Scrum and supports the team's effectiveness.

A Sprint has a fixed duration of one month or less. Two weeks is common, but Scrum does not prescribe it. Every Sprint has a single Sprint Goal and aims to produce at least one usable Increment that meets the Definition of Done. A new Sprint begins immediately after the previous one ends.

Scrum leaves teams free to add compatible engineering practices and workflow tools. Kanban boards, story points, velocity, pull requests, and software-specific labels can support delivery, but none belongs to Scrum's required framework. Teams assess these practices by whether they improve value, flow, quality, and learning.
### Executing work and managing flow
During Sprint Planning, the whole Scrum Team collaborates on the Sprint Goal. The Developers select Product Backlog items after discussing priorities and trade-offs with the Product Owner, then decide how to deliver them. The Sprint Goal, selected items, and delivery plan form the Sprint Backlog, which the Developers update as they learn.

A digital or physical board can make this work visible. When a team uses one, it keeps item status, ownership, reviews, and impediments current. Developers decide who undertakes each task and may use a pull policy that favours the highest-value available work. Neither individual self-assignment nor taking the top item is a Scrum rule.

Limiting work in progress can reduce context switching and expose bottlenecks. A developer generally finishes or helps finish current work before starting more. When an item is blocked, the team first seeks to remove the impediment or collaborate elsewhere. Starting another item remains a deliberate flow decision, not an automatic response. Pull requests, automated board transitions, and separate review columns can support delivery, but an item becomes Done only when it satisfies the team's Definition of Done.
### Daily Scrum
The Daily Scrum is a 15-minute event for the Developers on every working day of the Sprint, normally at the same time and place. The Developers inspect progress towards the Sprint Goal, adapt the Sprint Backlog, and produce an actionable plan for the next working day.

The Developers choose the format. Questions about completed work, planned work, and impediments can help, but Scrum does not require those three questions. Participants do not need to stand. The event is not a status report to management. A Product Owner or Scrum Master who actively works on Sprint Backlog items participates as a Developer. Detailed problem-solving can continue afterwards with the relevant people.

The team identifies impediments promptly. The Scrum Master causes impediments to be removed, but does not need to attend solely to collect or personally resolve every blocker.
### Monitoring progress
A burndown chart plots estimated work remaining against time. Its downward trend can help a team inspect whether its current pace supports the Sprint Goal or another delivery target. It does not measure completed and remaining work as two separate quantities, guarantee completion, or calculate a reliable probability of success.

Burndowns are optional forecasting aids rather than Scrum artefacts. Scope changes and late completion can distort their shape, so teams interpret them with the Sprint Goal, the current Increment, and other evidence. Burn-up charts and cumulative-flow diagrams can reveal different aspects of progress and flow.
### Sprint Review
The Sprint Review is a working session for the Scrum Team and key stakeholders. Participants inspect the Sprint outcome, discuss progress towards the Product Goal, consider changes in the operating environment, and collaborate on future adaptations. Demonstrations can support the discussion, but the event is broader than a presentation or approval meeting. Feedback may lead the Product Owner to reorder, clarify, add, or remove Product Backlog items.

The Product Owner does not accept or reject work as a separate Scrum gate. Work that meets the Definition of Done forms part of an Increment and may be released before the review. Work that does not meet the Definition of Done cannot form part of the Increment or be presented as completed. It returns to the Product Backlog for future consideration.
### Sprint Retrospective
The Sprint Retrospective concludes the Sprint and includes the entire Scrum Team, including the Product Owner. The team examines people, interactions, processes, tools, quality, and the Definition of Done to identify ways to improve quality and effectiveness. It may discuss what went well, what failed, and what to change, but Scrum does not prescribe those prompts.

Psychological safety supports candid discussion. The team selects the most useful improvements, addresses the highest-impact changes promptly, and may add them to the next Sprint Backlog. Excluding the Product Owner is not a standard Scrum practice.
### Measuring delivery performance
Useful metrics connect observed results to decisions. A team establishes a baseline, chooses an improvement goal, changes its system, and checks the trend. Counts such as website hits become vanity metrics when they lack context or do not guide action. Product outcomes, customer feedback, quality, and operational health remain important alongside delivery measures.

DORA's current model uses five software delivery performance metrics:
- Change lead time measures the time from committing a change to deploying it in production.
- Deployment frequency measures deployments over time or the interval between them.
- Failed deployment recovery time measures recovery from a deployment that requires immediate intervention.
- Change fail rate measures the proportion of deployments that require immediate intervention, such as a rollback or hotfix.
- Deployment rework rate measures the proportion of unplanned deployments made in response to production incidents.

The first three describe throughput, while the last two describe instability. Teams track them for one application or service over time, balance speed with stability, and avoid comparing dissimilar systems. Turning a metric into an arbitrary target can encourage gaming, so measurement serves learning and improvement rather than competition.
### Ending a Sprint and handling unfinished work
The Sprint ends when its timebox expires. Closing board columns, milestones, or project records may support a team's tools, but Scrum does not require a separate administrative close-out procedure.

An unfinished Product Backlog item does not earn partial completion credit. It returns to the Product Backlog, where the Product Owner may reorder it and the Developers may refine or resize the remaining work. The team does not label incomplete work as Done, reduce its estimate to manufacture completed points, or create a replacement item solely to protect velocity. Untouched items also return to current Product Backlog ordering rather than moving automatically into the next Sprint.

Velocity, when used, is a planning aid based on completed work and local team history. It is not a performance score. Sprint Planning selects work afresh according to the Product Goal, current priorities, capacity, past performance, and the Definition of Done.
### Scrum team health
Common warning signs include unclear Product Owner accountability, competing Product Owners, oversized teams, functional silos, weak self-management, chronic multitasking, and work that repeatedly misses the Definition of Done. Geographic distribution alone is not a Scrum anti-pattern. Distributed teams can succeed when they maintain effective collaboration, shared working hours where needed, transparent work, and clear goals.

Healthy Scrum Teams are cross-functional, empowered, and focused on one Product Goal. They maintain an ordered and transparent Product Backlog, a visible Sprint Backlog, a clear Sprint Goal, and a sustainable pace. They adapt daily, create a usable Increment each Sprint, invite meaningful stakeholder feedback, and turn retrospective insights into observable improvements. These practices reinforce transparency, inspection, adaptation, commitment, focus, openness, respect, and courage.