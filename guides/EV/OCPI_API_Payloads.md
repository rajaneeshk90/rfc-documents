## OCPI 2.2.1 API Payloads – Locations, Tariffs, Commands, Sessions, CDRs

Notes:
- Base path and role-specific prefixes vary by party (CPO vs eMSP). Examples use generic paths for clarity (e.g., `/ocpi/2.2/...`).
- Headers (typical): `Authorization: Token <token>`, `X-Request-ID`, `X-Correlation-ID`, `Content-Type: application/json`.
- Pagination: `offset`, `limit` query params; responses include `Link` header and `X-Total-Count`.
- Response wrapper: `{ "data": ..., "status_code": 1000, "status_message": "Success", "timestamp": "..." }` (values illustrative).

---

### 1. Locations API

#### 1.1 GET /ocpi/2.2/locations?offset=0&limit=20
Request: query filter and pagination; optionally `country_code`, `party_id` if supported by party.

Response (excerpt):
```json
{
  "data": [
    {
      "id": "LOC-DELHI-001",
      "name": "BlueCharge Connaught Place Station",
      "address": "Connaught Place, New Delhi",
      "coordinates": { "latitude": "28.6304", "longitude": "77.2177" },
      "time_zone": "Asia/Kolkata",
      "parking_type": "PARKING_GARAGE",
      "facilities": ["RESTAURANT", "TOILETS", "SHOPPING"],
      "evses": [
        {
          "uid": "EVSE-001",
          "evse_id": "IN*BCN*E001",
          "status": "AVAILABLE",
          "parking_restrictions": ["CUSTOMERS"],
          "connectors": [
            {
              "id": "1",
              "standard": "IEC_62196_T2",
              "format": "SOCKET",
              "power_type": "AC_3_PHASE",
              "max_voltage": 415,
              "max_amperage": 16,
              "max_electric_power": 10000,
              "tariff_ids": ["tariff-01"]
            }
          ],
          "last_updated": "2025-08-01T11:25:10Z"
        }
      ],
      "opening_times": { "twentyfourseven": true },
      "last_updated": "2025-08-01T11:27:05Z"
    }
  ],
  "status_code": 1000,
  "status_message": "Success",
  "timestamp": "2025-08-14T09:58:05Z"
}
```

**Field Descriptions:**

**Location Object:**
- `id` (required): Unique identifier for the charging location assigned by the CPO
- `name` (optional): Human-readable name/title of the charging location
- `address` (required): Street/block/building number of the charging location
- `coordinates.latitude` (required): Geographic latitude of the charging location
- `coordinates.longitude` (required): Geographic longitude of the charging location
- `time_zone` (optional): One of IANA tzdata time zone identifiers
- `parking_type` (optional): Type of parking at the charge point location
- `facilities` (optional): Optional list of facilities this charging location directly belongs to
- `opening_times` (optional): Hours when the EVSEs at the location can be accessed for charging
- `last_updated` (required): Timestamp when this location was last updated

**EVSE Object:**
- `uid` (required): Unique identifier of the EVSE within the CPO's domain
- `evse_id` (optional): Compliant with ISO-IEC-15118 standard EVSE identification
- `status` (required): Indicates the current status of the EVSE
- `parking_restrictions` (optional): List of restrictions that apply to the parking bay where this EVSE is located
- `last_updated` (required): Timestamp when this EVSE status was last updated

**Connector Object:**
- `id` (required): Identifier of the connector within the EVSE
- `standard` (required): The standard of the installed connector
- `format` (required): The format (socket/cable) of the connector
- `power_type` (required): Type of power supported by this connector
- `max_voltage` (optional): Maximum voltage supported by this connector
- `max_amperage` (optional): Maximum amperage supported by this connector
- `max_electric_power` (optional): Maximum electric power supported by this connector in Watts
- `tariff_ids` (optional): List of tariffs applicable to this connector

**Enums:**
- Location `parking_type`: ON_STREET, PARKING_GARAGE, PARKING_LOT, ON_DRIVEWAY, UNDERGROUND_GARAGE (ParkingType)
- Location `facilities`: RESTAURANT, TOILETS, SHOPPING, SUPERMARKET, HOTEL, SPORTS (Facility)
- EVSE `parking_restrictions`: CUSTOMERS, EMSP_CUSTOMERS, MOTORCYCLES, DISABLED, ELECTRIC_VEHICLES_ONLY, CARSHARING, PLUGGED, OTHER (ParkingRestriction)
- EVSE `status`: AVAILABLE, CHARGING, INOPERATIVE, OUTOFORDER, PLANNED, REMOVED, RESERVED, UNKNOWN (EVSEStatus)
- Connector `standard`: IEC_62196_T2, CHADEMO, IEC_62196_T2_COMBO, DOMESTIC_D, etc. (ConnectorType)
- Connector `format`: SOCKET, CABLE (ConnectorFormat)
- Connector `power_type`: AC_1_PHASE, AC_3_PHASE, DC (PowerType)

