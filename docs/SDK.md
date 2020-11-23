# IOWA Software Development Kit

## Content

doc
: The IOWA APIs reference document.

externals
: Open source code used by IOWA.

include
: The IOWA header files.

src
: The IOWA source code.

samples
: Sample applications.

## Samples

IOWA SDK comes with several sample applications.

### Samples Compilation

#### On Linux

**Prerequisites:** An x86-64 computer with a Linux distribution installed, the `cmake` utility, the `make` utility and a C compiler.

1. Create a build folder

   `mkdir build`

2. Go to this folder

   `cd build`

3. Launch cmake in debug mode

   `cmake -DCMAKE_BUILD_TYPE=Debug ..`

   (the last parameter point to the folder containing the CMakeLists.txt file of your target. In this case the one at the root of the repo including all the samples)

4. Build the client and the server

   `make -j 4`

   ( the `-j 4` parameter enables four parallel compilations)

After making some modifications to the code, only the step 4 is required.

#### On Windows

##### Using Visual Studio Code

1. Install the Microsoft C++ compiler as explained here: https://docs.microsoft.com/en-us/cpp/build/building-on-the-command-line?view=vs-2019
   1. Select the "Build Tools for Visual Studio 2019".
   2. In the Installer, make sure the following optional features are checked:
      * MSVC v142 - VS 2019 C++ x64/x86 build tools (Note that the version may differ)
      * Windows 10 SDK
      * C++ CMake tools for Windows
1. Install Visual Studio Code from https://code.visualstudio.com/
1. Launch Visual Studio Code.
1. Go to the "Extensions" panel (Ctrl+Shift+X) on the left side.
1. Install the "C/C++", "CMake", and "CMake Tools" extensions
1. Open the folder containing the IOWA Samples ("File" menu -> "Open Folder..." or "Ctrl+K Ctrl+O")
1. Open the CMake panel on the left side.
1. On the top bar of the CMake panel, click on the icon "Configure All Projects".
1. When prompted to select a kit, choose one of the Visual Studio Build Tools.
1. On the top bar of the CMake panel, click on the icon "Build All Projects".
1. Click on the sample of your choice.
1. Right-click on the application and select "Run in terminal"

##### Using Visual Studio with C++ support.

Visual Studio version must be at least 2017 for the CMake support.

1. In the Visual Studio menu bar, go to "File", "Open", "Folder". Select the IOWA Samples folder.
1. In the "Solution Explorer" windows, right-click on "CMakeList.txt" and choose "Set as Startup Item".
1. In the Visual Studio menu bar, go to "Build", "Build All"

### Testing

#### Without Security

- Launch the previously built 'server'.
- Launch the previously built 'client' on the same computer.

#### With PSK security

- Launch the 'server' previously built.
- Edit the *samples/client/main.c* file to go from:
```c
    // Add the information of a LwM2M Server to connect to
    result = iowa_client_add_server(iowaH, SERVER_SHORT_ID, SERVER_URI, 300, 0, IOWA_SEC_NONE);
    // or if you want to use a secure connection
    // result = iowa_client_add_server(iowaH, SERVER_SHORT_ID, SERVER_SECURE_URI, 300, 0, IOWA_SEC_PRE_SHARED_KEY);
```
to:
```c
    // Add the information of a LwM2M Server to connect to
    // result = iowa_client_add_server(iowaH, SERVER_SHORT_ID, SERVER_URI, 300, 0, IOWA_SEC_NONE);
    // or if you want to use a secure connection
    result = iowa_client_add_server(iowaH, SERVER_SHORT_ID, SERVER_SECURE_URI, 300, 0, IOWA_SEC_PRE_SHARED_KEY);
```
- Rebuild and launch 'client'.

## Build System

If you are using **cmake** as your build system, you can include the file `iowa.cmake` from the `src` folder. This file defines several variables:

- **IOWA_HEADERS** is the list of all the header files of IOWA,
- **IOWA_SOURCES** is the list of all the source files of IOWA,
- **IOWA_INCLUDE_DIR** is the path to IOWA header files.

Similar to the defines above, `iowa.cmake` provides additional defines if you only build a IOWA LwM2M Client or a IOWA LwM2M Server:

- **IOWA_CLIENT_HEADERS** is the list of all the header files of IOWA Client,
- **IOWA_CLIENT_SOURCES** is the list of all the source files of IOWA Client,
- **IOWA_CLIENT_DIR** is the path to IOWA Client header files,
- **IOWA_SERVER_HEADERS** is the list of all the header files of IOWA Server,
- **IOWA_SERVER_SOURCES** is the list of all the source files of IOWA Server,
- **IOWA_SERVER_DIR** is the path to IOWA Server header files.

