# WWCP_ESGP


This software will allow the communication between World Wide Charging Protocol
(WWCP) entities and entities implementing the *E-Mobility Statistics Gathering Protocol (ESGP)*.
The focus of this protocol are the communication aspects between charge point operators (CPOs)
and national contact points (NCPs) in e-mobility.


### Scope

This implementation aims to become one of the ESGP reference implementations. Therefore you will find a lot of inline documentation, many test cases and mockups.


### Specification

The internal (or at least they claim, that this would be "free" and "open") [OpenAPI specification](https://www.openapis.org) from the [Nationale Leitstelle](https://nationale-leitstelle.de): [OBELISdeutschlandnetz v0.9](Leitstelle/OBELISdeutschlandnetz_0.9-1.yaml).

Be aware, that this "OCPI specification" seems to be a quite strange specification. More details on design flaws and why this is not really OCPI can be found [here](Leitstelle/README.md).


### Related Work

- https://driveelectric.gov/resources/#ev-chart
- https://driveelectric.gov/files/ev-chart-data-guidance.pdf
- Electric Vehicle Charging Data Performance & Reporting (SAE report, 2023)

