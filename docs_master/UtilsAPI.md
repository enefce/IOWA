# Utils API Reference

The functions explained below are defined inside the file *include/iowa_utils.h*.

## Data types

### iowa_list_t

List structure used by the List APIs.

```c
typedef struct _iowa_list_t
{
    struct _iowa_list_t *nextP;
} iowa_list_t;
```

nextP
: Pointer to the next element in the list.

\clearpage

## Callbacks

### iowa_list_node_free_callback_t

This is the list node free callback, called to free a node.

```c
typedef void(*iowa_list_node_free_callback_t) (void *nodeP);
```

*nodeP*
: The node to free.

\clearpage

### iowa_list_node_find_callback_t

This is the list node find callback, called to find a node.

```c
typedef bool(*iowa_list_node_find_callback_t) (void *nodeP,
                                               void *criteria);
```

*nodeP*
: The current node in the list.

*criteria*
: The criteria to match.

\clearpage

## API

### iowa_utils_base64_get_encoded_size

** Prototype **

```c
size_t iowa_utils_base64_get_encoded_size(size_t rawBufferLen);
```
** Description **

`iowa_utils_base64_get_encoded_size()` calculates the length of a Base64 buffer based on a raw buffer represented by its length.

** Arguments **

*rawBufferLen*
: The length of the raw buffer.

** Return Value **

The length of the Base64 buffer.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_base64_get_decoded_size

** Prototype **

```c
size_t iowa_utils_base64_get_decoded_size(uint8_t *base64Buffer,
                                          size_t base64BufferLen);
```
** Description **

`iowa_utils_base64_get_decoded_size()` calculates the length of a raw buffer based on a Base64 buffer.

** Arguments **

*base64Buffer*
: The Base64 buffer.

*base64BufferLen*
: The length of the Base64 buffer.

** Return Value **

The length of the raw buffer. If any error the length will be 0.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_base64_encode

** Prototype **

```c
size_t iowa_utils_base64_encode(uint8_t * rawBuffer,
                                size_t rawBufferLen,
                                uint8_t * base64Buffer,
                                size_t base64BufferLen);
```

** Description **

`iowa_utils_base64_encode()` encodes a raw buffer using Base64.

** Arguments **

*rawBuffer*
: The raw buffer.

*rawBufferLen*
: The length of the raw buffer.

*base64Buffer*
: The preallocated Base64 buffer.

*base64BufferLen*
: The length of the preallocated Base64 buffer.

** Return Value **

The length of the encoded buffer. If any error the length will be 0.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_base64_decode

** Prototype **

```c
size_t iowa_utils_base64_decode(uint8_t * base64Buffer,
                                size_t base64BufferLen,
                                uint8_t * rawBuffer,
                                size_t rawBufferLen);
```

** Description **

`iowa_utils_base64_decode()` decodes a Base64 buffer into a raw buffer.

** Arguments **

*base64Buffer*
: The Base64 buffer.

*base64BufferLen*
: The length of the Base64 buffer.

*rawBuffer*
: The preallocated raw buffer.

*rawBufferLen*
: The length of the preallocated raw buffer.

** Return Value **

The length of the decoded buffer. If any error the length will be 0.

** Header File **

iowa_utils.h

\clearpage

-------------------

### iowa_utils_uri_to_sensor

** Prototype **

```c
iowa_sensor_t iowa_utils_uri_to_sensor(iowa_lwm2m_uri_t *uriP);
```

** Description **

`iowa_utils_uri_to_sensor()` converts an iowa_lwm2m_uri_t into an iowa_sensor_t.

** Arguments **

*uriP*
: Uri to convert.

** Return Value **

The corresponding iowa_sensor_t or IOWA_INVALID_SENSOR_ID in case of error.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_sensor_to_uri

** Prototype **

```c
iowa_lwm2m_uri_t iowa_utils_sensor_to_uri(iowa_sensor_t id);
```

