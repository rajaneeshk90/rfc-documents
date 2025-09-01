UEI Implementation Guide - EV Charging
Copyright Notice
Copyright (c) 2025 Beckn Protocol Foundation. All rights reserved.
Status of This Memo
This is a draft RFC for implementing EV charging using the Beckn Protocol. It provides implementation guidance for network participants to build EV charging applications that integrate with the Beckn network while maintaining compatibility with OCPI standards for CPO communication.
Abstract
This RFC presents a paradigm shift in electric vehicle charging accessibility by leveraging the Beckn Protocol's distributed commerce architecture to create a unified marketplace for EV charging services. Rather than forcing consumers to manage multiple charging network apps and accounts, this approach enables seamless discovery and booking of charging sessions across any participating Charge Point Operator (CPO) through a single consumer interface.
The architecture transforms EV charging from fragmented network silos to an interoperable ecosystem where consumers can find, compare, and book charging sessions at any participating station regardless of the underlying CPO. This marketplace-driven approach addresses the key barriers to EV adoption—charging anxiety, network fragmentation, and payment complexity—through standardized discovery, transparent pricing, and unified booking across the entire charging infrastructure landscape.
By combining Beckn Protocol's commerce capabilities with OCPI's technical interoperability, this implementation enables e-Mobility Service Providers (eMSPs) to aggregate charging services from multiple CPOs while providing consumers with a consistent, app-agnostic charging experience. The result is a scalable, future-proof foundation for the mass adoption of electric vehicles.
Introduction
This RFC provides comprehensive implementation guidance for deploying EV charging services using the Beckn Protocol ecosystem. It specifically addresses how consumer applications can provide unified access to charging infrastructure across multiple Charge Point Operators while maintaining technical compatibility with existing OCPI-based systems.
Scope
This document covers:
Architecture patterns for EV charging marketplace implementation using Beckn Protocol
Discovery and charging mechanisms for charging EVs across multiple CPOs
Real-time availability and pricing integration with OCPI-based systems
Session management and billing coordination between Beckn and OCPI protocols
This document does not cover:
Detailed OCPI protocol specifications (refer to OCPI 2.2.1 documentation)
Physical charging infrastructure requirements and standards
Regulatory compliance beyond technical implementation (varies by jurisdiction)
Smart grid integration and load management systems
Target Audience
Consumer Application Developers (BAP): Building EV driver-facing charging applications with unified cross-network access
e-Mobility Service Providers (eMSP/BPP): Implementing charging service aggregation platforms across multiple CPO networks
Charge Point Operators (CPO): Understanding integration requirements for Beckn-enabled marketplace participation
Technology Integrators: Building bridges between existing OCPI infrastructure and new Beckn-based marketplaces
System Architects: Designing scalable, interoperable EV charging ecosystems
Business Stakeholders: Understanding technical capabilities and implementation requirements for EV charging marketplace strategies
Standards Organizations: Evaluating interoperability approaches for future EV charging standards development
Conventions and Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.
Terminology
Acronym
Full Form/Description
BAP
Beckn App Platform: Consumer-facing application that initiates transactions
BPP
Beckn Provider Platform: Service provider platform that responds to BAP requests
eMSP
e-Mobility Service Provider: Service provider that aggregates multiple CPOs
CPO
Charge Point Operator: Entity that owns and operates charging infrastructure
EVSE
Electric Vehicle Supply Equipment: Individual charging station unit
OCPI
Open Charge Point Interface: Protocol for communication between eMSPs and CPOs



Use case - Discovery, order and fulfillment of EV Charging
Note: Pre booking a charging time slot is not covered in this use case.
This implementation guide covers a walking case where users discover the charger using third-party apps, word of mouth, or Beckn API, and then drive to the location.
The order flow happens while the user is at the charging station. This is due to the current readiness of the participants.
It is envisioned that a future version of this guide (once the NPs are ready) will add pre-booking of a slot at a charging station.
Use case:
Srilekha is an IT professional who travels for work five days a week. She uses her four-wheeler electric vehicle (EV) for her commute. She prefers to charge her vehicle during her work hours and looks for suitable charging stations near her workplace.
Discovery:
Srilekha identifies a charging station. This discovery can be facilitated through various methods, including Beckn APIs, Google Maps, or word-of-mouth recommendations.
She then proceeds to drive to the aforementioned charging station.
Order:
Srilekha uses her EV charging app to scan the QR code displayed on a charging point(EVSE) at the charging station.
Her EV charging app fetches the details of the charger point. Srilekha goes through the charging point details and proceeds. Srilekha receives terms of the order on her EV charging app.
She accepts the terms of order and is prompted to choose a payment option. She chooses to make payment through UPI, does the payment and places the order.
The provider verifies the payment, confirms the order and generates an order ID.
Fulfillment:
Srilekha plugs the charger into her vehicle and initiates the charging process.
She tracks the charging status on her EV charging app while her EV is getting charged.
She gets a notification on her EV charging app when the charging stops.
Srilekha removes the charger from her vehicle, thus ending the charging process.
Post Fulfilment:
Srilekha rates her experience using a 0-5 star rating.
General Beckn message flow and error handling
This section is relevant to all the message flows illustrated below and discussed further in the document.
Beckn is a synchronous protocol at its core.
When a network participant(NP1) sends a message to another participant(NP2), the other participant(NP2) immediately returns back an ACK/NACK(Acknowledgement or Negative Acknowledgement in case of error - usually with wrongly formed messages).
An ACK is an indicator that the receiving participant(NP2) will process this message and dispatch an on_xxxxxx message to original NP (NP1)
Subsequently after processing the message NP2 sends back the real response in the corresponding on_xxxxxx message, to which again the first participant(NP1).
This message can contain a message field (for success) or error field (for failure)
NP1 when it receives the on_xxxxxx message, sends back an ACK/NACK (Here in both the cases NP1 will not send any subsequent message).
In the Use case diagrams, this ACK/NACK is not illustrated explicitly to keep the diagrams crisp.
However when writing software we should be prepared to receive these NACK messages as well as error field in the on_xxxxxx messages
While this discussion is from a Beckn perspective, Adapters can provide synchronous modes. For example, the Protocol Server which is the reference implementation of the Beckn Adapter provides a synchronous mode by default. So if your software calls the support endpoint on the BAP Protocol Server, the Protocol Server waits till it gets the on_support and returns back that as the response.

