# Chapter 8: Transportation Intelligence
Transportation intelligence examines how people, goods, vehicles, weapons, services, and infrastructure move between places. It combines transport records, imagery, public disclosures, signals, organisational data, infrastructure information, and subject intelligence to answer questions about location, ownership, routing, intent, vulnerability, and future movement.

Analysts usually treat transport activity through five linked modes:
- Road, including cars, trucks, buses, motorcycles, bicycles, heavy equipment, and military vehicles
- Rail, including passenger trains, freight rail, metro systems, light rail, railcars, tracks, stations, yards, and signalling equipment
- Water, including cargo vessels, tankers, fishing vessels, passenger ships, naval vessels, ports, offshore platforms, pipelines, and undersea cables
- Air, including passenger aircraft, cargo aircraft, helicopters, drones, military aircraft, airports, airfields, and aerodromes
- Intermodal transport, where goods move across multiple modes such as ship, rail, truck, and air cargo

Transport data matters because modern life depends on continuous movement. Around 80 percent of international trade in goods by volume moves by sea, and interruptions in maritime, port, rail, trucking, and air cargo networks can disrupt food, fuel, electronics, medical supplies, and industrial production. The 2021 grounding of Ever Given in the Suez Canal showed how one blocked chokepoint can create global delays.

Transport intelligence also supports geopolitical, security, law enforcement, humanitarian, and business analysis. Analysts may need to assess the movement of troops, refugees, weapons, sanctioned cargo, stolen vehicles, illegal fishing fleets, smuggling routes, or commercial shipments. Subject intelligence becomes stronger when transport evidence shows where a person, vehicle, aircraft, ship, or shipment could realistically have been.

Common intelligence questions include:
- Which route did the subject or asset take
- Which ports, stations, airports, roads, or borders did it use
- Does the movement match normal economic behaviour
- Did the asset meet another vessel, vehicle, aircraft, or train
- Did any signal disappear, change identity, or report an impossible path
- Who owns, operates, leases, insures, services, or brokers the asset
- Which infrastructure or technology could create safety, security, or supply chain risk

Analysts should treat every source as partial. A signal can be spoofed, a schedule can change, a plate can be swapped, a social media post can be old, and a satellite image can be obscured by cloud or collected at the wrong time. Strong findings come from corroborating independent sources.

Transport selectors are the details that allow an analyst to pivot from one source to another. They include a vessel name, IMO number, MMSI, call sign, flag, draught, berth, container number, rail reporting mark, wagon number, service code, aircraft registration, ICAO address, call sign, flight number, airport code, vehicle plate, VIN, fleet number, company logo, route number, timestamp, and location. A single selector rarely answers the question. Its value comes from linking records and testing whether different sources describe the same asset.

Intermodal analysis follows cargo across hand-offs. A container may leave a factory by truck, enter a rail terminal, load onto a ship, transfer through a port, move by rail again, and finish by truck. Each hand-off creates records, images, schedules, contracts, gate movements, customs interactions, or public disclosures. Analysts can use those hand-offs to bridge gaps when one transport mode has poor public visibility.
## Collection methods
### Visual intelligence
Visual intelligence confirms what a transport asset looks like, where it appeared, and what identifiers it displayed. Analysts use direct sightings, photographs, video, social media, webcams, satellite imagery, street-level imagery, and specialised hobbyist collections.

Spotter communities provide useful transport evidence. Plane spotters, ship spotters, and rail enthusiasts often record dates, locations, registration numbers, liveries, reporting marks, names, and photographs. Their material can verify that a vehicle, aircraft, train, or vessel existed in a specific place at a specific time. Spotter evidence also helps compare visible identifiers against signal data, registries, and ownership records.

Social media creates transport intelligence when employees, passengers, coordinators, contractors, or hobbyists disclose more than intended. Crew videos may reveal bridge equipment, software versions, cargo operations, train yards, control rooms, aircraft interiors, or maintenance practices. Passenger posts can reveal routes, timestamps, stations, gates, ports, and pattern of life. Coordinators may disclose schedules, dispatch activity, operational constraints, or technology used to manage traffic.

