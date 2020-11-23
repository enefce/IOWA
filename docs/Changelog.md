# Changelog

## 2020-09.1

** Behavior Changes **

* When an Observation is set on asynchronous resources, the Client [event callback][iowa_event_callback_t] is called with an [IOWA_EVENT_READ][iowa_event_type_t] event.
* When using mbed TLS, the MTU is declared during the DTLS/TLS handshake.

** SDK Changes **

* [`iowa_server_configuration_set()`][iowa_server_configuration_set] is deprecated and renamed [`iowa_client_set_server_configuration()`][iowa_client_set_server_configuration].
* `iowa_dimmer.h` can now be imported without importing first `iowa_ipso.h`.

** Bug Fixes **

* Fix a possible core dump when IOWA reconnects a disconnected CoAP peer.
* Fix a but where the Client was rejecting an Object Instance creation using TLV if the payload was not containing the new Object Instance ID.
* Fix a possible memory leak if the registration reply from the Server does not contain a location path.
* Avoid a compilation error when [IOWA_DEVICE_RSC_FACTORY_RESET_REMOVE][Device] is defined.
* When using mbed TLS, fix a bug preventing to send a payload larger than 16384 bytes over TLS.

## 2020-09

** New Features **

* Add two APIs [`iowa_client_get_server_count()`][iowa_client_get_server_count] and [`iowa_client_get_server_array()`][iowa_client_get_server_array] to retrieve the bootstrapped and declared LwM2M Servers.
* Add an API [`iowa_server_configuration_set()`][iowa_server_configuration_set] to update the lifetime and the queue mode of a LwM2M Server.
* Add an API [`iowa_client_update_device_information()`][iowa_client_update_device_information] to update the device information at runtime.
* Add a new API [`iowa_clock_reset()`][iowa_clock_reset] to inform IOWA of clock issues.
* The API [`iowa_client_set_device_error_code()`][iowa_client_set_device_error_code] now allow user-defined error codes.

** Behavior Changes **

* For LwM2M Server applications, IOWA now takes care of the reassembly of CoAP block transfers.
* Events **IOWA_EVENT_OBSERVATION_STARTED** and **IOWA_EVENT_OBSERVATION_CANCELED** are now emitted after having updated the internal IOWA context.

- CoAP block-wise transfers use the negotiated block size during all the block-wise transfer.
- In the AT Command Object, the strings passed to [`iowa_client_at_command_set_response()`][iowa_client_at_command_set_response] are appended to the existing ones instead of replacing them.
- In CoAP messages, URI-Path option is not included when URI is "/".
- In the LwM2M layer, MAX_TRANSMIT_WAIT has a duration of 20 seconds for TCP peers and 120 seconds for LoRaWAN peers.
- The return value of [`iowa_system_security_data()`][iowa_system_security_data] is no longer checked when IOWA calls it for an **IOWA_SEC_FREE** operation.
- Add multiple parameters checks in Objects APIs.
- In [`iowa_client_light_control_set_state()`][iowa_client_light_control_set_state], [`iowa_system_gettime()`][iowa_system_gettime] is called only if the resource "On Time" is enabled.
- The LwM2M Client now ignores unexpected CoAP responses.

** SDK Changes **

* Remove Dimmer object from IPSO objects and add its own API.
* All [`iowa_apn_connection_profile_details_t`][iowa_apn_connection_profile_details_t] integer members are now unsigned.
* [`iowa_device_time_info_t::currentTime`][iowa_device_time_info_t] was changed from `uint32_t` to `int32_t`.
* Included mbed TLS was updated to version 2.24.0.

** Bug Fixes **

* Fix a bug when calling [`iowa_flush_before_pause()`][iowa_flush_before_pause] on TCP connections.
* Fix a possible deadlock when calling [`iowa_system_random_vector_generator()`][iowa_system_random_vector_generator].
* Fix a possible core dump when reading a Resource Instance containing a nil value.
* Fix a buffer overflow in serialization of integer CoAP option with a number greater than 269.
* Fix a bug when parsing large RFC8323 COAP headers.
* For Bootstrap Server applications, fix a bug where the unexpected disconnection of a TCP Client may lead to an invalid read.
* Fix a potential invalid read if the Client fails to send its registration message.
* Fix a possible bug when generating a new token.
* Fix several bugs when the Client uses CoAP block transfers for the registration message.
* For LwM2M Server applications, fix a potential stack corruption when the connection type is one of UDP, SMS, or LoRaWAN, and when iowa_system_random_vector_generator() fails.
* Fix a possible memory leak when creating a new CoAP peer.
* Fix a possible bug when generating a new token.
* In several Objects, fix a potential invalid read when string Resources under Observation are modified repeatedly.
* In the Location and GPS Objects, fix a potential segmentation fault when allocating memory for a string value fails.
* Fix a possible memory leak when deleting an APN Object Instance.
* Fix a potential invalid free when calling [`iowa_client_send_sensor_data()`][iowa_client_send_sensor_data].
* Fix a Bootstrap-Delete bug when ACL are enabled.

