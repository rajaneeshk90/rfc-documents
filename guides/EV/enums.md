# OCPI 2.2.1 Enums Reference

This document contains all enum types and their allowed values from the OCPI 2.2.1 specification, organized by module for easy reference.

---

## 6. Credentials & Registration Module

### 6.2.3. InterfaceRole enum
| Value | Description |
|-------|-------------|
| **SENDER** | Sender Interface implementation. Interface implemented by the owner of data, so the Receiver can Pull information from the data Sender/owner. |
| **RECEIVER** | Receiver Interface implementation. Interface implemented by the receiver of data, so the Sender/owner can Push information to the Receiver. |

### 6.2.4. ModuleID enum
| Module | ModuleID | Remark |
|--------|----------|---------|
| CDRs | `cdrs` | |
| Charging Profiles | `chargingprofiles` | |
| Commands | `commands` | |
| Credentials & Registration | `credentials` | **Required for all implementations.** The role field has no function for this module. |
| Hub Client Info | `hubclientinfo` | |
| Locations | `locations` | |
| Sessions | `sessions` | |
| Tariffs | `tariffs` | |
| Tokens | `tokens` | |

**Note:** Parties are allowed to create custom modules. Custom moduleIDs should use a prefix (e.g., country-code + party-id) to avoid conflicts with future OCPI modules.

### 6.2.5. VersionNumber enum
| Value | Description |
|-------|-------------|
| `2.0` | OCPI version 2.0 |
| `2.1` | OCPI version 2.1 (DEPRECATED, do not use, use 2.1.1 instead) |
| `2.1.1` | OCPI version 2.1.1 |
| `2.2` | OCPI version 2.2 (DEPRECATED, do not use, use 2.2.1 instead) |
| `2.2.1` | OCPI version 2.2.1 (this version) |

---

## 8. Locations Module

### 8.4.3. Capability enum
| Value | Description |
|-------|-------------|
| **CHARGING_PREFERENCES_CAPABLE** | The EVSE supports charging preferences. |
| **CREDIT_CARD_PAYABLE** | The EVSE supports credit card payment. |
| **DEBIT_CARD_PAYABLE** | The EVSE supports debit card payment. |
| **PED_TERMINAL** | The EVSE has a Pedestrian/Emergency Door terminal. |
| **REMOTE_START_STOP_CAPABLE** | The EVSE supports remote start/stop. |
| **RESERVABLE** | The EVSE can be reserved. |
| **RFID_READER** | The EVSE has an RFID reader. |
| **UNLOCK_CAPABLE** | The EVSE supports connector unlocking. |

### 8.4.4. ConnectorFormat enum
| Value | Description |
|-------|-------------|
| **CABLE** | The connector is a cable. |
| **SOCKET** | The connector is a socket. |

### 8.4.5. ConnectorType enum
| Value | Description |
|-------|-------------|
| **CHADEMO** | CHAdeMO connector. |
| **DOMESTIC_A** | Domestic outlet, type A, 2 pins. |
| **DOMESTIC_B** | Domestic outlet, type B, 2 pins. |
| **DOMESTIC_C** | Domestic outlet, type C, 2 pins. |
| **DOMESTIC_D** | Domestic outlet, type D, 2 pins. |
| **DOMESTIC_E** | Domestic outlet, type E, 2 pins. |
| **DOMESTIC_F** | Domestic outlet, type F, 2 pins. |
| **DOMESTIC_G** | Domestic outlet, type G, 2 pins. |
| **DOMESTIC_H** | Domestic outlet, type H, 2 pins. |
| **DOMESTIC_I** | Domestic outlet, type I, 2 pins. |
| **DOMESTIC_J** | Domestic outlet, type J, 2 pins. |
| **DOMESTIC_K** | Domestic outlet, type K, 2 pins. |
| **DOMESTIC_L** | Domestic outlet, type L, 2 pins. |
| **DOMESTIC_M** | Domestic outlet, type M, 2 pins. |
| **DOMESTIC_N** | Domestic outlet, type N, 2 pins. |
| **DOMESTIC_O** | Domestic outlet, type O, 2 pins. |
| **IEC_60309_2_single_16** | IEC 60309-2 single phase connector, 16A. |
| **IEC_60309_2_three_16** | IEC 60309-2 three phase connector, 16A. |
| **IEC_60309_2_three_32** | IEC 60309-2 three phase connector, 32A. |
| **IEC_60309_2_three_64** | IEC 60309-2 three phase connector, 64A. |
| **IEC_62196_T1** | IEC 62196 Type 1 connector. |
| **IEC_62196_T1_COMBO** | IEC 62196 Type 1 connector with Combo 2. |
| **IEC_62196_T2** | IEC 62196 Type 2 connector. |
| **IEC_62196_T2_COMBO** | IEC 62196 Type 2 connector with Combo 2. |
| **IEC_62196_T3A** | IEC 62196 Type 3A connector. |
| **IEC_62196_T3C** | IEC 62196 Type 3C connector. |
| **TESLA_R** | Tesla connector. |
| **TESLA_S** | Tesla connector. |