Webcams can confirm a live or near real-time view of ports, airports, rail corridors, border crossings, highways, tourist sites, weather stations, and traffic points. A webcam can disprove a claimed signal position when a vessel or vehicle remains visible elsewhere. During conflict or unrest, small public cameras can capture movements that larger surveillance systems miss.

Satellite imagery gives analysts a top-down view where ground access is difficult or unsafe. Optical imagery can show vessels in port, aircraft on ramps, vehicles in staging areas, port congestion, rail yards, highway traffic, construction, and military build-up. Landsat, Sentinel, MODIS, ASTER, Meteosat, and other public or public-facing sources support broad analysis, while commercial providers offer higher resolution and more frequent tasking.

Synthetic aperture radar, or SAR, actively emits microwave energy and records the returned signal. SAR can collect imagery at night and through cloud, smoke, haze, and some weather conditions. It is valuable when optical imagery fails, especially for maritime, disaster, flood, conflict, and infrastructure monitoring.

Satellite analysis requires care. Analysts should compare size, scale, shape, shadows, colour, season, location, surrounding activity, and known dimensions. A tanker at sea, a railcar in a yard, or an aircraft at an airport may look similar to many others from above. Confirming length, deck shape, wing form, livery, berth position, shadow, or nearby infrastructure reduces mistaken identification.

Visual work also needs disciplined image handling. Analysts should preserve the original file, note the collection date, separate upload time from capture time, and record whether metadata is original, stripped, edited, or platform-generated. A reposted image can be useful, but it should not anchor a timeline unless the analyst can establish when and where it was captured. Shadows, weather, vegetation, road layout, port cranes, rail geometry, runway markings, signage, and skyline features can all support geolocation and time assessment.

Transport images often contain small but decisive details. A vessel photograph may show its deck colour, funnel mark, crane layout, safety equipment, hull scars, or draught marks. A rail image may show reporting marks, hazardous goods placards, consist order, locomotive number, and trackside infrastructure. An aircraft image may show the registration under the wing, a temporary sticker, a sensor pod, a cargo door, or a refuelling boom. A road image may show a partial plate, mirror shape, bumper damage, roof equipment, or local inspection sticker.
### Signal intelligence from transport systems
Transport platforms often broadcast or depend on navigation and identification signals. Analysts can use public or low-cost tracking platforms to follow vessels, aircraft, and sometimes rail or road assets, but they must understand what each signal can and cannot prove.

GNSS is the global term for satellite navigation systems. GPS is one GNSS system. GNSS satellites broadcast timing and orbital data that receivers use to calculate position. AIS for vessels and ADS-B for aircraft depend on GNSS or other navigation sources to report position.

AIS transponders automatically broadcast vessel identity, position, course, speed, destination, and other data to nearby ships, coastal authorities, ground stations, and satellites. International carriage rules apply to ships of 300 gross tonnage and upwards on international voyages, cargo ships of 500 gross tonnage and upwards not on international voyages, and passenger ships regardless of size. AIS supports safety and traffic management, but it is not automatically truthful.

ADS-B combines aircraft positioning sources, avionics, and ground infrastructure. ADS-B Out broadcasts aircraft position, altitude, ground speed, and other data to ground stations and other aircraft. ADS-B In provides traffic and weather information to equipped aircraft. Mode S transmissions identify aircraft without necessarily carrying position. Multilateration, or MLAT, estimates a Mode S aircraft position by comparing the time difference of arrival across multiple receivers.

Analysts should distinguish a missing signal from illegal activity. A vessel or aircraft may appear dark because of equipment failure, poor receiver coverage, military activity, privacy filtering, safety requirements, or deliberate concealment. A dark period becomes more important when it appears near a sanctioned port, illegal fishing zone, ship-to-ship transfer point, conflict area, border crossing, or suspicious cargo movement.

