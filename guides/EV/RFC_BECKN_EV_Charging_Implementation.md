# RFC: Beckn Protocol Implementation for EV Charging

**RFC Number:** BECKN-EV-001  
**Category:** Informational  
**Status:** Draft  
**Version:** 0.1.0  
**Date:** 2025-08-29  
**Updated:** 2025-08-29  
**Authors:** Beckn Protocol EV Charging Working Group  
**Reviewers:** Ravi Prakash, Sujith Nair  
**Supersedes:** None  
**Keywords:** Beckn Protocol, EV Charging, OCPI, eMSP, CPO, BAP, BPP  

## Copyright Notice
Copyright (c) 2025 Beckn Protocol Foundation. All rights reserved.

## Status of This Memo
This is a draft RFC for implementing EV charging using the Beckn Protocol. It provides implementation guidance for network participants to build EV charging applications that integrate with the Beckn network while maintaining compatibility with OCPI standards for CPO communication.

## Abstract

This RFC provides a comprehensive implementation guide for using the Beckn Protocol to implement Electric Vehicle (EV) charging use cases. It defines how consumer-facing applications (BAPs) can discover, book, and manage EV charging sessions through e-Mobility Service Providers (eMSPs) acting as BPPs, while eMSPs communicate with Charge Point Operators (CPOs) using the OCPI protocol. The document covers the complete flow from discovery to fulfillment, including message schemas, API specifications, and implementation best practices.

## Table of Contents