#### 1.2 GET /ocpi/2.2/locations/{location_id}
Request: path param `location_id`.

Response: same Location object as above (single object in `data`).

---

### 2. Tariffs API

#### 2.1 GET /ocpi/2.2/tariffs?offset=0&limit=20
Response (excerpt):
```json
{
  "data": [
    {
      "id": "tariff-01",
      "currency": "INR",
      "elements": [
        {
          "price_components": [
            { "type": "ENERGY", "price": 18.0, "step_size": 1 }
          ]
        }
      ],
      "last_updated": "2024-12-10T06:53:43Z"
    }
  ],
  "status_code": 1000,
  "status_message": "Success",
  "timestamp": "2025-08-14T09:34:00Z"
}
```

**Field Descriptions:**

**Tariff Object:**
- `id` (required): Unique identifier for this tariff assigned by the CPO
- `currency` (required): ISO 4217 code of the currency of this tariff
- `elements` (required): List of tariff elements that define the pricing structure
- `last_updated` (required): Timestamp when this tariff was last updated

**TariffElement Object:**
- `price_components` (required): List of price components that make up the pricing of this tariff

**PriceComponent Object:**
- `type` (required): Type of tariff dimension (what is being charged for)
- `price` (required): Price per unit (excluding VAT) for this tariff dimension
- `step_size` (optional): Minimum amount to be billed. This unit will be billed in this step_size blocks

**Enums:**
- `price_components[].type`: ENERGY, TIME, PARKING_TIME, FLAT, MIN_CURRENT, MAX_CURRENT, MIN_POWER, MAX_POWER, RESERVATION (TariffDimensionType)

#### 2.2 GET /ocpi/2.2/tariffs/{tariff_id}
Response: single Tariff object in `data`.

---

### 3. Commands API

Pattern: Asynchronous commands initiated by eMSP to CPO. Immediate response indicates acceptance; execution result is sent to `response_url` callback.

#### 3.1 POST /ocpi/2.2/commands/RESERVE_NOW
Request:
```json
{
  "response_url": "https://emsp.example.com/ocpi/emsp/2.2/commands/RESERVE_NOW/cb_123",
  "token": {
    "country_code": "IN",
    "party_id": "EMSP",
    "uid": "driver_token_123",
    "type": "APP_USER"
  },
  "location_id": "LOC-DELHI-001",
  "evse_uid": "EVSE-001",
  "connector_id": "1",
  "reservation_id": "res_12345",
  "expiry_date": "2025-08-29T16:00:00Z",
  "authorization_reference": "auth_ref_789"
}
```

**Field Descriptions:**
- `response_url` (required): URL that the command result will be POSTed to by the CPO
- `token` (required): Token object containing driver identification and authorization details
- `token.country_code` (required): ISO 3166-1 alpha-2 country code of the token issuer
- `token.party_id` (required): ID of the party that issued this token
- `token.uid` (required): Unique identifier of the token
- `token.type` (required): Type of the token (RFID, APP_USER, OTHER)
- `location_id` (required): Location identifier of the charging location
- `evse_uid` (required): EVSE identifier of the charging point to reserve
- `connector_id` (optional): Connector identifier, required when evse_uid is not unique
- `reservation_id` (optional): Reservation identifier for this reservation
- `expiry_date` (optional): The date and time when the reservation expires
- `authorization_reference` (optional): Reference to the authorization given by the eMSP

Immediate response (CommandResponse):
```json
{
  "data": { "result": "ACCEPTED", "timeout": 30 },
  "status_code": 1000,
  "status_message": "Command accepted",
  "timestamp": "2025-08-29T10:15:00Z"
}
```

**Field Descriptions:**
- `result` (required): Result of the command request
- `timeout` (optional): Timeout for this command in seconds

**Enums:**
- `token.type`: RFID, APP_USER, OTHER (TokenType)
- `data.result`: ACCEPTED, REJECTED, UNKNOWN, NOT_SUPPORTED (CommandResult)