Spoofing creates false position or identity information. AIS spoofing can make a vessel appear somewhere it is not, create a vessel that does not exist, or hide activity behind a plausible route. Spoofed paths often show impossible jumps, hard straight lines across land, unrealistic speeds, duplicated identities, or positions that conflict with imagery and webcams. GNSS spoofing feeds receivers false satellite-like signals, which can corrupt dependent AIS or ADS-B positions.

Identity manipulation changes the static details broadcast by a platform. A vessel may alter its name, Maritime Mobile Service Identity, IMO number, vessel type, destination, or draught. Analysts should treat mismatched identifiers, recycled names, inconsistent dimensions, abrupt flag changes, and conflicting ownership records as warning signs. IMO numbers are intended to stay with a hull, so inconsistencies deserve close review.

Jamming degrades or masks satellite navigation by overpowering the receiver, transmitter, or local signal environment. It can affect ships, aircraft, vehicles, and unmanned systems. Meaconing intercepts and rebroadcasts navigation signals, usually at higher power, so a receiver accepts delayed or false navigation information. Jamming and meaconing can create safety risks, route deviations, collisions, false equipment faults, and geopolitical incidents.
## Maritime intelligence
Maritime intelligence covers vessels, ports, offshore infrastructure, undersea cables, pipelines, owners, operators, crews, cargo, and facilitators. It is central to supply chain, sanctions, fisheries, conflict, smuggling, piracy, environmental, and critical infrastructure analysis.

Key maritime entities include container ships, bulk carriers, tankers, passenger ships, naval vessels, submarines, offshore support vessels, fishing vessels, pleasure craft, dredgers, high-speed craft, roll-on roll-off ships, barges, buoys, offshore platforms, wind farms, undersea cables, pipelines, restricted zones, and autonomous maritime systems.

Basic vessel terminology improves visual and documentary analysis. The bow is the front, the stern is the back, the hull is the main body, the freeboard is the visible hull above the waterline, the draught is the submerged portion, the bridge is the navigation and control area, the accommodation is where crew live, and the mast can carry lights, antennas, radar, and other equipment. Draught changes can indicate cargo loading or unloading.

Maritime movement normally follows economic behaviour. Commercial vessels usually seek efficient routes that reduce time, fuel, risk, and cost. Long detours, unusual loitering, repeated meetings, dark activity, inconsistent cargo claims, or unexplained identity changes may indicate weather avoidance, piracy risk, sanctions evasion, conflict routing, smuggling, mechanical problems, or legitimate operational needs. Historical and geopolitical context decides whether a route is normal.

Useful maritime questions include:
- Where is the vessel now, and where has it been
- Does the path match normal trade, weather, port, and safety constraints
- Did the vessel go dark near a sensitive location
- Did it meet another vessel long enough to transfer cargo, fuel, crew, or stores
- Did its draught, destination, name, flag, owner, operator, or IMO details change
- Which companies, agents, brokers, port operators, insurers, or freight forwarders support it
- Which infrastructure did it approach, berth at, service, or threaten

Vessel paths come from AIS, satellite AIS, terrestrial receivers, port records, visual sightings, and commercial datasets. Most detailed historical AIS data sits behind paid or restricted platforms, so analysts often use public disclosures, news, company websites, ship spotter photographs, port cameras, satellite imagery, sanctions reporting, and social media to reconstruct history.

Ship-to-ship transfers occur when two vessels come alongside each other while stationary or moving to transfer oil, gas, fish, supplies, crew, or other cargo. Many transfers are lawful. Suspicious transfers may involve dark AIS, sanctioned goods, false origin documents, unusual meeting points, extended loitering, or vessels with opaque ownership. Analysts can test a suspected transfer by comparing vessel paths, meeting duration, draught changes, satellite imagery, port calls, cargo records, and ownership links.