Structure of a message with a NACK
{
    "message": {
        "ack": {
            "status": "NACK"
        }
    },
    "error": {
        "code": 400,
        "message": "OpenApiValidator Error at BAP-CLIENT",
    }
}
Structure of a on_select message with an error
{
    "context": {
        "action": "on_select",
        "version": "1.1.0",
        ...
    },
    "error": {
        "code": 30001,
        "message": "Requested provider is not in the database"
    }
}
API Calls and Schema
Search
Consumer searches for EV charging stations with specific criteria including location, connector type, and category filters.
This is like typing "EV charger" into Google Maps and saying "find me charging stations within 5km of this location that have CCS2 connectors." The app sends this request to find available charging stations that match your criteria.

```json
{
  "context": {
    "ttl": "PT10M",
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
    "timestamp": "2024-08-05T09:21:12.618Z",
    "message_id": "e138f204-ec0b-415d-9c9a-7b5bafe10bfe",
    "transaction_id": "2ad735b9-e190-457f-98e5-9702fd895996",
    "domain": "ev-charging:uei",
    "version": "1.1.0",
    "bap_id": "example-bap-id",
    "bap_uri": "https://example-bap-url.com"
  },
  "message": {
    "intent": {
      "descriptor": {
        "name": "EV charger"
      },
      "category": {
        "descriptor": {
          "code": "green-tariff"
        }
      },
      "fulfillment": {
        "type": "CHARGING",
        "stops": [
          {
            "location": {
              "circle": {
                "gps": "12.423423,77.325647",
                "radius": {
                  "value": "5",
                  "unit": "km"
                }
              }
            }
          }
        ],
        "tags": [
          {
            "list": [
              {
                "descriptor": {
                  "code": "connector-type"
                },
                "value": "CCS2"
              }
            ]
          }
        ]
      }
    }
  }
}
```

**Parameter Explanation:**

**Search Criteria:**
- `message.intent.descriptor.name`: Free text search for charging stations (user can enter any search terms like "EV charger", "fast charging", etc.)
- `message.intent.category.descriptor.code`: Category-based search filter (e.g., "green-tariff" for eco-friendly charging options)

**Location and Timing:**
- `message.intent.fulfillment.stops[0].location.circle.gps`: GPS coordinates for search center (REQUIRED - format: "latitude,longitude")
- `message.intent.fulfillment.stops[0].location.circle.radius.value`: Search radius value (REQUIRED - e.g., "5")
- `message.intent.fulfillment.stops[0].location.circle.radius.unit`: Unit of measurement (REQUIRED - e.g., "km", "miles")

**Connector Type Filtering:**
- `message.intent.fulfillment.tags[0].list[0].descriptor.code`: Connector type filter code (e.g., "connector-type")
- `message.intent.fulfillment.tags[0].list[0].value`: Specific connector type value (e.g., "CCS2", "CHAdeMO", "Type 2")
- Used by BPP to filter charging stations that match vehicle requirements

**Context Information:**
- `context.domain`: Service domain identifier (e.g., "ev-charging:uei")
- `context.action`: API action type (set to "search")
- `context.bap_id`: Consumer application identifier
- `context.transaction_id`: Unique transaction identifier for tracking this search request

