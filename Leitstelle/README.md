# Leitstelle

The following discusses the OpenAPI specification from the Nationale Leitstelle called: *OBELISdeutschlandnetz v0.9*.

Whether this specification is internal, because it was attached to a limited tender or really "free" and "open" is still unclear.

## OCPI Credentials Module

How to authenticate and authorize your communication with the Leitstelle is not defined. Therefore it is likely, that the normal OCPI credentials module should be used. But this is well-known totally broken as it trusts the client to send correct country codes and party identifications... which is *bullshit* security.

## OCPI Routing Headers

The specification seems to be based on OCPI v2.2 and uses the OCPI Hub routing HTTP header information. As the Leitstelle is not an OCPI hub and we can always take the same sender information from the HTTP connection credentials, this information seems to be redundant. Therefore it is unclear why this optional headers are mentioned in this use case.

```
OCPI-from-party-id      OPCI Party-ID of BoB
OCPI-from-countrycode   OPCI country BoB
OCPI-to-party-id        OPCI Party-ID of sender
OCPI-to-country-code    OPCI country code of sender
```


## Locations

### POST /locations and PUT|PAT|DELETE /locations/{locationId}

```
{
  "name": "Aldi-Süd Parkplatz",
  "address": "Am Stellwerk 1",
  "postalCode": "70197",
  "city": "Stuttgart",
  "state": "BW",
  "coordinates": {
    "latitude": 27.4,
    "longitude": 123.6
  },
  "evses": [
    {
      "id": "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
      "evseId": "DE*ABC*E45B*78C",
      "status": "CHARGING",
      "statusSchedule": [
        {
          "periodBegin": "2023-07-02T18:13:11.067Z",
          "periodEnd": "2023-07-02T18:13:11.067Z",
          "status": "RESERVED"
        }
      ],
      "capabilities": [
        "CREDIT_CARD_PAYABLE",
        "PED_TERMINAL"
      ],
      "connectors": [
        {
          "id": "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
          "standard": "CHADEMO",
          "powerType": "AC_1_PHASE",
          "format": "SOCKET",
          "voltage": 23,
          "amperage": 16,
          "maxPower": 23000
        }
      ],
      "manufacturer": "ChargeX",
      "model": "Aqueduct + Pro",
      "negligibleDefects": false,
      "predecessor": "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611"
    }
  ],
  "operator": "InCharge AB",
  "owner": "Aldi-Süd",
  "auxiliaryFacilities": [
    "FOOD_SERVICE",
    "GREEN_ROOF"
  ],
  "negligibleDamages": false,
  "lastUpdated": "2023-07-02T18:13:11.067Z"
}
```

#### State

An enum that represents the german states using exactly 2 letters (e.b. BB = Brandenburg, HE = Hessen, ...).
This *conflicts* with the OCPI v2.2.1 specification: string(20) vs. string(2)!

```
"state": "TH"

```


#### Auxiliary Facilities

An enum of type string that represents a auxiliary facility that is available at the location.

```
"auxiliaryFacilities": [
    "FOOD_SERVICE",
    "GREEN_ROOF",
    "ROOF",
    "FOOD_SERVICE",
    "SEATING",
    "SANITATION",
    "VEHICLE_SUPPLIES_AIR_PRESSURE",
    "VEHICLE_SUPPLIES_WINDSHIELD_WIPER_SYSTEM",
    "VEHICLE_SUPPLIES_VACUUM_CLEANER",
    "GREEN_ROOF",
    "PHOTOVOLTAIC_SYSTEM"
  ],
```

#### Negligible Damages

A boolean that describes if there are any negligible damages at the location or EVSE.

```
  "negligibleDamages": false
```

### POST /locations/{locationId}/renewal

Report renewal of multiple EVSEs at a certain location.

Not using the official EVSE Ids, but the CPO internal identifications is a very bad API design.

Having this at the EVSE level does not make much sense. It should be at the charging station level, as the charging station is the physical device. EVSEs are just logical devices.

What is the definition of a `SOFTWARE_FAULT` leading to ab renewal of a charging station?

For this endpoint only POST seems to be defined. But the definition of the endtime seems to indicate, that there might be at least a 1st error report and a final repair report. How do I know, that both belong together and are not two independent errors? When a repair is rescheduled multiple times, as a replacement part was not delivered in time how do we know, that all those reports belong to the same initial report? Currently we could only reason about the EVSE Ids and the starting time. But as soon as multiple different reports overlap, we end up in total chaos.