#### 3.2 POST /ocpi/2.2/commands/START_SESSION
Request:
```json
{
  "response_url": "https://emsp.example.com/ocpi/emsp/2.2/commands/START_SESSION/cb_456",
  "token": { "country_code": "IN", "party_id": "EMSP", "uid": "driver_token_123", "type": "APP_USER" },
  "location_id": "LOC-DELHI-001",
  "evse_uid": "EVSE-001",
  "connector_id": "1",
  "authorization_reference": "auth_ref_789"
}
```

**Field Descriptions:**
- Same as RESERVE_NOW, except `reservation_id` and `expiry_date` are not applicable

Response: same CommandResponse pattern as above.

#### 3.3 POST /ocpi/2.2/commands/STOP_SESSION
Request:
```json
{
  "response_url": "https://emsp.example.com/ocpi/emsp/2.2/commands/STOP_SESSION/cb_789",
  "session_id": "ocpi_session_456"
}
```

**Field Descriptions:**
- `response_url` (required): URL that the command result will be POSTed to by the CPO
- `session_id` (required): Session identifier of the session to be stopped

Response: same CommandResponse pattern as above.

---

### 4. Sessions API

Usage patterns:
- eMSP pulls sessions from CPO: `GET /ocpi/2.2/sessions?date_from=...&date_to=...&offset=...&limit=...`
- CPO pushes session updates to eMSP receiver: `PUT /ocpi/emsp/2.2/sessions/{session_id}`

#### 4.1 GET /ocpi/2.2/sessions?offset=0&limit=20
Response (excerpt):
```json
{
  "data": [
    {
      "id": "ocpi_session_456",
      "start_date_time": "2025-08-29T10:30:00Z",
      "end_date_time": null,
      "kwh": 3.2,
      "auth_id": "driver_token_123",
      "auth_method": "AUTH_REQUEST",
      "location_id": "LOC-DELHI-001",
      "evse_uid": "EVSE-001",
      "connector_id": "1",
      "meter_id": "MTR-0001",
      "currency": "INR",
      "total_cost": null,
      "last_updated": "2025-08-29T10:31:00Z"
    }
  ],
  "status_code": 1000,
  "status_message": "Success",
  "timestamp": "2025-08-29T10:31:30Z"
}
```

**Field Descriptions:**

**Session Object:**
- `id` (required): Unique identifier for this charging session
- `start_date_time` (required): ISO 8601 date and time when the session was started
- `end_date_time` (optional): ISO 8601 date and time when the session was completed
- `kwh` (required): How many kWh were charged in this session (0 if session is ongoing)
- `auth_id` (required): Identifier that was used by the Charge Point to identify the driver
- `auth_method` (required): Method used for authentication
- `location_id` (required): Location identifier where this session took place
- `evse_uid` (required): EVSE identifier where this session took place
- `connector_id` (required): Connector identifier where this session took place
- `meter_id` (optional): Optional identification of the kWh meter
- `currency` (optional): ISO 4217 code of the currency of this session
- `total_cost` (optional): Total cost of this session (excluding VAT)
- `last_updated` (required): Timestamp when this session was last updated

**Enums:**
- `auth_method`: AUTH_REQUEST, COMMAND, WHITELIST (AuthMethod)

#### 4.2 PUT /ocpi/emsp/2.2/sessions/{session_id} (Receiver – CPO → eMSP)
Request body (Session object): same fields as above; `end_date_time`, `total_cost` filled on stop.
Response: OCPI response wrapper with acknowledgment.

---

### 5. CDRs API

Usage patterns:
- eMSP pulls final records: `GET /ocpi/2.2/cdrs?date_from=...&date_to=...&offset=...&limit=...`
- CPO pushes to eMSP receiver: `PUT /ocpi/emsp/2.2/cdrs/{cdr_id}`

