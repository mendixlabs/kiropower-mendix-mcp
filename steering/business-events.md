---
inclusion: manual
---

# Business Events (Kafka / Message Broker)

Use this when implementing event-driven communication between Mendix apps using Mendix Business Events (Kafka-backed pub/sub).

## Requirements

- **BusinessEvents** marketplace module installed in both publisher and subscriber apps
- Kafka broker configured (Mendix Cloud provides this; on-premise needs Confluent or similar)
- Mendix 9.6+

## Core Concepts

| Concept | Description |
|---------|-------------|
| Business Event Service | Defines the event schema — what messages look like |
| Published Event | Message a service emits (PUBLISH operation) |
| Consumed Event | Message a service listens to (SUBSCRIBE operation) |
| PBE_ entity | Entity that holds the event payload, extends `BusinessEvents.PublishedBusinessEvent` |

## Publisher Workflow

### Step 1: Create the backing entity

Create a persistent entity in the domain model:
- Name prefix: `PBE_` (e.g., `PBE_OrderShipped`)
- Generalization: `BusinessEvents.PublishedBusinessEvent`
- Attributes: must exactly match the event message attributes — no extra, no missing

```
ped_update_document("DomainModels$DomainModel", "MyModule", operations)
→ add entity PBE_OrderShipped with generalization BusinessEvents.PublishedBusinessEvent
→ add attributes: OrderId (String), ShippedDate (DateTime), TrackingNumber (String)
```

### Step 2: Publish from a microflow

In the publishing microflow:
1. Create a `PBE_OrderShipped` object
2. Set all attributes
3. Commit the object
4. Call `BusinessEvents.PublishBusinessEvent_V2` Java action with the committed object

The commit triggers the event emission — the Business Events module listens for commits on PBE_ entities.

## Subscriber Workflow

### Step 1: Create an event handler microflow

Create a microflow named `EVT_OrderShipped_Handle` that accepts the event entity as a parameter and processes it (e.g., update local records, trigger further logic).

### Step 2: Register the subscriber

In the Business Event Service document (configured in Studio Pro or via the portal), link the SUBSCRIBE message to the handler microflow.

## Entity Attribute Constraints

**Critical:** The `PBE_` entity attributes must exactly match the Business Event Service message definition:
- Same attribute names (case-sensitive)
- Same attribute types
- No additional attributes beyond those in the message

If they don't match, events will fail to serialize/deserialize.

## Supported Attribute Types

`String`, `Integer`, `Long`, `Decimal`, `Boolean`, `DateTime`

## MCP Tools

```
ped_update_document("DomainModels$DomainModel", "MyModule", ops)  → add PBE_ entity
ped_get_schema(["DomainModels$Entity"], kind: "constructor")      → entity schema
ped_create_document([{documentType: "Microflows$Microflow", ...}]) → create handler
ped_check_errors([...])                                           → validate
```

The Business Event Service document itself is configured in Studio Pro's Marketplace module UI, not via MCP tools.

## Checklist

- [ ] `BusinessEvents` marketplace module installed
- [ ] `PBE_` entity has `generalization: "BusinessEvents.PublishedBusinessEvent"`
- [ ] Entity attributes exactly match the event message definition
- [ ] Publishing microflow commits the object before calling `PublishBusinessEvent_V2`
- [ ] Subscriber microflow exists and is registered in the event service
- [ ] Run `ped_check_errors` on the domain model and handler microflow