## 2020-06.2

** Bug Fixes **

* Fix an interoperability bug in SenML JSON related to base64 encoding of opaque values.
* Fix a potential invalid read when deleting an ACL Object instance.
* Fix a potential memory leak when creating a new CoAP peer fails.
* Correct several conversion compiler warnings.
* Fix an offset error when parsing long CoAP over TCP headers.
* Fix interoperability issue with CoAP FETCH method when using Block-Transfer.
* Fix a possible memory leak when deleting an APN Object Instance.

## 2020-06.1

** Behavior Changes **

* For RFC7252 CoAP connections, e.g. LwM2M over UDP, the initial message ID is now random (using [`iowa_system_random_vector_generator()`][iowa_system_random_vector_generator]).
* The LwM2M Registration will now fail if the transmission was successful but no response was received after a delay set to EXCHANGE_LIFETIME from RFC7252.

** Bug Fixes **

* Fix a bug when the Current Time resource was read asynchronously.

## 2020-06

** New Features **

* Add support of the MQTT Broker Object (ID: 18330) and of the MQTT Publication Object (ID: 18831). See [MQTT Object API][MQTT Object API].
* The CoAP retransmission parameters can be modified at runtime. See [`iowa_coap_peer_configuration_set()`][iowa_coap_peer_configuration_set] and [`iowa_coap_peer_configuration_get()`][iowa_coap_peer_configuration_get].
* New APIs to retrieve the CoAP peer associated to a LwM2M Server to ease usage of the previous APIs. See [`iowa_client_get_server_coap_peer()`][iowa_client_get_server_coap_peer] and [`iowa_client_get_bootstrap_server_coap_peer()`][iowa_client_get_bootstrap_server_coap_peer].
* Support of the "Reset Error Code" Resource of the [Device Object][Device Object].
* For Client applications, add a new event when an notification for an Observation was generated and is about to be sent. See **IOWA_EVENT_OBSERVATION_NOTIFICATION** in [iowa_event_type_t][iowa_event_type_t].
* For Client applications, the events related to Observations now include the ID of the specific resource under observation if any.
* Introduce a new special value **IOWA_DEVICE_TIME_SENSOR_ID** of an [`iowa_sensor_t`][iowa_sensor_t] to refer to the Current Time of the Device.

** Behavior Changes **

* The API [`iowa_client_notification_lock()`][iowa_client_notification_lock] now also prevents registration updates.
* The **IOWA_EVENT_BS_PENDING** event is emitted when connecting to the LwM2M Bootstrap Server instead of after receiving the reply to the Bootstrap-Request operation.
* The **IOWA_EVENT_REG_REGISTERING** event is emitted when connecting to the LwM2M Server instead of after sending the Registration message.
* Returned error codes for Object related APIs are more consistent.
* Stricter checks of the parameters passed to [`iowa_client_IPSO_add_sensor()`][iowa_client_IPSO_add_sensor].
* Improved OSCORE support for a better interoperability.

** SDK Changes **

* [`iowa_client_device_update_battery()`][iowa_client_device_update_battery] can only be called when IOWA is built WITHOUT the flag **IOWA_DEVICE_RSC_BATTERY_REMOVE**.
* [`iowa_client_..._device_power_source()`][iowa_client_add_device_power_source] can only be called when IOWA is built WITHOUT the flag **IOWA_DEVICE_RSC_POWER_SOURCE_REMOVE**.
* The file `src/core/iowa_comms.c` was removed.
* **INVALID_SENSOR_ID** is deprecated and renamed **IOWA_INVALID_SENSOR_ID**.

** Bug Fixes **

