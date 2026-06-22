# Chapter 4: Search Engine Techniques
Search engines give investigators a practical way to navigate the public web, but they do not show the whole internet. They index material that crawlers can discover, access and process. They usually miss private databases, paywalled material, pages blocked by robots rules, content behind forms, ephemeral social media content, and much of the dark web. Search results also change with location, language, device, account state, ranking systems and time.

The original 2017 search figures should be read as historical. Netcraft reported about 1.8 billion sites in January 2017, not 1.8 billion billion sites. Google remains the dominant search engine, with about 90 per cent of global search traffic in early 2026, followed by Bing, Yahoo, Yandex, DuckDuckGo and Baidu. Search market share affects OSINT because it shapes visibility. It does not prove that one engine has the most useful answer for every investigation.
## Search strategy
Effective searching begins with disciplined keyword selection. Investigators should identify the language that a source is likely to use, not only the language used by the investigator. Medical, legal, technical, corporate and government sources often use formal terms. Social media, forums and leaked references often use slang, abbreviations, usernames, product names, misspellings and local terminology.

Keyword research expands a query beyond a single phrase. It should include:
- Synonyms and near synonyms
- Acronyms, aliases and former names
- Local spellings and translations
- Industry terms and regulatory terms
- File names, domain names, usernames and email domains
- Date ranges, locations and named events

Searchers should test the same concept across more than one search engine. Google, Bing, DuckDuckGo, Yandex, Baidu and regional engines rank and index differently. The same query can expose different pages, documents, images and snippets. Privacy-focused engines reduce profiling risks, but they do not make a search fully anonymous. Browser history, device telemetry, DNS, workplace monitoring, account logins and network records can still reveal activity.

Investigators should keep a search log. The log should record the query, search engine, date, time, location or VPN exit country, browser state, account state, filters and result pages checked. A search log makes later review possible and helps explain why one investigator saw a result that another person did not.
## Core operators
Advanced operators reduce noise when they are used carefully. Search engines change operator behaviour over time, so investigators should treat operators as aids rather than guarantees.

Reliable Google operators include:
- `"exact phrase"` to search for a phrase
- `-term` to exclude a term
- `site:example.com` to search a site or domain
- `filetype:pdf` to target a file type
- `before:2024-01-01` and `after:2023-01-01` to restrict by indexed or detected date
- `OR` to search for alternatives

Google Advanced Search also supports filters for language, region, last update, site or domain, where terms appear, file type and usage rights. It is useful for precise queries without manually typing every operator.

Some older operators and shortcuts no longer deserve reliance. The tilde synonym operator is obsolete. Google's visible cached page feature was retired, so investigators should use web archives and their own evidence capture instead of expecting `cache:` to work. The `related:` and `info:` style shortcuts are less dependable than modern related searches, entity panels and manual pivoting.

Bing supports its own advanced keywords, including `filetype:`, `ext:`, `contains:`, `site:`, `inbody:`, `intitle:` and `language:`. Bing syntax overlaps with Google syntax, but it is not identical. Queries should be tested directly rather than copied without validation.
## Google dorking and exposure discovery
Google dorking combines keywords and operators to find specific classes of indexed material. It can locate public files, exposed directories, configuration pages, error messages, login portals, device banners and documents with sensitive terms. The Google Hacking Database remains a useful source of examples, but it must be used lawfully.

Search operators do not authorise access. An investigator may record that a sensitive file is publicly indexed, but opening, downloading, copying, attempting login, bypassing controls or using exposed credentials can cross legal and ethical boundaries. The safest practice is to preserve minimal evidence, avoid interacting with systems beyond ordinary public access, and report exposure through an approved channel.
## Search engines and privacy
Google gives broad coverage and strong ranking, but personalisation and account-linked activity can distort what an investigator sees. Bing provides a useful second view, especially for Microsoft-indexed material, maps, news and file searches. DuckDuckGo, Startpage, Qwant, Swisscows, Brave Search and Mojeek are useful when an investigator wants less profiling or an independent index.

Startpage offers Google-derived results through a privacy-protective layer. DuckDuckGo states that it does not save or share search or browsing history. Those claims help reduce exposure to the search provider, but they do not remove the need for operational security, compartmentalised browsers, clean accounts, VPN or Tor where lawful and appropriate, and careful evidence handling.
## Business and organisational searches
Corporate OSINT draws on annual reports, securities filings, company registers, procurement portals, litigation records, sanctions lists, insolvency notices, patents, trademarks, job advertisements, archived web pages and social media.

Useful sources include official registers and disclosure systems. The US Securities and Exchange Commission's EDGAR system publishes filings for US reporting companies. The UK Companies House service allows free search and download of company information. Canada's legacy SEDAR system has moved to SEDAR+. OpenCorporates, Crunchbase, Kompass and other business directories can add context, but official registries should carry more weight than commercial profiles.

Business searches should connect entities across names, addresses, officers, shareholders, subsidiaries, domains, email patterns, phone numbers and filing histories. Investigators should verify current status because businesses change names, dissolve, merge, re-register and use nominee or service addresses.

Source quality matters as much as discovery. Primary records, official registers and original publications should outrank copied summaries. Commercial databases, social media posts and scraped profiles can provide leads, but they need confirmation from independent evidence.
## Metasearch, code search, FTP and IoT search
Metasearch engines query multiple sources and combine results. They can broaden discovery and reduce the bias of any single index. Their ranking depends on partner engines, so investigators should still inspect source results and rerun important queries on the original engines.

