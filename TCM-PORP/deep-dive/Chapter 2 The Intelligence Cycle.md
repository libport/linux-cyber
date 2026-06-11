# Chapter 2: The Intelligence Cycle
## Purpose and structure
The intelligence cycle gives analysts a repeatable way to turn requirements, collection and assessment into usable intelligence. A common public model moves through planning, collection, processing, analysis and dissemination. Lowenthal's expanded model treats consumption and feedback as explicit parts of the process because stakeholders do not always read, understand or act on a finished product without further engagement.

The cycle should not operate as a rigid checklist. It works best as a disciplined framework that keeps analysts aligned with stakeholder needs, records why collection occurs and forces reassessment when new questions appear.
## Planning and requirements
Planning starts with the intelligence need. Stakeholders should define the decision, the questions that matter, the intended audience, the reporting format and any limits on time, cost, sources or handling. Analysts then convert those requirements into a collection and reporting plan.

A sound plan identifies the analysts required, the sources to be used, the tools needed, the expected output, collaboration methods, storage arrangements, access controls, legal constraints and feedback path. The team should also identify the intelligence discipline involved, such as open-source intelligence, human intelligence, signals intelligence, imagery intelligence or measurement and signature intelligence. This helps analysts judge source value, product sensitivity and the difference between single-source and all-source assessment.

Poor planning creates wasted collection, unfocused analysis and reporting that does not answer the user's decision problem. Early stakeholder agreement reduces those risks. The plan should also define success. A fraud investigation might require attribution, a threat intelligence task might require indicators and mitigations, and a corporate investigation might require evidence suitable for legal review. Different outcomes require different collection depth, confidence language and documentation standards.
## Collection
Collection puts the plan into action. Analysts gather only the data needed to answer the agreed questions. Open-source intelligence collection may use search engines, public records, social media, maritime or geolocation services, breach data, application programming interfaces, crawlers and paid databases. Tools can accelerate collection, but they do not replace human judgement.

Legal and ethical limits shape collection. The European Union's General Data Protection Regulation is a regulation, not a set of regulations. It requires a lawful basis for personal data processing and imposes principles such as fairness, transparency, purpose limitation, data minimisation, accuracy, storage limitation, integrity, confidentiality and accountability. EU law enforcement processing is also governed by Directive (EU) 2016/680 and national law. Personal, household, journalistic, research and law enforcement contexts can be treated differently, so analysts should seek legal advice before collecting or storing personal data.

Open-source does not mean permission-free. In the United States, the Ninth Circuit held in hiQ Labs v. LinkedIn that scraping publicly available data was likely not access without authorisation under the Computer Fraud and Abuse Act. That ruling did not make all scraping lawful. Contract, privacy, copyright, anti-circumvention, platform access controls and local law can still create risk.

Data minimisation is essential. Analysts should collect, retain and report only what the requirement justifies. A travel question does not justify broad historical collection of unrelated breach data.
## Pivoting and analytical discipline
Pivoting follows one data point to another. An email address may lead to usernames, breach records, social media accounts, phone numbers, domains, wallet addresses or locations. Each new selector can become another starting point. Effective pivoting depends on curiosity, record keeping and constant relevance testing.

Analysts must balance detail and context. Close inspection can reveal hidden relationships, while wide review can show patterns that isolated facts obscure. The COBRA method supports this discipline: concentrate on what is camouflaged, inspect one thing at a time, take breaks, realign expectations and ask another person to review the material.

Large datasets create a second risk. Analysts can mistake coincidence for meaning. Apophenia describes the tendency to see meaningful patterns in random or weakly related data. Analysts reduce that risk by corroborating findings, recording uncertainty, testing alternative explanations and seeking peer review.
## Roadblocks and gap analysis
The absence of information can itself be significant. A missing record, unusually small online footprint or failed search may show concealment, poor indexing, privacy controls, jurisdictional limits or simply no relevant data. Analysts should report meaningful gaps rather than hide them.

The RESET technique helps analysts regain perspective during a stalled investigation. The analyst refreshes routines, notices emotional pressure, severs attention from the task for a short period, explores new tools or ideas and thinks beyond the current search path.

