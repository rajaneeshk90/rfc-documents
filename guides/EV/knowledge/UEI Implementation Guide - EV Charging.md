# **UEI Implementation Guide \- EV Charging**

#### **Version 1.2**

## **Version History**

| Date | Version | Description |
| ----- | ----- | ----- |
| 07-08-2024 | 1.0 | Initial Version |
| 09-09-2024 | 1.1 | Incorporated input from Participants during Winroom |
| 14-11-2024 | 1.2 | Incorporated feedback from Participants |
| 25-11-2024 | 1.3 | Updated moving connector specification tag groups from "fulfillments" to "items." |

## **Introduction**

This document provides material that helps network participants build and integrate their application with the UEI Network for EV Charging. This document is part of the starter kit that provides information about the network, learning resources, network participant checklist etc. This document only focuses on the implementation of the seeker/provider platform. It assumes the reader has a good overview of the Beckn network, its APIs, the overall structure of the schema etc.

## **Structure of the document**

This document has the following parts:

1. Outcome Visualization \- This is a pictorial or descriptive representation of the different use cases that are supported by the network.  
2. Flow diagrams \- This section provides a pictorial representation of the message flows that happen during the use case.  
3. API Calls and Schema \- This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.  
4. Taxonomy and layer 2 configuration \- This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.  
5. Notes on writing/integrating with your own software \- This section describes ways in which you can integrate (Becknify) your new or existing software  
6. Links to downloadable resources \- This section contains the downloadable files referenced in this document.  
7. Sandbox Details \- Sandbox links to BAP, Regitry/Gateway and BPP.

## **Outcome Visualisation**

**![][image1]**

### **Use case \- Discovery, order and fulfillment of EV Charging**

This use cases uses the names "Pulse Energy" and "Kazam" as examples for illustration.

* Srilekha is an IT professional who travels for work five days a week. She uses her four-wheeler electric vehicle (EV) for her commute. She prefers to charge her vehicle during her work hours and looks for suitable charging stations near her workplace.

Discovery:

* Srilekha begins to browse the Pulse Energy app to search for nearby EV chargers for four-wheelers.  
* She receives a catalogue of 4 available chargers provided by Pulse energy & Kazam. Among them, she finds one which is located at a distance of 500 metres, which was in the catalogue of Kazam.

Order:

* Srilekha selects the charger, opting for a charging session at the cost of Rs. 13/kWh for a duration of 1 hour.  
* She accepts the terms of order and is prompted to choose a payment method (card or link).  
* She chooses to make payment through the link and confirms the order.  
* The order is confirmed by Kazam’s charger provider and verifies the payment and generates an order ID

Fulfillment:

* Srilekha plugs the kazam’s charger into her vehicle and initiates the charging process. After an hour, the charging stops, and she is prompted to either remove the charger or continue charging. Srilekha removes the charger from her vehicle, thus ending the charging process.

Post Fulfilment:

* Srilekha rates her experience using a 0-5 star rating.

## **Flow diagrams**

### **General Beckn message flow and error handling**

This section is relevant to all the messages flows illustrated below and discussed further in the document.

Beckn is a aynchronous protocol at its core.

* When a network participant(NP1) sends a message to another participant(NP2), the other participant(NP2) immediately returns back an ACK/NACK(Acknowledgement or Negative Acknowledgement in case of error \- usually with wrongly formed messages).  
* An ACK is an indicator that the receiving participant(NP2) will process this message and dispatch an on\_xxxxxx message to original NP (NP1)  
* Subsequently after processing the message NP2 sends back the real response in the corresponding on\_xxxxxx message, to which again the first participant(NP1).  
* This message can contain a message field (for success) or error field (for failure)  
* NP1 when it receives the on\_xxxxxx message, sends back an ACK/NACK (Here in both the cases NP1 will not send any subsequent message).  
* In the Use case diagrams, this ACK/NACK is not illustrated explicitly to keep the diagrams crisp.  
* However when writing software we should be prepared to receive these NACK messages as well as error field in the on\_xxxxxx messages  
* While this discussion is from a Beckn perspective, Adapters can provide synchronous modes. For example, the Protocol Server which is the reference implementation of the Beckn Adapter provides a synchronous mode by default. So if your software calls the support endpoint on the BAP Protocol Server, the Protocol Server waits till it gets the on\_support and returns back that as the response.

