# Chapter 6: Social Media Analysis
## Purpose and scope
Social media intelligence applies structured OSINT methods to social media data that analysts can access lawfully and ethically. It examines accounts, posts, media, connections, interactions, metadata and recurring behaviour. It covers social networks, messaging platforms, forums, media-sharing services, gaming communities, blogs and microblogs.

Social media analysis often anchors subject intelligence because users disclose names, locations, relationships, work history, routines, opinions and images across many platforms. Useful platforms include Facebook, Instagram, LinkedIn, Reddit, YouTube, TikTok, Discord, Snapchat, Pinterest, VK, Telegram, WhatsApp, WeChat, QQ, Gab, MeWe, 4chan, 8kun and X, formerly Twitter. Platform names, privacy settings and data access change regularly, so analysts must verify current functionality before collection.

Common use cases include cybercrime, fraud, financial crime, child exploitation, trafficking, terrorism, organised crime, community monitoring, trend analysis, misinformation analysis and pattern-of-life assessment.
## Account elements that create pivots
A social media account contains several useful data categories.

- User data: names, usernames, dates of birth, profile photos, biographies, education, employment, hometowns, places lived, contact details, relationships and life events.
- Connections: friends, followers, followed accounts, family, colleagues, groups and memberships.
- Interactions: comments, likes, reactions, emojis, tags, follow-backs and repeated engagement patterns.
- Post content: text, hashtags, linked accounts, locations, personal details, opinions, stress indicators, habits and event references.
- Media content: images and video showing the subject, companions, vehicles, tattoos, clothing, hobbies, locations and other identifying features.
- Metadata: timestamps, platform-displayed device details, location labels and file metadata where platforms preserve it.

Each category creates pivot points. A profile photo can lead to another account. A tagged friend can lead to a relationship. A background landmark can lead to a location. A repeated commenter can identify a close associate. Analysts need organised notes and link charts because social media cases quickly accumulate accounts, aliases, screenshots, URLs and inferred relationships.

Analysts should also capture what was not found. Negative results, failed searches and excluded hypotheses prevent duplicated work and help reviewers understand why a conclusion rests on the available evidence.
## Collection and account correlation
Collection works best when analysts focus on method rather than tools. They should document the query, platform, account, URL, date, time, screenshot, observed data and inference. Evidence needs preservation because posts disappear, accounts become private and usernames change.

Many subjects use multiple accounts on the same platform. Some accounts represent businesses, hobbies or public identities. Others separate audiences, such as a family-facing account and a private account for close friends. Analysts correlate accounts through shared usernames, reused profile images, overlapping friends, repeated interactions, shared contact details, similar writing style, matching images, linked bios and common locations.

The strongest correlations rely on converging indicators. A shared username alone may identify an unrelated person. A matching profile image, overlapping friend group, repeated self-interactions and matching biographical details create a stronger assessment. Analysts should label associations by confidence and distinguish confirmed facts from reasonable inferences.

A repeatable collection workflow improves accuracy. Analysts should begin with known selectors, then search exact names, usernames, email addresses, phone numbers, domains, profile images and distinctive phrases. They should then pivot from each reliable result to linked profiles, archived pages, tagged people, group memberships, media, domain records and public records where permitted. Each step needs a reason, a timestamp and a confidence assessment.

Account ownership needs particular care. A username search may return parody accounts, fan accounts, reused handles or different people with the same alias. A profile image may be stolen. A location field may reflect aspiration, humour or deliberate deception. Analysts should avoid treating an account as belonging to a subject until several independent indicators support that conclusion.
## Associations, interactions and risk signals
Interactions often reveal relationships more clearly than profile fields. Frequent comments, mutual likes, repeated tags, private jokes, shared locations and sustained engagement suggest real associations. Analysts can map these links in an association matrix or link chart. Dense connections can highlight family members, close friends, leaders, rivals, accomplices or communities of interest.

Posts can disclose risk-relevant information. A marketplace post might expose a phone number, meeting point and valuable property. A travel post might show that a person is away from home. A workplace photo might reveal an employer, clock time, security context or exact building. A public group post about weapons, threats or self-harm indicators may require escalation when combined with distress, intent and capability indicators.

Responsible analysis avoids overclaiming. A single post rarely proves intent, location or relationship. Analysts assess content in context, corroborate with independent sources and record uncertainty.