```
{
  "evseIds": [                                  // Set of unique EVSE IDs that belong to EVSEs that were renewed. Omitting this set indicates that all EVSEs were renewed.
    "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
    "5459fd2c-b75e-4a93-8fcb-2dde5d2fc612"
  ],
  "renewedComponents": [                        // Set of renewed EVSE components. At least one element must be provided.
    {
      "chargePointComponent":  "DISPLAY",       // CABLE_COOLING_UNIT|NETWORKING_ELECTRONICS|POWER_MODULE|PAYMENT_DEVICE|VEHICLE_COMMUNICATION_UNIT|BACKEND_COMMUNICATION_UNIT|DISPLAY|DC_METER|AIR_FILTER|OTHERS
      "reason":                "DAMAGE"         // DAMAGE|HARDWARE_FAULT|SOFTWARE_FAULT|RETROFITTING|OTHERS
    }
  ],
  "start":  "2023-07-02T18:20:22.986Z",        // UTC datetime for when the renewal work has started/will start.
  "end":    "2023-07-02T18:20:22.986Z"         // UTC datetime for when the renewal work was finished/is scheduled to finish.
}
```

The HTTP response will return multiple report instances, each having its unique report identification which was generated by the Leitstelle. It is very bad API design, that the receiver of data chooses an unique identification, because now the sender (and owner) of this data has to store this data along with his own (internal) identifications. It is also very bad API design to use UUIDs for this, as this does not allow to create unique identifications in a distributed way. A simple format like `DE*GEF*REPORT*RENEWAL*5459fd2c-b75e-4a93-8fcb-2dde5d2fc611` would have solved so many problems.

Also we now have to update multiple reports when there is a problem at the missing charging station level. This is very bad API design.

```
{
  "data": [
    {
      "id":           "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",         // The renewal report identification generated by the Leitstelle!
      "locationId":   "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
      "evseId":       "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
      "renewedComponents": [
        {
          "chargePointComponent": "DISPLAY",
          "reason": "DAMAGE"
        }
      ],
      "start": "2023-07-02T18:48:53.939Z",
      "end": "2023-07-02T18:48:53.939Z"
    }
  ],
  "timestamp": "2022-10-24T12:00:00Z",
  "status_code": 1000,
  "status_message": "Request was processed successfully"
}
```



### POST /locations/{locationId}/ratings and PUT|PATCH /locations/{locationId}/ratings/{month}

```
{
  "month": "09-2022",     // MM-yyyy or (0[1-9]|1[0-2])-[0-9]{4}. Only within POST requests!
  "rating": 2             // Average customer location rating in german school grades (1 = best, 6 = worst)
}
```


### /locations/{locationId}/ad-hoc-tariff/{month}