### on_search
BPP returns a comprehensive catalog of available charging stations from multiple CPOs with detailed specifications, pricing, and location information.
Multiple providers (CPOs) with their charging networks
Detailed location information with GPS coordinates
Individual charging station specifications and pricing
Connector types, power ratings, and availability status
This is the response you get back after searching - like getting a list of all nearby restaurants from Google Maps. It shows you all the charging stations available, their locations, prices, and what type of connectors they have. Think of it as a "charging station directory" for your area.
{
  "context": {
    "domain": "ev-charging:uei",
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
    "timestamp": "2023-07-16T04:41:16Z"
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
                  },
                }
              ]
            }
          ],
          "items": [
            {
              "id": "pe-charging-01",
              "descriptor": {
			 "name": "EV Charger #1 (AC Fast Charger)"
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
				  "code":"connector-cpecifications"
                      "name": "Connector Specifications"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "connector Id",
                            "code": "connector-id"
                        },
                        "value": "1"
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
                            "name": "connector ID",
                            "code": "connector-id"
                        },
                        "value": "2"
                    },
                    {
                        "descriptor": {
                            "name": "Power Type",
                            "code": "power-type"
                        },
                        "value": "DC"
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
                        "value": "40kW"
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

**Parameter Explanation:**

**Provider Information:**
- `message.catalog.providers[0].id`: Unique identifier for the charging network provider (CPO)
- `message.catalog.providers[0].descriptor.name`: Display name of the charging network (e.g., "CPO1 EV charging Company")
- `message.catalog.providers[0].descriptor.short_desc`: Brief description of provider's services and coverage area

**Location Details:**
- `message.catalog.providers[0].locations[0].id`: Unique location identifier for the charging station
- `message.catalog.providers[0].locations[0].gps`: GPS coordinates of the charging station (format: "latitude,longitude")
- `message.catalog.providers[0].locations[0].descriptor.name`: Human-readable name of the charging station
- `message.catalog.providers[0].locations[0].address`: Full address of the charging station

**Charging Station Specifications (Items):**
- `message.catalog.providers[0].items[0].id`: Unique identifier for the specific charging point/EVSE
- `message.catalog.providers[0].items[0].descriptor.name`: Human-readable name of the charging point
- `message.catalog.providers[0].items[0].price.value`: Charging rate per unit (e.g., "18" for ₹18/kWh)
- `message.catalog.providers[0].items[0].price.currency`: Currency and unit basis (e.g., "INR/kWH")

**Technical Specifications (Tags):**
- `connector-id`: Physical connector identifier at the charging station
- `power-type`: Type of power delivery (e.g., "AC_3_PHASE", "DC")
- `connector-type`: Connector standard (e.g., "CCS2", "CHAdeMO", "Type 2")
- `charging-speed`: Relative charging speed classification (e.g., "FAST", "SLOW")
- `power-rating`: Maximum power output in kilowatts (e.g., "30kW", "40kW")
- `status`: Current availability status (e.g., "Available", "In Use", "Maintenance")

**Fulfillment and Category Links:**
- `fulfillment_ids`: Links to fulfillment options (charging service delivery methods)
- `category_ids`: Links to program categories (e.g., "green-tariff" for eco-friendly options)
- `location_ids`: Links to specific charging station locations

### init
This is like filling out a hotel booking form - you're telling the charging station "I want to charge here, here's my contact info and billing details." It's the first step in actually doing transacting.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "init",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "cpo1.com"
      },
      "items": [
        {
          "id": "pe-charging-01",
"tags": [
                {
                  "descriptor": {
				  "code":"charging-options"
                      "name": "Charging Options"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "Charging By",
                            "code": "charging-by"
                        },
                        "value": "amount"
                    },
                    {
                        "descriptor": {
                            "name": "Charging Limit",
                            "code": "chaging-limit"
                        },
                        "value": "100INR"
                    }
			    ]
			}
		 ]
        }
      ],
      "billing": {
        "name": "Ravi Kumar",
        "organization": {
          "descriptor": { "name": "GreenCharge Pvt Ltd" }
        },
        "address": "Apartment 123, MG Road, Bengaluru, Karnataka, 560001, India",
        "state": { "name": "Karnataka" },
        "city": { "name": "Bengaluru" },
        "email": "ravi.kumar@greencharge.com",
        "phone": "+918765432100",
        "time": { "timestamp": "2025-07-30T12:02:00Z" },
        "tax_id": "GSTIN29ABCDE1234F1Z5"
      },
      "fulfillments": [
        {
          "id": "1",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Ravi Kumar"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          }
        }
      ]
    }
  }
}
```

**Parameter Explanation:**

**Order Information:**
- `message.order.provider.id`: Selected charging network provider identifier (from on_search response)
- `message.order.items[0].id`: Specific charging point/EVSE identifier (from on_search response)

**Charging Options:**
- `message.order.items[0].tags[0].list[0].descriptor.code`: Charging method identifier (e.g., "charging-by")
- `message.order.items[0].tags[0].list[0].value`: Charging method value (e.g., "amount" for pay-per-use)
- `message.order.items[0].tags[0].list[1].descriptor.code`: Charging limit identifier (e.g., "chaging-limit")
- `message.order.items[0].tags[0].list[1].value`: Maximum charging amount (e.g., "100INR" for ₹100 limit)

**Billing Information:**
- `message.order.billing.name`: Customer's full name for billing purposes
- `message.order.billing.organization.descriptor.name`: Company name if charging for business use
- `message.order.billing.address`: Complete billing address for tax and invoice purposes
- `message.order.billing.email`: Contact email for billing communications
- `message.order.billing.phone`: Contact phone number for billing inquiries
- `message.order.billing.tax_id`: GST number for business billing and tax compliance

**Fulfillment Details:**
- `message.order.fulfillments[0].id`: Unique fulfillment identifier for this charging session
- `message.order.fulfillments[0].stops[0].type`: Session type (set to "start" for charging initiation)
- `message.order.fulfillments[0].stops[0].time.timestamp`: Requested charging start time
- `message.order.fulfillments[0].customer.person.name`: Customer name for session identification
- `message.order.fulfillments[0].customer.contact.phone`: Customer phone for session coordination

**Context Information:**
- `context.transaction_id`: Unique transaction identifier linking all related API calls
- `context.bap_id` and `context.bpp_id`: Consumer and provider application identifiers
- `context.domain`: Service domain (e.g., "ev-charging:uei")

### on_init
This is like getting a hotel quote back - "Your charging session will cost ₹100, here are the payment options." It's the charging station saying "I can accommodate your request, here are the terms and how to pay."
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_init",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "cpo1.com",
        "descriptor": {
            "name": "CPO1 EV charging Company",
            "short_desc": "CPO1 provides EV charging facility across India",
            "images": [
              {
                "url": "https://cpo1.com/images/logo.png"
              }
            ]
          }
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
				  "code":"connector-cpecifications"
                      "name": "Connector Specifications"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "connector Id",
                            "code": "connector-id"
                        },
                        "value": "1"
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
                },
 {
                  "descriptor": {
				  "code":"charging-options"
                      "name": "Charging Options"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "Charging By",
                            "code": "charging-by"
                        },
                        "value": "amount"
                    },
                    {
                        "descriptor": {
                            "name": "Charging Limit",
                            "code": "chaging-limit"
                        },
                        "value": "100INR"
                    }
			    ]
			}
              ]
        }
      ],
      "fulfillments": [
        {
          "id": "fulfillment-001",
          "type": "CHARGING",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              },
		    "location": {
  "gps": "28.345345,77.389754",
            "descriptor": {
              "name": "BlueCharge Connaught Place Station"
              },
              "address": "Connaught Place, New Delhi"
},
"instructions": {
	"short_desc": "Ground floor, Pillar Number 4"
}
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "100",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (5 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "90",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "10"
            }
          }
        ]
      },
"billing": {
        "name": "Ravi Kumar",
        "organization": {
          "descriptor": { "name": "GreenCharge Pvt Ltd" }
        },
        "address": "Apartment 123, MG Road, Bengaluru, Karnataka, 560001, India",
        "state": { "name": "Karnataka" },
        "city": { "name": "Bengaluru" },
        "email": "ravi.kumar@greencharge.com",
        "phone": "+918765432100",
        "time": { "timestamp": "2025-07-30T12:02:00Z" },
        "tax_id": "GSTIN29ABCDE1234F1Z5"
      },
      "payments": [
        {
         "id": "payment-123e4567-e89b-12d3-a456-426614174000",
          "collected_by": "BPP",
          "url": "https://payments.bluechargenet-aggregator.io/pay?transaction_id=$transaction_id&amount=$amount",
          "params": {
            "amount": "100.00",
            "currency": "INR"
          },
          "type": "PRE-ORDER",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ]
    }
  }
}
```