Port calls provide strong context. A port is not just a loading point, it is a critical infrastructure hub with terminals, customs, security, shipping agents, stevedores, longshore workers, storage, cranes, road links, rail links, and support businesses. Berthing reports can reveal expected arrivals, departures, agents, estimated times, services, and sometimes cargo. Webcams and satellite imagery can confirm whether a vessel berthed, loaded, unloaded, or remained idle.

Maritime ownership analysis follows the money and control structure. A shipment can involve an importer, exporter, freight forwarder, shipping agent, broker, shipper, consignee, stevedore, terminal operator, customs broker, insurer, manager, charterer, beneficial owner, and port authority. Link analysis helps map relationships among vessels, companies, directors, subsidiaries, contracts, sanctions records, cargo movements, and public announcements.

The shipping process creates many pivot points. An importer orders goods, a freight forwarder arranges export, a shipping agent manages carriage, a broker may connect shipper and carrier, stevedores load containers, a terminal operator manages the berth, customs clears the movement, and a consignee or appointed broker handles arrival. A bill of lading records the contract and receipt for carriage, while a manifest lists cargo carried by the vessel. Public leaks, court records, procurement notices, port documents, insurance records, sanctions releases, company fleet pages, and trade databases can expose parts of this chain.

Cargo analysis should separate what is known from what is inferred. A draught change may suggest loading or unloading, but ballast changes can also alter draught. A tanker meeting may suggest oil transfer, but it may involve bunkering, stores, or maintenance. A port call in a sanctioned state may matter, but the cargo, beneficial owner, charterer, and timing decide its significance. Strong maritime conclusions link movement, vessel identity, cargo evidence, ownership, and context.

Potential maritime illegal activity includes sanctions evasion, arms smuggling, drug trafficking, people smuggling, illegal fishing, piracy, hijacking, shipbreaking violations, illegal dumping, terrorism support, and proliferation. Indicators should not be treated as proof without corroboration, because legitimate shipping can also produce unusual patterns.

Maritime critical infrastructure includes vessel control systems, bridge systems, engine control rooms, water ingress detection, rudders, safety systems, cargo tracking, shore-based navigation, cranes, port security, automated port systems, offshore platforms, undersea cables, and pipelines. Industrial control systems include SCADA, distributed control systems, programmable logic controllers, and other control components. Outdated systems, exposed interfaces, weak authentication, unmanaged contractors, and poor segmentation can create operational risk.

Undersea cables carry more than 95 percent of international data and support communications, finance, government, cloud services, and military activity. They remain vulnerable to anchors, fishing, accidents, sabotage, espionage, and network intrusion. Pipelines and offshore platforms face physical and digital risks. Public cable and pipeline maps, maritime charts, satellite imagery, AIS tracks, ownership records, and repair vessel movements can support lawful infrastructure risk analysis.
## Railway intelligence
Railway intelligence covers passenger trains, freight trains, railcars, locomotives, stations, platforms, yards, terminals, depots, routes, schedules, bridges, tunnels, signalling, power, communications, and control systems. Rail is efficient for heavy freight and high-volume passenger movement, and it often forms the inland leg of maritime and air cargo supply chains.

Visual identification starts with line, company, livery, logo, route, reporting marks, railcar type, locomotive type, graffiti, cargo markings, placards, and visible numbers. Freight railcars often display reporting marks that identify the owner or lessee. Hazard placards, container numbers, waybills, and company markings can indicate cargo class or handling requirements. Analysts should compare visual evidence with official registries, company fleets, public photographs, and rail enthusiast records.

Rail routes and schedules can expose both routine and abnormal movement. Passenger schedules often appear on operator websites, ticketing systems, transport apps, station signs, and real-time service feeds. Freight schedules are less public, but rail crossings, webcams, spotter logs, port intermodal records, industrial customer announcements, public contracts, and satellite imagery can reveal likely corridors and time windows.

