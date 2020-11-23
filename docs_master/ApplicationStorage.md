# Application storage

Depending on features enabled during the IOWA build, IOWA can rely on the Application storage to store data.

Two main features rely on the Application storage:

* The Context storage,
* The Notification storage.

## Context storage

Context saving/restoring can be enabled with the flag `IOWA_STORAGE_CONTEXT_SUPPORT`. When this flag is set two additional platform functions must be defined: [`iowa_system_store_context()`](AbstractionLayer.md#iowa_system_store_context) and [`iowa_system_retrieve_context()`](AbstractionLayer.md#iowa_system_retrieve_context).

Note: Currently this feature is only available for the LwM2M Client.

### Client pseudo code

```c
#include "iowa_client.h"
#include "iowa_ipso.h"

int main(int argc,
         char *argv[])
{
    iowa_context_t iowaH;
    iowa_status_t result;
    iowa_device_info_t devInfo;
    iowa_sensor_t sensorId;

    /******************
    * Initialization
    */

    iowaH = iowa_init(NULL);

    devInfo.manufacturer = "IOTEROP";
    devInfo.deviceType = "Context Storage example device";
    devInfo.modelNumber = "1";
    devInfo.serialNumber = NULL;
    devInfo.hardwareVersion = NULL;
    devInfo.softwareVersion = NULL;
    devInfo.optFlags = 0;
    result = iowa_client_configure(iowaH, "IOWA_Sample_Client", devInfo, NULL);

    result = iowa_client_IPSO_add_sensor(iowaH,
                                         IOWA_IPSO_VOLTAGE, 12.0,
                                         "V", "Test DC", 0.0, 0.0,
                                         &sensorId);

    result = iowa_load_context(iowaH);
    if (result != IOWA_COAP_NO_ERROR)
    {
        result = iowa_client_add_server(iowaH, 1234, "coap://localhost:5683", 0, 0, IOWA_SEC_NONE);
    }

    /******************
    * "Main loop"
    */

    while (!quit
           && result == IOWA_COAP_NO_ERROR)
    {
        float sensorValue;

        result = iowa_step(iowaH, 5);

        sensorValue = read_battery_voltage();
        result = iowa_client_IPSO_update_value(iowaH,
                                               sensorId,
                                               sensorValue);
    }

    if (result == IOWA_COAP_NO_ERROR)
    {
        result = iowa_save_context_snapshot(iowaH);
    }

    iowa_client_IPSO_remove_sensor(iowaH, sensorId);
    iowa_close(iowaH);

    return 0;
}
```

### Saving the context

Context saving can be controlled through the APIs [`iowa_save_context()`](CommonAPI.md#iowa_save_context) and [`iowa_save_context_snapshot()`](CommonAPI.md#iowa_save_context_snapshot). Each call to theses APIs will call the platform function [`iowa_system_store_context()`](AbstractionLayer.md#iowa_system_store_context).

The data passed to the platform is a serialized non-encrypted version of the IOWA Context. The buffer / buffer length pair can, depending of the Application use case, be serialized without modification or be encrypted before the serialization.

The data is encoded using the CoAP option scheme which is a Type Length Value format with size optimizations. Each stored piece of information is prepended by a header between 1 and 4 bytes in size.

On the Client side, the size of the buffer to save will depend on the presence of a LwM2M Bootstrap Server Account and the number of LwM2M Server Accounts configured at the time the Context is saved. Using [`iowa_save_context_snapshot()`](CommonAPI.md#iowa_save_context_snapshot) saves additional information for each server, including registration information, observations and attributes.

#### Bootstrap Server and Server Accounts

Both account types are composed of:

| Data | Type | Bytes used |
| ------------------------------- | ------ | ---- |
| Server Short ID | uint16_t | 2 |
| Security Object Instance ID | uint16_t | 2 |
| Server Object Instance ID | uint16_t | 2 |
| Registration lifetime | int32_t | 4 |
| Binding mode | uint8_t | 1 |
| Default Minimum Period | uint32_t | 4 |
| Default Maximum Period | uint32_t | 4 |
| Notification Storing | bool | 1 |
| Bootstrap-Server | bool | 1|
| Disable Timeout | int32_t | 4 |
| Security Mode | uint8_t | 1 |
| Server URI | string | Variable (max length: 65535) |
| Server MSISDN | string | Variable (max length: 15) |

Summing up the fields except the last two ones, the maximum static size is 26 bytes.

The remaining variables do not have a static length and depend on the Server URI or Server MSISDN. The maximum length for a Server MSISDN is 15 bytes. For a Server URI, an approximation can be made by taking the following approach:

The Server URI can be usually decomposed into three parts: `[COAP_SCHEMA]://[HOSTNAME]:[PORT]`. With this decomposition in mind, the following Server URI handles 90% of the cases: `coaps://255.255.255.255:65535` which leads to an conservative size of 29 bytes.

The encoding adds an average overhead of 22 bytes per Server Account.

So, the LwM2M Bootstrap Server Account and LwM2M Server Accounts are mostly stored inside 77 bytes. Now let's take the following example:

1. A LwM2M Client is launched, no LwM2M server is configured but a LwM2M Bootstrap Server is added by calling [`iowa_client_add_bootstrap_server`](ClientAPI.md#iowa_client_add_bootstrap_server). The Server URI is: `coaps://217.182.95.250:5784`. If the Context is saved at this stage, the serialized buffer will have a length of 77 bytes.

2. After the Bootstrap procedure, a LwM2M Server Account has been added to the LwM2M Client with the Server URI: `coaps://18.195.192.63:5684`. If the Context is now saved, the serialized buffer will have a length of 77 bytes (LwM2M Bootstrap Server Account) + 80 bytes (LwM2M Server) = 157 bytes.

#### Access Control

If [**IOWA_SUPPORT_ACCESS_CONTROL_LIST_OBJECT**][IOWA_SUPPORT_ACCESS_CONTROL_LIST_OBJECT] is set.

| Data | Type | Bytes used |
| ------------------------------- | ------ | ---- |
| ACL Instance ID | uint16_t | 2 |
| Object ID | uint16_t | 2 |
| Instance ID | uint16_t | 2 |
| Owner ID | uint16_t | 2 |
| ACL Flags | Buffer | Variable |

ACL Flags may be present several times. The ACL Flags buffer is made of the following data:

| Data | Type | Bytes used |
| ------------------------------- | ------ | ---- |
| Server ID | uint16_t | 2 |
| Flags | uint8_t | 1 |

#### Runtime Information

When using [`iowa_save_context_snapshot()`](CommonAPI.md#iowa_save_context_snapshot), runtime information, including observations and attributes, will also be stored when the context is saved.

For LwM2M Server Account, runtime information are composed of:

| Data | Type | Bytes used |
| ------------------------------- | ------ | ---- |
| Runtime Status | uint8_t | 1 |
| Location | string | Variable |
| Retry Count | uint8_t | 1 |
| Sequence Retry Count | uint8_t | 1 |
| Attributes | Buffer | Variable |
| Observations | Buffer | Variable |
| Storage Queue | Buffer | Variable |

Both observations and attributes key may be present several times in order to serialize each observation and attributes of the desired LwM2M Server Account.

If **LWM2M_STORAGE_QUEUE_SUPPORT** or **LWM2M_STORAGE_QUEUE_PEEK_SUPPORT** flags are set, IOWA will call [`iowa_system_queue_backup()`](AbstractionLayer.md#iowa_system_queue_backup) and save the returned buffer. When loading the context, IOWA calls [`iowa_system_queue_restore()`](AbstractionLayer.md#iowa_system_queue_restore) with the saved buffer. The application must be able to restore the queue and the stored notifications using this buffer.

##### Attributes

For each URI with attributes, only set attributes will be saved. The size of this buffer depends on the LwM2M Client state.

| Data | Type | Bytes used |
| ------------------------------- | ------ | ---- |
| URI | uint8_t[] | max size: 8 |
| Minimum Period | uint32_t | 4 |
| Maximum Period | uint32_t | 4 |
| Greater Than | double | 8 |
| Less Than | double | 8 |
| Step | double | 8 |
| Minimum Evaluation Period | uint32_t | 4 |
| Maximum Evaluation Period | uint32_t | 4 |

##### Observations

Each observation will have a static size of 14 bytes. The remaining variables do not have a static length and depends on the URI and the number of token. Observation can also have several URI key. This leads to a maximum overhead of 8 bytes plus 9 bytes for each URI.

| Data | Type | Bytes used |
| ------------------------------- | ------ | ---- |
| URI | uint8_t[] | max size: 8 |
| URI flags | uint8_t | 1 |
| Content Format | uint16_t | 2 |
| Token | uint8_t[] | max size: 8 |
| Last Notification Time | uint32_t | 4 |
| Notification Counter | uint32_t | 4 |

### Restoring the context

On the other side, Context restoring can be controlled through the API [`iowa_load_context()`](CommonAPI.md#iowa_load_context). Each call to this API will call the platform function [`iowa_system_retrieve_context()`](AbstractionLayer.md#iowa_system_retrieve_context).

The data retrieved from the platform must be the serialized non-encrypted version of the IOWA Context. The buffer passed by argument has to be allocated by the Application. But the [`iowa_system_retrieve_context()`](AbstractionLayer.md#iowa_system_retrieve_context) function doesn't need to free the memory. This will be done by IOWA once the Context has been loaded into memory.

### Automatic Context saving

Automatic Context saving can be done when the flag `IOWA_STORAGE_CONTEXT_AUTOMATIC_BACKUP` is set. This feature only saves the Bootstrap Server and Server accounts just as [`iowa_save_context()`](CommonAPI.md#iowa_save_context).

On the Client, the Context is then saved when the following events occur:

* Calling the functions [`iowa_client_add_bootstrap_server`](ClientAPI.md#iowa_client_add_bootstrap_server), [`iowa_client_remove_bootstrap_server`](ClientAPI.md#iowa_client_remove_bootstrap_server), [`iowa_client_add_server`](ClientAPI.md#iowa_client_add_server) and [`iowa_client_remove_server`](ClientAPI.md#iowa_client_remove_server),
* After a successful Bootstrap sequence,
* A LwM2M Server writes a new value on the resources Lifetime (ID:1), Default Minimum Period (ID:2), Default Maximum Period (ID:3), Disable Timeout (ID:5), or Notification Storing When Disabled or Offline (ID:6) of the [`Server Object`][Server Object] (ID:1).

## Notification storage

Note: This feature is only available for the LwM2M Client. This is used by the Client when the Server is not available and the notifications need to be stored until the connection with the Server is reestablished.

Notification storing is enabled when the resource `Notification Storing When Disabled or Offline` is set to true in the Server object. This can be done either by using the API [`iowa_client_use_reliable_notifications()`](ClientAPI.md#iowa_client_use_reliable_notifications) or when a LwM2M Server writes in the resource `1/x/6`.

When notification storing is enabled, notifications are stored in Queue if one the flags `LWM2M_STORAGE_QUEUE_SUPPORT` or `LWM2M_STORAGE_QUEUE_PEEK_SUPPORT` is set.

When `LWM2M_STORAGE_QUEUE_SUPPORT` flag is set four additional system abstraction functions must be defined: [`iowa_system_queue_create()`](AbstractionLayer.md#iowa_system_queue_create), [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue), [`iowa_system_queue_dequeue()`](AbstractionLayer.md#iowa_system_queue_dequeue), and [`iowa_system_queue_delete()`](AbstractionLayer.md#iowa_system_queue_delete).

When `LWM2M_STORAGE_QUEUE_PEEK_SUPPORT` flag is set five additional system abstraction functions must be defined: [`iowa_system_queue_create()`](AbstractionLayer.md#iowa_system_queue_create), [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue), [`iowa_system_queue_peek()`](AbstractionLayer.md#iowa_system_queue_peek), [`iowa_system_queue_remove()`](AbstractionLayer.md#iowa_system_queue_remove), and [`iowa_system_queue_delete()`](AbstractionLayer.md#iowa_system_queue_delete).

When the `LWM2M_STORAGE_QUEUE_SUPPORT` or the `LWM2M_STORAGE_QUEUE_PEEK_SUPPORT` are defined, the system abstraction function [`iowa_system_queue_create()`](AbstractionLayer.md#iowa_system_queue_create) is called to create a queue to save notifications. This queue must be a FIFO (First In, First Out).
Likewise, when the Observation is cancelled by the Server, if a queue was created, the system abstraction function [`iowa_system_queue_delete()`](AbstractionLayer.md#iowa_system_queue_delete) is called to remove it.

As said earlier, the notifications are stored when the Server is no more reachable. A Server is considered no more available when:

- On a Registration message, no acknowledgement has been received,
- On a Registration Update message, no acknowledgement has been received,
- On a Confirmable Notification message, no acknowledgement has been received,
- After the Server triggers the resource Disable (ID: 4) of the [`Server Object`][Server Object] (ID:1).

### Saving a notification

The data passed to the platform are a serialized non-encrypted version of a Notification encoded with TLV Content Format. The pair buffer / buffer length can, depend on the Application use case, be serialized without modification or be encrypted before the serialization.

The size of the buffer to save will depend on the URI observed, the buffer is composed of:

| Data | Type | Bytes used |
| ---- | ---- | ---------- |
| Counter | uint32_t | 4 |
| Token length | uint8_t | 1 |
| Token | uint8_t[] | 1-8 |
| Serialized data | uint8_t[] | Variable |

Summing up the bytes used except the last one, the static size obtained is: 13 bytes. The remaining variable does not have a static length and highly depends on the URI observed. If the URI observed is an Object Instance, the *Serialized data* will have a longer buffer than if the URI was observed a Resource. But again it depends on the Resources data types.

To estimate the serialized data, the TLV specification should be used. The complete specification of this Content Format can be found in the part *6.4.3* of the document *OMA-TS-LightweightM2M-V1_0_2-20180209-A*.

With that in mind, let's take the following example:

An Observation is placed by the Server on the URI /3303/0. The Object Instance has the Resources:

- Sensor Value (ID:5700),
- Min Measured Value (ID:5601),
- Max Measured Value (ID:5602),
- Min Range Value (ID:5603),
- Max Range Value (ID:5604),
- Sensor Units (ID:5701).

On a Registration Update, no response is received from the Server and thus becomes no more reachable. The Client updates periodically the value of the Resource Sensor Value (ID:5700).

The first notification is saved into the Application Storage by calling:

- [`iowa_system_queue_create()`](AbstractionLayer.md#iowa_system_queue_create) to create the Queue,
- [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue) to save the serialized Notification.

Each subsequent call will only call [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue).

An example of the serialized Data can be:

```
Payload: 41 bytes
  E4 16 44 41  BC 71 C4 E4  15 E1 41 BB  29 2C E4 15   |..DA.q....A.),..|
  E2 41 C6 96  1C E3 16 45  43 65 6C E4  15 E3 C1 A0   |.A.....ECel.....|
  00 00 E4 15  E4 42 48 00  00                         |.....BH..|
```

| Type byte (hex) | ID byte(s) (hex) | Length byte(s) | Value | Total bytes |
| --------- | ---------- | -------------- | ----- | ----------- |
| E4 | 16 44 (ID:5700) | No length | 0x41BC71C4 | 7 |
| E4 | 15 E1 (ID:5601) | No length | 0x41BB292C | 7 |
| E4 | 15 E2 (ID:5602) | No length | 0x41C6961C | 7 |
| E3 | 16 45 (ID:5701) | No length | 0x43656C | 6 |
| E4 | 15 E3 (ID:5603) | No length | 0xC1A00000 | 7 |
| E4 | 15 E4 (ID:5604) | No length | 0x42480000 | 7 |

The total length of the serialized Notification passed to the application will be: 4 bytes (Counter) + 1 byte (Token length) + 8 bytes maximum (Token) + 41 bytes (Serialized data) = 54 bytes.

If the Observation was on the URI /3303/0/5700:

The serialized Data would have been:

```
Payload: 7 bytes
  E4 16 44 41  BC 71 C4                                |..DA.q.|
```

| Type byte (hex) | ID byte(s) (hex) | Length byte(s) | Value | Total bytes |
| --------- | ---------- | -------------- | ----- | ----------- |
| E4 | 16 44 (ID:5700) | No length | 0x41BC71C4 | 7 |

As said earlier, for proper estimation of the Serialized data, the TLV specification should be used to estimate the serialization length depending on which URI the Observation will be placed.

### Loading a stored notification

On the other side, IOWA reads frequently the stored Notifications. System abstraction functions are called only if the queue has been created on the Application side.

When `LWM2M_STORAGE_QUEUE_SUPPORT` is defined, reading the stored notifications is done by calling the system abstraction function [`iowa_system_queue_dequeue()`](AbstractionLayer.md#iowa_system_queue_dequeue). Then the system abstraction function [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue) may be called depending if the following conditions are met:

- The Observation has no parameter, and the Server is not reachable. The value will be put back into the Queue.
- The Observation has parameters such as Minimum Period, Maximal Period, etc, and the Server is not reachable. If the conditions are not met, the value is put back into the Queue.

When `LWM2M_STORAGE_QUEUE_PEEK_SUPPORT` is used, reading the stored notification is done by calling the system abstraction function [`iowa_system_queue_peek()`](AbstractionLayer.md#iowa_system_queue_peek). Then the system abstraction function [`iowa_system_queue_remove()`](AbstractionLayer.md#iowa_system_queue_remove) is called if the notification was successfully received by the Server.

Once the Server becomes reachable, the Queue is emptied and all the saved Notifications are sent to the Server.

Note: When the Server becomes reachable, the Queue is not deleted once it's emptied. It will only be deleted when the Observation is cancelled.