**Parameter Explanation:**

**Quote and Pricing:**
- `message.order.quote.price.value`: Total estimated cost for the charging session
- `message.order.quote.breakup`: Detailed cost breakdown showing energy cost and service fees
- `message.order.quote.breakup[0].title`: Description of each cost component (e.g., "Charging session cost (5 kWh @ ₹18.00/kWh)")

**Payment Information:**
- `message.order.payments[0].id`: Unique payment identifier for tracking
- `message.order.payments[0].collected_by`: Who collects the payment (BPP in this case)
- `message.order.payments[0].url`: Payment gateway URL for processing the transaction
- `message.order.payments[0].type`: Payment timing (PRE-ORDER for advance payment)
- `message.order.payments[0].status`: Current payment status (NOT-PAID initially)

**Charging Options:**
- `message.order.items[0].tags[0].list[0].value`: Charging method (e.g., "amount" for pay-per-use)
- `message.order.items[0].tags[0].list[1].value`: Charging limit (e.g., "100INR" for ₹100 maximum)

**Location and Instructions:**
- `message.order.fulfillments[0].stops[0].location.gps`: Charging station GPS coordinates
- `message.order.fulfillments[0].stops[0].instructions.short_desc`: Specific location details (e.g., "Ground floor, Pillar Number 4")

### confirm
This is like clicking "Confirm Booking" on a hotel website after you've completed the payment. You're saying "Yes, I accept these terms and want to proceed with this charging session." The payment has already been processed (you can see the transaction ID in the message), and this is the final confirmation step before your charging session is officially booked.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "confirm",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "cpo1.com"
      },
      "items": [
        {
          "id": "pe-charging-01",
"tags": [
                {
                  "descriptor": {
				  "code":"charging-options"
                      "name": "Charging Options"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "Charging By",
                            "code": "charging-by"
                        },
                        "value": "amount"
                    },
                    {
                        "descriptor": {
                            "name": "Charging Limit",
                            "code": "chaging-limit"
                        },
                        "value": "100INR"
                    }
			    ]
			}
		 ]
        }
      ],
      "billing": {
        "name": "Ravi Kumar",
        "organization": {
          "descriptor": { "name": "GreenCharge Pvt Ltd" }
        },
        "address": "Apartment 123, MG Road, Bengaluru, Karnataka, 560001, India",
        "state": { "name": "Karnataka" },
        "city": { "name": "Bengaluru" },
        "email": "ravi.kumar@greencharge.com",
        "phone": "+918765432100",
        "time": { "timestamp": "2025-07-30T12:02:00Z" },
        "tax_id": "GSTIN29ABCDE1234F1Z5"
      },
      "fulfillments": [
        {
          "id": "fulfillment-001",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            }
            }
          ],
          "customer": {
            "person": {
              "name": "Ravi kumar"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          }
        }
      ],
      "payments": [
        {       
"id": "payment-123e4567-e89b-12d3-a456-426614174000",
          "collected_by": "BPP",
          "url": "https://payments.bluechargenet-aggregator.io/pay?transaction_id=$transaction_id&amount=$amount",
          "params": {
            "transaction_id": "123e4567-e89b-12d3-a456-426614174000",
            "amount": "100.00",
            "currency": "INR"
          },
          "type": "PRE-ORDER",
          "status": "PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ]
    }
  }
}
```

**Parameter Explanation:**

**Order Confirmation:**
- `message.order.provider.id`: Selected charging network provider identifier
- `message.order.items[0].id`: Specific charging point/EVSE identifier
- `message.order.fulfillments[0].id`: Fulfillment identifier for this session

**Payment Confirmation:**
- `message.order.payments[0].id`: Payment identifier matching the on_init response
- `message.order.payments[0].status`: Payment status (changed from "NOT-PAID" to "PAID")
- `message.order.payments[0].params.transaction_id`: External payment system transaction ID
- `message.order.payments[0].params.amount`: Confirmed payment amount

**Session Details:**
- `message.order.fulfillments[0].stops[0].time.timestamp`: Confirmed charging start time
- `message.order.fulfillments[0].customer.person.name`: Customer name for session identification
- `message.order.fulfillments[0].customer.contact.phone`: Customer contact for coordination

**Charging Options:**
- `message.order.items[0].tags[0].list[0].value`: Confirmed charging method (e.g., "amount")
- `message.order.items[0].tags[0].list[1].value`: Confirmed charging limit (e.g., "100INR")

### on_confirm
This is like getting a hotel confirmation email - "Your booking is confirmed! Here's your reservation number." The charging station is saying "Great! Your charging session is booked and ready. Here's your order ID and all the details."
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_confirm",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "cpo1.com",
        "descriptor": {
            "name": "CPO1 EV charging Company",
            "short_desc": "CPO1 provides EV charging facility across India",
            "images": [
              {
                "url": "https://cpo1.com/images/logo.png"
              }
            ]
          }
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
				  "code":"connector-cpecifications"
                      "name": "Connector Specifications"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "connector Id",
                            "code": "connector-id"
                        },
                        "value": "1"
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
                },
 {
                  "descriptor": {
				  "code":"charging-options"
                      "name": "Charging Options"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "Charging By",
                            "code": "charging-by"
                        },
                        "value": "amount"
                    },
                    {
                        "descriptor": {
                            "name": "Charging Limit",
                            "code": "chaging-limit"
                        },
                        "value": "100INR"
                    }
			    ]
			}
            ]
        }
      ],
"fulfillments": [
        {
          "id": "fulfillment-001",
          "type": "CHARGING",
           "state": {
            "descriptor": {
              "code": "PENDING",
              "name": "Charging Pending"
            },
            "updated_at": "2025-07-30T12:06:02Z",
            "updated_by": "bluechargenet-aggregator.io"
          },
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              },
		    "location": {
  "gps": "28.345345,77.389754",
            "descriptor": {
              "name": "BlueCharge Connaught Place Station"
              },
              "address": "Connaught Place, New Delhi"
},
"instructions": {
	"short_desc": "Ground floor, Pillar Number 4"
}
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "100",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (5 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "90",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "10"
            }
          }
        ]
      },
"billing": {
        "name": "Ravi Kumar",
        "organization": {
          "descriptor": { "name": "GreenCharge Pvt Ltd" }
        },
        "address": "Apartment 123, MG Road, Bengaluru, Karnataka, 560001, India",
        "state": { "name": "Karnataka" },
        "city": { "name": "Bengaluru" },
        "email": "ravi.kumar@greencharge.com",
        "phone": "+918765432100",
        "time": { "timestamp": "2025-07-30T12:02:00Z" },
        "tax_id": "GSTIN29ABCDE1234F1Z5"
      },
      "payments": [
        {
         "id": "payment-123e4567-e89b-12d3-a456-426614174000",
          "collected_by": "BPP",
          "url": "https://payments.bluechargenet-aggregator.io/pay?transaction_id=$transaction_id&amount=$amount",
          "params": {
            "transaction_id": "123e4567-e89b-12d3-a456-426614174000",
            "amount": "100.00",
            "currency": "INR"
          },
          "type": "PRE-ORDER",
          "status": "PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ]
    }
  }
}
```