Train movement analysis should consider the whole network. A train cannot leave the track, and its route depends on gauge, operator rights, signalling, power supply, bridges, tunnels, yard access, terminals, maintenance windows, border controls, and congestion. A suspicious movement may simply follow the only viable line, avoid damage, use a permitted corridor, or respond to a temporary closure.

Rail passenger analysis can help reconstruct a subject's movement when a station, timetable, ticket class, platform, surveillance still, social media post, or witness statement gives a time window. Analysts should avoid overclaiming from schedules alone. A scheduled train does not prove a person boarded it, and service disruptions can alter times and routes.

Railway evidence also appears in industrial and intermodal settings. Mines, ports, military depots, grain terminals, refineries, container terminals, and automotive plants often have private sidings or dedicated loading points. Satellite imagery can show rolling stock, container stacks, loading arms, gantries, and changes in yard occupancy. Local planning documents, environmental approvals, procurement notices, and infrastructure maps can reveal new lines, closures, upgrades, and high-value chokepoints.

Cargo handling adds another layer. Intermodal containers retain identifiers across truck, rail, and ship movements. Tank cars, hopper cars, autoracks, flatcars, refrigerated cars, and well cars indicate different commodity classes. Hazard placards and UN numbers can reveal regulated goods, although analysts should account for empty returns, residue, and placard errors. Freight analysis becomes stronger when visual rail evidence aligns with port calls, ship manifests, road haulage, and buyer or seller disclosures.

Rail ownership and operations include infrastructure managers, train operating companies, freight carriers, leasing companies, terminal operators, maintenance providers, government agencies, port rail operators, intermodal carriers, and private industrial railways. Organisational intelligence can reveal contracts, fleet ownership, government funding, commodity flows, defence logistics, and service dependencies.

Rail critical infrastructure includes onboard control systems, train backbone networks, signalling, points, crossings, positive train control or equivalent safety systems, communications, power, dispatch centres, bridges, tunnels, stations, yards, and maintenance facilities. Exposed devices, weak remote access, outdated systems, and public technical disclosures can create risk. Analysts should identify and report vulnerabilities only within lawful authority.
## Aircraft intelligence
Aircraft intelligence covers aircraft, airports, airfields, aerodromes, air traffic control, operators, owners, cargo, passengers, maintenance, ground services, flight plans, NOTAMs, registries, ADS-B data, Mode S data, MLAT positions, photographs, and airport imagery.

Aircraft categories include gliders, ultralights, balloons, helicopters, propeller aircraft, turboprops, business jets, airliners, cargo aircraft, military aircraft, drones, and specialised mission aircraft. Visual identification relies on registration markings, livery, make and model, wing shape, engine placement, tail design, windows, doors, landing gear, antennas, pods, sensors, and unusual modifications.

Registration systems vary by country. In the United States, civil aircraft registration numbers begin with N and are known as N-numbers. Other states use different prefixes and formats. Aircraft also have a manufacturer serial number and a 24-bit ICAO address used in transponder and ADS-B systems. A registration can change, but serial numbers and physical features help track continuity.

Flight paths come from ADS-B, MLAT, Mode S receivers, flight tracking websites, airport websites, schedules, spotter posts, ATC audio, NOTAMs, public disclosures, satellite imagery, and airport cameras. Analysts should compare several flight tracking platforms because coverage, filtering, delay, receiver density, military restrictions, privacy programmes, and data sources differ.

Privacy and filtering complicate aviation analysis. The FAA LADD programme can filter data from FAA feeds or from participating public displays. It does not stop an aircraft from broadcasting ADS-B. The FAA PIA programme allows eligible US-registered aircraft to use an alternate temporary ICAO address, which reduces the ability of low-cost receivers to connect a broadcast to a registered owner. Neither programme makes visual sightings, spotter photographs, ATC audio, or independent receivers irrelevant.