![ACK NACK for messages][image2]

Structure of a message with a NACK

```
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
```

Structure of a on\_select message with an error

```
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
```

### **Use case \- Discovery, order and fulfillment of EV Charging**

* This usecase does not allow pre-booking of the charging station. So essentially after the user searches the charging station, the rest of the messages (select, init, confirm and update) all happen while the user is at the charging station. This is due to the current readiness of the participants.  
* It is envisioned that a future version of this guide (once the NPs are ready) will add pre-booking of a slot at a charging station.

Search for EV Charging stations nearby

## **API Calls and Schema**

### **search**

A search request can contain one or more search criteria within it. Use the following list on how to specify the criterion.

* The location to search around is specified in the message-\>intent-\>fulfillment-\>stops\[0\]-\>location field.  
* The connector type required is specified in message-\>intent-\>fulfillment-\>tags\[0\]-\>list\[0\]. Refer to the taxonomy section below for the list of connector types supported.  
* If searching by free text, it is specified in message-\>intent-\>descriptor-\>name  
* If searching by category, it is specified in message-\>intent-\>category-\>descriptor-\>code

```
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

### **on\_search**

on\_search with catalog of results

* The catalog that comes back has a list of providers.  
* Each provider has a list of items.  
* Each item is the catalog listing for a charging station.

```
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
          ],
          "fulfillments": [
            {
              "id": "",
              "type": "CHARGING",
              "tags": [
                {
                  "descriptor": {
                    "name": "Charging Point Specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Pillar Number 4",
                        "code": "charger-id"
                      },
                      "value": "charg1"
                    },
                    {
                      "descriptor": {
                          "code": "Availability"
                      },
                      "value": "Available"
                    }
                  ]
                }
              ]
            },
            {
              "id": "3",
              "type": "CHARGING",
              "tags": [
                {
                  "descriptor": {
                    "name": "Charging Point Specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Pillar Number 3",
                        "code": "charger-id"
                      },
                      "value": "charg3"
                    },
                    {
                      "descriptor": {
                          "code": "Availability"
                      },
                      "value": "Unvailable"
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

### **select**

sending a select request

* Choose the item(s) from the list from on\_search and request quote  
* The chosen item is in message-\>order-\>item\_id

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "select",
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
          "id": "pe-charging-01"
        }
      ],
      "fulfillments": [
        {
          "id": "fulfillment-001",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ]
        }
      ]
    }
  }
}
```

### **on\_select**

* The BPP returns back with a quote for the selection  
* It is in message-\>order-\>quote

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_select",
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
      }
    }
  }
}
```

### **init**

send init request

* The draft order includes billing details.  
* Billing details specified in message-\>order-\>billing

```
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
          "id": "pe-charging-01"
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
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
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

### **on\_init**

* Contains payment terms. Payment terms specified in message-\>order-\>payments  
* Cancellation terms specified in message-\>order-\>cancellation\_terms  
* Here we show the BPP as a payment collector. In case the BAP specifies that it collects the payment in the init, the url field within payments will be empty

```
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
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### **confirm**

* Confirm order including payment info (when applicable).  
* It is in message-\>order-\>payments

```
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
          "id": "pe-charging-01"
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
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
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
          "collected_by": "bpp",
          "url": "https://payments.bluechargenet-aggregator.io/pay?transaction_id=$transaction_id&amount=$amount",
          "params": {
            "transaction_id": "123e4567-e89b-12d3-a456-426614174000",
            "amount": "100.00",
            "currency": "INR"
          },
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ]
    }
  }
}
```