**Parameter Explanation:**

**Order Status:**
- `message.order.id`: Unique order identifier assigned by the BPP
- `message.order.fulfillments[0].state.descriptor.code`: Current order status (e.g., "PENDING")
- `message.order.fulfillments[0].state.updated_at`: Timestamp of last status update
- `message.order.fulfillments[0].state.updated_by`: System that updated the status

**Charging Point Details:**
- `message.order.items[0].descriptor.name`: Human-readable name of the charging point
- `message.order.items[0].price.value`: Charging rate per unit (e.g., "18" for ₹18/kWh)
- `message.order.items[0].tags`: Technical specifications and charging options

**Session Information:**
- `message.order.fulfillments[0].type`: Service type (set to "CHARGING")
- `message.order.fulfillments[0].stops[0].location.gps`: Charging station GPS coordinates
- `message.order.fulfillments[0].stops[0].instructions.short_desc`: Specific location instructions

**Payment Confirmation:**
- `message.order.payments[0].status`: Payment status (confirmed as "PAID")
- `message.order.payments[0].params.transaction_id`: External payment transaction ID
- `message.order.payments[0].type`: Payment type (PRE-ORDER for advance payment)

### update (start charging)
This is like pressing the "Start" button on a washing machine. You're telling the charging station "I'm ready to start charging now, please begin the session." It's the moment when you actually start using the charging service you booked.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "update",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "update_target": "order.fulfillments[0].state",
    "order": {
	 "id": "123e4567-e89b-12d3-a456-426614174000",
      "fulfillments": [
        {
          "id": "fulfillment-001",
          "type": "CHARGING",
          "state": {
            "descriptor": {
              "code": "start-charging"
            }
          },
		"stops": [
   {
	"authorization": {
    "token": "AUTH-GCN-20250730-001"
 }
   }
]
        }
      ]
    }
  }
}
```

**Parameter Explanation:**

**Update Target:**
- `message.update_target`: Specifies which part of the order to update (e.g., "order.fulfillments[0].state")
- `message.order.id`: Order identifier from the confirmed booking

**State Change:**
- `message.order.fulfillments[0].state.descriptor.code`: New state value (e.g., "start-charging")
- `message.order.fulfillments[0].type`: Service type (set to "CHARGING")

**Authorization:**
- `message.order.fulfillments[0].stops[0].authorization.token`: Authorization token for the charging session
- This token validates that the user is authorized to start charging at this station

**Context Information:**
- `context.transaction_id`: Links this update to the original booking transaction
- `context.action`: Set to "update" to indicate state modification

### on_update (start charging)
This is like getting a "Washing Started" notification from your washing machine. The charging station is saying "Perfect! Your charging session has begun. You can now track your progress and see how much energy is being delivered to your vehicle."
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_update",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
 "message": {
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "cpo1.com",
        "descriptor": {
            "name": "CPO1 EV charging Company",
            "short_desc": "CPO1 provides EV charging facility across India",
            "images": [
              {
                "url": "https://cpo1.com/images/logo.png"
              }
            ]
          }
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
				  "code":"connector-cpecifications"
                      "name": "Connector Specifications"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "connector Id",
                            "code": "connector-id"
                        },
                        "value": "1"
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
                },
 {
                  "descriptor": {
				  "code":"charging-options"
                      "name": "Charging Options"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "Charging By",
                            "code": "charging-by"
                        },
                        "value": "amount"
                    },
                    {
                        "descriptor": {
                            "name": "Charging Limit",
                            "code": "chaging-limit"
                        },
                        "value": "100INR"
                    }
			    ]
			}
              ]
          }
      ],
"fulfillments": [
        {
          "id": "fulfillment-001",
          "type": "CHARGING",
           "state": {
            "descriptor": {
              "code": "ACTIVE",
              "name": "Charging started"
            },
            "updated_at": "2025-07-30T12:06:02Z",
            "updated_by": "bluechargenet-aggregator.io"
          },
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              },
		    "location": {
  "gps": "28.345345,77.389754",
            "descriptor": {
              "name": "BlueCharge Connaught Place Station"
              },
              "address": "Connaught Place, New Delhi"
},
"instructions": {
	"short_desc": "Ground floor, Pillar Number 4"
}
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "100",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (5 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "90",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "10"
            }
          }
        ]
      },
"billing": {
        "name": "Ravi Kumar",
        "organization": {
          "descriptor": { "name": "GreenCharge Pvt Ltd" }
        },
        "address": "Apartment 123, MG Road, Bengaluru, Karnataka, 560001, India",
        "state": { "name": "Karnataka" },
        "city": { "name": "Bengaluru" },
        "email": "ravi.kumar@greencharge.com",
        "phone": "+918765432100",
        "time": { "timestamp": "2025-07-30T12:02:00Z" },
        "tax_id": "GSTIN29ABCDE1234F1Z5"
      },
      "payments": [
        {
         "id": "payment-123e4567-e89b-12d3-a456-426614174000",
          "collected_by": "bpp",
          "url": "https://payments.bluechargenet-aggregator.io/pay?transaction_id=$transaction_id&amount=$amount",
          "params": {
            "transaction_id": "123e4567-e89b-12d3-a456-426614174000",
            "amount": "100.00",
            "currency": "INR"
          },
          "type": "PRE-ORDER",
          "status": "PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" }
        }
      ]
    }
  }
}
```