Likewise, the folder `externals` contains the files `mbedtls.cmake` and `tinydtls.cmake` which define similar variables to compile mbed TLS or TinyDTLS.

## Configuration

IOWA relies on the compilation flags described below to enable or disable features. These compilation flags can be set in the header file `iowa_config.h`. Every source file of IOWA includes this header file.

To create your product, a template is provided in the `include` folder.

### Platform Configuration

#### LWM2M_BIG_ENDIAN and LWM2M_LITTLE_ENDIAN

Define one and only one of these two to specify the endianness of your platform.

#### IOWA_BUFFER_SIZE

When using a packet switching transport (e.g. UDP), IOWA stores the received data in a static buffer. **IOWA_BUFFER_SIZE** defines the size in bytes of this static buffer.

#### IOWA_USE_SNPRINTF

When using a text content format, IOWA uses snprintf to serialize float which absolute value is greater than INT64_MAX.

### IOWA Configuration

#### Transports

IOWA can support various transports. These transports are enabled by defining the following:

**IOWA_UDP_SUPPORT**
  : Support for UDP transport. URI scheme is in the form "coap://" or "coaps://".

**IOWA_TCP_SUPPORT**
  : Support for TCP transport. URI scheme is in the form "coap+tcp://" or "coaps+tcp://".

**IOWA_SMS_SUPPORT**
  : Support for SMS transport. URI scheme is in the form "coap+sms://" for text SMS or "sms://" for binary SMS.

**IOWA_LORAWAN_SUPPORT**
  : Support for LoRaWAN transport. URI scheme is in the form "lorawan://".

##### Additional flags

**IOWA_COAP_BLOCK_SUPPORT**
  : Support for full packet fragmentation at CoAP level as defined in RFC 7959 "Block-Wise Transfers in the Constrained Application Protocol (CoAP)".

**IOWA_COAP_BLOCK_MINIMAL_SUPPORT**
  : Support for reassembly of fragmented packets at CoAP level. Automatically defined with **IOWA_COAP_BLOCK_SUPPORT**. Useful for constrained devices using the "Push" method of [Device Update][Device Update].

**IOWA_COAP_OSCORE_SUPPORT**
  : Support for security at the CoAP message level using Object Security for Constrained RESTful Environments (RFC 8613).

#### IOWA_SECURITY_LAYER

IOWA can use different DTLS/TLS stacks to secure communication between LwM2M Clients and Servers. The possible values for this define are:

**IOWA_SECURITY_LAYER_NONE**
  : No security features can be used. This is the default value if **IOWA_SECURITY_LAYER** is not defined.

**IOWA_SECURITY_LAYER_USER**
  : To provide your security stack. Refer to the [Providing your security implementation][Providing your security implementation] section for details.

**IOWA_SECURITY_LAYER_MBEDTLS**
  : Use mbed TLS as the DTLS/TLS stack. The sources of mbed TLS are provided in the `externals/mbedtls` folder.

**IOWA_SECURITY_LAYER_MBEDTLS_OSCORE_ONLY**
  : Use mbed TLS as the cryptographic stack restricted to OSCORE mode. The sources of mbed TLS are provided in the `externals/mbedtls` folder.

**IOWA_SECURITY_LAYER_MBEDTLS_PSK_ONLY**
  : Use mbed TLS as the DTLS/TLS stack restricted to Pre-Shared Key mode. The sources of mbed TLS are provided in the `externals/mbedtls` folder.

**IOWA_SECURITY_LAYER_TINYDTLS**
  : Use tinydtls as the DTLS/TLS stack. The sources of tinydtls are provided in the `externals/mbedtls` folder. Note that tinydtls does not handle certificate based security modes.