### 8.4.8. EnergySourceCategory enum
| Value | Description |
|-------|-------------|
| **NUCLEAR** | Nuclear energy. |
| **GENERAL_FOSSIL** | General fossil energy. |
| **COAL** | Coal energy. |
| **GAS** | Gas energy. |
| **GENERAL_GREEN** | General green energy. |
| **SOLAR** | Solar energy. |
| **WIND** | Wind energy. |
| **WATER** | Water energy. |

### 8.4.10. EnvironmentalImpactCategory enum
| Value | Description |
|-------|-------------|
| **NUCLEAR_WASTE** | Nuclear waste. |
| **CARBON_DIOXIDE** | Carbon dioxide. |

### 8.4.12. Facility enum
| Value | Description |
|-------|-------------|
| **HOTEL** | Hotel. |
| **RESTAURANT** | Restaurant. |
| **CAFE** | Cafe. |
| **MALL** | Mall. |
| **SUPERMARKET** | Supermarket. |
| **SPORT** | Sport facility. |
| **RECREATION_AREA** | Recreation area. |
| **NATURE** | Nature area. |
| **MUSEUM** | Museum. |
| **BUSINESS** | Business center. |
| **PARKING** | Parking facility. |

### 8.4.16. ImageCategory enum
| Value | Description |
|-------|-------------|
| **CHARGER** | Image of the charger. |
| **ENTRANCE** | Image of the entrance. |
| **LOCATION** | Image of the location. |
| **NETWORK** | Image of the network. |
| **OPERATOR** | Image of the operator. |
| **OTHER** | Other image. |
| **OWNER** | Image of the owner. |

### 8.4.17. ParkingRestriction enum
| Value | Description |
|-------|-------------|
| **EV_ONLY** | Parking only for EVs. |
| **PLUGGED** | Parking only for plugged EVs. |
| **DISABLED** | Parking only for disabled people. |
| **CUSTOMERS** | Parking only for customers. |
| **MOTORCYCLES** | Parking only for motorcycles. |

### 8.4.18. ParkingType enum
| Value | Description |
|-------|-------------|
| **ALONG_MOTORWAY** | Parking along motorway. |
| **PARKING_GARAGE** | Parking garage. |
| **PARKING_LOT** | Parking lot. |
| **ON_DRIVEWAY** | Parking on driveway. |
| **ON_STREET** | Parking on street. |
| **UNDERGROUND_GARAGE** | Underground garage. |

### 8.4.19. PowerType enum
| Value | Description |
|-------|-------------|
| **AC_1_PHASE** | AC single phase. |
| **AC_2_PHASE** | AC two phase. |
| **AC_2_PHASE_SPLIT** | AC two phase split. |
| **AC_3_PHASE** | AC three phase. |
| **DC** | DC. |

### 8.4.22. Status enum
| Value | Description |
|-------|-------------|
| **AVAILABLE** | The EVSE/Connector is able to start a new charging session. |
| **BLOCKED** | The EVSE/Connector is not accessible because of a physical barrier, i.e. a car. |
| **CHARGING** | The EVSE/Connector is in use. |
| **INOPERATIVE** | The EVSE/Connector is not yet active, or temporarily not available for use, but not broken or defect. |
| **OUTOFORDER** | The EVSE/Connector is currently out of order, some part/components may be broken/defect. |
| **PLANNED** | The EVSE/Connector is planned, will be operating soon. |
| **REMOVED** | The EVSE/Connector was discontinued/removed. |
| **RESERVED** | The EVSE/Connector is reserved for a particular EV driver and is unavailable for other drivers. |
| **UNKNOWN** | No status information available (also used when offline). |

---

## 9. Sessions Module

### 9.4.1. ChargingPreferencesResponse enum
| Value | Description |
|-------|-------------|
| **ACCEPTED** | The charging preferences have been accepted. |
| **REJECTED** | The charging preferences have been rejected. |

### 9.4.2. ProfileType enum
| Value | Description |
|-------|-------------|
| **CHEAP** | Cheap charging profile. |
| **FAST** | Fast charging profile. |
| **GREEN** | Green charging profile. |
| **REGULAR** | Regular charging profile. |