### **on\_confirm**

* Order confirmed.

```
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
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### **update (start charging)**

* Update the state of the fulfillment to start-charging.  
* This will trigger charging start.

```
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

### **on\_update (start charging)**

* Confirmation of successful start of charging operation  
* State in message-\>order-\>fulfillments\[\]-\>state

```
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
},

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
},
"authorization": {
    "token": "AUTH-GCN-20250730-001"
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
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### 

### **track**

* Request for a tracking URL for the order. Order Id must be specified in message-\>order\_id

```
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

### **on\_track**

* Tracking URL with custom UI on charging process.  
* The tracking URL will be in message-\>tracking-\>url

```
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

### 

### **update (stop charging)**

* Request to change fulfillment state to end-charging

```
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
              "code": "stop-charging"
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

### **on\_update (stop charging)**

* Confirmation of successful operation to end charging  
* The state will be in message-\>order-\>fulfillments\[\]-\>state

```
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
},

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
},
"authorization": {
    "token": "AUTH-GCN-20250730-001"
}
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "91",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (4.5 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "81",
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
            "amount": "91.00",
            "currency": "INR"
          },
          "type": "POST-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### **Asynchronous on\_update (stop charging)**

When the BPP detects a change that alters the agreed-upon payment terms (but does not cancel the order), it must send an unsolicited on\_update with the new quote and payments.

```
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
},

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
},
"authorization": {
    "token": "AUTH-GCN-20250730-001"
}
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "112",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (3 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "54",
              "currency": "INR"
            }
          },
          {
            "title": "Charging session cost (2 kWh @ ₹25.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "50",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "8"
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
            "amount": "112.00",
            "currency": "INR"
          },
          "type": "ON-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}

```

### 

### **status**

* Request for status on order. order\_id is specified in message-\>order\_id

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "status",
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
    "order_id": "6743e9e2-4fb5-487c-92b7"
  }
}
```

### **on\_status**

* Status of requested order.  
* Primarily the fulfillment status is specified in message-\>order-\>fulfillments\[\]-\>state  
* The message-\>order-\>fulfillments\[\]-\>state-\>descriptor-\>long\_desc can be used to specify the OCPP status of the charge point. This can help the BAP to construct a custom detailed UI for charging status.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_status",
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
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### 

### **Unsolicited on\_status**

The BPP should also push on\_status whenever informational OCPI session statuses change. These updates keep the BAP in sync with charger state but do not affect payment terms.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_status",
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
              "code": "SUSPENDED_EVSE",
              "name": "Charging Paused by Station"
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
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}

```

### **cancel**

* Request to cancel an order. order\_id to be specified in message-\>order\_id  
* In the case where there is no advance booking of the charging slot, this does not have much relevance

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "cancel",
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
    "order_id": "6743e9e2-4fb5-487c-92b7",
    "cancellation_reason_id": "5",
    "descriptor": {
      "short_desc": "User Requested Cancellation"
    },
    
  }
}
```

### **on\_cancel**

* Confirmation of cancelled order. The cancelled order will be in message-\>order  
* In cases where advanced booking of charging station is not supported, this message is not relevant.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_cancel",
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
              "code": "CANCELLED",
              "name": "Session cancelled by customer"
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
          "value": "60",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (3 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "54",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "6"
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
            "amount": "60.00",
            "currency": "INR"
          },
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ],
      "cancellation": {
        "cancelled_by": "CONSUMER",
	   "reason": {
  "descriptor": {
          "code": "UNKNOWN",
          "name": "Reason not specified"
        }
   }
    }
  }
}

