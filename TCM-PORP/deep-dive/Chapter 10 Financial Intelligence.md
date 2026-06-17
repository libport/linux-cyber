# Chapter 10: Financial Intelligence
Financial intelligence (FININT) collects and analyses financial indicators to understand entity behaviour and forecast likely activity. It commonly supports investigations into money laundering, tax evasion, fraud, organised crime and terrorism financing. OSINT analysts usually lack bank records, so they use lawful public sources to connect people, accounts, businesses, assets, purchases, payments, sanctions records, public filings, cryptocurrency wallets and adverse media.
## Financial intelligence organisations
Financial intelligence units receive, analyse and disseminate suspicious transaction reports, suspicious activity reports, transaction reports and other financial intelligence. They support domestic investigations and share intelligence with foreign counterparts through formal networks.

FinCEN is a bureau of the US Department of the Treasury and the financial intelligence unit of the United States. It administers Bank Secrecy Act responsibilities, receives financial transaction data and disseminates intelligence for law enforcement, tax, regulatory, intelligence and counter-terrorism purposes. SAR confidentiality matters because disclosure can alert a subject and compromise an investigation.

FATF sets global anti-money-laundering and counter-terrorism-financing standards. The FDIC provides public US banking data through BankFind Suite, including current and former insured institutions, FDIC certificate numbers, financial trends, deposits, failures and assistance. The IMF supports monetary cooperation and financial stability across 191 member countries. The FFIEC BSA/AML Examination Manual helps analysts understand regulatory expectations and risk indicators. OFAC administers and enforces US economic and trade sanctions, which makes its lists and programme material useful for researching sanctioned people, companies, addresses, vessels, aircraft and digital wallet identifiers.
## Organised crime, PEPs and corruption
Organised crime relies on profitable markets such as drug trafficking, migrant smuggling, human trafficking, firearms trafficking, counterfeit goods, cybercrime and money laundering. Transnational criminal organisations coordinate activity across borders and use violence, corruption and legitimate businesses to protect revenue. Analysts can start with sanctions lists, INTERPOL notices, court records, corporate registries, investigative reporting and regional crime indexes. INTERPOL Red Notices request that law enforcement locate and provisionally arrest a person pending extradition, surrender or similar legal action, but they are not international arrest warrants.

A politically exposed person (PEP) is a person who is or has been entrusted with a prominent public function. The category can include senior politicians, senior judicial or military officials, senior executives of state-owned corporations, important political party officials, senior international organisation officials, family members and close associates. PEP status is a risk indicator, not an allegation of criminality. Analysts should look for source-of-wealth gaps, unexplained intermediaries, inconsistent asset declarations, shell companies, high-risk industries, procurement links and family or associate ownership.
## AML, CFT and financial crime patterns
Money laundering conceals the source, ownership, location or movement of criminal proceeds. A common model uses three stages: placement introduces illicit funds into the financial system, layering creates distance through transfers or transactions, and integration returns value to the economy with an apparently lawful source. Structuring, also called smurfing, breaks transactions into smaller amounts to avoid reporting thresholds.

Anti-money-laundering (AML) programmes use customer due diligence, know-your-customer checks, beneficial-ownership information, source-of-funds questions and ongoing monitoring to detect suspicious behaviour. Counter-terrorism financing work tracks funds that support terrorism and may involve lawful-looking sources, charities, businesses, informal value transfer, trafficking and laundering.

OSINT strengthens financial investigations by adding context. It can link aliases to people, identify businesses and associates, locate assets, compare lifestyle with lawful income, trace public cryptocurrency activity and reveal links to sanctions, court records and adverse media. It can also expose red flags for tax evasion, tax fraud, embezzlement, Ponzi schemes, trade-based money laundering and cyber-enabled fraud.

Tax evasion uses concealment or deceit to avoid tax. Tax fraud involves knowingly false tax reporting, such as omitted income or overstated expenses. Embezzlement occurs when a trusted person diverts assets they were meant to manage. The FTX case should be treated as fraud and misappropriation rather than as a tax-fraud example. Sam Bankman-Fried was convicted of fraud and conspiracy offences, including conspiracy to commit money laundering, and was sentenced in 2024 to 25 years in prison.
## Identifiers and source selection
Financial identifiers guide pivots. An issuer identification number or bank identification number identifies the issuer and payment network for a card, but six-digit lookups can be incomplete because major schemes now support eight-digit BINs. ABA routing numbers identify US banks for domestic transfers. SWIFT means Society for Worldwide Interbank Financial Telecommunication. A BIC is an 8- or 11-character Business Identifier Code, and not every BIC connects to the SWIFT network.

VAT registrations can connect business names, addresses, status and tax identifiers. ISO 3166 alpha-2, alpha-3 and numeric country codes help analysts interpret financial documents. Currency codes and banknote images can help geolocate material in photos and videos. Public lookup sites are useful for orientation, but analysts should confirm key facts through official registers, regulator data, court records, corporate filings or primary documents.
## Analytical workflow
Begin with a reliable clue: username, email, phone number, subject name, business, wallet address, vehicle, property or suspicious purchase. Then ask targeted questions:
- Does the subject have organised-crime links?
- Is the subject a PEP, family member or close associate?
- Does lifestyle align with lawful income?
- Do public posts show windfalls, luxury purchases or unexplained travel?
- Do usernames, emails or phone numbers appear in forums, advertisements, leaks, marketplaces or payment notes?
- Is the subject linked to businesses, charities, state-owned enterprises, sanctions or court records?
- Are transactions linked to cryptocurrency wallets, trade routes, invoices, shell companies or high-risk jurisdictions?

Use adverse-media strings that match the search engine. For Google, a corrected starting point is:

`"ENTITY NAME" (arrest OR conviction OR criminal OR fraud OR lawsuit OR "money laundering" OR OFAC OR Ponzi OR terrorist OR violation OR "honorary consul" OR consul OR "Panama Papers" OR embezzlement OR evasion OR OCCRP)`

Drug-financing cases may require current slang, controlled-substance schedules, pill images, trafficking methods and regional price information. Organised-crime cases benefit from sanctions material, law-enforcement notices, court filings, corporate records, investigative journalism and regional crime analysis. Tools change, but the method remains stable: validate identity, connect entities, verify sources, preserve evidence and record uncertainty.