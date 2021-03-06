---
title: Authenticate event delivery to event handlers (Azure Event Grid)
description: This article describes different ways of authenticating delivery to event handlers in Azure Event Grid. 
ms.topic: conceptual
ms.date: 01/07/2021
---

# Authenticate event delivery to event handlers (Azure Event Grid)
This article provides information on authenticating event delivery to event handlers. 

## Overview
Here are the different authentication methods that are used to deliver events to event handlers. 

| Authentication method | Supported handlers | Description  |
|--|--|--|
Access key | <ul><li>Event Hubs</li><li>Service Bus</li><li>Storage Queues</li><li>Relay Hybrid Connections</li><li>Azure Functions</li><li>Storage Blobs (Deadletter)</li></ul> | Access keys are fetched using Event Grid credentials. The permissions are granted to Event Grid when you register the Event Grid resource provider in their Azure subscription. |  
Managed System Identity <br/>&<br/> Role-based access control |  <ul><li>Event Hubs</li><li>Service Bus</li><li>Storage Queues</li><li>Storage  Blobs (Deadletter)</li></ul> | Enable managed system identity for the topic and add it to the appropriate role on the destination. See the following articles:<br/><ul><li>[Enable managed identity for a system topic](enable-identity-system-topics.md)</li><li>[Enable managed identity for a custom topic or a domain](enable-identity-custom-topics-domains.md)</li><li>[Grand identity the access to Event Grid destination](add-identity-roles.md)</li><li>[Create an event subscription that uses the identity](managed-service-identity.md)</li></ul> |
|Bearer token authentication with Azure AD protected webhook | Webhook | See [Create an app registration in Azure AD & use role-based access control](secure-webhook-delivery.md). |
Client secret as a query parameter | Webhook | Webhook URL must contain access keys or secrets in the URL. See [Using client secret as a query parameter](#using-client-secret-as-a-query-parameter). |

## Use system-assigned identities for event delivery
You can enable a system-assigned managed identity for a topic or domain and use the identity to forward events to supported destinations such as Service Bus queues and topics, event hubs, and storage accounts.

Here are the steps: 

1. Create a topic or domain with a system-assigned identity, or update an existing topic or domain to enable identity. 
1. Add the identity to an appropriate role (for example, Service Bus Data Sender) on the destination (for example, a Service Bus queue).
1. When you create event subscriptions, enable the usage of the identity to deliver events to the destination. 

For detailed step-by-step instructions, see [Event delivery with a managed identity](managed-service-identity.md).


## Authenticate event delivery to webhook endpoints
The following sections describe how to authenticate event delivery to webhook endpoints. Use a validation handshake mechanism irrespective of the method you use. See [Webhook event delivery](webhook-event-delivery.md) for details. 


### Using Azure Active Directory (Azure AD)
You can secure the webhook endpoint that's used to receive events from Event Grid by using Azure AD. You'll need to create an Azure AD application, create a role and service principal in your application authorizing Event Grid, and configure the event subscription to use the Azure AD application. Learn how to [Configure Azure Active Directory with Event Grid](secure-webhook-delivery.md).

### Using client secret as a query parameter
You can also secure your webhook endpoint by adding query parameters to the webhook destination URL specified as part of creating an Event Subscription. Set one of the query parameters to be a client secret such as an [access token](https://en.wikipedia.org/wiki/Access_token) or a shared secret. Event Grid service includes all the query parameters in every event delivery request to the webhook. The webhook service can retrieve and validate the secret. If the client secret is updated, event subscription also needs to be updated. To avoid delivery failures during this secret rotation, make the webhook accept both old and new secrets for a limited duration before updating the event subscription with the new secret. 

As query parameters could contain client secrets, they are handled with extra care. They are stored as encrypted and are not accessible to service operators. They are not logged as part of the service logs/traces. When retrieving the Event Subscription properties, destination query parameters aren't returned by default. For example: [--include-full-endpoint-url](/cli/azure/eventgrid/event-subscription#az_eventgrid_event_subscription_show) parameter is to be used in Azure [CLI](/cli/azure).

For more information on delivering events to webhooks, see [Webhook event delivery](webhook-event-delivery.md)

> [!IMPORTANT]
> Azure Event Grid only supports **HTTPS** webhook endpoints. 

## Endpoint validation with CloudEvents v1.0
If you're already familiar with Event Grid, you might be aware of the endpoint validation handshake for preventing abuse. CloudEvents v1.0 implements its own [abuse protection semantics](webhook-event-delivery.md) by using the **HTTP OPTIONS** method. To read more about it, see [HTTP 1.1 Web Hooks for event delivery - Version 1.0](https://github.com/cloudevents/spec/blob/v1.0/http-webhook.md#4-abuse-protection). When you use the CloudEvents schema for output, Event Grid uses the CloudEvents v1.0 abuse protection in place of the Event Grid validation event mechanism. For more information, see [Use CloudEvents v1.0 schema with Event Grid](cloudevents-schema.md). 


## Next steps
See [Authenticate publishing clients](security-authenticate-publishing-clients.md) to learn about authenticating clients publishing events to topics or domains. 
