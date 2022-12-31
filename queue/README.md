# OCI queue

## [Use STOMP](https://docs.oracle.com/en-us/iaas/Content/queue/messages-stomp.htm)
- supports STOMP specifications 1.0, 1.1, and 1.2
- port `61613`
- Authentication: uses authentication tokens. It refers to [OCI IAM Auth Token](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcredentials.htm#Working)

### Produce
Use the `SEND` frame with headers
- `destination`: (Required) The OCID (Oracle Cloud Identifier) of the queue to publish the messages to.
- `receipt`: (Optional) If the SEND frame contains a receipt header, Queue sends a RECEIPT frame to the client upon successful publishing. 

The Queue service sends a `RECEIPT` frame to the STOMP client after the service successfully processes a `SEND` frame that requested a receipt.

A RECEIPT frame includes header
- `receipt-id`: a value that matches the receipt header in the `SEND` frame.

## Subscribe
The SUBSCRIBE frame is equivalent to a long polling GetMessages request

Use the `SUBSCRIBE` frame with headers
- destination: (Required) The OCID (Oracle Cloud Identifier) of the queue to consume messages from.
- id: (Required) Identifies the subscription. The same ID must be passed by the client in an UNSUBSCRIBE frame to stop consuming messages.
- ack: (Required) Only the client-individual value is supported. This means that an ACK frame only deletes the message identified in the ACK frame and not all messages delivered before.
- visibility: (Optional) How long the consumed messages should only be visible for the client making the request. If the header is omitted, Queue will use the queue's default visibility timeout.

Use the `UNSUBSCRIBE` frame with header
- id: (Required) Identifies which subscription to stop. This ID was used in the corresponding SUBSCRIBE frame.


### Message
Any messages received from the Queue will be delivered to the client as `MESSAGE` frames

A MESSAGE frame contains the following headers:
- message-id:
  - For STOMP v1.2: Deprecated. A unique, internal identifier for the message. Used for debugging purposes only.
  - For STOMP v1.1 and v1.0: The message receipt to use with ACK and NACK frames.
- subscription: The ID used in the SUBSCRIBE frame.
- destination: The OCID (Oracle Cloud Identifier) of the queue that contained the messages.
- ack: For STOMP v.1.2 only, the message receipt.
- content-length: The length of the payload, or message.
- expire-after: The message expiration time, as milliseconds since epoch.
- visible-after: The message visibility time, as milliseconds since epoch.
- delivery-count: The number of times this message has been delivered
- oci-message-id : An internal message ID.


### Acknowledgement
Use the `ACK` frame to delete a message after it's been received and processed.

Use the NACK frame to notify Queue that the message hasn't been successfully processed. Using the NACK frame updates the visibility of the message to make it immediately visible for other consumers.

Both frames accept the following headers:
- `id`: (Required for STOMP v1.2) The ID of the message to delete or update.
- `message-id`: (Required for STOMP v1.1 or 1.0) The ID of the message to delete or update.


### Limit
- Transactional frames or any custom headers are not supported (STOMP frames `BEGIN/COMMIT/ABORT`)
   - it will return an `ERROR` frame and closes the connection if above frames are used.