```

### 

### **Unsolicited on\_cancel**

The BPP should push an unsolicited on\_cancel when the order must be cancelled by the provider due to external events.

Triggering Scenarios:

1. Charger Hardware Failure Station reports EVSE offline or faulty → no session possible.  
2. Connector Incompatibility Vehicle cannot connect to the charger standard → session invalid.  
3. Payment Timeout User fails to pay before quote.ttl expires.  
4. No-Show at Reservation User does not arrive or start a session within the reserved window.  
5. Station Closure / Maintenance CPO closes stations unexpectedly or for emergency work.  
6. Regulatory / Grid Emergency Shutdown Grid operator forces charging halt.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_cancel",
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
              "code": "CANCELLED",
              "name": "Session cancelled by customer"
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
          "value": "60",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "Charging session cost (3 kWh @ ₹18.00/kWh)",
            "item": {
		    "id": "pe-charging-01"
            },
            "price": {
              "value": "54",
              "currency": "INR"
            }
          },
          {
            "title": "Service Fee",
            "price": {
                "currency": "INR",
                "value": "6"
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
            "amount": "60.00",
            "currency": "INR"
          },
          "type": "PRE-FULFILLMENT",
          "status": "NOT-PAID",
          "time": { "timestamp": "2025-07-30T14:59:00Z" },
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ],
      "cancellation": {
        "cancelled_by": "PROVIDER",
	   "reason": {
  "descriptor": {
          "code": "CHARGER_HW_FAILURE",
          "name": "Charger hardware failure"
        }
   }
    }
  }
}
```

### **get\_rating\_categories**

* BAP can fetch a list of categoried for which a BPP supports prviding rating  
* BAP does not need to send anything in the message field.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "get_rating_categories",
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
  }
}
```

### **rating\_categories**

* The BPP responds with a list of categories in which ratings can be provided.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "get_rating_categories",
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
    "rating_categories": [
      Item,
      Order,
      Fulfillment,
      Provider
    ]
  }
}
```

### **rating**

* Rate different categories including fulfillment. Specify in message-\>ratings  
* Specify a rating category from the list recieved in the rating\_categories callback from the BPP.  
* Value should be a decimal between 0 and 5\.  
* Id can be the order id.

```
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

### **on\_rating**

* Confirmation of rating.  
* Optionally send a form for additional input (message-\>feedback\_form). Empty message otherwise.

```
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

### **support**

* Request for support information.  
* If regarding a specific order, specify the order\_id in message-\>support-\>ref\_id

```
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

### **on\_support**

* Contains support information. If integrated with a CMS, the URL could contain order specific support info  
* In all other cases, contains general support info (message-\>support)

```
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

## 

## **Taxonomy and layer 2 configuration**