* Fix a bug where an intermediate runtime state of a LwM2M Server was saved when calling [`iowa_save_context()`][iowa_save_context], preventing resuming.
* Fix a bug when creating multiple Magnetometer instances with different resources.
* Fix a bug when creating multiple Light Control instances with different resources.
* Fix a bug when reading an unknown Resource Instance.
* Fix a possible memory leak if `iowa_coap_peer_new()` failed.
* Fix a possible segmentation fault when an Object Resource value is a nil-string.
* Fix a bug when receiving a COAP block-transfer message and neither **IOWA_COAP_BLOCK_MINIMAL_SUPPORT** nor **IOWA_COAP_BLOCK_SUPPORT** are defined.
* Fix a bug when updating Magnetometer Object's compass direction.

## 2020-03.6

** Bug Fixes **

* Fix a bug where an intermediate runtime state of a LwM2M Server was saved when calling [`iowa_save_context()`][iowa_save_context], preventing resumption.

## 2020-03.5

** Bug Fixes **

* Fix a bug where switching the resource "Notification Storing When Disabled or Offline" to false would stop the Notification queue flushing.
* Fix a bug where Notifications of canceled Observations were not always removed from the queue in peek mode, blocking other notifications.

** Behavior Changes **

* When receiving an Execute operation with arguments, the LwM2M Client will assume the arguments are encoded using text/plain when the CoAP option content-type is not present.

## 2020-03.4

** Bug Fixes **

* Fix a race condition issue where simultaneous registration update may generate a registration update failure event.

## 2020-03.3

** Bug Fixes **

* Fix a memory leak when receiving a block transfer.

## 2020-03.2

** Bug Fixes **

* Fix a potential issue with the size of the last block of a streamable resource.

## 2020-03.1

** Bug Fixes **

* Fix a potential deadlock in iowa_client_object_set_mode() when the sensor ID was unknown.
* Fix a possible wrong behaviour in the FSM of the registration when the Device is registered on the Server but not connected on the Security level.

## 2020-03

** New Features **

* For Client applications, add a new event when the *epmin* and *epmax* attributes are modified by the Server. See **IOWA_EVENT_EVALUATION_PERIOD** in [iowa_event_type_t][iowa_event_type_t].
* Add new helper functions to convert an `iowa_sensor_t` to an `iowa_lwm2m_uri_t` and vice versa. See [`iowa_utils_uri_to_sensor()`][iowa_utils_uri_to_sensor].
* Add a new event when the LwM2M Server requests a Read of a LwM2M Objects or custom LwM2M Objects resources which are defined as "asynchronous". See [`iowa_client_object_set_mode()`][iowa_client_object_set_mode] and **IOWA_EVENT_READ** in [iowa_event_type_t][iowa_event_type_t].
* In custom LwM2M Objects, resources can be defined as streamable. See [Custom Object Streaming APIs][Custom Object Streaming APIs].
* Add support of the "Factory Reset" resource of the Device Object. See[`iowa_client_factory_reset_callback_t`][iowa_client_factory_reset_callback_t].
* Add a new CoAP API [`iowa_coap_block_request_block_number()`][iowa_coap_block_request_block_number] to ease resuming of Firmware Update download.
* Add new helper functions to retrieve the security session associated to a LwM2M Server or a LwM2M Client. See [`iowa_security_get_server_session()`][iowa_security_get_server_session].
* Expose IOWA linked list functions. See [Example: Linked List usage][Example: Linked List usage].

** Behavior Changes **

