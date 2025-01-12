---
id: overview
title: Flow Manager Router
sidebar_label: Overview
---
The `Flow Manager Router` behaves as a proxy of events for the flow manager: based on the `Main Flow Manager` saga Id, the service send the request to the correct `Sub Flow Manager`.

The body of the request is the same event body expected by flow manager service.

:::info
This service communicates via REST by default, but it can also rely on KAFKA if the environment variable `MODE` is set to `KAFKA`.
:::

## Endpoints

The plugin exposes the following endpoints:
- `/saga`: create new saga on both the main and the sub flow managers
- `/event`: send an event to the sub flow manager
- `/notify`: send an event to the main flow manager and an optional external service
- `/status`: retrieve the status of both main and sub flow managers. Below is available an example of the response:
```json
{
  "sagaId": "main_flow_saga_id",
  "mainSaga": {
    "currentState": "main_flow_state",
    "businessStateDescription": "main_flow_business_state"
  },
  "identificationSaga": {
    "currentState": "identification_flow_state",
    "businessStateDescription": "identification_flow_business_state"
  }
}
```
- `/sub-flow/saga`: retrieve the sub flow saga from the ID of the main saga

## Architecture
The `Flow Manager Router` plugin allow you to manage different flow managers and used them as a unique flow, in particular you can define:
- `Main Flow Manager`
- a set of `Sub Flow Manager`

Based only on the `Main Flow Manager` saga ID, the  `Flow Manager Router` sends the events to the correct `Sub Flow Manager`.

The `Sub Flow Manager` can send an event to the `Flow Manager Router` that will be sent to the `Main Flow Manager` and an optional external service. 
Only data inside the `MainFlowData` object on the `Sub Flow Manager` metadata are included in the message payload of the event sent to both the `Main Flow Manager` and an optional external service.

## Sub Flow matching rules
In order to identify the correct `Sub Flow Manager` to send the event to is necessary to define a configuration file with a list of rules. 

A rule has the following properties:
- `id`: string to identify the rule
- `info`: an object with the information about the flow manager service to use and the related collection
- `rules`: an object with the values that needs to match in order to use the rule

The first valid rule found will be used to retrieve the needed information. The rules are checked in order.

A rule is considered verified if all the keys defined in the object `rules` are present also in the `Main Flow Manager` metadata and the associated values are equals.

More detail about the schema of the configuration file are available on the [dedicated section](./20_configuration.md).

## Metadata Upsert
The router service perform an upsert on the metadata sent to each flow manager: that means that object not in the first level of the metadata are not overwritten but instead are merged in order to not lost any information between the new metadata in the event and the already existing metadata.
Different merging strategies can be defined to merge array objects; you can define which policy use with the `ARRAY_MERGE_MODE` environment variable:
- `ALL_NEW`: keep only the incoming elements
- `SHA`: depending on the type of the elements inside the array:
  - if the elements are objects an upsert will be performed on the objects based on a key defined within the object. The final array will then contain only values with a unique key. The reference key must be defined inside a `sha` field of type string.
  - if at least one element does not contain a `sha` field or the elements are not objects then keep only the incoming elements
  

## Notify
The `/notify` endpoint is used to send an event to the main flow manager and an optional external service.
the router send the same messageLabel defined by the incoming event but only an object defined in the event payload: you can define the object key in the `MAIN_FLOW_DATA_KEY` environment variable.
As last action the router service send to the `Sub Flow Manager` an event with the `routerNotifyCompleted` label.

## Retry on http requests failure
This service will make call to CRUD service and a generic external service, if defined.
Through environment variables MAX_RETRIES and RETRIES_DELAY_MS, a retry system can be defined in case of network or connection failure.

:::warning
The total delay time in case of all retries are performed should not exceed 30 seconds, due to timeout issues.
:::