Association analysis should also account for negative and weak ties. Arguments, blocks, hostile comments and one-off mentions can matter, but they do not carry the same weight as sustained mutual interaction. Analysts should record the type of interaction, the direction of the link and the date range. A person who liked a post five years ago may not be relevant to a current case.

Patterns of life emerge when locations, times, associates and activities repeat. A gym photo every weekday morning, a regular workplace tag, a school uniform, a recurring cafe background or a repeated travel route can reveal routine. These findings need sensitive handling because routine information can expose safety risks for subjects and bystanders.
## Media, metadata and obfuscation
Social media platforms often strip embedded EXIF metadata from uploaded images to protect privacy, but they may still display metadata such as post time, date, device source, location tag or application. Where original files are available, metadata can reveal camera settings, device details, timestamps, coordinates or editing software. Analysts should treat metadata as one lead among many because platforms alter files and users can manipulate data.

Users may also obfuscate identity and location. They can rotate accounts, share passwords, post through friends, recycle images, use VPNs, backdate content, crop photos, mirror images, remove EXIF data or create false biographies. Cooperative obfuscation can confuse algorithmic location signals and advertising profiles. Analysts need independent corroboration before relying on social media-derived location, identity or chronology.
## Continuous community monitoring
Continuous monitoring observes online communities over time to understand behaviour, leadership, rhetoric, mobilisation, threats and changes in activity. Communities may operate on Facebook groups, Reddit, Discord, Telegram, 4chan, 8kun, niche forums and alternative platforms. Some groups remain public. Others use invitation rules, challenge questions or private channels.

Monitoring requires clear authority, legal review and operational security. Stakeholders must approve sock-puppet accounts, private-group access and any interaction. Passive observation is usually safer and cleaner than engagement. Some communities require interaction to maintain access, which creates legal and ethical risk.

A monitoring plan should define the goal, scope, platforms, keywords, accounts, collection period, evidence-handling process, archiving method and reporting threshold. Collecting everything wastes time and increases privacy risk. Collection should match the intelligence requirement, such as identifying leaders, tracking planned activity, verifying claims or understanding a group's operating method.

Operational security underpins the plan. Analysts should separate work identities from personal accounts, avoid exposing personal phone numbers or emails, manage browser fingerprints, use approved virtual machines or devices where appropriate and avoid clicking unknown files or links without a safe workflow. Higher-risk communities may attempt doxxing, social engineering or malware delivery.

Evidence handling also matters. Screenshots need visible URLs, dates and relevant context. Important pages should be archived when lawful. Chat exports, downloaded media and captured files need controlled storage, chain-of-custody notes and access limits. Analysts should minimise collection of uninvolved people's personal information.

Telegram needs precise handling. Public channels can broadcast messages to subscribers and may be visible without an account. Private channels require access. Telegram Cloud Chats use client-server and server-client encryption. Only Secret Chats add end-to-end encryption, and channels are not the same as end-to-end encrypted one-to-one Secret Chats.

When access to a closed group is unavailable, analysts can still monitor public spillover. Members may repost links, screenshots, slogans, usernames, invitation links or claims on other platforms. Search engines, archives, cached pages and the Wayback Machine can preserve content that later disappears.
## Image and video analysis
Imagery analysis extracts information from photographs and video by examining context, foreground, background, map markings and anomalies. Analysts should slow down and describe what the image shows before searching. Useful clues include signs, shopfronts, road markings, billboards, licence plates, architecture, roof shapes, waterways, vegetation, terrain, shadows, vehicles, weather, clothing and reflections.

Reverse image search helps identify logos, buildings, stolen images, uncropped originals, reused profile photos and earlier appearances of a meme or video frame. It works unevenly across platforms and search engines, so analysts should test multiple crops and engines.

Geolocation identifies where an image or video was captured by matching visual clues with maps, street imagery, satellite imagery and other open sources. Good practice starts with visible clues, tests hypotheses and verifies the final location from several features. Mirrored images, old street-view captures, seasonal changes and copied media can mislead analysis. The correct location for the Plein Street case is Plein Street and the R403 in Vosburg, Northern Cape, South Africa, not northern Cape Town.

Real-time geolocation adds time pressure. During crises, analysts may need to verify attacks, fires, troop movements or damage quickly. The Centre for Information Resilience used satellite imagery, drone footage and open video to analyse Russian activity near Chernobyl's exclusion zone in 2022. Such work shows why analysts must combine location verification, chronology, safety context and cautious language.