OCPI has a [tariffs module](https://github.com/ocpi/ocpi/blob/release-2.2.1-bugfixes/mod_tariffs.asciidoc). It is unclear why it was reinvented for the special case of "ad hoc"-tariffs.

Having a start and end date within the ad hoc tariff data structure and additionally a `{month}` parameter in the URL seems to be a design flaw.

```
{
  "cost":        12,
  "additionalCosts": [
    {
      "description":  "Parking fee in ct per hour [h].",
      "cost":          12
    }
  ],
  "validFrom":  "2023-07-02T10:53:36.982Z",
  "validTo":    "2023-07-02T10:53:36.982Z"
}
```



### /locations/{locationId}/facility-damages/{facilityDamageId}





## (Missing) Charging Stations

In OCPI the charging station level is traditionally missing, and everybody hates it. For the use case of the Leitstelle this is even more disturbing, as many problems and damages relate not to locations or EVSEs (electrical circuits with one or more charging connectors), but to charging stations. Therefore it is unclear how to report e.g. a a broken display: Should it be reported at the location level once, or multiple times at the EVSE level of a charging station having 4 or 6 EVSEs?


## EVSEs

### POST /locations/{locationId}/evses and PUT|PATCH|DELETE /locations/{locationId}/evses/{evseId}

When updating an EVSE there is an optional JSON property called `predecessor`, how this should be used is not described. Changing EVSEs is a highly problematic process, as we are within a distributed system an can not make sure, that at every moment in time everyone has the same information about an EVSE. What the concept of `EVSE Id` really means is no where defined correctly. It could be the unique identification of an EVSE hardware (but this is part of a charging station), the unique identification of a logical EVSE at a CPO or the unique identification of a charging possibility on a digital map. All mean the same as long as nothing changes, but during change processes those unique identifications mean something completely different.

```
"predecessor": "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611"
```

The JSON properties `manufacturer` and `model` are again arrays of defined values. This is very bad API design as it leads to a protocol and API versioning hell.




### POST /locations/{locationId}/evses/{evseId}/renewal and PUT|PATCH /locations/{locationId}/evses/{evseId}/renewal/{renewalId}

Report the renewal of a specific EVSE.

These requests have the same problems like the renewal reports at the location level.



### POST /locations/{locationId}/evses/{evseId}/ratings and PUT|PATCH /locations/{locationId}/evses/{evseId}/ratings/{month}

Monthly reports for charging session ratings of a specific EVSE.

These reports have the same problems like the rating reports at the location level.

```
{
  "month":            "09-2022",     // MM-yyyy or (0[1-9]|1[0-2])-[0-9]{4}. Only within POST requests!
  "rating":            2,            // Average customer rating for charging sessions at the EVSE for the current month in german school grades (1 = best, 6 = worst).
  "totalComplaints":   8             // The amount of the received complaints for charging sessions at the EVSE in question.
}
```


### POST /locations/{locationId}/evses/{evseId}/defects and PUT|PATCH /locations/{locationId}/evses/{evseId}/defects/{defectId}

```
{
  "type":           "SOFTWARE_FAULT",             // Enum that represents the type of error/defect that occurred at an EVSE: DAMAGE|HARDWARE_FAULT|SOFTWARE_FAULT|NETWORK_FAILURE|POWER_FAILURE|ROUTE_CLOSURE|SITE_CLOSURE|DANGEROUS_MALFUNCTION|USAGE_IMPOSSIBLE|RENEWAl_NECESSARY|OTHER
  "status":         "TECHNICIAN_INFORMED",        // Enum that represents the status of a charge point defect that occurred at an EVSE: TECHNICIAN_INFORMED|TECHNICIAN_ON_THE_WAY|ERROR_SOLVED
  "activities":     "Remote access",              // Necessary activities for fixing the error: REMOTE_ACCESS|ON_SITE
  "emergenceDate":  "2023-07-02T19:08:40.998Z",   // UTC datetime for when the defect was detected.
  "forecastDate":   "2023-07-02T19:08:40.998Z",   // UTC datetime for when the defect should be fixed.
  "fixDate":        "2023-07-02T19:08:40.998Z"    // UTC datetime for when the defect was fixed [optional]
}
```

The HTTP response will return an unique defect report identification which was generated by the Leitstelle. It is very bad API design, that the receiver of data chooses an unique identification, because now the sender (and owner) of this data has to store this data along with his own (internal) identifications. It is also very bad API design to use UUIDs for this, as this does not allow to create unique identifications in a distributed way. A simple format like `DE*GEF*REPORT*DEFECT*5459fd2c-b75e-4a93-8fcb-2dde5d2fc611` would have solved so many problems.

Also we have to send and update multiple reports when there is a problem at the missing charging station level. This is very bad API design.

```
{
  "data": {
    "id":              "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",         // The defect report identification generated by the Leitstelle!
    "locationId":      "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
    "evseId":          "5459fd2c-b75e-4a93-8fcb-2dde5d2fc611",
    "type":            "SOFTWARE_FAULT",
    "status":          "TECHNICIAN_INFORMED",
    "activities":      "Remote access",
    "emergenceDate":   "2023-07-02T19:08:41.002Z",
    "forecastDate":    "2023-07-02T19:08:41.002Z",
    "fixDate":         "2023-07-02T19:08:41.002Z"
  },
  "timestamp":       "2022-10-24T12:00:00Z",
  "status_code":      1000,
  "status_message":  "Request was processed successfully"
}
```


## Charging Sessions

### /locations/{locationId}/evses/{evseId}/charging-sessions/{chargingSessionId}

The charging sessions are not anonymized. What kind of "hash" is ment with `"DE-ABC-hashedPart"`? Even cryptographical secure hashes on short strings like eMAIds or RFID UIDs do not provide any security as we can use rainbow tables against them, or even just calculate all possibilites. Also hashed user identifications still have the same privacy risk as clear text user identifications, as they stil allow to track user movements.

The charging sessions include the signed meter values of the *German Calibration Law*. As this data includes unhashed personal data and the Leitstelle does not have any legal authority to demand personal data, this seems to be a direct violation of the European General Data Protection Regulation (GDPR)!

According to verbal explanations of the Leitstelle (29. June 2023) those charging sessions should not be send in real time (so no load management use case), but within 24 hours after the charging session. Therefore the use case of this data is unclear. When the Leitstelle really just wants to "understand" charging sessions, we could remove all privacy sensitive parts and only send a minor probe (~2%) of all charging sessions, or just "interesting/unusual ones". But when they do not want real time data, why is the `PATCH`method defined?

```
{
  "start": "2023-07-02T11:00:04.402Z",
  "end": "2023-07-02T11:00:04.402Z",
  "emaId": "DE-ABC-hashedPart",
  "tariffType": "PROFILE_GREEN",
  "chargingPeriods": [
    {
      "start": "2023-07-02T11:00:04.402Z",
      "dimensions": [
        {
          "type": "ENERGY",
          "value": 12.5
        }
      ],
      "encodingMethod":  "MyEncodingMethod",
      "publicKey":       "MyPublicKey",
      "signedValues": [
        {
          "nature":      "CHARGING_SESSION_START",
          "plainData":   "12000",
          "signedData":  "SignedDataString"
        }
      ]
    }
  ],
  "totalCost": 34.5,
  "totalEnergy": 25000,
  "totalTime": 3600,
  "totalParkingTime": 600,
  "reason": "POWERLOSS"
}
```


## Hotline

### POST /hotline-ratings and PUT|PATCH /hotline-ratings/{month}

In which way this hotline rating data can be assumed to be correct? A CPO had no incentive to report correct data and the Leitstelle will not be able to verify it.

The API design is a bit strange:
 - There is not GET method to verify the uploaded data.
 - The JSON property "month" is not a month, but complex data. Having just a "year" and a "month" property would have been easier.

```
{
  "month":        "09-2022",     // MM-yyyy or (0[1-9]|1[0-2])-[0-9]{4}. Only within POST requests!
  "hotlineCalls":  1570,         // Amount of hotline calls (int32, so -23 is ok ;) )
  "fcrRate":       0.67,         // First call resolution rate [0..1]
  "asa":           64.7,         // Average speed of answer in seconds.
  "rating":        2             // Average customer hotline rating in german school grades (1 = best, 6 = worst)
}
```


## Nondiscriminatory-Access

### POST /nondiscriminatory-access and PUT|PATCH /nondiscriminatory-access/{providerGroup}/{month}

In which way this hotline rating data can be assumed to be correct? A CPO had no incentive to report correct data and the Leitstelle will not be able to verify it.

Using an array of "well-known" EMP Ids is surely NOT a proper way to validate EMP Ids. This is very bad API design as it leads to a protocol and API versioning hell.

The API design is a bit strange:
 - There is not GET method to verify the uploaded data.
 - The JSON property "month" is not a month, but complex data. Having just a "year" and a "month" property would have been easier.
 - When this "contract" has a start- and enddate in which way is the "month" still relevant? Why reporting this contract every month, when it does not change during that time?

```
{
  "month":                    "09-2022",                         // MM-yyyy or (0[1-9]|1[0-2])-[0-9]{4}. Only within POST requests!
  "providerGroup":            "DEVFL",                           // Unique BDEW code of the provider group in question, defined by an array of 1070 EMP Ids. Only within POST requests!
  "cost":                      12,                               // Fee/Cost [int32] in cents [ct] per kilowatt hour [kWh].
  "additionalCosts": [
    {
      "description":   "Parking fee in ct per hour [h].",        // Description for the additional fee/cost.
      "cost":           12                                       // Additional fee/cost [int32] value in _verbaly_ described unit.
    }
  ],
  "validFrom":                "2023-07-02T17:29:34.922Z",
  "validTo":                  "2023-07-02T17:29:34.922Z",
  "contractualBasis":         "ROAMING_CONTRACT",                // Enum that represents the contractual basis for charge access at a location: ROAMING_CONTRACT|BILATERAL_CONTRACT
  "reasoningDifferentCosts":  "Some reason."                     // Reason why the provider group has a different fee than all others.
}
```