* Connector-Type is a tag used in multiple messages above. The values allowed for the connector types are specified in the OCPI Standard. This [page](https://ev2go.io/ocpiconnectors) lists some of them. This list will be extended with those types which are present in India, but not listed in the OCPI list. The code to be used will be the OCPI connector Value shown in the above webpage in all lowercase characters (e.g.chademo, iec\_62196\_t3a)  
* The Charging Status in fulfillment object has a long\_desc field. This field should have the [OCPP](https://openchargealliance.org/protocols/open-charge-point-protocol/) status json object. The BAP can in on\_status get this charging status value and show custom UI uniform across different BPPs.

Below is a list of standardised codes used in this implementation. Each of these codes also have a list of standardised enum values associated with them. These codes and enums are expected to evolve with time as more NPs implement the UEI EV charging use case.

| SN | Path | Tag Code | Meaning | Enums |
| ----- | ----- | ----- | ----- | ----- |
| 1 | Charging Point Specifications | identifier |  |  |
| 2 | Charging Point Specifications | manufacturer |  | EVSE, OEM |
| 3 | Charging Point Specifications | availability |  | Online, Offline |
| 4 | Charging Point Specifications | charger-type |  |  |
| 5 | Charging Point Specifications | power-rating |  |  |
| 6 | Connector Specification | charger-type |  | AC, DC |
| 7 | Connector Specification | connector-type |  | Enums defined in the below table |
| 8 | Connector Specification | power-rating |  |  |
| 9 | Connector Specification | availability |  | Available, Unavailable, Preparing, Finishing, Charging, SuspendedEVSE, SuspendedEV, Reserved, Faulted |
| 10 | quote breakup | energy-cost |  |  |
| 11 | quote breakup | service-charge |  |  |
| 12 | quote breakup | platform-fee |  |  |
| 13 | quote breakup | parking-fee |  |  |
| 14 | quote breakup | gst |  |  |
| 15 | quote breakup | total |  |  |
| 16 | payment | payment\_type |  | PRE-ORDER |
| 17 | payment | payment\_status |  | PAID, NOT-PAID |
| 18 | payment | collected\_by |  | bap, bpp |
| 19 | fulfillment | state |  | start-chargin, end-chargin, charging-started, charging-ended, order-initiated, payment-completed |
| 20 | Connector Charging Details | energy-delivered |  |  |
| 21 | Connector Charging Details | soc |  |  |
| 22 | Connector Charging Details | start-time |  |  |
| 23 | Connector Charging Details | stop-time |  |  |
| 24 | Connector Charging Details | meter-start |  |  |
| 25 | Connector Charging Details | meter-stop |  |  |
| 26 | Connector Charging Details | current |  |  |
| 27 | Connector Charging Details | power |  |  |
| 28 | Connector Charging Details | voltage |  |  |

List of enums values defined for connector-type code

* CHADEMO \- The connector type is CHAdeMO, DC  
* CHAOJI \-  The ChaoJi connector. The new generation charging connector, harmonized between CHAdeMO and GB/T. DC.  
* DOMESTIC\_A  \- Standard/Domestic household, type "A", NEMA 1-15, 2 pins  
* DOMESTIC\_B  \- Standard/Domestic household, type "B", NEMA 5-15, 3 pins  
* DOMESTIC\_C  \- Standard/Domestic household, type "C", CEE 7/17, 2 pins  
* DOMESTIC\_D  \- Standard/Domestic household, type "D", 3 pin  
* DOMESTIC\_E  \- Standard/Domestic household, type "E", CEE 7/5 3 pins  
* DOMESTIC\_F  \- Standard/Domestic household, type "F", CEE 7/4, Schuko, 3 pins  
* DOMESTIC\_G  \- Standard/Domestic household, type "G", BS 1363, Commonwealth, 3 pins  
* DOMESTIC\_H  \- Standard/Domestic household, type "H", SI-32, 3 pins  
* DOMESTIC\_I  \- Standard/Domestic household, type "I", AS 3112, 3 pins  
* DOMESTIC\_J  \- Standard/Domestic household, type "J", SEV 1011, 3 pins  
* DOMESTIC\_K  \- Standard/Domestic household, type "K", DS 60884-2-D1, 3 pins  
* DOMESTIC\_L  \- Standard/Domestic household, type "L", CEI 23-16-VII, 3 pins  
* DOMESTIC\_M  \- Standard/Domestic household, type "M", BS 546, 3 pins  
* DOMESTIC\_N  \- Standard/Domestic household, type "N", NBR 14136, 3 pins  
* DOMESTIC\_O  \- Standard/Domestic household, type "O", TIS 166-2549, 3 pins  
* GBT\_AC \-  Guobiao GB/T 20234.2 AC socket/connector  
* GBT\_DC \-  Guobiao GB/T 20234.3 DC connector  
* IEC\_60309\_2\_single\_16 \- IEC 60309-2 Industrial Connector single phase 16 amperes (usually blue)  
* IEC\_60309\_2\_three\_16 \- IEC 60309-2 Industrial Connector three phases 16 amperes (usually red)  
* IEC\_60309\_2\_three\_32 \-  IEC 60309-2 Industrial Connector three phases 32 amperes (usually red)  
* IEC\_60309\_2\_three\_64 \-  IEC 60309-2 Industrial Connector three phases 64 amperes (usually red)  
* IEC\_62196\_T1 \-  IEC 62196 Type 1 "SAE J1772"  
* IEC\_62196\_T1\_COMBO \-  Combo Type 1 based, DC  
* IEC\_62196\_T2 \-  IEC 62196 Type 2 "Mennekes"  
* IEC\_62196\_T2\_COMBO \-  Combo Type 2 based, DC  
* IEC\_62196\_T3A \-  IEC 62196 Type 3A  
* IEC\_62196\_T3C \-  IEC 62196 Type 3C "Scame"  
* NEMA\_5\_20 \- NEMA 5-20, 3 pins  
* NEMA\_6\_30 \- NEMA 6-30, 3 pins  
* NEMA\_6\_50 \- NEMA 6-50, 3 pins  
* NEMA\_10\_30 \- NEMA 10-30, 3 pins  
* NEMA\_10\_50 \- NEMA 10-50, 3 pins  
* NEMA\_14\_30 \- NEMA 14-30, 3 pins, rating of 30 A  
* NEMA\_14\_50 \- NEMA 14-50, 3 pins, rating of 50 A  
* PANTOGRAPH\_BOTTOM\_UP \- On-board Bottom-up-Pantograph typically for bus charging  
* PANTOGRAPH\_TOP\_DOWN \- Off-board Top-down-Pantograph typically for bus charging  
* TESLA\_R \- Tesla Connector "Roadster"-type (round, 4 pin)  
* TESLA\_S \- Tesla Connector "Model-S"-type (oval, 5 pin)  
* CHOGORI  
* TYPE\_6  
* TYPE\_7  
* IS\_17017  
* ANDERSON  
* SWAPPING

## **Integrating with your software**

This section gives a general walkthrough of how you would integrate your software with the Beckn network (say the sandbox environment). Refer to the starter kit for details on how to register with the sandbox and get credentials.

Beckn-ONIX is an initiative to promote easy installation and maintenance of a Beckn Network. Apart from the Registry and Gateway components that are required for a network facilitator, Beckn-ONIX provides a Beckn Adapter. A reference implementation of the Beckn-ONIX specification is available at [Beckn-ONIX repository](https://github.com/beckn/beckn-onix). The reference implementation of the Beckn Adapter is called the Protocol Server. Based on whether we are writing the seeker platform or the provider platform, we will be installing the BAP Protocol Server or the BPP Protocol Server respectively.

### **Integrating the seeker platform**

If you are writing the seeker platform software, the following are the steps you can follow to build and integrate your application.

1. Identify the use cases from the above section that are close to the functionality you plan for your application.  
2. Design and develop the UI that implements the flow you need. Typically you will have an API server that this UI talks to and it is called the Seeker Platform in the diagram below.  
3. The API server should construct the required JSON message packets required for the different endpoints shown in the API section above.  
4. Install the BAP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL) and a subscriber\_id (typically the same as subscriber URL without the "https://" prefix)  
5. Install the layer 2 file for the domain (Link is in the last section of this document)  
6. Check with your network tech support to enable your BAP Protocol Server in the registry.  
7. Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,  
   * you write the Seeker Platform(dark blue)  
   * install the BAP Protocol Server (light blue)  
   * the remaining components are provided by the sandbox environment  
8. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

![Seeker platform testing sandbox][image3]

### **Integrating the provider platform**

If you are writing the provider platform software, the following are the steps you can follow to build and integrate your application.

1. Identify the use cases from the above section that are close to the functionality you plan for your application.  
2. Design and develop the component that accepts the Beckn requests and interacts with your software to do transactions. It has to be an endpoint(it is called as webhook\_url in the description below) which receives all the Beckn requests (search, select etc). This endpoint can either exist outside of your marketplace/shop software or within it. That is a design decision that will have to be taken by you based on the design of your existing marketplace/shop software. This component is also responsible for sending back the responses to the Beckn Adaptor.  
3. Install the BPP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL), a subscriber\_id (typically the same as subscriber URL without the "https://" prefix) and the webhook\_url that you configured in the step above. Also the address of the BPP Protocol Server Client will have to be configured in your component above. This address hosts all the response endpoints (on\_search,on\_select etc)  
4. Install the layer 2 file for the domain (Link is in the last section of this document)  
5. Check with your network tech support to enable your BPP Protocol Server in the registry.  
6. Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,  
   * you write the Provider Platform(dark blue) \- Here the component you wrote above in point 2 as well as your marketplace/shop software is together shown as Provider Platform  
   * install the BPP Protocol Server (light blue)  
   * the remaining components are provided by the sandbox environment  
   * Use the postman collection to test your Provider Platform  
7. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

![Provider platform testing sandbox][image4]

## **Links to artefacts**

* [Postman collection for UEI EV Charging](https://github.com/beckn/missions/blob/main/UEI/postman/ev-charging_uei_postman_collection.json)  
* [Layer2 config for UEI EV Charging](https://github.com/beckn/missions/blob/main/UEI/layer2/EV-charging/3.1/energy_EV_1.1.0_openapi_3.1.yaml)  
* When installing layer2 using Beckn-ONIX use this web address ([https://raw.githubusercontent.com/beckn/missions/refs/heads/main/UEI/layer2/EV-charging/3.1/energy\_EV\_1.1.0\_openapi\_3.1.yaml](https://raw.githubusercontent.com/beckn/missions/refs/heads/main/UEI/layer2/EV-charging/3.1/energy_EV_1.1.0_openapi_3.1.yaml))

## **Sandbox Details**

**![UEI Alliance][image5]**

### **BAP:**

* BAP Client Sandbox: [https://bap-ps-client-sandbox.ueialliance.org/](https://bap-ps-client-sandbox.ueialliance.org/)  
* BAP Network Sandbox: [https://bap-ps-network-sandbox.ueialliance.org/](https://bap-ps-network-sandbox.ueialliance.org/)

### **Registry/Gateway:**

* Gateway Sandbox: [https://gateway-sandbox.ueialliance.org/](https://gateway-sandbox.ueialliance.org/)  
* Registry Sandbox: [https://registry-sandbox.ueialliance.org/](https://registry-sandbox.ueialliance.org/)

### **BPP:**

* BPP Client Sandbox: [https://bpp-ps-client-sandbox.ueialliance.org/](https://bpp-ps-client-sandbox.ueialliance.org/)  
* BPP Network Sandbox: [https://bpp-ps-network-sandbox.ueialliance.org/](https://bpp-ps-network-sandbox.ueialliance.org/)  
* BPP Playground Sandbox: [https://bpp-playground-sandbox.ueialliance.org/](https://bpp-playground-sandbox.ueialliance.org/)

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,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>

[image3]: <data:image/png;base64,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>

[image4]: <data:image/png;base64,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>

[image5]: <data:image/png;base64,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EEIIIURGSGgJIYQQQmSEhJYQQgghREZIaAkhhBBCZISElhBCCCFERkhoCSGEEEJkhISWEEIIIURGSGgJIYQQQmSEhJYQQgghREZIaAkhhBBCZISElhBCCCFERkhoCSGEEEJkhISWEEIIIURGSGgJIYQQQmSEhJYQQgghREZIaAkhhBBCZISElhBCCCFERkhoCSGEEEJkhISWEEIIIURGSGgJIYQQQmSEhJYQQgghREZIaAlRByxYsCD68MMPC9aNGDHCfW7dutUth3bgwIGC/Yz3338/ts6+jxw50n2+++670aZNm/xdMuXXX3+N+vbtG1166aXR+PHjw81lMWPGjOibb76JXnnllejIkSPh5rIJy64c7r///nCVEKKbIqElRB1w6623Rk8//XTBuosuar89582bF11xxRXu07fjx48X7GfwPWmd/7lkyZJo7dq13h7Zwnk/+uij6IcffoiuvfbaWP7K4aabbor27NkT3XLLLU6MVotq5K0aaQghmgP9GwhRB3QmtG6++eaCbT5+o47niO9XXXVV9Oyzz8b2SRJamzdvjnr06BHdc8890cmTJ9268+fPR6NGjYoefvjh6PLLL4+2b9/u1sPgwYNdOhMmTMive+mll6LLLrssmjx5cn6d8cUXX8SEB2kaX375pfN2IZgQTsC5EWU33HBDwbVzff369YsefPDBAqG1YcMGd82s82HfSy65JL/+8OHD0e23357ffuedd0Y//vhj/jtYXn/77bdo2LBh0ZVXXhktXbrUreusXK6++uro+eefz6fxxx9/RL169XJls379+oJ9wzKcNm2ay+vUqVPz6wYNGhQ9+uij7vi9e/fm10P//v3d+fbv31+wXghRX0hoCVEHdCa0aNSnT59eYOF+gOigwaYrzV8fCi26x15++eWCdYgQW/7222/dMmLs2LFj+fUIF8TP77//Hl188cXRqlWrnBBA5ACCJkkUcjw2adKk6OzZs7FtiBrW23lIj2WECsIIEWj7IgwPHjzolk1o2b6+yOFz9erV0Z9//unyaesRM0OGDInuvvtuJzBD/ON37tzpjmcZQVasXCiL4cOHu31mzZpVkMa2bdsK0k0qw6FDh0YDBgxw2xFm1r3KMYjIXC7nljmGc7O8bt266KeffnLLv/zyi9tfCFF/XPgnFkJ0GbfddluHQouuw1dffbXAwv1smS5FjGUEjL+PffpCC4FCY4/3xLaboDDC44G0zYPG8YgazN/Hh+49BCXbw32IT0Nc2HqElsWQnThxwgkSxBWfBvuY0PK7QZPyGn5PyoPhr//000+jF154wa3j/GnKxf+OqCPPy5cvj20DK0NAuPFbs/99990X2xdxhifS8m7ljXCrVtybEKL6JP/TCCFqCkHqLS0tBeuskU3bdYgoYBkxg9HdhNfG38c+TWjhgWEdws0XESwjEozweB/WrVy5ssB8nnzySRe47sMxeIpsecqUKdFXX32VTx8RZcH+p0+fduID4eWfH8+PCS0Eh2Fph3n1v7Mcbjf8ayWonXJF6JrQSlMu/vfPP//c/X6sI3g/3BduvPFG10W5Zs0aJ8qShNZdd90VzZ07N593v7zxvAkh6pP4HS+EqDmICRpP80ARG2SNbFqhhRhhxKFh3Ur+PvZpQgsRhKcEfG9UMUHBOd555538MudD0NFdCYxm9L1OYALQ8AUdAfK2fOjQofxyktACPx2WTWhZXnfv3l1wrdaldurUqfx6Rj6OGzfOdRsicELCsrLlzoQW+Qf/XHyaoKScKOOkMmS/c+fO5ffzhRbdhf4y3ah+uqSxY8cOtyyEqD8u/JMIIbqUFStWuAYUQwwYTP1g632bM2eO2+436iGsCxt+wINGLJGtwxBfnHf06NHRd999lygo/P0ROAaNPetCkWXMnj27IO++MLBj6QJjRCIB3r7QOnPmTD7d//qv/8qnMXDgQBcgTj4QjbbehAnH++dEyIRdmyzjSfOx7Rb7xbmty65YuXz//ff587Dd1psXDsMr5h+HWRma2MKOHj2aP97WYX7X8q5du/LriTcTQtQv8X9mIYQQdYEJLiFE46K7WAgh6hQJLSEaH93FQgghhBAZIaElhBBCCJERElpCCCGEEBkhoSWEEEIIkRESWkIIIYQQGSGhJYQQQgiRERJaQgghhBAZIaElhBBCCJERElpCCCGEEBkhoSWEEEIIkRESWkIIIYQQGSGhJYQQQgiRERJaQgghhBAZIaElhBBCCJERElpCCCGEEBnx/wEIxUuZb3r41wAAAABJRU5ErkJggg==>