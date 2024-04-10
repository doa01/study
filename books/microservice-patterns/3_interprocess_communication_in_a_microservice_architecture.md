# 3. interprocess communication in a microservice architecture

## interaction styles

### categories

- processed by
  - one-to-one: each client request is processed by one service(orthogonal to IPC technologies)
  - one-to-many: each request is processed by multiple services
- sync/async
  - synchronous: the client expects timely response, and might event block while it waits
  - asynchronous: the client doesn't block, and the response, if any, isn't necessarily sent immediately

|              | one-to-one                                                                    | one-to-many                                     |
| ------------ | ----------------------------------------------------------------------------- | ----------------------------------------------- |
| synchronous  | request/response                                                              | -                                               |
| asynchronous | asynchronous request/response <br/> one-way notifications(reply not expected) | publish/subscribe <br/> publish/async responses |