1. [Introduction](#1-introduction)
2. [Conventions and Terminology](#2-conventions-and-terminology)
3. [Background and Context](#3-background-and-context)
4. [Architecture Overview](#4-architecture-overview)
5. [Implementation Guide](#5-implementation-guide)
6. [Message Flows](#6-message-flows)
7. [API Specifications](#7-api-specifications)
8. [Best Practices](#8-best-practices)
9. [Examples](#9-examples)
10. [References](#10-references)
11. [Appendix](#11-appendix)

## 1. Introduction

### 1.1 Purpose

This RFC provides implementation guidance for network participants to build EV charging applications using the Beckn Protocol. It defines the integration patterns between consumer applications (BAPs), e-Mobility Service Providers (eMSPs) acting as BPPs, and Charge Point Operators (CPOs) using OCPI protocol.

### 1.2 Scope

This RFC covers:
- Beckn Protocol implementation for EV charging discovery and booking
- Integration between Beckn Protocol and OCPI protocol
- Message flows from discovery to fulfillment
- API specifications and message schemas
- Implementation best practices and examples

**Out of Scope:**
- OCPI protocol implementation details
- CPO infrastructure requirements
- Payment gateway integrations
- Regulatory compliance requirements

### 1.3 Target Audience

- **BAP Developers**: Building consumer-facing EV charging applications
- **eMSP Developers**: Implementing BPP interfaces for EV charging services
- **System Architects**: Designing EV charging network integrations
- **Business Stakeholders**: Understanding the implementation approach

### 1.4 Prerequisites

Readers should have:
- Basic understanding of the Beckn Protocol
- Familiarity with REST APIs and JSON
- Understanding of EV charging concepts
- Knowledge of OCPI protocol basics

## 2. Conventions and Terminology

### 2.1 Keywords

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### 2.2 Terminology

- **BAP (Beckn App Platform)**: Consumer-facing application that initiates transactions
- **BPP (Beckn Provider Platform)**: Service provider platform that responds to BAP requests
- **eMSP (e-Mobility Service Provider)**: Service provider that aggregates multiple CPOs
- **CPO (Charge Point Operator)**: Entity that owns and operates charging infrastructure
- **EVSE (Electric Vehicle Supply Equipment)**: Individual charging station unit
- **OCPI (Open Charge Point Interface)**: Protocol for communication between eMSPs and CPOs

## 3. Background and Context

### 3.1 Problem Domain

The EV charging ecosystem faces several challenges:
- **Fragmentation**: Multiple CPOs with different systems and protocols
- **User Experience**: Consumers need to use multiple apps for different charging networks
- **Interoperability**: Lack of standardized communication between different stakeholders
- **Discovery**: Difficulty in finding available charging stations across networks

### 3.2 Current State

Currently, EV charging is implemented through:
- **Direct CPO Apps**: Limited to specific charging networks
- **Aggregator Apps**: Limited integration capabilities
- **OCPI Protocol**: Standard for eMSP-CPO communication but lacks consumer interface

### 3.3 Motivation

The Beckn Protocol provides:
- **Unified Consumer Experience**: Single app for multiple charging networks
- **Standardized Communication**: Consistent API patterns across the network
- **Network Effects**: Value increases with more participants
- **Open Standards**: Vendor-neutral implementation approach

## 4. Architecture Overview

### 4.1 System Components

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Consumer  │    │     eMSP    │    │     CPO     │
│    (BAP)    │◄───Beckn Protocol──►│    (BPP)    │◄───OCPI Protocol──►│   (OCPI)    │
└─────────────┘    └─────────────┘    └─────────────┘
```

### 4.2 Data Flow

1. **Discovery**: BAP searches for charging stations through eMSP BPP
2. **Selection**: BAP selects charging station and receives catalog
3. **Order**: BAP initiates booking through eMSP BPP
4. **Fulfillment**: eMSP communicates with CPO using OCPI
5. **Completion**: Session completion and billing through OCPI

## 5. Implementation Guide

### 5.1 Overview

The implementation involves three main components:
1. **BAP Implementation**: Consumer application using Beckn Protocol
2. **BPP Implementation**: eMSP platform implementing Beckn Protocol
3. **OCPI Integration**: Communication with CPOs using OCPI protocol

### 5.2 Prerequisites

- Beckn Protocol Server/Adapter
- OCPI client implementation
- EV charging domain knowledge
- Network participant registration

### 5.3 Step-by-Step Implementation

#### 5.3.1 Step 1: BAP Implementation

**Input:** Beckn Protocol specification, consumer requirements
**Output:** Consumer application with search and booking capabilities

**Key Components:**
- Search interface for charging stations
- Catalog display with charging options
- Booking and payment integration
- Session management interface

#### 5.3.2 Step 2: BPP Implementation

**Input:** Beckn Protocol specification, eMSP business logic
**Output:** BPP platform handling EV charging requests

**Key Components:**
- Beckn Protocol message handlers
- OCPI client for CPO communication
- Business logic for charging aggregation
- User and session management

#### 5.3.3 Step 3: OCPI Integration

**Input:** OCPI specification, CPO endpoints
**Output:** Communication layer with CPOs

**Key Components:**
- OCPI client implementation
- Token management and authorization
- Real-time status updates
- Billing and session management

## 6. Message Flows

### 6.1 Discovery Flow

```
BAP ──search──► BPP ──OCPI locations──► CPO
BAP ──search──► BPP ──OCPI tariffs──► CPO
BAP ◄─on_search── BPP ◄─OCPI response── CPO
```

### 6.2 Selection Flow

```
BAP ──select──► BPP ──OCPI locations (narrow)──► CPO
BAP ──select──► BPP ──OCPI tariffs (narrow)──► CPO
BAP ◄─on_select── BPP ◄─OCPI response── CPO
```

### 6.3 Order Flow

```
BAP ──init──► BPP ──OCPI commands──► CPO
BAP ◄─on_init── BPP ◄─OCPI response── CPO
```

### 6.4 Fulfillment Flow

```
BAP ──confirm──► BPP ──OCPI session──► CPO
BAP ◄─on_confirm── BPP ◄─OCPI response── CPO
```

## 7. API Specifications

### 7.1 Beckn Protocol Messages

#### 7.1.1 Search Message

```json
{
  "context": {
    "domain": "ev-charging:beckn",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://example-bpp.com",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2025-08-29T10:00:00Z"
  },
  "message": {
    "intent": {
      "fulfillment": {
        "type": "CHARGING",
        "stops": [
          {
            "location": {
              "gps": "28.7041,77.1025",
              "address": {
                "full": "Delhi, India"
              }
            }
          }
        ]
      }
    }
  }
}
```

#### 7.1.2 On Search Response

```json
{
  "context": {
    "domain": "ev-charging:beckn",
    "action": "on_search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://example-bpp.com",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2025-08-29T10:00:00Z"
  },
  "message": {
    "catalog": {
      "providers": [
        {
          "id": "cpo1.com",
          "descriptor": {
            "name": "CPO1 EV charging Company",
            "short_desc": "CPO1 provides EV charging facility across India",
            "images": [
              {
                "url": "https://cpo1.com/images/logo.png"
              }
            ]
          },
          "categories": [
            {
              "id": "category-gt",
              "descriptor": {
                "code": "green-tariff",
                "name": "green tariff"
              }
            }
          ],
          "locations": [
            {
              "id": "LOC-DELHI-001",
              "gps": "28.345345,77.389754",
              "descriptor": {
                "name": "BlueCharge Connaught Place Station"
              },
              "address": "Connaught Place, New Delhi"
            },
            {
              "id": "LOC-DELHI-002",
              "gps": "28.247934,77.876987",
              "descriptor": {
                "name": "BlueCharge Saket Station"
              },
              "address": "824 Saket, New Delhi"
            }
          ],
          "fulfillments": [
            {
              "id": "fulfillment-001",
              "type": "CHARGING",
              "stops": [
                {
                  "location": {
                    "gps": "28.6304,77.2177",
                    "address": {
                      "full": "Connaught Place, New Delhi"
                    }
                  }
                }
              ]
            },
            {
              "id": "fulfillment-002",
              "type": "CHARGING",
              "stops": [
                {
                  "location": {
                    "gps": "28.6310,77.2200",
                    "address": {
                      "full": "Saket, New Delhi"
                    }
                  }
                }
              ]
            }
          ],
          "items": [
            {
              "id": "pe-charging-01",
              "descriptor": {
                "name": "EV Charger #1 (AC Fast Charger)",
                "code": "energy"
              },
              "price": {
                "value": "18",
                "currency": "INR/kWH"
              },
              "fulfillment_ids": [
                "fulfillment-001"
              ],
              "category_ids": [
                "category-gt"
              ],
              "location_ids": [
                "LOC-DELHI-001"
              ],
              "tags": [
                {
                  "descriptor": {
                    "code": "connector-specifications",
                    "name": "Connector Specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "connector Id",
                        "code": "connector-id"
                      },
                      "value": "con1"
                    },
                    {
                      "descriptor": {
                        "name": "Power Type",
                        "code": "power-type"
                      },
                      "value": "AC_3_PHASE"
                    },
                    {
                      "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                      },
                      "value": "CCS2"
                    },
                    {
                      "descriptor": {
                        "name": "Charging Speed",
                        "code": "charging-speed"
                      },
                      "value": "FAST"
                    },
                    {
                      "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                      },
                      "value": "30kW"
                    },
                    {
                      "descriptor": {
                        "name": "Status",
                        "code": "status"
                      },
                      "value": "Available"
                    }
                  ]
                }
              ]
            },
            {
              "id": "pe-charging-02",
              "descriptor": {
                "name": "EV Charger #2 (DC Fast Charger)",
                "code": "energy"
              },
              "price": {
                "value": "25",
                "currency": "INR/kWH"
              },
              "fulfillment_ids": [
                "fulfillment-002"
              ],
              "category_ids": [
                "category-gt"
              ],
              "location_ids": [
                "LOC-DELHI-002"
              ],
              "tags": [
                {
                  "descriptor": {
                    "name": "Connector Specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "connector 1",
                        "code": "connector-id"
                      },
                      "value": "con4"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  }
}
```

### 7.2 OCPI Integration Messages

#### 7.2.1 OCPI Locations Request

```http
GET /ocpi/cpo/2.2/locations?offset=0&limit=100
Authorization: Token {cpo_api_token}
```

#### 7.2.2 OCPI Tariffs Request

```http
GET /ocpi/cpo/2.2/tariffs?offset=0&limit=100
Authorization: Token {cpo_api_token}
```

#### 7.2.3 OCPI Commands Request

```http
POST /ocpi/cpo/2.2/commands/RESERVE_NOW
Authorization: Token {cpo_api_token}
Content-Type: application/json

{
  "response_url": "https://emsp-provider.com/ocpi/emsp/2.2/commands/RESERVE_NOW/callback",
  "token": {
    "country_code": "IN",
    "party_id": "EMSP",
    "uid": "driver_token_123"
  },
  "location_id": "LOC-DELHI-001",
  "evse_uid": "evse-001"
}
```

## 8. Best Practices

### 8.1 Beckn Protocol Implementation

- **Message Validation**: Always validate incoming messages against Beckn schema
- **Error Handling**: Implement proper error responses with meaningful messages
- **Transaction Management**: Maintain transaction state throughout the flow
- **Rate Limiting**: Implement appropriate rate limiting for API endpoints

### 8.2 OCPI Integration

- **Token Management**: Implement proper token lifecycle management
- **Real-time Updates**: Use OCPI push model for real-time status updates
- **Error Recovery**: Implement retry mechanisms for failed OCPI calls
- **Data Synchronization**: Maintain consistency between Beckn and OCPI data

### 8.3 Security Considerations

- **Authentication**: Implement proper authentication for all API endpoints
- **Authorization**: Validate user permissions for all operations
- **Data Privacy**: Ensure sensitive user data is properly protected
- **API Security**: Use HTTPS and implement proper input validation

## 9. Examples

### 9.1 Complete Search Flow

**BAP Search Request:**
```json
{
  "context": {
    "action": "search",
    "domain": "ev-charging:beckn"
  },
  "message": {
    "intent": {
      "fulfillment": {
        "type": "CHARGING",
        "stops": [
          {
            "location": {
              "gps": "28.7041,77.1025"
            }
          }
        ]
      }
    }
  }
}
```

**BPP Processing:**
1. Receive search request
2. Query OCPI locations from multiple CPOs
3. Query OCPI tariffs from multiple CPOs
4. Aggregate and map locations with tariffs
5. Return formatted catalog with pricing

**BAP Response:**
```json
{
  "context": {
    "action": "on_search",
    "domain": "ev-charging:beckn"
  },
  "message": {
    "catalog": {
      "providers": [
        {
          "id": "cpo1.com",
          "descriptor": {
            "name": "CPO1 EV charging Company",
            "short_desc": "CPO1 provides EV charging facility across India"
          },
          "items": [
            {
              "id": "pe-charging-01",
              "descriptor": {
                "name": "EV Charger #1 (AC Fast Charger)",
                "code": "energy"
              },
              "price": {
                "value": "18",
                "currency": "INR/kWH"
              },
              "tags": [
                {
                  "descriptor": {
                    "name": "Connector Specifications",
                    "code": "connector-specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                      },
                      "value": "CCS2"
                    },
                    {
                      "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                      },
                      "value": "30kW"
                    },
                    {
                      "descriptor": {
                        "name": "Status",
                        "code": "status"
                      },
                      "value": "Available"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    }
  }
}
```

### 9.2 Booking Flow

**BAP Init Request:**
```json
{
  "context": {
    "action": "init",
    "domain": "ev-charging:beckn"
  },
  "message": {
    "order": {
      "provider": {
        "id": "cpo1.com"
      },
      "items": [
        {
          "id": "pe-charging-01",
          "quantity": {
            "count": 1
          }
        }
      ]
    }
  }
}
```

**BPP Processing:**
1. Receive init request
2. Validate EVSE availability via OCPI locations
3. Get current pricing via OCPI tariffs
4. Call OCPI RESERVE_NOW command
5. Handle CPO response
6. Return confirmation

## 10. References

### 10.1 Beckn Protocol

- [Beckn Protocol Specification](https://github.com/beckn/protocol-specifications)
- [Beckn Implementation Guide](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md)

### 10.2 OCPI Protocol

- [OCPI 2.2.1 Specification](https://github.com/ocpi/ocpi)
- [OCPI Implementation Guide](https://evroaming.org/ocpi/)

### 10.3 EV Charging Standards

- [IEC 61851](https://webstore.iec.ch/publication/5968) - EV conductive charging system
- [ISO 15118](https://www.iso.org/standard/55366.html) - EV communication interface

## 11. Appendix

### 11.1 Message Flow Diagrams

#### 11.1.1 Complete EV Charging Flow

```
Consumer App (BAP)    eMSP (BPP)    CPO (OCPI)
      │                    │            │
      │─── search ────────►│            │
      │                    │─── GET locations ──►│
      │                    │◄── locations ──────│
      │◄── on_search ─────│            │
      │                    │            │
      │─── select ────────►│            │
      │                    │─── GET tariffs ───►│
      │                    │◄── tariffs ───────│
      │◄── on_select ─────│            │
      │                    │            │
      │─── init ─────────►│            │
      │                    │─── POST RESERVE_NOW ─►│
      │                    │◄── confirmation ───│
      │◄── on_init ───────│            │
      │                    │            │
      │─── confirm ───────►│            │
      │                    │─── POST START_SESSION ─►│
      │                    │◄── session_started ─│
      │◄── on_confirm ────│            │
      │                    │            │
      │                    │◄── PUT session updates ─│
      │                    │            │
      │                    │─── POST STOP_SESSION ─►│
      │                    │◄── session_stopped ─│
      │                    │            │
      │                    │◄── PUT CDR ──────────│
```

### 11.2 Error Handling

#### 11.2.1 Common Error Scenarios

- **CPO Unavailable**: Handle OCPI communication failures
- **Invalid Token**: Manage authentication errors
- **EVSE Unavailable**: Handle booking conflicts
- **Network Issues**: Implement retry mechanisms

#### 11.2.2 Error Response Format

```json
{
  "context": {
    "action": "on_search",
    "domain": "ev-charging:beckn"
  },
  "error": {
    "code": "EVSE_UNAVAILABLE",
    "message": "Selected EVSE is currently unavailable",
    "details": {
      "reason": "MAINTENANCE",
      "estimated_availability": "2024-12-16T10:00:00Z"
    }
  }
}
```

### 11.3 Testing and Validation

#### 11.3.1 Test Scenarios

- **Happy Path**: Complete successful booking flow
- **Error Scenarios**: Handle various failure modes
- **Load Testing**: Test with multiple concurrent users
- **Integration Testing**: Test OCPI communication

#### 11.3.2 Validation Checklist

- [ ] Beckn Protocol message validation
- [ ] OCPI protocol compliance
- [ ] Error handling and recovery
- [ ] Security and authentication
- [ ] Performance and scalability
- [ ] User experience validation

---

*This RFC provides implementation guidance for EV charging using the Beckn Protocol. For questions or clarifications, please contact the Beckn Protocol EV Charging Working Group.* 