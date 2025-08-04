# Socket nodes

| ❗ This document is in DRAFT stage |
|------------------------------------|

Socket nodes provide bidirectional reliable bytes stream. These nodes should not
have any child nodes (`*:ls` should always return `[]`).

SHV RPC is by design request-response communication protocol and most notably
messages can be dropped without a retransmit nor notice and thus lost. At the
same time there are use cases when it is desirable to emulate byte streams such
as remote terminal. This requires improved reliability in the form of ensured
delivery and both sides transmission control.

In the RPC communication there are always two sides, the asking peer (caller)
and the answering peer (answerer). To get the bidirectional communication we
exchange bytes between caller and answerer with method call. This gives full
control over the communication flow to the caller that must initiate every
exchange.

## `*:sctl`

| Name   | SHV Path | Flags | Param Type | Result Type | Access          |
|--------|----------|-------|------------|-------------|-----------------|
| `sctl` | Any      |       | `{?}`      |             | Write or higher |



## `*:sread`

| Name   | SHV Path | Flags | Param Type | Result Type | Access          |
|--------|----------|-------|------------|-------------|-----------------|
| `sread` | Any      |       | `i`        | `b`         | Write or higher |

```
=> <id:4, method:"sread", path:"test/socket">i{1:128}
<= <id:4>i{4:.0}
<= <id:4>i{2:b"sh> "}
```

## `*:swrite`

| Name     | SHV Path | Flags | Param Type | Result Type | Access          |
|----------|----------|-------|------------|-------------|-----------------|
| `swrite` | Any      |       | `b`        | `i`         | Write or higher |

```
=> <id:5, method:"swrite", path:"test/socket">i{1:b""}
<= <id:5>i{2:64}
=> <id:6, method:"swrite", path:"test/socket">i{1:b"help\n"}
<= <id:5>i{2:59}
...
<= <id:47>i{2:0}
=> <id:48, method:"swrite", path:"test/socket">i{1:b""}
<= <id:48>i{4:.0}
<= <id:48>i{4:.0}
<= <id:48>i{2:7}
```

## Walkthrough of the client

## Walkthrough of the server


This method provides a way to perform read and write operations. It heavily
relies on delayed responses to get increased responsiveness.

The connection is automatically opened with first method call. The method is
special because it differentiates between sessions by `CallerIds`. For specific
sequence there can be only one session at the time. Clients just have to use
different `CallerIds` initial value (instead of empty list) if they want to have
more than one session.

The connection can be closed by either side by sending Integer with negative
value.



Caller-answerer communication example on simple remote shell:

```
<= <id:4, method:"socket", path:"test/node">i{1:128}
=> <id:4>i{4:.0}
<= <id:5, method:"socket", path:"test/node">i{1:b""}
=> <id:5>i{2:64}
=> <id:4>i{2:b"sh> "}
<= <id:5, method:"socket", path:"test/node">i{1:128}
=> <id:5>i{4:.0}
# Now user types "help\n". The first character is captured and sent.
<= <id:6, method:"socket", path:"test/node">i{1:b"h"}
=> <id:6>i{2:63}
=> <id:5>i{2:[64,b"h"]}
# Shell cosumes and echoes the input.
<= <id:7, method:"socket", path:"test/node">i{1:[128,b"ello\n"]}
=> <id:7>i{2:[59]}

```