If security is in use, the platform abstraction functions [`iowa_system_security_data()`](AbstractionLayer.md#iowa_system_security_data) and [`iowa_system_random_vector_generator()`](AbstractionLayer.md#iowa_system_random_vector_generator) must be implemented.

Be aware, security increases the ROM/RAM footprints. The mbed TLS or tinyDTLS layer have been configured to keep a good ratio between the ROM/RAM footprints and optimal security. But on constrained devices, the Flash used is small and designed to be optimal (fully used). Thus, if the security layer exceeds the footprints, additional defines can be set depending of the device. These defines are outside of the scope of IOWA.

For mbed TLS layer, the following defines can be set in `externals/mbedtls/include/mbedtls/config.h`:

- **MBEDTLS_AES_ROM_TABLES**: Store the pre-computed AES tables in the ROM instead of the RAM. It can reduce RAM usage by ~8kb, but increase ROM usage by ~8kb.
- **MBEDTLS_AES_FEWER_TABLES**: Store a smaller pre-computed AES tables in the ROM/RAM. It can reduce the usage by ~6kb, and thus pre-computed AES tables cost only ~2kb.

#### IOWA_LOG_LEVEL and IOWA_LOG_PART

These defines configure the traces provided by IOWA. Obviously, having more traces increase the code footprint of IOWA. Note that all traces are provided to the platform abstraction function [`iowa_system_trace()`](AbstractionLayer.md#iowa_system_trace).

**IOWA_LOG_LEVEL** possible values are:

* **IOWA_LOG_LEVEL_NONE**: No traces are generated. This is the default value if **IOWA_LOG_LEVEL** is not defined.
* **IOWA_LOG_LEVEL_ERROR**: Only the most critical errors are reported like calling IOWA APIs with wrong parameters or memory allocation failures.
* **IOWA_LOG_LEVEL_WARNING**: Recoverable errors are also reported.
* **IOWA_LOG_LEVEL_INFO**: IOWA reports information on major steps of its execution. This is the recommended setting during integration.
* **IOWA_LOG_LEVEL_TRACE**: IOWA reports every step of its execution. It is advised to use this value only to provide details when contacting the support.

**IOWA_LOG_PART** is useful to restrict traces to some components of IOWA. It is a combination of the following:

* **IOWA_PART_BASE** IOWA APIs and execution.
* **IOWA_PART_COAP** the CoAP layer.
* **IOWA_PART_COMM** the communication handling.
* **IOWA_PART_DATA** the serialization/deserialization payload packet handling.
* **IOWA_PART_LWM2M** the Lightweight M2M layer.
* **IOWA_PART_OBJECT** the object layer.
* **IOWA_PART_SECURITY** the security layer.
* **IOWA_PART_SYSTEM** the platform abstraction layer.

Additionally, **IOWA_PART_ALL** is defined as enabling traces of all components. This is the default value if **IOWA_LOG_PART** is not defined.

#### IOWA_THREAD_SUPPORT

When using IOWA in a multithreaded system, defining **IOWA_THREAD_SUPPORT** enables thread safety in IOWA.

This feature requires the platform abstraction functions [`iowa_system_connection_interrupt_select()`](AbstractionLayer.md#iowa_system_connection_interrupt_select), [`iowa_system_mutex_lock()`](AbstractionLayer.md#iowa_system_mutex_lock), and [`iowa_system_mutex_unlock()`](AbstractionLayer.md#iowa_system_mutex_unlock) to be implemented.

#### IOWA_STORAGE_CONTEXT_SUPPORT

IOWA can save and restore its context through the APIs [`iowa_save_context()`](CommonAPI.md#iowa_save_context), [`iowa_save_context_snapshot()`](CommonAPI.md#iowa_save_context_snapshot) and [`iowa_load_context()`](CommonAPI.md#iowa_load_context). This feature requires this compilation flag to be set.

This feature allows external data to be saved and restored with the context through callback. Callbacks can be added and deleted through the APIs [`iowa_backup_register_callback()`](CommonAPI.md#iowa_backup_register_callback) and [`iowa_backup_deregister_callback()`](CommonAPI.md#iowa_backup_deregister_callback).

This feature requires the platform abstraction functions [`iowa_system_store_context()`](AbstractionLayer.md#iowa_system_store_context) and [`iowa_system_retrieve_context()`](AbstractionLayer.md#iowa_system_retrieve_context) to be implemented.

#### IOWA_STORAGE_CONTEXT_AUTOMATIC_BACKUP

When this flag is set, IOWA would save the LwM2M Client context after every modification by a LwM2M Server or LwM2M Bootstrap Server. This feature does not save the server's runtime information. This is only relevant when IOWA is in Client mode and requires **IOWA_STORAGE_CONTEXT_SUPPORT** to be set.

#### IOWA_CONFIG_SKIP_SYSTEM_FUNCTION_CHECK

This define allows disabling system function checks such as memory allocation. It assumes that system functions can not fail. This define is useful to reduce the code footprint.

#### IOWA_CONFIG_SKIP_ARGS_CHECK

This define allows disabling check functions' arguments. It assumes that the functions' arguments are valid. This define is useful to reduce the code footprint.

#### IOWA_LOGGER_USER

This define allows implementing your Logger's functions. If not defined, use the IOWA Logger implementation.

If this define is set, the platform abstraction functions [`iowa_log()`](LoggerAPI.md#iowa_log), [`iowa_log_arg()`](LoggerAPI.md#iowa_log_arg), [`iowa_log_buffer()`](LoggerAPI.md#iowa_log_buffer) and [`iowa_log_arg_buffer()`](LoggerAPI.md#iowa_log_arg_buffer) must be implemented.

#### IOWA_PEER_IDENTIFIER_SIZE

This is only relevant when IOWA is in Server and/or Bootstrap Server mode. This define is used to set the maximum size of the peer identifier on the network. This is used when the endpoint name is not found in the registration payload and the stack calls [`iowa_system_connection_get_peer_identifier`](AbstractionLayer.md#iowa_system_connection_get_peer_identifier).

### LwM2M Configuration

#### LwM2M Role

Lightweight M2M defines three possible roles for the elements of a LwM2M system: Client, Server, or Bootstrap Server. You can define the role of your device by defining one of
**LWM2M_CLIENT_MODE**, **LWM2M_SERVER_MODE**, or **LWM2M_BOOTSTRAP_SERVER_MODE**.

##### LWM2M_BOOTSTRAP

This is only relevant when IOWA is in Client mode. This allows the LwM2M Client to be configured by a LwM2M Bootstrap Server.

#### LwM2M Version

##### LWM2M_VERSION_1_0_REMOVE

This disables the default Lightweight M2M version of this stack: LwM2M version 1.0.

##### LWM2M_VERSION_1_1_SUPPORT

This enables the support of the LwM2M version 1.1.

#### LwM2M Data Encoding

##### LWM2M_SUPPORT_JSON

This enables the support of JSON encoding for LwM2M payload. This support is optional for LwM2M Clients and mandatory for LwM2M Servers. Thus, the feature is enabled automatically when **LWM2M_SERVER_MODE** or **LWM2M_BOOTSTRAP_SERVER_MODE** are set. Note that JSON is a verbose encoding in LwM2M.

Note that JSON encoding is deprecated in LwM2M 1.1.

##### LWM2M_SUPPORT_SENML_JSON

This enables the support of SenML JSON encoding for LwM2M 1.1 payload. This support is optional for LwM2M Clients and mandatory for LwM2M Servers:

- For LwM2M Clients, at least one of the following format must be supported: SenML CBOR or SenML JSON if **LWM2M_VERSION_1_1_SUPPORT** is set.
- For LwM2M Servers, the support is enabled automatically when **LWM2M_SERVER_MODE** or **LWM2M_BOOTSTRAP_SERVER_MODE** are set.

Note that SenML JSON is a verbose encoding in LwM2M.

##### LWM2M_SUPPORT_CBOR

This enables the support of CBOR encoding for LwM2M 1.1 payloads containing a single resource value. This support is optional for LwM2M Clients and mandatory for LwM2M Servers. This support is enabled automatically when **LWM2M_SERVER_MODE** or **LWM2M_BOOTSTRAP_SERVER_MODE** are set.

##### LWM2M_SUPPORT_SENML_CBOR

This enables the support of SenML CBOR encoding for LwM2M 1.1 payload. This support is optional for LwM2M Clients and mandatory for LwM2M Servers:

- For LwM2M Clients, at least one of the following format must be supported: SenML CBOR or SenML JSON if **LWM2M_VERSION_1_1_SUPPORT** is set.
- For LwM2M Servers, the support is enabled automatically when **LWM2M_SERVER_MODE** or **LWM2M_BOOTSTRAP_SERVER_MODE** are set.

##### LWM2M_SUPPORT_TLV

This enables the support of TLV encoding for LwM2M payload.

This support is mandatory in LwM2M 1.0, this means that this feature is automatically set if **LWM2M_VERSION_1_0_REMOVE** is NOT set.

In LwM2M 1.1 and later, this support is optional for LwM2M Clients and mandatory for LwM2M Servers. Thus, the feature is enabled automatically when **LWM2M_SERVER_MODE** or **LWM2M_BOOTSTRAP_SERVER_MODE** are set if **LWM2M_VERSION_1_0_REMOVE** is set.

Note that TLV encoding is deprecated in LwM2M 1.1.

#### LWM2M_STORAGE_QUEUE_SUPPORT

When a LwM2M Server observing some resources is not reachable, the LwM2M Client stores the notifications until the connectivity is restored. By default, IOWA stores the last notifications in memory. When this flag is set, IOWA discharges the storage of these notifications to the platform.

This feature requires the system abstraction functions [`iowa_system_queue_create()`](AbstractionLayer.md#iowa_system_queue_create), [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue), [`iowa_system_queue_dequeue()`](AbstractionLayer.md#iowa_system_queue_dequeue), and [`iowa_system_queue_delete()`](AbstractionLayer.md#iowa_system_queue_delete) to be implemented.

#### LWM2M_STORAGE_QUEUE_PEEK_SUPPORT

When a LwM2M Server observing some resources is not reachable, the LwM2M Client stores the notifications until the connectivity is restored. By default, IOWA stores the last notifications in memory. When this flag is set, IOWA discharges the storage of these notifications to the platform. New version using a peek/remove mechanism instead of a dequeue mechanism.

This feature requires the system abstraction functions [`iowa_system_queue_create()`](AbstractionLayer.md#iowa_system_queue_create), [`iowa_system_queue_enqueue()`](AbstractionLayer.md#iowa_system_queue_enqueue), [`iowa_system_queue_peek()`](AbstractionLayer.md#iowa_system_queue_peek), [`iowa_system_queue_remove()`](AbstractionLayer.md#iowa_system_queue_remove), and [`iowa_system_queue_delete()`](AbstractionLayer.md#iowa_system_queue_delete) to be implemented.

#### LWM2M_SUPPORT_TIMESTAMP

This enables the support of the timestamp for notifications. Timestamp can only be used with the following Content format:

* JSON with [`LWM2M_SUPPORT_JSON`][LWM2M_SUPPORT_JSON]
* SenML JSON with [`LWM2M_SUPPORT_SENML_JSON`][LWM2M_SUPPORT_SENML_JSON]
* SenML CBOR with [`LWM2M_SUPPORT_SENML_CBOR`][LWM2M_SUPPORT_SENML_CBOR]

#### LWM2M_ALTPATH_SUPPORT

By default, the LwM2M Objects are located under the root path. However, devices might be hosting other CoAP Resources on an endpoint, and there may be the need to place LwM2M Objects under an alternate path.

This define allows the use of the Alternate Path. For LwM2M Servers, the support is enabled automatically when **LWM2M_SERVER_MODE** is set.

#### LWM2M_CLIENT_INCOMING_CONNECTION_SUPPORT

This is only relevant when IOWA is in Client mode. When set, this define enables the [`iowa_client_new_incoming_connection()`](ClientAPI.md#iowa_client_new_incoming_connection) API.

#### LWM2M_CLIENT_ASYNCHRONOUS_OPERATION_SUPPORT

This is only relevant when IOWA is in Client mode. When this compilation flag is activated, operations on LwM2M Objects can be declared as asynchronous. See [`iowa_client_object_set_mode()`](ClientAPI.md#iowa_client_object_set_mode).

#### Composite operations

The composite operations are automatically enabled when **LWM2M_SERVER_MODE** and **LWM2M_VERSION_1_1_SUPPORT** are set.

Those features require that either **LWM2M_SUPPORT_SENML_JSON** or **LWM2M_SUPPORT_SENML_CBOR** are set.

##### LWM2M_READ_COMPOSITE_SUPPORT

This enables the support of the Read-Composite operation defined in LwM2M 1.1.

##### LWM2M_OBSERVE_COMPOSITE_SUPPORT

This enables the support of Observe-Composite operation and Read-Composite operation defined in LwM2M 1.1. This means that **LWM2M_READ_COMPOSITE_SUPPORT** will be automatically set.

##### LWM2M_WRITE_COMPOSITE_SUPPORT

This enables the support of the Write-Composite operation defined in LwM2M 1.1.

#### LWM2M_DATA_PUSH_SUPPORT

This enables the support of data push operation defined in LwM2M 1.1. This feature requires that either **LWM2M_SUPPORT_SENML_JSON** or **LWM2M_SUPPORT_SENML_CBOR** are set.

### Objects Configuration

#### OMA Objects

##### IOWA_SUPPORT_ACCESS_CONTROL_LIST_OBJECT

This enables the support of Access Control List OMA Object. Refer to the [`Access Control List Object`][Access Control List Object] for details.

##### IOWA_SUPPORT_FIRMWARE_UPDATE_OBJECT

This enables the support of Firmware Update OMA Object. Refer to the [`Firmware Update Object`][Firmware Update Object] for details.

##### IOWA_FIRMWARE_UPDATE_MAX_BLOCK_INTERVAL

When the LwM2M Client supports the "Push" method of [Device Update][Device Update], and  **IOWA_COAP_BLOCK_MINIMAL_SUPPORT**, this defines the maximum time in seconds to wait between blocks before considering the connection as lost.

##### IOWA_SUPPORT_SOFTWARE_COMPONENT_OBJECT

This enables the support of Software Component OMA Object. Refer to the [`Software Component Object`][Software Component Object] for details.

##### IOWA_SUPPORT_SOFTWARE_MANAGEMENT_OBJECT

This enables the support of Software Management OMA Object. Refer to the [`Software Management Object`][Software Management Object] for details.

#### APN Connection Profile

By default, all the resources of the APN Connection Profile Object are enabled. But they can be disabled by using the following defines:

- **IOWA_APN_CONNECTION_PROFILE_RSC_APN_REMOVE**: Disable the resource "APN" (Id: 1)
- **IOWA_APN_CONNECTION_PROFILE_RSC_AUTO_SELECT_APN_DEVICE_REMOVE**: Disable the resource "Auto select APN by device" (Id: 2)
- **IOWA_APN_CONNECTION_PROFILE_RSC_ENABLE_STATUS_REMOVE**: Disable the resource "Enable status" (Id: 3)
- **IOWA_APN_CONNECTION_PROFILE_RSC_USER_NAME_REMOVE**: Disable the resource "User Name" (Id: 5)
- **IOWA_APN_CONNECTION_PROFILE_RSC_SECRET_REMOVE**: Disable the resource "Secret" (Id: 6)
- **IOWA_APN_CONNECTION_PROFILE_RSC_RECONNECT_SCHEDULE_REMOVE**: Disable the resource "Reconnect Schedule" (Id: 7)
- **IOWA_APN_CONNECTION_PROFILE_RSC_VALIDITY_REMOVE**: Disable the resource "Validity (MCC, MNC)" (Id: 8)
- **IOWA_APN_CONNECTION_PROFILE_RSC_CONN_ESTABLISHMENT_TIME_REMOVE**: Disable the resource "Connection establishment time" (Id: 9)
- **IOWA_APN_CONNECTION_PROFILE_RSC_CONN_ESTABLISHMENT_RESULT_REMOVE**: Disable the resource "Connection establishment result" (Id: 10)
- **IOWA_APN_CONNECTION_PROFILE_RSC_CONN_ESTABLISHMENT_REJECT_CAUSE_REMOVE**: Disable the resource "Connection establishment reject cause" (Id: 11)
- **IOWA_APN_CONNECTION_PROFILE_RSC_CONNECTION_END_TIME_REMOVE**: Disable the resource "Connection end time" (Id: 12)
- **IOWA_APN_CONNECTION_PROFILE_RSC_TOTAL_BYTES_SENT_REMOVE**: Disable the resource "TotalBytesSent" (Id: 13)
- **IOWA_APN_CONNECTION_PROFILE_RSC_TOTAL_BYTES_RECEIVED_REMOVE**: Disable the resource "TotalBytesReceived" (Id: 14)
- **IOWA_APN_CONNECTION_PROFILE_RSC_IP_ADDRESS_REMOVE**: Disable the resource "IP address" (Id: 15)
- **IOWA_APN_CONNECTION_PROFILE_RSC_PREFIX_LENGTH_REMOVE**: Disable the resource "Prefix length" (Id: 16)
- **IOWA_APN_CONNECTION_PROFILE_RSC_SUBNET_MASK_REMOVE**: Disable the resource "Subnet mask" (Id: 17)
- **IOWA_APN_CONNECTION_PROFILE_RSC_GATEWAY_REMOVE**: Disable the resource "Gateway" (Id: 18)
- **IOWA_APN_CONNECTION_PROFILE_RSC_PRIMARY_DNS_ADDRESS_REMOVE**: Disable the resource "Primary DNS address" (Id: 19)
- **IOWA_APN_CONNECTION_PROFILE_RSC_SECONDARY_DNS_ADDRESS_REMOVE**: Disable the resource "Secondary DNS address" (Id: 20)
- **IOWA_APN_CONNECTION_PROFILE_RSC_QCI_REMOVE**: Disable the resource "QCI" (Id: 21)
- **IOWA_APN_CONNECTION_PROFILE_RSC_TOTAL_PACKETS_SENT_REMOVE**: Disable the resource "TotalPacketsSent" (Id: 23)
- **IOWA_APN_CONNECTION_PROFILE_RSC_PDN_TYPE_REMOVE**: Disable the resource "PDN Type" (Id: 24)
- **IOWA_APN_CONNECTION_PROFILE_RSC_APN_RATE_CONTROL_REMOVE**: Disable the resource "APN Rate Control" (Id: 25)

#### Bearer Selection

By default, all the resources of the Bearer Selection Object are enabled. But they can be disabled by using the following defines:

- **IOWA_BEARER_SELECTION_RSC_PREFERRED_COMM_BEARER_REMOVE**: Disable the resource "Preferred Communications Bearer" (Id: 0)
- **IOWA_BEARER_SELECTION_RSC_ACCEPTABLE_RSSI_GSM_REMOVE**: Disable the resource "Acceptable RSSI (GSM)" (Id: 1)
- **IOWA_BEARER_SELECTION_RSC_ACCEPTABLE_RSCP_UMTS_REMOVE**: Disable the resource "Acceptable RSCP (UMTS)" (Id: 2)
- **IOWA_BEARER_SELECTION_RSC_ACCEPTABLE_RSRP_LTE_REMOVE**: Disable the resource "Acceptable RSRP (LTE)" (Id: 3)
- **IOWA_BEARER_SELECTION_RSC_ACCEPTABLE_RSSI_EV_DO_REMOVE**: Disable the resource "Acceptable RSSI (1xEV-DO)" (Id: 4)
- **IOWA_BEARER_SELECTION_RSC_CELL_LOCK_LIST_REMOVE**: Disable the resource "Cell lock list" (Id: 5)
- **IOWA_BEARER_SELECTION_RSC_OPERATOR_LIST_REMOVE**: Disable the resource "Operator list" (Id: 6)
- **IOWA_BEARER_SELECTION_RSC_OPERATOR_LIST_MODE_REMOVE**: Disable the resource "Operator list mode" (Id: 7)
- **IOWA_BEARER_SELECTION_RSC_AVAILABLE_PLMNS_REMOVE**: Disable the resource "List of available PLMNs" (Id: 8)
- **IOWA_BEARER_SELECTION_RSC_ACCEPTABLE_RSRP_NB_IOT_REMOVE**: Disable the resource "Acceptable RSRP (NB-IoT" (Id: 10)
- **IOWA_BEARER_SELECTION_RSC_PLMN_SEARCH_TIMER_REMOVE**: Disable the resource "Higher Priority PLMN Search Timer" (Id: 11)
- **IOWA_BEARER_SELECTION_RSC_ATTACH_WO_PDN_CONNECTION_REMOVE**: Disable the resource "Attach without PDN connection" (Id: 12)

** Note **

You can not set all these defines at once, the Object would not have any resource.

#### Cellular Connectivity

By default, all the resources of the Cellular Connectivity Object are enabled. But they can be disabled by using the following defines:

- **IOWA_CELLULAR_CONNECTIVITY_RSC_SMSC_ADDRESS_REMOVE**: Disable the resource "SMSC address" (Id: 0)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_DISABLE_RADIO_PERIOD_REMOVE**: Disable the resource "Disable radio period" (Id: 1)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_MODULE_ACTIVATION_CODE_REMOVE**: Disable the resource "Module activation code" (Id: 2)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_PSM_TIMER_REMOVE**: Disable the resource "PSM Timer" (Id: 4)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_ACTIVE_TIMER_REMOVE**: Disable the resource "Active Timer" (Id: 5)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_PLMN_RATE_CONTROL_REMOVE**: Disable the resource "Serving PLMN Rate control" (Id: 6)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_EDRX_PARAM_IU_MODE_REMOVE**: Disable the resource "eDRX parameters for Iu mode" (Id: 7)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_EDRX_PARAM_WB_S1_MODE_REMOVE**: Disable the resource "eDRX parameters for WB-S1 mode" (Id: 8)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_EDRX_PARAM_NB_S1_MODE_REMOVE**: Disable the resource "eDRX parameters for NB-S1 mode" (Id: 9)
- **IOWA_CELLULAR_CONNECTIVITY_RSC_EDRX_PARAM_A_GB_MODE_REMOVE**: Disable the resource "eDRX parameters for A/Gb mode" (Id: 10)

#### Connectivity Monitoring

By default, all the resources of the Connectivity Monitoring Object are enabled. But they can be disabled by using the following defines:

- **IOWA_CONNECTIVITY_MONITORING_RSC_LINK_QUALITY_REMOVE**: Disable the resource "Link Quality" (Id: 3)
- **IOWA_CONNECTIVITY_MONITORING_RSC_ROUTER_IP_ADDR_REMOVE**: Disable the resource "Router IP Addresses" (Id: 5)
- **IOWA_CONNECTIVITY_MONITORING_RSC_LINK_USAGE_REMOVE**: Disable the resource "Link Utilization" (Id: 6)
- **IOWA_CONNECTIVITY_MONITORING_RSC_APN_REMOVE**: Disable the resource "APN" (Id: 7)
- **IOWA_CONNECTIVITY_MONITORING_RSC_CELL_ID_REMOVE**: Disable the resource "Cell ID" (Id: 8)
- **IOWA_CONNECTIVITY_MONITORING_RSC_SMNC_REMOVE**: Disable the resource "SMNC" (Id: 9)
- **IOWA_CONNECTIVITY_MONITORING_RSC_SMCC_REMOVE**: Disable the resource "SMCC" (Id: 10)

#### Device

By default, all the resources of the Device Object are enabled. But they can be disabled by using the following defines:

- **IOWA_DEVICE_RSC_MANUFACTURER_REMOVE**: Disable the resource "Manufacturer" (Id: 0)
- **IOWA_DEVICE_RSC_MODEL_NUMBER_REMOVE**: Disable the resource "Model Number" (Id: 1)
- **IOWA_DEVICE_RSC_SERIAL_NUMBER_REMOVE**: Disable the resource "Serial Number" (Id: 2)
- **IOWA_DEVICE_RSC_FIRMWARE_VERSION_REMOVE**: Disable the resource "Firmware Version" (Id: 3)
- **IOWA_DEVICE_RSC_FACTORY_RESET_REMOVE**: Disable the resource "Factory Reset" (Id: 5)
- **IOWA_DEVICE_RSC_POWER_SOURCE_REMOVE**: Disable the resources "Available Power Sources" (Id: 6), "Power Source Voltage" (Id: 7) and "Power Source Current" (Id: 8)
- **IOWA_DEVICE_RSC_BATTERY_REMOVE**: Disable the resources "Battery Level" (Id: 9) and "Battery Status" (Id: 20)
- **IOWA_DEVICE_RSC_RESET_ERROR_REMOVE**: Disable the resource "Reset Error Code" (Id: 12)
- **IOWA_DEVICE_RSC_CURRENT_TIME_REMOVE**: Disable the resource "Current Time" (Id: 13)
- **IOWA_DEVICE_RSC_UTC_OFFSET_REMOVE**: Disable the resource "UTC Offset" (Id: 14)
- **IOWA_DEVICE_RSC_TIMEZONE_REMOVE**: Disable the resource "Timezone" (Id: 15)
- **IOWA_DEVICE_RSC_DEVICE_TYPE_REMOVE**: Disable the resource "Device Type" (Id: 17)
- **IOWA_DEVICE_RSC_HARDWARE_VERSION_REMOVE**: Disable the resource "Hardware Version" (Id: 18)
- **IOWA_DEVICE_RSC_SOFTWARE_VERSION_REMOVE**: Disable the resource "Software Version" (Id: 19)

#### Server

By default, all the resources of the Server Object are enabled. But they can be disabled by using the following defines:

- **IOWA_SERVER_RSC_DISABLE_TIMEOUT_REMOVE**: Disable resources "Disable" (Id: 4) and "Disable Timeout" (Id: 5)
- **IOWA_SERVER_RSC_DEFAULT_PERIODS_REMOVE**: Disable resources "Default Minimum Period" (Id: 2) and "Default Maximum Period" (Id: 3)
- **IOWA_SERVER_RSC_BOOTSTRAP_TRIGGER_REMOVE**: Disable the resource "Bootstrap-Request Trigger" (Id: 9). This resource is also disabled when **LWM2M_BOOTSTRAP** is not defined.
- **IOWA_SERVER_RSC_REGISTRATION_BEHAVIOUR_REMOVE**: Disable resources "Registration Priority Order" (Id: 13), "Initial Registration Delay Timer" (Id: 14), "Registration Failure Block" (Id: 15) and "Bootstrap on Registration Failure" (Id: 16). These resources are also disabled when **LWM2M_VERSION_1_1_SUPPORT** is not defined.
- **IOWA_SERVER_RSC_COMMUNICATION_ATTEMPTS_REMOVE**: Disable resources "Communication Retry Count" (Id: 17), "Communication Retry Timer" (Id: 18), "Communication Sequence Delay Timer" (Id: 19) and "Communication Sequence Retry Count" (Id: 20). These resources are also disabled when **LWM2M_VERSION_1_1_SUPPORT** is not defined.
- **IOWA_SERVER_RSC_MUTE_SEND_REMOVE**: Disable resources "Mute Send" (Id: 23). Only relevant when LWM2M_DATA_PUSH_SUPPORT is set.

As well, some resources contain default value which can be updated using the following defines:

- **IOWA_SERVER_RSC_DISABLE_TIMEOUT_DEFAULT_VALUE**: Update default resource value of "Disable Timeout" (Id: 5)
- **IOWA_SERVER_RSC_STORING_DEFAULT_VALUE**: Update default resource value of "Notification Storing When Disabled or Offline" (Id: 6)
- **IOWA_SERVER_RSC_MUTE_SEND_DEFAULT_VALUE**: Update default resource value of "Mute Send" (Id: 23)

#### Light Control

By default, all the resources of the Light Control Object are enabled. But they can be disabled by using the following defines:

- **IOWA_LIGHT_CONTROL_RSC_DIMMER_REMOVE**: Disable the resource "Dimmer" (Id: 5851)
- **IOWA_LIGHT_CONTROL_RSC_ON_TIME_REMOVE**: Disable the resource "On time" (Id: 5852)
- **IOWA_LIGHT_CONTROL_RSC_CUMULATIVE_ACTIVE_POWER_REMOVE**: Disable the resource "Cumulative active power" (Id: 5805)
- **IOWA_LIGHT_CONTROL_RSC_POWER_FACTOR_REMOVE**: Disable the resource "Power factor" (Id: 5820)
- **IOWA_LIGHT_CONTROL_RSC_COLOUR_REMOVE**: Disable the resources "Colour" (Id: 5706) and "Sensor Units" (Id: 5701)