### 9.4.3. SessionStatus enum
| Value | Description |
|-------|-------------|
| **ACTIVE** | The session has been accepted and is active. All pre-conditions were met: Communication between EV and EVSE (for example: cable plugged in correctly), EV or driver is authorized. EV is being charged, or can be charged. Energy is, or is not, being transfered. |
| **COMPLETED** | The session has been finished successfully. No more modifications will be made to the Session object using this state. |
| **INVALID** | The Session object using this state is declared invalid and will not be billed. |
| **PENDING** | The session is pending, it has not yet started. Not all pre-conditions are met. This is the initial state. The session might never become an active session. |
| **RESERVATION** | The session is started due to a reservation, charging has not yet started. The session might never become an active session. |

---

## 10. CDRs Module

### 10.4.1. AuthMethod enum
| Value | Description |
|-------|-------------|
| **AUTH_REQUEST** | Authentication request has been sent to the eMSP. |
| **COMMAND** | Command like StartSession or ReserveNow used to start the Session, the Token provided in the Command was used as authorization. |
| **WHITELIST** | Whitelist used for authentication, no request to the eMSP has been performed. |

### 10.4.3. CdrDimensionType enum
| Value | Session Only | Description |
|-------|--------------|-------------|
| **CURRENT** | Y | Average charging current during this ChargingPeriod: defined in A (Ampere). When negative, the current is flowing from the EV to the grid. |
| **ENERGY** | | Total amount of energy (dis-)charged during this ChargingPeriod: defined in kWh. When negative, more energy was feed into the grid then charged into the EV. Default step_size is 1. |
| **ENERGY_EXPORT** | Y | Total amount of energy feed back into the grid: defined in kWh. |
| **ENERGY_IMPORT** | Y | Total amount of energy charged, defined in kWh. |
| **MAX_CURRENT** | | Sum of the maximum current over all phases, reached during this ChargingPeriod: defined in A (Ampere). |
| **MIN_CURRENT** | | Sum of the minimum current over all phases, reached during this ChargingPeriod, when negative, current has flowed from the EV to the grid. Defined in A (Ampere). |
| **MAX_POWER** | | Maximum power reached during this ChargingPeriod: defined in kW (Kilowatt). |
| **MIN_POWER** | | Minimum power reached during this ChargingPeriod: defined in kW (Kilowatt), when negative, the power has flowed from the EV to the grid. |
| **PARKING_TIME** | | Time during this ChargingPeriod not charging: defined in hours, default step_size multiplier is 1 second. |
| **POWER** | Y | Average power during this ChargingPeriod: defined in kW (Kilowatt). When negative, the power is flowing from the EV to the grid. |
| **RESERVATION_TIME** | | Time during this ChargingPeriod Charge Point has been reserved and not yet been in use for this customer: defined in hours, default step_size multiplier is 1 second. |
| **STATE_OF_CHARGE** | Y | Current state of charge of the EV, in percentage, values allowed: 0 to 100. |
| **TIME** | | Time charging during this ChargingPeriod: defined in hours, default step_size multiplier is 1 second. |

---

## 11. Tariffs Module

### 11.4.1. DayOfWeek enum
| Value | Description |
|-------|-------------|
| **MONDAY** | Monday. |
| **TUESDAY** | Tuesday. |
| **WEDNESDAY** | Wednesday. |
| **THURSDAY** | Thursday. |
| **FRIDAY** | Friday. |
| **SATURDAY** | Saturday. |
| **SUNDAY** | Sunday. |

### 11.4.3. ReservationRestrictionType enum
| Value | Description |
|-------|-------------|
| **RESERVATION** | Reservation is required. |
| **RESERVATION_EXPIRES** | Reservation expires. |

### 11.4.5. TariffDimensionType enum
| Value | Description |
|-------|-------------|
| **ENERGY** | Energy consumption. |
| **FLAT** | Flat rate. |
| **PARKING_TIME** | Parking time. |
| **RESERVATION_TIME** | Reservation time. |
| **TIME** | Time consumption. |

### 11.4.7. TariffType enum
| Value | Description |
|-------|-------------|
| **AD_HOC_PAYMENT** | Ad hoc payment. |
| **PROFILE_CHEAP** | Cheap profile. |
| **PROFILE_FAST** | Fast profile. |
| **PROFILE_GREEN** | Green profile. |
| **REGULAR** | Regular tariff. |

---

## 12. Tokens Module

### 12.4.1. AllowedType enum
| Value | Description |
|-------|-------------|
| **ALLOWED** | Token is allowed. |
| **BLOCKED** | Token is blocked. |
| **EXPIRED** | Token has expired. |
| **INVALID** | Token is invalid. |
| **PENDING** | Token is pending. |

