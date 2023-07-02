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

### /locations/{locationId}

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

A boolean that describes if there are any negligible damages at the location.

```
  "negligibleDamages": false
```

### /locations/{locationId}/ratings/{month}



### /locations/{locationId}/ad-hoc-tariff/{month}

OCPI has a [tariffs module](https://github.com/ocpi/ocpi/blob/release-2.2.1-bugfixes/mod_tariffs.asciidoc). It is unclear why it was reinvented for the special case of "ad hoc"-tariffs.

Having a start and end date within the ad hoc tariff data structure and additionally a `{month}` parameter in the URL seems to be a design flaw.

```
{
  "cost": 12,
  "additionalCosts": [
    {
      "description": "Parking fee in ct per hour [h].",
      "cost": 12
    }
  ],
  "validFrom": "2023-07-02T10:53:36.982Z",
  "validTo": "2023-07-02T10:53:36.982Z"
}
```



### /locations/{locationId}/facility-damages/{facilityDamageId}




## EVSEs

### /locations/{locationId}/evses/{evseId}/renewal/{renewalId}


### /locations/{locationId}/evses/{evseId}/ratings/{month}


### /locations/{locationId}/evses/{evseId}/defects/{defectId}



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
      "encodingMethod": "MyEncodingMethod",
      "publicKey": "MyPublicKey",
      "signedValues": [
        {
          "nature": "CHARGING_SESSION_START",
          "plainData": "12000",
          "signedData": "SignedDataString"
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

### /hotline-ratings/{month}



## Nondiscriminatory-Access

### /nondiscriminatory-access/{providerGroup}/{month}