Cargo tracking often requires an air waybill number, carrier data, customs information, or public shipment disclosure. Without that, analysts can use company announcements, procurement records, contracts, route history, cargo aircraft schedules, airport handling records, social media, and intermodal evidence to infer possibilities. Such inferences need clear confidence language.

NOTAMs publish essential, abnormal, time-sensitive information for flight operations. They may cover closed runways, hazards, military activity, airspace restrictions, unmanned aircraft operations, navigation outages, VIP movement, or temporary flight restrictions. NOTAMs are useful, but they use abbreviations that require aviation knowledge and should be checked against official systems.

Airport and aerodrome analysis combines terminal maps, gate data, runway layouts, airport codes, webcams, satellite imagery, ground handling firms, fixed-base operators, hangars, fuel providers, maintenance organisations, and local road access. Geolocation can use runway orientation, apron layout, hangar shape, mountains, coastlines, shadows, signage, aircraft stands, and background structures.

Air traffic control audio can provide context that is absent from tracking sites. Calls may reveal diversions, runway changes, emergencies, routing, call signs, aircraft type, cargo handling concerns, weather, or restrictions. Coverage depends on feeder locations and local law, and live audio should be handled cautiously because it can involve active safety operations.

Airports use several code systems. IATA codes often appear in passenger travel and airline booking contexts, while ICAO airport codes appear in operational aviation contexts. Analysts should verify which code system a source uses before joining records. Similar names, shared metropolitan regions, and multiple airports serving one city can cause false links.

Aviation ownership and operation may involve registered owners, beneficial owners, trusts, leasing firms, charter operators, airlines, maintenance firms, shell companies, management companies, brokers, and ground handlers. Link analysis can connect aircraft to people, companies, contracts, sanctions, trips, cargo, and political or military activity.

Aviation critical infrastructure includes aircraft avionics, airport operational technology, baggage systems, boarding systems, fuel systems, radar, radio, navigation aids, access control, surveillance, power, lighting, weather systems, and maintenance platforms. Analysts should focus on identification, exposure, and risk reporting, not exploitation.
## Road and vehicle intelligence
Road intelligence covers personal vehicles, commercial fleets, buses, motorcycles, bicycles, logistics vehicles, trailers, construction machinery, agricultural machinery, emergency vehicles, and military vehicles. It supports investigations into subject movement, logistics, theft, smuggling, protests, organised crime, military activity, and infrastructure risk.

Vehicle identification begins with make, model, body style, year range, colour, trim, wheels, lights, grille, badges, damage, accessories, stickers, roof racks, cargo, and modifications. Reverse image search, model comparison, street-view imagery, manufacturer brochures, sales listings, enthusiast forums, and vehicle recognition tools can narrow a vehicle type, but analysts should not treat automated recognition as conclusive.

Number plates, also called licence plates, connect a vehicle to a registration system, although the registered keeper may not be the legal owner and plates can be stolen, cloned, obscured, or swapped. Plate design, colour, size, script, symbols, and regional marks can support geolocation. Public access to plate databases varies by jurisdiction, so analysts often rely on search engines, public posts, vehicle photo databases, auction sites, insurance write-off listings, and social media.

A vehicle identification number, or VIN, is intended as a permanent unique identifier for a vehicle. In the United States and Canada, vehicles from the 1981 model year onwards use a 17-character VIN format. VINs can reveal manufacturer, vehicle attributes, model year, plant, and serial information, but physical VIN plates can be tampered with. Analysts should compare the VIN across the dashboard, door label, title, registration, service records, sale listings, and official decoder results where lawful.

Route analysis can draw on street-level imagery, traffic cameras, toll disclosures, road closure notices, dashcam videos, social media, logistics portals, local news, weather, fuel stops, border crossings, weigh stations, ferry schedules, and satellite imagery. A vehicle's route depends on road access, fuel range, traffic, checkpoints, bridges, tunnels, weight limits, roadworks, ferries, and terrain.

