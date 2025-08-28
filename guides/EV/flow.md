# EV Charging Session Booking Flow - High-Level Design

## Overview
This document outlines the complete flow for booking and managing an EV charging session, showing the interaction between three layers:
- **UI Layer**: Driver's mobile app or website interface
- **eMSP Layer**: e-Mobility Service Provider backend system
- **CPO Layer**: Charge Point Operator infrastructure

---



## Complete Flow: UI ↔ eMSP ↔ CPO

### 1. UI Search Request - POST /api/charging-stations/search

**Flow:**
1. Driver opens app and searches for charging stations
2. UI calls eMSP backend: `POST /api/charging-stations/search`

**UI Payload:**
```json
{
  "city": "Delhi",
  "latitude": 28.5503906,
  "longitude": 77.2584309,
  "radius": 10,
  "search_type": "city_and_gps"
}
```

**Parameters Passed:**
- **UI → eMSP:** City name, GPS coordinates, search radius
- **eMSP → UI:** Acknowledgment of search request

---

## Data Mapping Process (eMSP Internal)

**Why Data Mapping is Required:**
- **Locations API** returns charging stations with `tariff_ids` (no pricing)
- **Tariffs API** returns pricing with no location information
- **eMSP must combine both** to show complete information to customer

**How eMSP Solves This:**
1. **eMSP receives** search request from UI
2. **eMSP calls each CPO twice:**
   - `GET /ocpi/cpo/2.2/locations` (gets locations with tariff_ids)
   - `GET /ocpi/cpo/2.2/tariffs` (gets pricing information)
3. **eMSP maps** locations to tariffs using tariff_ids
4. **eMSP filters** results by city and GPS radius
5. **eMSP returns** unified catalog with complete pricing to UI

**Example:**
- **Locations data**: "Thunder Test CS" with `tariff_ids: ["abc123"]`
- **Tariffs data**: `"abc123"` = "8.50 INR/kWh + 2.00 INR/hour"
- **eMSP combines**: "Thunder Test CS - 8.50 INR/kWh + 2.00 INR/hour"

---



### 3. GET /ocpi/cpo/2.2/locations - Get Location Details

---

### 4. GET /ocpi/cpo/2.2/tariffs - Get Pricing Information

**Flow:**
1. eMSP calls CPO: `GET /ocpi/cpo/2.2/tariffs?offset=0&limit=100`
2. CPO returns all available tariffs
3. eMSP processes tariff data for mapping

**Parameters Passed:**
- **eMSP → CPO:** Authorization token, pagination parameters
- **CPO → eMSP:** Array of tariff objects with pricing elements

---

### 5. eMSP Returns Complete Catalog to UI

**Flow:**
1. eMSP completes data mapping and filtering
2. eMSP returns unified catalog with locations and pricing to UI
3. UI displays charging stations with complete information to driver