Gap analysis turns confusion into a structured problem. Analysts ask four questions:
1. What is already known?
2. What does it mean?
3. What remains unknown?
4. How can it be found out?

This method works for small tasks such as geolocating an image and for larger investigations. It separates evidence, interpretation, knowledge gaps and next actions.
## Data volume and automation
The scale of online data makes manual collection insufficient for many OSINT tasks. Industry forecasts from IDC and Seagate used zettabyte-scale growth to show why analysts increasingly rely on aggregators, application programming interfaces, crawlers and enrichment platforms. Those tools can surface leads, correlate selectors and reduce repetitive work. They can also amplify error, bias and overcollection if analysts accept tool output without verification. Automation should therefore support collection and triage, while analysts retain responsibility for scope, corroboration, interpretation and lawful handling.
## Documentation
Documentation records what analysts found, where they found it and how they reached each conclusion. Notes must be clear enough for the original analyst, collaborators, reviewers and legal users. Weak notes can force rework, damage credibility and prevent timely action.

A documentation plan should be agreed before collection begins. Analysts should record sources, URLs, dates, times, screenshots, captions, selectors, process steps and pivot points. Links should be defanged where needed. Notes should assume later sharing.

The best tool depends on the case. Word processors suit short reports. Spreadsheets organise selectors and structured data. Shared workspaces support collaboration. Browser-based investigation tools can capture pages, screenshots, URLs and notes while the analyst works. Mind maps help show pivots. Link charts help show relationships between people, accounts, infrastructure and events. The tool matters less than consistency, security and recoverability.
## Processing and evaluation
Collection produces raw data, not intelligence. Processing cleans, translates, decodes, normalises and organises that data. CyberChef can help with encoding, decoding, hashing, conversion and cryptographic operations, but it does not decrypt material without the necessary method and key.

Evaluation tests reliability and credibility. The Admiralty Code rates source reliability separately from information credibility. A source may be reliable while a particular claim remains unconfirmed, and a new source may provide information that cannot yet be judged. Analysts should grade evidence carefully, preserve uncertainty and avoid treating a tool match as proof.

Scoping may occur at any stage. If stakeholder questions change, analysts should refocus the investigation and restart affected parts of the cycle. Data enrichment can then fill gaps by combining open-source data with approved first-party or third-party sources. Enrichment should strengthen correlations without exceeding the requirement or breaching legal limits.
## Analysis, production and reporting
Analysis explains what the processed data means and why it matters. Analysts should answer the original intelligence questions, identify confidence and uncertainty, and produce a product suitable for the reader. Intelligence products may include quick alerts, technical notes, investigative reports, executive summaries, presentations or continuing assessments.

Effective reporting is timely, clear, tailored and easy to act on. It should use bottom-line-up-front structure when readers need fast decisions. The executive summary should state the main finding, confidence level, impact and recommended action. The body should explain the evidence, method and reasoning. The summary should restate the key findings. Recommendations should give practical next steps. Appendices should hold large tables, images and supporting material.

Reports should use precise, active language and avoid unsupported absolutes. A report should say that an email address is highly likely to belong to a subject when the evidence supports probability rather than certainty. Visualisations should clarify the analysis. Tables suit selectors, graphs suit measurable comparisons, mind maps suit internal exploration and link charts suit relationship analysis. Design choices should serve comprehension, with clear headings, consistent captions, readable tables and source details that allow reviewers to retrace the work.
## Dissemination, consumption and feedback
Dissemination does not end the cycle. Stakeholders must receive, understand and use the intelligence. Critical findings may require a tipper or alert before the final report. A tipper should give the bottom line, confidence, evidence summary and immediate action.

Feedback tests whether the product answered the requirement. It may create new questions, require further collection or show that the format did not suit the audience. Analysts and stakeholders should maintain communication from planning through post-report review.

The intelligence cycle has limits. Hulnick argues that requirements and collection often work in parallel rather than in a neat sequence, and that analysts should not wait passively for perfect stakeholder guidance. Security barriers, institutional habits and confirmation bias can also separate collection from analysis or cause stakeholders to reject inconvenient findings. OSINT teams should treat the cycle as a flexible model, not a guarantee of sound judgement.