* If the registration using LwM2M 1.1 fails, the Client tries to register using LwM2M 1.0.
* During the Bootstrap procedure, if the LwM2M Bootstrap Server did not provide the security credentials of a LwM2M Server, IOWA checks if credentials are known by the Application. See [iowa_security_operation_t][iowa_security_operation_t].
* There is now one notifications storage queue per LwM2M Server instead of one per Observation. Unsent reliable notifications are no longer stored in memory.
* **IOWA_LWM2M_ID_ALL** can be used as parameter in [`iowa_client_set_notification_default_periods()][iowa_client_set_notification_default_periods].
* [`iowa_client_remove_server()`][iowa_client_remove_server] called with **IOWA_LWM2M_ID_ALL** as parameter, also removes bootstrapped LwM2M Servers.
* Better handling of NaN and Infinity floating-point values in CBOR.

** SDK Changes **

* IOWA has now internal checks done by using the `assert()` macro.
* Add a new flag to use the libc `snprintf()` when converting to text large floating-point values. See [IOWA_USE_SNPRINTF][IOWA_USE_SNPRINTF].
* Rename the header file "iowa_security_user.h", "iowa_security.h".
* Clean up cross inclusion of header files.
* Deprecate several compilation flags. See [Deprecated Compilation Flags][Deprecated Compilation Flags].
* Remove some functions deprecated in previous releases: `iowa_client_add_lorawan_server()`, `iowa_server_dm_read()`, `iowa_server_dm_observe()`, `iowa_server_dm_observe_cancel()`, `iowa_server_dm_write()`, `iowa_server_dm_write_attributes()`, and `iowa_server_set_content_format()`.
* Prototype of `iowa_bearer_selection_update_state_callback_t`, `iowa_client_bearer_selection_update`, `iowa_client_cellular_connectivity_update`, `iowa_cellular_connectivity_update_state_callback_t` and `iowa_client_connectivity_monitoring_update` have been updated to use a pointer to access to the structure.

** Bug Fixes **

* Fix a bug where the Client was not always resending its new LwM2M Object if a previous Registration Update failed.
* Fix a bug where the Client was not always resetting its internal registration sequence counters, when registering to a LwM2M 1.1 Server.
* Improve the Client registration process resilience to network disconnections.
* Fix text and JSON encoding of very large float values.
* Fix the usage of the URI query in [`iowa_coap_peer_get()`][iowa_coap_peer_get].
* Keep the previously declared LwM2M Bootstrap Server when failing to load a context snapshot containing a LwM2M Bootstrap Server.
* Fix pushing the firmware package with CoAP block which was rejected by the LwM2M Client.
* Fix an issue where modifying the Objects list or the lifetime during registration, or while disconnected, was not always triggering a Registration Update.
* Fix a possible derivation between the Registration timer and the Registration Update timer.

## 2019-12.1

** Bug Fixes **

* Allow the Context Storage feature to be built with Single Server mode.
* Use the correct defines to remove the resources UTC Offset and Timezone in the Device Object.
* Allow the ACL feature to be built with Single Server mode.
* Strengthen the checking of the loaded context version.
* Update the Power Source defines to reflect the version 1.1 of the Device Object.

## 2019-12

** New Features **

* Add the possibility to choose LwM2M protocol version to support. See [LwM2M Version][LwM2M Version].
* LwM2M 1.1: The LwM2M operations Read, Write, Write-Attribute, and Observe can target Resource Instances.
* LwM2M 1.1: Add support of the *epmin* and *epmax* notification attributes.
* LwM2M 1.1: Add support of the Read-Composite, Write-Composite, and Observe-Composite operations.
* LwM2M 1.1: Add support of the LwM2M 1.1 Send operation. See the samples **samples/client_1_1** and **samples/server_1_1**.
* LwM2M 1.1: Add support of the new registration conditions. See [iowa_client_set_server_registration_behaviour][iowa_client_set_server_registration_behaviour].
* LwM2M 1.1: Add support of the resource type Unsigned Integer.
* LwM2M 1.1: Add support of the Bootstrap Trigger resource.
* LwM2M 1.1: Add support of the Bootstrap Read operation.
* Add new CoAP APIs to ease downloading during Firmware Update. See [CoAP API Reference][CoAP API Reference] and the sample **samples/fw_update_client**.
* Add support of the Access Control List Object. See [Access Control List Object API][Access Control List Object API].
* Add support of the Error Code resource in the Device Object. See [iowa_client_set_device_error_code][iowa_client_set_device_error_code].
* Add new API for the application to provide data to be saved with the IOWA context. See [iowa_backup_register_callback][iowa_backup_register_callback].
* The runtime information can be stored, allowing the Client to resume its operations seamlessly. See [Context Storage][Context Storage].

** Behavior Changes **

* Rework of the context storage feature: The format of the saved data has changed for a more flexible one.
* MSISDN can no longer be set on the Client side when SMS support is not enabled.

** SDK Changes **

* The functions of the Security component are no longer exposed in the header file *iowa_security.h*. See [Security Component][Security Component].
* Deprecated APIs: `iowa_client_add_lorawan_server()` and `iowa_server_set_content_format`.

** Advanced Features **

These features have not undergone full interoperability testings.

* LwM2M 1.1: Add support of OSCORE security. See [Security][Security].

## 2019-08

** New Features **

* During Firmware Update Pushs, a time limit can be set for blocks reception. See the ["Push" method]["Push" method] section in [Device Update][Device Update].
* New Client API to set the Bootstrap Hold Off Time. See [iowa_client_set_bootstrap_server_hold_off][iowa_client_set_bootstrap_server_hold_off].
* New Server APIs to validate a Client registration ([iowa_server_set_verify_client_callback][iowa_server_set_verify_client_callback]) and to close a connection to a Client ([iowa_server_close_client_connection][iowa_server_close_client_connection]).

** Behavior Changes **

* For LwM2M Clients, if [iowa_step][iowa_step] returns an error because no LwM2M Server can be reached, subsequent call to [iowa_step][iowa_step], will retry to register to the known LwM2M Servers, instead of staying in the failed state.
* Changes were made to the [Context storage][Context storage] feature making the saved contexts no longer compatible with the one of the previous version.
* The "Binding" resource of the [Device Object][Device Object] is now dependant on the compilation flags.

** SDK Changes **

* Change the way provide a custom security implementation. See [Providing your security implementation][Providing your security implementation].
* The [iowa_lwm2m_data_t][iowa_lwm2m_data_t] definition no longer depends on the [LWM2M_SUPPORT_TIMESTAMP][LWM2M_SUPPORT_TIMESTAMP] compilation flag.
* More compilation flags were added to reduce the code footprint of the LwM2M Client version. See [SDK Configuration][Configuration].
* Add a new log part to filter logs generated by LwM2M Objects. See [IOWA_LOG_LEVEL and IOWA_LOG_PART][IOWA_LOG_LEVEL and IOWA_LOG_PART].
* Update the provided MbedTLS to version 2.18.1.
* Samples are now usable on Windows. See [Samples Compilation][Samples Compilation].

** Experimental Features **

These features have not undergone interoperability testings. They are provided for prototyping purpose only.

* Add the possibility to choose LwM2M version to support.
* The LwM2M operations Read, Write, Write-Attribute, and Observe can target Resource Instances.
* Add support of the *epmin* and *epmax* notification attributes.
* Add support of the LwM2M 1.1 operations Read-Composite, Write-Composite, and Observe-Composite.
* Add support of the LwM2M 1.1 Send operation.

## 2019-05

** New Features **

* Support of timestamped values in IPSO Objects and custom Objects. See [`iowa_client_IPSO_update_values`][iowa_client_IPSO_update_values] and [Custom Object][Monitoring the temperature during a period] section.
* Support of SenML JSON, SenML CBOR and CBOR data encodings.
* Support of the [Software Management Objects][Software Management] for FOTA.
* Support of the power source and timezone information in the LwM2M Device Object. See [`iowa_client_add_device_power_source`][iowa_client_add_device_power_source] and [`iowa_client_update_device_time_information`][iowa_client_update_device_time_information].
* Support of CoAP Block-Wise transfer for the Device Update push method. See **IOWA_COAP_BLOCK_SUPPORT** and **IOWA_COAP_BLOCK_MINIMAL_SUPPORT** in [IOWA Configuration][IOWA Configuration].
* **Experimental** support of the TCP transport. This feature has not undergone interoperability testings.

** Behavior Changes **

* In the custom Objects [iowa_RWE_callback_t][iowa_RWE_callback_t], new operation **IOWA_DM_FREE** to allow the release of the memory potentially allocated when the callback was called with **IOWA_DM_READ**.
* In the security layer, the certificates are parsed in DER format.
* New APIs to manipulate the [APN Connection Profile Object][APN Connection Profile Object]. Previous ones are marked as deprecated but are still functionnal.
* The user application can implement its own log functions. See [Logger Component][Logger Component].

** SDK Changes **

* All iowa files had their name prepended with `iowa_` to avoid filename collisions on some build systems.
* Numerous compilation flags were added to reduce the code footprint of the LwM2M Client version. See [SDK Configuration][Configuration].
* Using the Firmware Update feature now requires the definition of the compilation flag **IOWA_SUPPORT_FIRMWARE_UPDATE_OBJECT**. See [Firmware Update Object][Firmware Update Object].
* APIs related to iowa-supported LwM2M Objects were moved in their separate header and source files. The header files are located in the "include/objects" folder.
* [`iowa_lwm2m_data_type_t`][iowa_lwm2m_data_type_t] enumeration was converted to a list of defines.
* the `const` keyword was added to several APIs arguments.
* Remove client build dependency on files object_connectivity_stats.c and object_light_control.c
* The abstraction layer function `iowa_system_connection_is_same_peer()` is no longer required.