** Description **

`iowa_utils_sensor_to_uri()` converts an iowa_sensor_t into an iowa_lwm2m_uri_t.

** Arguments **

*id*
: Id to convert.

** Return Value **

An iowa_lwm2m_uri_t.

** Header File **

iowa_utils.h

\clearpage

-------------------

### iowa_utils_list_add

** Prototype **

```c
iowa_list_t * iowa_utils_list_add(iowa_list_t *headP,
                                  iowa_list_t *nodeP);
```

** Description **

`iowa_utils_list_add()` adds a node to a list.

** Arguments **

*headP*
: Head of the current list.

*nodeP*
: Node to add to the list.

** Return Value **

The list with the new element.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_list_remove

** Prototype **

```c
iowa_list_t * iowa_utils_list_remove(iowa_list_t *headP,
                                     iowa_list_t *nodeP);
```

** Description **

`iowa_utils_list_remove()` removes a node from a list.

** Arguments **

*headP*
: Head of the current list.

*nodeP*
: Node to remove from the list.

** Return Value **

The updated list.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_list_free

** Prototype **

```c
void iowa_utils_list_free(iowa_list_t *headP,
                          iowa_list_node_free_callback_t freeCb);
```

** Description **

`iowa_utils_list_free()` adds a node to a list.

** Arguments **

*headP*
: List to free.

*freeCb*
: Callback used to free the list.

** Return Value **

None.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_list_find

** Prototype **

```c
iowa_list_t * iowa_utils_list_find(iowa_list_t *headP,
                                   iowa_list_node_find_callback_t findCb,
                                   void *criteriaP);
```

** Description **

`iowa_utils_list_find()` finds a node in a list.

** Arguments **

*headP*
: List to search on.

*findCb*
: Callback used to find the node in the list.

*criteriaP*
: Criteria used to find the node in the list.

** Return Value **

The node if found else NULL.

** Header File **

iowa_utils.h

\clearpage

### iowa_utils_list_find_and_remove

** Prototype **

```c
iowa_list_t * iowa_utils_list_find_and_remove(iowa_list_t *headP,
                                              iowa_list_node_find_callback_t findCb,
                                              void *criteriaP,
                                              iowa_list_t **nodeP);
```

** Description **

`iowa_utils_list_find_and_remove()` finds a node in a list and removes it.

** Arguments **

*headP*
: List to search on.

*findCb*
: Callback used to find the node in the list.

*criteriaP*
: Criteria used to find the node in the list.

*nodeP*
: OUT. Node removed from the list. Can be nil.

** Return Value **

The updated list.

** Header File **

iowa_utils.h

\clearpage

## Example: Linked List usage

When declaring the linked list data structure, the first member **must** be a pointer. This pointer will contain the address of the next element of the list.

Example:
```c
struct myData
{
    struct myData *nextP;   // Used by the linked list functions
    char          *aString;
    int            anInt;
};
```

The head of the list is a pointer to your data structure.

Example:
```c
struct myData *listHead;
```

You can now add or remove elements to the list by using the functions [`iowa_utils_list_add()`](iowa_utils_list_add) and [`iowa_utils_list_remove()`](iowa_utils_list_remove).

But to avoid compiler warnings or multiple cast making the code unreadable, the following macros can be used:

* IOWA_UTILS_LIST_ADD(H, N)
* IOWA_UTILS_LIST_REMOVE(H, N)
* IOWA_UTILS_LIST_FREE(H, F)
* IOWA_UTILS_LIST_FIND(H, F, C)
* IOWA_UTILS_LIST_FIND_AND_REMOVE(H, F, C, N)

Instead of calling:

```c
listHead = (struct myData *)iowa_utils_list_add((iowa_list_t *)listHead, (iowa_list_t *)newDataP);
```

You can use:

```c
listHead = (struct myData *)IOWA_UTILS_LIST_ADD(listHead, newDataP);
```