Vehicle analysis often depends on narrowing possibilities rather than proving identity immediately. A partial number plate, unusual paint, damage, aftermarket wheels, roof lights, company decal, or load can reduce the search set. Analysts can compare the vehicle against manufacturer catalogues, dealership listings, auction archives, recall notices, fleet photographs, and local registration designs. A single visual trait can be misleading because parts can be replaced, copied, or modified. Multiple independent traits carry more weight.

Road freight adds commercial selectors. A trailer number, container number, dangerous goods placard, depot sticker, telematics unit, weighbridge reference, or carrier logo can connect road movement to a company, contract, or cargo chain. Border crossings, ferry bookings, weigh stations, port gates, logistics parks, warehouses, and fuel stops often provide the strongest route anchors.

Commercial vehicle analysis often pivots to organisational intelligence. Trucks, trailers, vans, buses, and heavy equipment may display company names, Department of Transportation numbers, fleet IDs, hazardous material placards, cargo markings, depot locations, contractor logos, phone numbers, QR codes, or asset tags. These selectors can identify operators, subcontractors, clients, permits, contracts, safety records, and movement patterns.

Modern vehicles contain electronic control units, sensors, telematics, infotainment systems, remote diagnostics, tyre pressure systems, GPS receivers, cellular modules, Bluetooth, Wi-Fi, keyless entry, cameras, and advanced driver assistance systems. Many internal components communicate across in-vehicle networks such as the CAN bus. Connectivity improves safety and efficiency but expands the attack surface. Publicly exposed telematics, weak credentials, outdated software, poor vendor management, and insecure remote access can create safety and privacy risks.

Analysts assessing vehicle technology should work from lawful authority, visible disclosures, product documentation, manufacturer information, vulnerability databases, and approved security assessments. Risk findings should describe exposure, potential impact, and remediation paths without enabling misuse.
## Analytical discipline
Transport intelligence works best when analysts build timelines and test each claim against independent evidence. A strong timeline records dates, times, time zones, locations, source type, confidence, and contradictions. The same event may appear differently in AIS, ADS-B, station schedules, port reports, social media, satellite imagery, and eyewitness material.

Analysts should prioritise primary and high-confidence sources, including official registries, port and airport authorities, transport operators, regulators, satellite imagery, original photos, live cameras, and direct technical documentation. Secondary reporting and social media are useful leads, but they need verification.

A useful analytic file separates evidence from assessment. Evidence records what the source shows. Assessment explains what the evidence may mean. Confidence language should reflect source quality, corroboration, and the number of plausible alternatives. Words such as confirmed, likely, possible, and unverified should be used consistently.

Time handling deserves particular care. Transport data crosses time zones, daylight saving changes, UTC timestamps, local schedules, delayed feeds, archived captures, edited videos, and platform upload times. Analysts should normalise time zones, keep original timestamps, and state which time standard a timeline uses.

Ethical handling is part of accuracy. Transport intelligence can expose the location of private people, crews, refugees, critical infrastructure, and active operations. Analysts should minimise unnecessary personal data, avoid doxxing, respect legal limits on tracking and communications monitoring, and avoid publishing tactical details that create immediate risk. Sensitive findings should go to authorised stakeholders through controlled channels.

A good transport assessment ends with clear judgements and clear limits. It should identify the asset, route, timeframe, supporting sources, contradictions, unresolved gaps, and the most plausible explanations. It should also state what would change the judgement, such as a later port record, clearer imagery, official registry update, additional receiver coverage, or confirmation from a primary document.

Errors often arise from assuming that a signal equals truth, that absence equals concealment, that a schedule equals presence, that a registration equals beneficial ownership, or that an unusual route equals crime. Analysts should state uncertainty, explain alternative explanations, and avoid publishing sensitive details that could endanger people, active operations, or critical infrastructure.

Transport intelligence ultimately links movement to meaning. It identifies what moved, how it moved, where it moved, who controlled it, what infrastructure enabled it, what risks it created, and what evidence supports the conclusion.