**eMSP Response:**
```json
{
  "locations": [
    {
      "name": "Thunder Test CS",
      "city": "Delhi",
      "address": "Test, Delhi",
      "distance": "2.3 km",
      "evses": [
        {
          "uid": "tLAj42jrr0dYtDqdqTDK_1",
          "status": "AVAILABLE",
          "connectors": [
            {
              "id": "1",
              "standard": "IEC_62196_T2",
              "pricing": {
                "energy_rate": "8.50 INR/kWh",
                "time_rate": "2.00 INR/hour",
                "estimated_cost": "150-200 INR for full charge"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

**Parameters Passed:**
- **eMSP → UI:** Complete catalog with locations, EVSEs, and pricing
- **UI → Driver:** Displays charging stations with costs for selection

---

### 6. Driver Selects EVSE and Views Details

**Flow:**
1. Driver selects one EVSE from the location catalog
2. eMSP performs fresh narrow search for real-time status
3. eMSP fetches latest EVSE details and pricing
4. eMSP displays complete EVSE information to driver

**eMSP Actions:**
1. **Fresh EVSE Status Check:**
   ```http
   GET https://api.cpo1.com/ocpi/2.2/locations/LOC-DELHI-001/EVSE-001
   ```
   - Get real-time EVSE availability
   - Check current status (AVAILABLE, CHARGING, RESERVED, etc.)
   - Verify connector status and technical details

2. **Specific Tariff Fetch:**
   ```http
   GET https://api.cpo1.com/ocpi/2.2/tariffs/tariff_abc123
   GET https://api.cpo1.com/ocpi/2.2/tariffs/tariff_def456
   ```
   - Fetch specific tariffs by ID from EVSE response
   - Get exact pricing for this EVSE's connectors
   - More efficient than fetching all tariffs

**What Driver Sees:**
- **Real-time availability** status
- **Current pricing** (energy rate, time rate, connection fees)
- **Technical details** (connector types, power ratings)
- **Estimated costs** for their charging session
- **Booking options** if EVSE is available

**Benefits of Narrow Search:**
- ✅ **Faster response** - targeted data fetch
- ✅ **Real-time accuracy** - latest status and pricing
- ✅ **Better UX** - driver sees current information
- ✅ **Efficient** - only fetch what's needed

---

### 7. UI Booking Request - POST /api/book-evse

**Flow:**
1. Driver clicks "Book Now" with selected time/duration
2. UI calls eMSP: `POST /api/book-evse`

**UI Booking Payload:**
```json
{
  "evse_uid": "tLAj42jrr0dYtDqdqTDK_1",
  "connector_id": "1",
  "location_id": "HrKNUxJ9Ara6AL6PaUlF",
  "booking_date": "2024-12-15",
  "start_time": "14:00",
  "end_time": "16:00",
  "duration_hours": 2,
  "driver_token": "driver_token_123",
  "estimated_cost": "150-200 INR",
  "vehicle_info": {
    "make": "Tesla",
    "model": "Model 3",
    "battery_capacity": "75 kWh"
  }
}
```

3. eMSP generates reservation_id and calls CPO

#### OCPI Limitation - No Future Time Slot Support
- **OCPI RESERVE_NOW** does NOT support future time slot booking
- **Only supports** immediate reservation with expiry_date
- **eMSP must manage** time slot availability internally
- **eMSP sends OCPI command** when time slot actually starts

4. eMSP calls CPO: `POST /ocpi/cpo/2.2/commands/RESERVE_NOW`

**Important Note - Driver Token Source:**
- **eMSP creates and stores** driver token during registration
- **eMSP uses stored token** directly (no need to call CPO tokens API)
- **CPO validates token** by calling eMSP's tokens endpoint

**eMSP → CPO Payload:**
```json
{
  "response_url": "https://emsp-app.com/ocpi/emsp/2.2/commands/RESERVE_NOW/booking_123",
  "token": {
    "country_code": "IN",
    "party_id": "EMSP",
    "uid": "driver_token_123",
    "type": "APP_USER",
    "valid": true,
    "whitelist": "ALLOWED"
  },
  "expiry_date": "2024-12-15T16:00:00Z",
  "reservation_id": "res_12345",
  "location_id": "HrKNUxJ9Ara6AL6PaUlF",
  "evse_uid": "tLAj42jrr0dYtDqdqTDK_1"
}
```

5. **Two-Stage Token Authorization (Recommended OCPI Pattern)**

**Stage 1: eMSP Pre-Authorizes Token**
```http
# eMSP calls CPO to pre-authorize driver token
POST https://api.cpo1.com/ocpi/2.2/tokens/IN/EMSP/driver_token_123/authorize?type=APP_USER
Authorization: Token {cpo_api_token}
Content-Type: application/json

