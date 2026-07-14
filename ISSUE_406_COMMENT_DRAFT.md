# COMMENT DRAFT FOR ISSUE #406 - Ready to Submit

## HGA Freight Forwarder Position: Subscription-Based Notifications Model

---

### **Operating Context**
Airsell Cargo - Freight Forwarder at Egal International Airport (HGA), Hargeisa, Somaliland

### **Our Position**

We agree with @aloccid-iata's clarification on subscription-based notifications. The answer to Issue #406 is **not binary (1 vs. 101)** — it depends on **subscription configuration** following the Publisher-Initiated Workflow outlined in the ONE Record 2025-07 API specification.

### **Operational Reality at HGA**

When Airsell consolidates 100+ individual shipments:

```
Consolidation Scenario:
├── Shipper A → House AWB-A → demands notification
├── Shipper B → House AWB-B → demands notification
├── [... 98 more shippers ...]
└── Master AWB-M created by Airsell → demands 1 notification

Current ambiguity:
❌ Do we send 101 notifications (noisy)?
❌ Do we send 1 notification only (incomplete)?

Correct answer:
✅ Send notifications per subscription settings
```

### **Recommended Implementation: Three-Tier Subscription Model**

Following the publisher-initiated workflow in section "Get Subscription information as Publisher":

#### **1. Airsell Freight Forwarder (Publisher)**
```
GET /subscriptions?topicType=LOGISTICS_OBJECT_TYPE&topic=Waybill
Query Parameters:
  - TopicType: LOGISTICS_OBJECT_TYPE
  - Topic: Waybill (MASTER only via filter)
  - SubscriptionEventType: LOGISTICS_OBJECT_CREATED, LOGISTICS_OBJECT_UPDATED

Response: 1 Subscription for MASTER AWBs
Result: Airsell receives 1 notification per MASTER creation
```

#### **2. Individual Shippers (Subscribers)**
```
POST /subscriptions?topicType=LOGISTICS_OBJECT_IDENTIFIER&topic=https://airsell.so/logistics-objects/house-awb-A
Query Parameters:
  - TopicType: LOGISTICS_OBJECT_IDENTIFIER
  - Topic: Specific House AWB URI
  - SubscriptionEventType: LOGISTICS_OBJECT_CREATED

Result: Shipper A receives 1 notification for their House AWB only
```

#### **3. HGA Customs Authority (Compliance)**
```
GET /subscriptions?topicType=LOGISTICS_OBJECT_TYPE&topic=Waybill
Query Parameters:
  - TopicType: LOGISTICS_OBJECT_TYPE
  - Topic: Waybill (ALL)
  - Filter: departureAirport=HGA

Result: Receives all waybills departing Hargeisa (regulatory compliance)
```

### **Key Advantages**

| Metric | Without Filtering | With Subscription Filtering |
|--------|------------------|----------------------------|
| **Notifications per consolidation** | 101 (noisy, expensive) | ~3 (efficient) |
| **System integration cost** | High (parse irrelevant data) | Low (receive only relevant) |
| **Customs compliance** | Manual collection | Automatic by filter |
| **Shipper confusion** | "Which AWB is mine?" | Direct subscription link |
| **Scalability** | Fails at 1000+ AWBs | Scales efficiently |

### **Compliance with Somaliland Regulations**

- ✅ **Each shipment tracked independently** — shipper-level subscriptions
- ✅ **Authorities monitor all departures** — customs-level subscriptions  
- ✅ **Audit trails** — subscription logs + notification delivery logs
- ✅ **Real-time visibility** — event-driven model

### **Request to ONE Record Team**

**Issue #406 should be clarified to emphasize**:

1. **Subscription filtering is the standard model** (not optional per provider)
2. **Notification scope = Subscription topic** (not determined by business logic)
3. **Publishers MUST respect subscription filters** when sending notifications
4. **No way to "opt out of HOUSE if subscribed to MASTER"** — subscriptions are independent

### **Specification Reference**

This approach is aligned with:
- ONE Record 2025-07 API Specification: "Get Subscription information as Publisher" (Section 2.2.2)
- `topicType` parameter: `LOGISTICS_OBJECT_TYPE` or `LOGISTICS_OBJECT_IDENTIFIER`
- `includeSubscriptionEventType`: `LOGISTICS_OBJECT_CREATED`, `LOGISTICS_OBJECT_UPDATED`

### **Implementation Commitment**

Airsell Cargo commits to:
- ✅ Implement subscription management per ONE Record specification
- ✅ Configure topic-based filtering for shippers, freight forwarders, and customs
- ✅ Maintain audit trails for all subscriptions and notifications
- ✅ Report operational feedback on this model

### **Conclusion**

The answer to "1 notification or 101 notifications?" is: **Neither — it depends on subscription settings.**

We recommend ONE Record **standardize and mandate** this subscription-filtering model for all implementations, especially critical for regional operators at smaller airports where compliance and shipper communication are paramount.

---

**Submitted by**: Airsell Cargo (Freight Forwarder, Egal International Airport, Hargeisa, Somaliland)  
**Date**: July 14, 2026  
**Reference Issue**: [#406](https://github.com/IATA-Cargo/ONE-Record/issues/406)
