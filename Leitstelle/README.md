# Leitstelle

The YAML defines the OCPI Hub routing HTTP header information, but as the Leitstelle is not an OCPI hub and we can take the 'from* information from the credentials those information seems to be redundant.

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


### /locations/{locationId}/facility-damages/{facilityDamageId}




## EVSEs

### /locations/{locationId}/evses/{evseId}/renewal/{renewalId}


### /locations/{locationId}/evses/{evseId}/ratings/{month}


### /locations/{locationId}/evses/{evseId}/defects/{defectId}



## Charging Sessions

### /locations/{locationId}/evses/{evseId}/charging-sessions/{chargingSessionId}



## Hotline

### /hotline-ratings/{month}



## Nondiscriminatory-Access

### /nondiscriminatory-access/{providerGroup}/{month}