#### 5.1 GET /ocpi/2.2/cdrs?offset=0&limit=20
Response (excerpt):
```json
{
  "data": [
    {
      "id": "cdr_abc123",
      "start_date_time": "2025-08-29T10:30:00Z",
      "end_date_time": "2025-08-29T11:05:00Z",
      "cdr_token": { "uid": "driver_token_123", "type": "APP_USER", "contract_id": "GCN-IND-00001" },
      "auth_method": "AUTH_REQUEST",
      "authorization_reference": "auth_ref_789",
      "location_id": "LOC-DELHI-001",
      "evse_uid": "EVSE-001",
      "connector_id": "1",
      "currency": "INR",
      "total_cost": 112.0,
      "total_energy": 6.2,
      "total_time": 2100,
      "charging_periods": [
        {
          "start_date_time": "2025-08-29T10:30:00Z",
          "dimensions": [
            { "type": "ENERGY", "volume": 6.2 }
          ]
        }
      ],
      "tariffs": [ { "id": "tariff-01" } ],
      "last_updated": "2025-08-29T11:06:00Z"
    }
  ],
  "status_code": 1000,
  "status_message": "Success",
  "timestamp": "2025-08-29T11:06:30Z"
}
```

**Field Descriptions:**

**CDR Object:**
- `id` (required): Unique identifier for this CDR
- `start_date_time` (required): ISO 8601 date and time when the session was started
- `end_date_time` (required): ISO 8601 date and time when the session was completed
- `cdr_token` (required): Token used to start this charging session
- `cdr_token.uid` (required): Unique identifier of the token
- `cdr_token.type` (required): Type of the token
- `cdr_token.contract_id` (optional): Contract ID this token belongs to
- `auth_method` (required): Method used for authentication
- `authorization_reference` (optional): Reference to the authorization given by the eMSP
- `location_id` (required): Location identifier where this session took place
- `evse_uid` (required): EVSE identifier where this session took place
- `connector_id` (required): Connector identifier where this session took place
- `currency` (required): ISO 4217 code of the currency of this CDR
- `total_cost` (optional): Total cost of this transaction (excluding VAT)
- `total_energy` (required): Total energy charged during this transaction in kWh
- `total_time` (optional): Total time the connector was connected in seconds
- `charging_periods` (optional): List of charging periods that make up this transaction
- `tariffs` (optional): List of tariffs that were used during this transaction
- `last_updated` (required): Timestamp when this CDR was last updated

**ChargingPeriod Object:**
- `start_date_time` (required): Start timestamp of this charging period
- `dimensions` (required): List of relevant values for this charging period

**CdrDimension Object:**
- `type` (required): Type of CDR dimension
- `volume` (required): Volume of the dimension consumed during the period

**Enums:**
- `cdr_token.type`: RFID, APP_USER, OTHER (TokenType)
- `auth_method`: AUTH_REQUEST, COMMAND, WHITELIST (AuthMethod)
- `charging_periods[].dimensions[].type`: ENERGY, TIME, PARKING_TIME, FLAT, etc. (TariffDimensionType)

#### 5.2 PUT /ocpi/emsp/2.2/cdrs/{cdr_id} (Receiver – CPO → eMSP)
Request body: CDR object as above.
Response: OCPI response wrapper with acknowledgment.

---

### Headers & Pagination (Reference)
- Headers to send: `Authorization`, `X-Request-ID`, `X-Correlation-ID`, `Content-Type: application/json`.
- Pagination params: `offset` (default 0), `limit` (party-defined max). Use `Link` header for navigation and `X-Total-Count` for total items.

### Enum References (Quick List)
- EVSEStatus: AVAILABLE, CHARGING, INOPERATIVE, OUTOFORDER, PLANNED, REMOVED, RESERVED, UNKNOWN
- ConnectorType (sample): IEC_62196_T2, IEC_62196_T2_COMBO, CHADEMO, DOMESTIC_D, ...
- ConnectorFormat: SOCKET, CABLE
- PowerType: AC_1_PHASE, AC_3_PHASE, DC
- ParkingType: ON_STREET, PARKING_GARAGE, PARKING_LOT, ON_DRIVEWAY, UNDERGROUND_GARAGE
- ParkingRestriction: CUSTOMERS, EMSP_CUSTOMERS, MOTORCYCLES, DISABLED, ELECTRIC_VEHICLES_ONLY, CARSHARING, PLUGGED, OTHER
- Facility (examples): RESTAURANT, TOILETS, SHOPPING, SUPERMARKET, HOTEL, SPORTS
- TariffDimensionType: ENERGY, TIME, PARKING_TIME, FLAT, MIN_CURRENT, MAX_CURRENT, MIN_POWER, MAX_POWER, RESERVATION
- TokenType: RFID, APP_USER, OTHER
- AuthMethod: AUTH_REQUEST, COMMAND, WHITELIST
- CommandResult: ACCEPTED, REJECTED, UNKNOWN, NOT_SUPPORTED 