### 12.4.4. TokenType enum
| Value | Description |
|-------|-------------|
| **AD_HOC_USER** | One time use Token ID generated by a server (or App.) The eMSP uses this to bind a Session to a customer, probably an app user. |
| **APP_USER** | Token ID generated by a server (or App.) to identify a user of an App. The same user uses the same Token for every Session. |
| **OTHER** | Other type of token. |
| **RFID** | RFID Token. |

**Note:** The eMSP is RECOMMENDED to push Tokens with type: AD_HOC_USER or APP_USER with whitelist set to NEVER. Whitelists are very useful for RFID type Tokens, but the AD_HOC_USER/APP_USER Tokens are used to start Sessions from an App etc. so whitelisting them has no advantages.

### 12.4.5. WhitelistType enum
| Value | Description |
|-------|-------------|
| **ALWAYS** | Token always has to be whitelisted, realtime authorization is not possible/allowed. CPO shall always allow any use of this Token. |
| **ALLOWED** | It is allowed to whitelist the token, realtime authorization is also allowed. The CPO may choose which version of authorization to use. |
| **ALLOWED_OFFLINE** | In normal situations realtime authorization shall be used. But when the CPO cannot get a response from the eMSP (communication between CPO and eMSP is offline), the CPO shall allow this Token to be used. |
| **NEVER** | Whitelisting is forbidden, only realtime authorization is allowed. CPO shall always send a realtime authorization for any use of this Token to the eMSP. |

---

## 13. Commands Module

### 13.4.1. CommandResponseType enum
| Value | Description |
|-------|-------------|
| **ACCEPTED** | Command has been accepted and will be executed. |
| **REJECTED** | Command has been rejected and will not be executed. |

### 13.4.2. CommandResultType enum
| Value | Description |
|-------|-------------|
| **ACCEPTED** | Command has been accepted and executed. |
| **CANCELED_RESERVATION** | Command has been accepted and executed, reservation has been canceled. |
| **EVSE_NOT_FOUND** | Command has been rejected, EVSE not found. |
| **FAILED** | Command has been accepted but execution failed. |
| **NOT_SUPPORTED** | Command has been rejected, not supported. |
| **REJECTED** | Command has been rejected. |
| **TIMEOUT** | Command has been accepted but execution timed out. |
| **UNKNOWN_RESERVATION** | Command has been rejected, reservation not found. |

### 13.4.3. CommandType enum
| Value | Description |
|-------|-------------|
| **CANCEL_RESERVATION** | Cancel a reservation. |
| **RESERVE_NOW** | Reserve an EVSE. |
| **START_SESSION** | Start a charging session. |
| **STOP_SESSION** | Stop a charging session. |
| **UNLOCK_CONNECTOR** | Unlock a connector. |

---

## 14. Charging Profiles Module

### 14.6.2. ChargingRateUnit enum
| Value | Description |
|-------|-------------|
| **A** | Ampere. |
| **W** | Watt. |

### 14.6.5. ChargingProfileResponseType enum
| Value | Description |
|-------|-------------|
| **ACCEPTED** | Charging profile has been accepted. |
| **REJECTED** | Charging profile has been rejected. |

### 14.6.6. ChargingProfileResultType enum
| Value | Description |
|-------|-------------|
| **ACCEPTED** | Charging profile has been accepted and executed. |
| **REJECTED** | Charging profile has been rejected. |

---

## 15. Hub Client Info Module

### 15.5.1. ConnectionStatus enum
| Value | Description |
|-------|-------------|
| **CONNECTED** | Connected. |
| **DISCONNECTED** | Disconnected. |

---

## 16. Registration Module

### 16.5.1. Role enum
| Value | Description |
|-------|-------------|
| **CPO** | Charge Point Operator. |
| **EMSP** | e-Mobility Service Provider. |
| **HUB** | Hub. |
| **NAP** | National Access Point. |
| **OTHER** | Other role. |

---

## Usage Notes

- **Case Sensitivity:** All strings in messages and enumerations are case sensitive, unless explicitly stated otherwise.
- **Custom Values:** Parties are allowed to create custom enum values for specific use cases, but these should be documented and agreed upon between parties.
- **Deprecated Values:** Some enum values may be deprecated in newer versions. Always use the latest recommended values.
- **Extensibility:** The OCPI specification allows for custom modules and enum extensions, but these should use appropriate prefixes to avoid conflicts.

---

*This document is based on OCPI 2.2.1 specification. For the most up-to-date information, refer to the official OCPI documentation.* 




Connector-type
power-type
connector-format
charging-speed
charger-status
fulfillments.state.descriptor.code