Public trace-an-object programmes also show the value of visual analysis. Europol and Australia's ACCCE publish sanitised background objects from child sexual abuse material so the public can help identify locations or origins. Analysts should submit relevant tips through official channels and avoid sharing sensitive material publicly.
## Verification and information disorder
Verification assesses accuracy, credibility and reliability. Social media accelerates false claims because users repost content without checking it, influencers benefit from attention and coordinated campaigns can amplify narratives.

Information disorder has three useful categories.

- Misinformation: false or misleading information shared without intent to deceive.
- Disinformation: false or misleading information shared deliberately.
- Malinformation: genuine information used or framed to cause harm.

Verification starts with core questions. Who posted the claim? What exactly does it claim? Where did it first appear? Who benefits? What evidence supports it? What evidence contradicts it? Does the account show coordinated behaviour? Does the content use old media, cropped images, copied text, fabricated branding or emotional framing?

A sound verification workflow separates source verification from content verification. Source verification examines the account, posting history, affiliations, incentives and prior reliability. Content verification examines whether the image, video, text or claim matches the alleged time, place and event. A reliable source can share false content, and an unreliable source can occasionally post true material.

Chronology deserves separate attention. Analysts should compare upload time, claimed event time, time zone, daylight, weather, shadows, platform timestamps and any earlier appearances of the same media. Old content presented as new is one of the most common forms of misleading social media evidence.

Analysts should search for the earliest appearance of a claim, archive relevant pages, preserve URLs and screenshots, compare image and video frames, inspect account history and look for independent reporting. Many false narratives contain fragments of truth. The misleading element may be the date, place, caption, source, framing or omitted context rather than the entire item.
## Bots, networks and coordinated behaviour
Automated and semi-automated accounts can amplify narratives, harass users, hijack hashtags, inflate popularity and create the appearance of consensus. Bot detection depends on patterns, not labels. Indicators include high posting frequency, recent account creation, default or cartoon avatars, formulaic usernames, duplicated text, synchronised posting, repeated retweets or reposts and dense interaction with accounts showing the same behaviours.

Network analysis helps analysts understand scale and influence. Nodes represent accounts or entities. Edges represent relationships such as follows, mentions, reposts or shared links. Weight shows interaction strength. Degree, centrality and community clustering can highlight influential accounts, bridge accounts and coordinated clusters. Visualisations assist analysis, but underlying data quality determines the value of the graph.

Coordination is not always automation. Real people can coordinate through shared documents, private chats, influencer prompts or campaign instructions. Analysts should distinguish automation, coordination and ordinary popularity. Similar content across many accounts may reflect a copied press release, a trending joke, a grassroots movement or an organised influence operation.
## Manipulated media and synthetic content
Manipulated media ranges from simple cropping and false captions to edited photos, cloned image regions, fabricated screenshots, reused video-game footage and AI-generated deepfakes. Analysts should not treat a single forensic tool as proof. Error-level analysis and clone detection can flag regions that deserve further review, but they are subjective and can mislead. Strong assessments combine provenance, original-file acquisition, metadata, visual inspection, source history and corroborating evidence.

Deepfakes and other synthetic media are increasingly accessible and difficult to detect. Visual cues such as unnatural blinking, inconsistent lighting, poor teeth detail, warped glasses, facial-edge artefacts or mismatched skin tones can still help, but modern models often reduce those signs. Verification should prioritise source provenance, known filming context, independent confirmation, platform history, frame-level analysis and expert review for high-stakes claims.
## Integrated case analysis
The puppy scam case demonstrates integrated social media analysis. Victim reviews identified a business name, domain, phone numbers and alleged subject name. Website analysis found suspicious pricing, personal-data collection, reused images, poor translation and linked social accounts. Domain history tied multiple sites to common email addresses. Social media accounts added usernames, locations and related businesses. Reverse image search on testimonial videos indicated hired actors rather than real customers. Gap analysis then directed further collection toward usernames, phone numbers, associates, locations, identity confirmation and financial context.

Strong investigations move from selectors to corroborated relationships. Names, emails, domains, phone numbers, accounts, images, videos and locations become useful when analysts test links, record confidence, preserve evidence and identify gaps. The final product should separate facts, inferences and unanswered questions so decision-makers understand both the findings and the limits of the analysis.