## 1. Background
There are 2 entities:
- BRPL
- Bus Depot

Lets consider 2 cases.
- case1: Bus Depot as a BAP and BRPL as a BPP
- case2: BPRL as a BAP and Bus Depot as a BPP

The first case seems more logical but lets analyse both the cases.

## 2. Case1: Bus Depot as a BAP and BRPL as a BPP

### 2.1 Supporting discovery and subscription of DF programs before getting DF requests.

Flow:
- Bus Depots can discover and register to DF programs.
  - Search - confirm APIs can be used.
- BPRL sending a DF request: must be an /on_update request with request type as "query", and containing subscription id.
- Bus depot sending a reply: must be an /update request.
- BRPL sends DFPCC catalog to Bus depot: must be an /on_update request with request type as "reminder", and containing subscription id.
- Bus depot sending a reply: must be an /update request.

pros:
- supports discovery and subscription of DF programs by the clients.
- Each DF request is against a subscription.

cons:
- forced to use lesser used Beckn protocol constructs
  - unsolicited /on_update request
  - transaction started by BPP

### 2.2 Sending DF requests without subscriptions

Flow:
- BPRL sending a DF request: an /on_init request with request type as "query".
- Bus depot sending a reply: must be an /confirm request.
- BRPL sends DFPCC catalog to Bus depot: must be an /on_init request with request type as "reminder".
- Bus depot sending a reply: must be an /confirm request.
- BPRL sends confirms client participation in that DF instance,noting time period, clients promises, attaching an id to this participation.
- clients participation is evaluated using this stored state, and incentive is calculated on the basis of this object.
- client checks incentive status against a participation using status API.
- Grid sends participation status, incentive status using on_status API.

pros:
- supports a wide range of actions.

cons:
- client registration to a subscription is not supported.

### 2.3 Sending DF requests with discovery and subscription

Flow:
- Clients can discover a list of DF programs using search API.
- Client can subscribe to a DF program using confirm API (with flag: `request_type: "program_subscription"`).
- BPRL sending a DF request: an /on_init request with request type as "query".
- Bus depot sending a reply: must be an /confirm request (with flag: `request_type: "event_participation"`).
- BRPL sends DFPCC catalog to Bus depot: must be an /on_init request with request type as "reminder".
- Bus depot sending a reply: must be an /confirm request (with flag: `request_type: "event_participation"`).
- BPRL confirms client participation in that DF instance,noting time period, clients promises, attaching an id to this participation. sends an /on_confirm
- clients participation is evaluated using this stored state, and incentive is calculated on the basis of this object.
- client checks incentive status against a participation using status API.
- Grid sends participation status, incentive status using on_status API.

pros:
- supports discovery and subscription of DF programs.
- supports a wide range of actions.
- clear differentiation between subscription and participation confirmations.

cons:
- requires additional API parameters to differentiate confirm calls.

## 3. Case2: BRPL as a BAP and Bus Depot as a BPP

### 3.1 Analysis of BRPL as BAP approach

Making BRPL a BAP does not seem logical for the following reasons:

**1. BRPL has seller characteristics:**
- BRPL offers the DF programs and sets the incentive prices
- These are typically seller/provider features in the Beckn ecosystem
- BAPs are usually buyers/consumers, not sellers

**2. Scalability concerns:**
- If BRPL is a BAP and initiates DF events, it would need to call too many BPPs
- This becomes particularly problematic when utility household consumer-related DFs are activated
- The number of BPP calls would scale with the number of participating consumers, creating performance and reliability issues

**Conclusion:** The BRPL as BAP approach is not recommended due to its misalignment with typical Beckn roles and the scalability challenges it presents.

