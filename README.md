# WWCP_ESGP


This software will allow the communication between World Wide Charging Protocol
(WWCP) entities and entities implementing the *E-Mobility Statistics Gathering Protocol (ESGP)*.
The focus of this protocol are the communication aspects between charge point operators (CPOs)
and national contact points (NCPs) in e-mobility.


### Specification

The internal (or at least they claim, that this would be "free" and "open") [OpenAPI specification](https://www.openapis.org) from the [Nationale Leitstelle](https://nationale-leitstelle.de): [OBELISdeutschlandnetz v0.9](Leitstelle/OBELISdeutschlandnetz_0.9-1.yaml).

Be aware, that this "OCPI specification" seems to be a quite strange specification. More details on design flaws and why this is not really OCPI can be found [here](Leitstelle/README.md).

Main problems of this protocol specification are:
- A bit based on the bad parts of OCPI v2.2, e.g. missing charging station concept, broken authentication/authorization.
- Most unique data identificators are generated on the receiver side, this makes the process of submitting and later updating data, e.g. a repair process, complicated for no good reason.
- Predefined strings for `EMP Ids`, `Charging Station Manufacturers` and `Charging Station Models` will lead to unneccessary protocol versioning issues. *"Closed world"* protocol designs do not work in e-mobility!
- General very bad API design, e.g. GET methods for validating data already sent are missing completely. This protocol is more a postcard service.
- If the data sent should have any regulatory or legal importance (in the future) any data should be digitally signed. This also means that some kind of organization, key and certificate management is required.

Currently it does not make much sense to implement this protocol!


### Suggestions

- Energy related (real-time) information esp. for locations having more charging stations than the energy uplink (or multiple energy related bottlenecks) can handle concurrently and thus have to use some sort of load balancing should be made available.
- Charging sessions under load balancing might not be handled equally. A common case are priorities based on user identifications or special backend management capabilities because a user needs to leave earlier than another one. So to understand concurrent charging sessions those priorities should be included within the charging session information.
- Each CPO should also report monthly IT security incidents. Each RFID and AutoCharge authentication should be counted as one IT security incident. Each ISO 15118 authentication should be counted as one privacy incident.


### Related Work

- https://driveelectric.gov/resources/#ev-chart
- https://driveelectric.gov/files/ev-chart-data-guidance.pdf
- Electric Vehicle Charging Data Performance & Reporting (SAE report, 2023)

