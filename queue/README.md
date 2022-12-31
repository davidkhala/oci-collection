# OCI queue

## [Use STOMP](https://docs.oracle.com/en-us/iaas/Content/queue/messages-stomp.htm)
- supports STOMP specifications 1.0, 1.1, and 1.2
- port `61613`

Limit
- Does not support transactions (STOMP frames `BEGIN/COMMIT/ABORT`)
   - it will return an `ERROR` frame and closes the connection if above frames are used.