{
  "location_id": "HrKNUxJ9Ara6AL6PaUlF",
  "evse_uids": ["tLAj42jrr0dYtDqdqTDK_1"]
}
```

**CPO Response with Authorization Code:**
```json
{
  "data": {
    "allowed": "ALLOWED",
    "authorization_reference": "auth_ref_789",
    "token": {
      "country_code": "IN",
      "party_id": "EMSP",
      "uid": "driver_token_123",
      "type": "APP_USER",
      "valid": true,
      "whitelist": "ALLOWED"
    },
    "location": {
      "location_id": "HrKNUxJ9Ara6AL6PaUlF",
      "evse_uids": ["tLAj42jrr0dYtDqdqTDK_1"]
    }
  }
}
```

**Stage 2: eMSP Uses Authorization Code in Commands**
```javascript
// eMSP includes authorization_reference for faster processing
POST /ocpi/2.2/commands/RESERVE_NOW
{
  "response_url": "https://emsp-app.com/ocpi/emsp/2.2/commands/RESERVE_NOW/booking_123",
  "token": {
    "country_code": "IN",
    "party_id": "EMSP",
    "uid": "driver_token_123"
  },
  "authorization_reference": "auth_ref_789",  // ← Pre-authorized code
  "expiry_date": "2024-12-15T16:00:00Z",
  "reservation_id": "res_12345",
  "location_id": "HrKNUxJ9Ara6AL6PaUlF",
  "evse_uid": "tLAj42jrr0dYtDqdqTDK_1"
}
```

**Benefits of Two-Stage Authorization:**
- ✅ **Faster Commands** - CPO uses cached authorization (no re-validation)
- ✅ **Better Performance** - Single authorization for multiple commands
- ✅ **Reduced Latency** - No network calls for token validation on each command
- ✅ **Improved UX** - Driver experiences faster response times

**OCPI Endpoint Structure:**
```
POST /ocpi/2.2/tokens/{country_code}/{party_id}/{token_uid}/authorize[?type={token_type}]
```

**Authorization Reference Usage:**
- **`authorization_reference`** is returned by CPO during pre-authorization
- **eMSP includes this reference** in subsequent commands
- **CPO uses cached authorization** for faster command processing
- **Enables audit trail** for compliance and billing tracking

**Parameters Passed:**
- **eMSP → CPO:** 
  - response_url (eMSP's callback URL)
  - token (driver's authorization token)
  - authorization_reference (from pre-authorization)
  - expiry_date (reservation end time)
  - reservation_id (unique booking identifier)
  - location_id (charging location)
  - evse_uid (specific EVSE)
- **CPO → eMSP:** Command acceptance response
- **CPO → eMSP (async):** Command execution result

**Note:** The same `authorization_reference` can be used in subsequent commands (START_SESSION, STOP_SESSION) for the same charging session, eliminating the need for repeated token validation.

---

### 8. POST /ocpi/cpo/2.2/commands/START_SESSION - Start Charging

**Flow:**
1. Driver arrives at location and connects car
2. Driver authenticates (RFID, app, etc.)
3. UI calls eMSP: `POST /api/start-charging`
4. eMSP calls CPO: `POST /ocpi/cpo/2.2/commands/START_SESSION`
5. CPO starts charging session and responds
6. CPO sends async result to eMSP
7. UI shows charging session started

**Parameters Passed:**
- **eMSP → CPO:**
  - response_url (eMSP's callback URL)
  - token (driver's authorization token)
  - location_id (charging location)
  - evse_uid (specific EVSE)
  - connector_id (specific connector)
- **CPO → eMSP:** Command acceptance response
- **CPO → eMSP (async):** Command execution result

---

### 9. PUT /ocpi/emsp/2.2/sessions/{session_id} - Receive Session Updates

**Flow:**
1. CPO continuously monitors charging session
2. CPO pushes session updates to eMSP
3. CPO calls eMSP: `PUT /ocpi/emsp/2.2/sessions/{session_id}`
4. eMSP receives real-time session data
5. eMSP pushes updates to UI via WebSocket/SSE
6. UI displays real-time charging progress

**Parameters Passed:**
- **CPO → eMSP:**
  - session_id (unique session identifier)
  - current_kwh (energy consumed)
  - current_cost (cost so far)
  - status (ACTIVE, COMPLETED, etc.)
  - last_updated (timestamp)
- **eMSP → UI:** Real-time session updates

---

### 10. POST /ocpi/cpo/2.2/commands/STOP_SESSION - Stop Charging

**Flow:**
1. Driver disconnects car or stops via app
2. UI calls eMSP: `POST /api/stop-charging`
3. eMSP calls CPO: `POST /ocpi/cpo/2.2/commands/STOP_SESSION`
4. CPO stops charging session and responds
5. CPO sends async result to eMSP
6. UI shows session ended

**Parameters Passed:**
- **eMSP → CPO:**
  - response_url (eMSP's callback URL)
  - session_id (session to stop)
- **CPO → eMSP:** Command acceptance response
- **CPO → eMSP (async):** Command execution result

---

### 11. PUT /ocpi/emsp/2.2/cdrs/{cdr_id} - Receive Billing Data

**Flow:**
1. CPO generates Charging Data Record (CDR)
2. CPO pushes CDR to eMSP
3. CPO calls eMSP: `PUT /ocpi/emsp/2.2/cdrs/{cdr_id}`
4. eMSP receives complete billing information
5. eMSP processes billing and sends to UI
6. UI displays final receipt and cost breakdown

**Parameters Passed:**
- **CPO → eMSP:**
  - cdr_id (unique billing record identifier)
  - session_id (related session)
  - total_kwh (total energy consumed)
  - total_cost (final cost with VAT)
  - charging_periods (detailed time breakdown)
  - tariff_used (pricing structure applied)
- **eMSP → UI:** Complete billing summary and receipt

---

## Key Integration Points

### Real-Time Communication
- **WebSocket/SSE** for UI updates during charging
- **Async callbacks** for command results
- **Push notifications** for session status changes

### Data Flow
- **UI** handles user interaction and display
- **eMSP** orchestrates business logic and CPO communication
- **CPO** executes physical charging operations

### Error Handling
- **Network failures** between eMSP and CPO
- **Command rejections** by CPO
- **Session timeouts** and failures
- **Billing discrepancies** and resolution

---

## Business Flow Summary

1. **Discovery**: Driver finds charging station and sees pricing
2. **Booking**: Driver reserves EVSE for specific time
3. **Arrival**: Driver arrives and starts charging session
4. **Monitoring**: Real-time updates during charging
5. **Completion**: Driver stops and receives final billing
6. **Settlement**: eMSP bills driver and pays CPO

---

## OCPI Protocol Benefits

- **Standardized communication** between eMSP and CPO
- **Real-time updates** for better user experience
- **Seamless roaming** across different charging networks
- **Automated billing** and settlement
- **Clear separation** of responsibilities between parties 