**Parameter Explanation:**

**Session Status Update:**
- `message.order.fulfillments[0].state.descriptor.code`: Current session status (changed to "ACTIVE")
- `message.order.fulfillments[0].state.updated_at`: Timestamp when charging started
- `message.order.fulfillments[0].state.updated_by`: System that initiated the charging session

**Charging Point Information:**
- `message.order.items[0].descriptor.name`: Human-readable name of the active charging point
- `message.order.items[0].price.value`: Charging rate per unit (e.g., "18" for ₹18/kWh)
- `message.order.items[0].tags`: Technical specifications and charging options

**Session Details:**
- `message.order.fulfillments[0].type`: Service type (set to "CHARGING")
- `message.order.fulfillments[0].stops[0].time.timestamp`: Actual charging start time
- `message.order.fulfillments[0].stops[0].location.gps`: Charging station GPS coordinates
- `message.order.fulfillments[0].stops[0].instructions.short_desc`: Specific location details

**Payment Status:**
- `message.order.payments[0].status`: Payment status (confirmed as "PAID")
- `message.order.payments[0].type`: Payment type (PRE-ORDER for advance payment)

### track
This is like asking "Where's my package?" on an e-commerce website. You're requesting a link to monitor your charging session in real-time - how much energy has been delivered, how much it's costing, and when it will be complete. Think of it as getting a "live dashboard" for your charging session.
{
    "context": {
        "domain": "ev-charging:uei",
        "action": "track",
        "location": {
            "city": {
                "code": "std:080"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "example-bap-id",
        "bap_uri": "https://example-bap-url.com",
        "bpp_id": "example-bpp-id",
        "bpp_uri": "https://example-bpp-url.com",
        "transaction_id": "e0a38442-69b7-4698-aa94-a1b6b5d244c2",
        "message_id": "6ace310b-6440-4421-a2ed-b484c7548bd5",
        "timestamp": "2023-02-18T17:00:40.065Z",
        "version": "1.0.0",
        "ttl": "PT10M"
    },
    "message": {
        "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38"
    }
}
```

**Parameter Explanation:**

**Tracking Request:**
- `message.order_id`: Unique order identifier for the charging session to track
- This links the tracking request to the specific booking

**Context Information:**
- `context.transaction_id`: Links this tracking request to the original booking transaction
- `context.action`: Set to "track" to indicate tracking request
- `context.ttl`: Time-to-live for the tracking request (e.g., "PT10M" for 10 minutes)

### on_track
This is like getting a FedEx tracking link - "Click here to see your package's journey." The charging station is giving you a special webpage where you can watch your charging session live, see the current power being delivered, and get real-time updates on your charging progress.
{
    "context": {
        "domain": "ev-charging:uei",
        "action": "on_track",
        "location": {
            "city": {
                "code": "std:080"
            },
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "bap_id": "example-bap-id",
        "bap_uri": "https://example-bap-url.com",
        "bpp_id": "example-bpp-id",
        "bpp_uri": "https://example-bpp-url.com",
        "transaction_id": "e0a38442-69b7-4698-aa94-a1b6b5d244c2",
        "message_id": "6ace310b-6440-4421-a2ed-b484c7548bd5",
        "timestamp": "2023-02-18T17:00:40.065Z",
        "version": "1.0.0",
        "ttl": "PT10M"
    },
    "message": {
        "tracking": {
            "id": "TRACK-SESSION-9876543210",
            "url": "
https://track.bluechargenet-aggregator.io/session/SESSION-9876543210",
            "status": "active"
        }
    }
}
```

**Parameter Explanation:**

**Tracking Response:**
- `message.tracking.id`: Unique tracking identifier for the charging session
- `message.tracking.url`: Live tracking dashboard URL for monitoring charging progress
- `message.tracking.status`: Current tracking status (e.g., "active" for ongoing session)

**Context Information:**
- `context.transaction_id`: Links this tracking response to the original booking transaction
- `context.action`: Set to "on_track" to indicate tracking response

### Asynchronous on_update (stop charging)
This is like getting a "Washing Complete" notification from your washing machine. The charging station is saying "Your charging session has finished! Here's the final bill and session summary."
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_update",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "cpo1.com",
        "descriptor": {
            "name": "CPO1 EV charging Company",
            "short_desc": "CPO1 provides EV charging facility across India",
            "images": [
              {
                "url": "https://cpo1.com/images/logo.png"
              }
            ]
          }
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
				  "code":"connector-cpecifications"
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
                },
 {
                  "descriptor": {
				  "code":"charging-options"
                      "name": "Charging Options"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "Charging By",
                            "code": "charging-by"
                        },
                        "value": "amount"
                    },
                    {
                        "descriptor": {
                            "name": "Charging Limit",
                            "code": "chaging-limit"
                        },
                        "value": "100INR"
                    }
			    ]
			 }
            ]
        }
      ],
