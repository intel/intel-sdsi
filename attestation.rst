Intel On Demand Attestation Service
-----------------------------------

Attestation support is indicated when the ATTESTATION_ENABLE bit is set in the
ENABLED_FEATURES register. Intel On Demand uses the Security Protocol and Data
Model Specification (SPDM) to perform the attestation service. This service
is based on the `SPDM 1.0`__ protocol. The attestation service allows for the
authentication of the Intel On Demand hardware and verification of its firmware
measurements. The Intel On Demand measurements that can be verified are the
state certificate and meter certificate.

.. __: https://www.dmtf.org/sites/default/files/standards/documents/DSP0274_1.0.0.pdf

SPDM attestation is performed through the mailbox operation using the
ATTESTATION command. The Mailbox DATA will contain the SPDM message. The
following SPDM messages are supported by Intel On Demand hardware when the
ATTESTATION_ENABLE bit is set:

* GET_VERSION
* GET_DIGESTS
* GET_CERTIFICATE
* CHALLENGE
* GET_MEASUREMENTS
* GET_CAPABILITIES
* NEGOTIATE_ALGORITHMS

The attestation flow is the sequence of commands and corresponding responses
exchanged between the Platform Root of Trust (Requester) and Intel On Demand
hardware (Responder).  The valid command order is defined by the SPDM standard.
Per the standard, the possible request orderings after Power on Reset are listed
below explicitly:

* GET_VERSION, GET_CAPABILITY, NEGOTIATE_ALGORITHMS, GET_DIGESTS, GET_CERTIFICATE, CHALLENGE
* GET_VERSION, GET_CAPABILITY, NEGOTIATE_ALGORITHMS, GET_DIGESTS, CHALLENGE
* GET_VERSION, GET_CAPABILITY, NEGOTIATE_ALGORITHMS, CHALLENGE

Intel On Demand hardware doesn't support caching capability over reset (reported
in the GET_CAPABILITIES response, CACHE_CAP flag). As a result, after reset
sequences shall start with the GET_VERSION, GET_CAPABILITY, NEGOTIATE_ALGORITHMS
command sequence. Following one successful negotiation, the ordering below will
be supported as well:

* GET_DIGESTS, GET_CERTIFICATE, CHALLENGE
* GET_DIGESTS, CHALLENGE
* GET_DIGESTS
* CHALLENGE

After reset, the negotiation steps (GET_VERSION, GET_CAPABILITY,
NEGOTIATE_ALGORITHMS) must be repeated.

Structure of SPDM messages in the On Demand mailbox
---------------------------------------------------

Because the mailbox requires QWORD alignment, padding needs to added at the end
of the SPDM message as needed to ensure an 8-byte alignment. Because of the
variable size of some SPDM messages, an additional 8-byte field is used to
indicate the actual non-padded size of the SPDM message so that the correct data
and length may be sent. After this field, the last field is the ATTESTATION
mailbox command.

Example for a GET_VERSION Request:

+---------------+---------------+---------------+---------------+-------+
|    Byte 3     |    Byte 2     |    Byte 1     |    Byte 0     | DWORD |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=======+
|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-------+
|            GET_VERSION requestor message (4 bytes)            |   0   |
+---------------------------------------------------------------+-------+
|            Padding (4 bytes)                                  |   1   |
+---------------------------------------------------------------+-------+
|            SPDM command length = 4                            |   2   |
|                                                               +-------+
|                                                               |   3   |
+---------------------------------------------------------------+-------+
|            ATTESTATION mailbox command = 0x1012               |   4   |
|                                                               +-------+
|                                                               |   5   |
+---------------------------------------------------------------+-------+

* The GET_VERSION message is only 4 bytes and requires an additional 4 bytes of padding.
* The next QWORD contains the actual length of the GET_VERSION message, 4 bytes.
* The last QWORD contamailbox command
* Note: The total size of the mailbox message is 24 bytes. This is the value set for both the packet size and message size fields in the mailbox control register.

Supported Algorithms
--------------------

Supported algorithms will be indicated in the NEGOTIATE_ALGORITHMS response.
The following are the current algorithms supported by the attestation service.

+------------------------------------------+-----------------------------+
| Measurement Hash Algorithm               | TPM_ALG_SHA_384             |
+------------------------------------------+-----------------------------+
| Base Asymmetric Key Signature Algorithms | TPM_ALG_ECDSA_ECC_NIST_P384 |
+------------------------------------------+-----------------------------+
| Base Hashing Algorithms                  | TPM_ALG_SHA_384             |
+------------------------------------------+-----------------------------+

Certificate Chains
-------------------

A single certificate is supported and is retrievable from the first slot of the
GET_CERTIFICATE message. A hash of this certificate chain is retrievable in the
first slot of the GET_DIGESTS message response.

Measurements
------------

Upon successful completion of the attestation flow, hardware measurements may be
collected using the GET_MEASUREMENTS command. Intel On Demand hardware supports
measurement for two entries, entry 0 which is a hash of the NVRAM region used
to store the state certificate, and entry 1 which is a hash of the region used
to store the meter certificate. Measurements may be signed if requested with
support indicated in the CAPABILITIES response.