Code search matters when investigations involve exposed secrets, software provenance, malware analysis, reused code, leaked API keys or vulnerable snippets. GitHub code search supports Boolean operators and regular expressions across public code. Searches for credentials or private keys must be handled carefully, with rapid responsible disclosure and no unauthorised use.

FTP search is less central than it was. FTP is old, insecure by design because it can transmit data in clear text, and major browsers removed built-in FTP support. Dedicated FTP clients and specialist indexes may still find public FTP material, but investigators should treat open FTP servers as potentially misconfigured systems and avoid unnecessary downloading.

IoT search engines such as Shodan index internet-connected devices by collecting publicly visible service banners and metadata. They help defenders measure exposure across routers, cameras, industrial systems, databases and remote access services. They should not be used to access devices, test passwords or manipulate systems without authorisation.
## Directories, translation, archives and monitoring
Web directories once provided curated lists of websites. They now play a smaller role, but niche directories, professional association lists, government portals and industry catalogues remain useful for discovery.

Translation tools help investigators search across languages. Good practice includes searching in the local language, checking transliterations, and validating machine translations before relying on fine details. Proper nouns, idioms, legal terms and technical phrases often need human review.

Web archives are essential because web pages change, disappear and get edited. The Internet Archive's Wayback Machine can show earlier versions of public pages, although it does not capture every page and may fail on dynamic or restricted content. Google now offers archive links in some result panels rather than its old cache feature. Investigators should capture their own evidence early, including URL, access time, screenshots, downloads, hashes and notes about context.

Monitoring tools support ongoing OSINT. Google Alerts can email new search results for a query. RSS feeds publish structured updates from sites that support them. Page change monitors can alert investigators when a web page changes. Monitoring should record when a change was observed and preserve before-and-after evidence.
## News and fact checking
News searches require source evaluation. A credible report should be traced to its original source, checked against independent reporting, and compared with primary evidence where possible. Social media posts should not be treated as reliable merely because they are widely shared.

Fact checking should use lateral reading. Investigators should leave the page, search for the source, inspect ownership and expertise, compare claims across independent sources, and look for corrections or archived versions. The International Fact-Checking Network provides standards for professional fact checkers. Google Fact Check Explorer, Snopes, Reuters Fact Check, AFP Fact Check, FactCheck.org and other reputable services can help identify previously assessed claims, but no fact-checking database is complete.
## Searching for digital files
Documents, spreadsheets, presentations, images, videos and archives often contain more investigative value than web pages. Search operators such as `filetype:pdf`, `filetype:xlsx`, `filetype:docx`, `site:drive.google.com`, `site:docs.google.com`, `site:s3.amazonaws.com` and `site:onedrive.live.com` can locate publicly indexed files. Searchers should avoid opening or downloading files that appear private, sensitive, malicious or unlawfully exposed unless they have authority and a safe environment.

Common document formats include PDF, DOCX, XLSX, PPTX, TXT, HTML and OpenDocument formats such as ODT, ODS and ODP. File extensions can mislead, so investigators should check file signatures, metadata and hashes. PRONOM, DROID, ExifTool and file signature tables help identify formats and metadata.

Google Programmable Search Engine can build a focused search over selected sites. This is useful for repeat investigations, sector monitoring and controlled source lists. A custom engine should be documented so others can reproduce its scope and limits.

Grey literature includes research reports, theses, dissertations, preprints, standards, government papers, conference papers, policy submissions, working papers and technical reports. Search sources include Google Scholar, OpenAlex, institutional repositories, government portals, standards bodies and university libraries. Grey literature is valuable because it often contains details that never reach commercial publishing.
## Data leaks and breach information
Leaked data can contain personal, financial, corporate and security information. Its existence may matter to an investigation, but possession and use can create legal, ethical and safety risks. Investigators should avoid redistributing leaked personal data, should minimise collection, and should use lawful breach-checking services when possible.

Have I Been Pwned allows authorised breach checks for email addresses and domains and stores breach involvement metadata separately from its pwned password corpus. In Australia, eligible data breaches involving personal information may require notification to affected individuals and the Office of the Australian Information Commissioner. OSINT work should support harm reduction, not amplify exposure.
## Image and video search
Image search starts with keywords, captions, filenames and page context. Reverse image tools such as Google Lens and TinEye can find visually similar images, earlier appearances, altered copies and possible source pages. Investigators should compare the earliest known appearance, the highest quality copy, surrounding text, publication dates and the account or site that first posted it.

Image verification should combine metadata, reverse search, visual inspection and external evidence. Metadata can show device, time, software and location, but it can be stripped or forged. Visual clues such as weather, shadows, signage, road markings, architecture, vegetation and licence plates support geolocation and chronolocation.

Video search should examine titles, descriptions, thumbnails, transcripts, key frames and comments. Tools such as the InVID-WeVerify plugin help extract key frames and run verification checks. Automated manipulation detectors can support triage, but human review remains essential because false positives and false negatives are common.
## Evidence management and local search
OSINT produces many files. Investigators need a repeatable system for storage, naming, hashing, indexing and notes. Screenshots should include the source URL and time. Downloads should be hashed. Important pages should be saved as PDF or web archive files where lawful. Notes should separate observation from inference.

Local search tools such as Everything, Recoll, DocFetcher, dtSearch and Open Semantic Search help search large collections. Indexing should cover filenames, extracted text, metadata and tags. Sensitive collections should be encrypted, access controlled and backed up.

Search engine work is not a single query. It is an iterative process of hypothesis, search, pivot, verification, preservation and review. Strong OSINT depends on varied sources, careful syntax, current knowledge of search tools, ethical limits and documented reasoning.