"fulfillments": [
        {
          "id": "fulfillment-001",
          "type": "CHARGING",
           "state": {
            "descriptor": {
              "code": "COMPLETED",
              "name": "Charging completed"
            },
            "updated_at": "2025-07-30T13:07:02Z",
            "updated_by": "bluechargenet-aggregator.io"
          },
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              },
		    "location": {
  "gps": "28.345345,77.389754",
            "descriptor": {
              "name": "BlueCharge Connaught Place Station"
              },
              "address": "Connaught Place, New Delhi"
},
"instructions": {
	"short_desc": "Ground floor, Pillar Number 4"
}
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              },
		    "location": {
  "gps": "28.345345,77.389754",
            "descriptor": {
              "name": "BlueCharge Connaught Place Station"
              },
              "address": "Connaught Place, New Delhi"
},
"instructions": {
	"short_desc": "Ground floor, Pillar Number 4"
}
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "100",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (5 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "90",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "10"
            }
          }
        ]
      },
"billing": {
        "name": "Ravi Kumar",
        "organization": {
          "descriptor": { "name": "GreenCharge Pvt Ltd" }
        },
        "address": "Apartment 123, MG Road, Bengaluru, Karnataka, 560001, India",
        "state": { "name": "Karnataka" },
        "city": { "name": "Bengaluru" },
        "email": "ravi.kumar@greencharge.com",
        "phone": "+918765432100",
        "time": { "timestamp": "2025-07-30T12:02:00Z" },
        "tax_id": "GSTIN29ABCDE1234F1Z5"
      },
      "payments": [
        {
         "id": "payment-123e4567-e89b-12d3-a456-426614174000",
          "collected_by": "bpp",
          "url": "https://payments.bluechargenet-aggregator.io/pay?transaction_id=$transaction_id&amount=$amount",
          "params": {
            "transaction_id": "123e4567-e89b-12d3-a456-426614174000",
            "amount": "100.00",
            "currency": "INR"
          },
          "type": "PRE-ORDER",
          "status": "PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ]
     }
  }
}
```

**Parameter Explanation:**

**Session Completion:**
- `message.order.fulfillments[0].state.descriptor.code`: Final session status (changed to "COMPLETED")
- `message.order.fulfillments[0].state.updated_at`: Timestamp when charging session ended
- `message.order.fulfillments[0].state.updated_by`: System that completed the session

**Session Timeline:**
- `message.order.fulfillments[0].stops[0].time.timestamp`: Session start time
- `message.order.fulfillments[0].stops[1].time.timestamp`: Session end time
- `message.order.fulfillments[0].stops[1].type`: Set to "finish" indicating session completion

**Final Billing:**
- `message.order.quote.price.value`: Total final cost for the completed session
- `message.order.quote.breakup`: Detailed cost breakdown for energy and service fees
- `message.order.payments[0].status`: Payment status (confirmed as "PAID")

**Location Information:**
- `message.order.fulfillments[0].stops[0].location.gps`: Charging station GPS coordinates
- `message.order.fulfillments[0].stops[0].instructions.short_desc`: Specific location details

### Rating
This is like leaving a review on Amazon or rating your Uber driver. You're giving feedback on your charging experience - how easy it was to find the station, how fast the charging was, and overall satisfaction. It helps other users and improves the service.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "rating",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "ratings": [
      {
        "id": "fulfillment-001",
        "rating_category": "Fulfillment",
        "value": "5"
      }
    ]
  }
}
```

**Parameter Explanation:**

**Rating Information:**
- `message.ratings[0].id`: Identifier for what is being rated (e.g., "fulfillment-001" for the charging session)
- `message.ratings[0].rating_category`: Category of the rating (e.g., "Fulfillment" for service quality)
- `message.ratings[0].value`: Rating value (e.g., "5" for 5-star rating, scale typically 1-5)

**Context Information:**
- `context.transaction_id`: Links this rating to the original charging session transaction
- `context.action`: Set to "rating" to indicate rating submission

### on_rating
This is like getting a "Thank you for your review!" message from Amazon. The charging station is acknowledging your rating and might ask for more detailed feedback.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_rating",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "feedback_form": {
      "form": {
        "url": "https://example-bpp.comfeedback/portal",
   "mime_type": "application/xml"

      },
      "required": false
    }
  }
}
```

**Parameter Explanation:**

**Feedback Form:**
- `message.feedback_form.form.url`: URL to additional feedback form or survey
- `message.feedback_form.form.mime_type`: Format of the feedback form (e.g., "application/xml")
- `message.feedback_form.required`: Whether additional feedback is mandatory (set to "false")

**Context Information:**
- `context.transaction_id`: Links this rating response to the original charging session transaction
- `context.action`: Set to "on_rating" to indicate rating acknowledgment

### support
This is like calling customer service when you have a problem with your hotel booking. You're asking for help with your charging session - maybe the charger isn't working, you have a billing question, or you need to report an issue. It's your way of getting assistance when something goes wrong.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "support",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "support": {
      "ref_id": "6743e9e2-4fb5-487c-92b7",
	 "callback_phone": "+911234567890",
      "email": "ravi.kumar@bookmycharger.com"
    }
  }
}
```

**Parameter Explanation:**

**Support Request:**
- `message.support.ref_id`: Reference ID linking to the specific order or transaction
- `message.support.callback_phone`: Phone number for customer service callback
- `message.support.email`: Customer's email for support communication

**Context Information:**
- `context.transaction_id`: Links this support request to the original charging session transaction
- `context.action`: Set to "support" to indicate support request

### on_support
This is like getting a customer service response - "Here's our support phone number, email, and a link to create a support ticket." The charging station is providing you with contact information and ways to get help, similar to how a hotel would give you their front desk number and manager's contact details.
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_support",
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
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "support": {
      "ref_id": "6743e9e2-4fb5-487c-92b7",
      "phone": "18001080",
      "email": "support@bluechargenet-aggregator.io",
      "url": "https://support.bluechargenet-aggregator.io/ticket/SUP-20250730-001"
    }
  }
}
```

**Parameter Explanation:**

**Support Response:**
- `message.support.ref_id`: Echoed reference ID from the support request
- `message.support.phone`: Support hotline number for immediate assistance
- `message.support.email`: Support email address for written inquiries
- `message.support.url`: Support ticket creation URL for issue tracking

**Context Information:**
- `context.transaction_id`: Links this support response to the original charging session transaction
- `context.action`: Set to "on_support" to indicate support response

## Integrating with your software
This section gives a general walkthrough of how you would integrate your software with the Beckn network (say the sandbox environment). Refer to the starter kit for details on how to register with the sandbox and get credentials.
Beckn-ONIX is an initiative to promote easy installation and maintenance of a Beckn Network. Apart from the Registry and Gateway components that are required for a network facilitator, Beckn-ONIX provides a Beckn Adapter. A reference implementation of the Beckn-ONIX specification is available at Beckn-ONIX repository. The reference implementation of the Beckn Adapter is called the Protocol Server. Based on whether we are writing the seeker platform or the provider platform, we will be installing the BAP Protocol Server or the BPP Protocol Server respectively.
Integrating the seeker platform
If you are writing the seeker platform software, the following are the steps you can follow to build and integrate your application.
Identify the use cases from the above section that are close to the functionality you plan for your application.
Design and develop the UI that implements the flow you need. Typically you will have an API server that this UI talks to and it is called the Seeker Platform in the diagram below.
The API server should construct the required JSON message packets required for the different endpoints shown in the API section above.
Install the BAP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL) and a subscriber_id (typically the same as subscriber URL without the "https://" prefix)
Install the layer 2 file for the domain (Link is in the last section of this document)
Check with your network tech support to enable your BAP Protocol Server in the registry.
Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,
you write the Seeker Platform(dark blue)
install the BAP Protocol Server (light blue)
the remaining components are provided by the sandbox environment
Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

Integrating the provider platform
If you are writing the provider platform software, the following are the steps you can follow to build and integrate your application.
Identify the use cases from the above section that are close to the functionality you plan for your application.
Design and develop the component that accepts the Beckn requests and interacts with your software to do transactions. It has to be an endpoint(it is called as webhook_url in the description below) which receives all the Beckn requests (search, select etc). This endpoint can either exist outside of your marketplace/shop software or within it. That is a design decision that will have to be taken by you based on the design of your existing marketplace/shop software. This component is also responsible for sending back the responses to the Beckn Adaptor.
Install the BPP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL), a subscriber_id (typically the same as subscriber URL without the "https://" prefix) and the webhook_url that you configured in the step above. Also the address of the BPP Protocol Server Client will have to be configured in your component above. This address hosts all the response endpoints (on_search,on_select etc)
Install the layer 2 file for the domain (Link is in the last section of this document)
Check with your network tech support to enable your BPP Protocol Server in the registry.
Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,
you write the Provider Platform(dark blue) - Here the component you wrote above in point 2 as well as your marketplace/shop software is together shown as Provider Platform
install the BPP Protocol Server (light blue)
the remaining components are provided by the sandbox environment
Use the postman collection to test your Provider Platform
Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

Links to artefacts
Postman collection for UEI EV Charging
Layer2 config for UEI EV Charging
When installing layer2 using Beckn-ONIX use this web address (https://raw.githubusercontent.com/beckn/missions/refs/heads/main/UEI/layer2/EV-charging/3.1/energy_EV_1.1.0_openapi_3.1.yaml)
Sandbox Details

BAP:
BAP Client Sandbox: https://bap-ps-client-sandbox.ueialliance.org/
BAP Network Sandbox: https://bap-ps-network-sandbox.ueialliance.org/
Registry/Gateway:
Gateway Sandbox: https://gateway-sandbox.ueialliance.org/
Registry Sandbox: https://registry-sandbox.ueialliance.org/
BPP:
BPP Client Sandbox: https://bpp-ps-client-sandbox.ueialliance.org/
BPP Network Sandbox: https://bpp-ps-network-sandbox.ueialliance.org/
BPP Playground Sandbox: https://bpp-playground-sandbox.ueialliance.org/


