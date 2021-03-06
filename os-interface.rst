========================================================
Intel® Software Defined Silicon (SDSi) In-band Interface
========================================================

SDSi Capability Enumeration
---------------------------

SDSi is enumerated, per socket, as a PCI Express Vendor Specific Capability
(VSEC) on a PCIe endpoint device. The table below shows the entire VSEC header
for SDSi including the PCI Express Extended capability header. The VSEC is used
by software to locate the SDSi Discovery Structure.

Refer to the PCI Express Specification for details on the Vendor Specific
Capability definitions.

+---------------+---------------+---------------+---------------+
|    Byte 3     |    Byte 2     |    Byte 1     |    Byte 0     |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Next                  | Cap   |  PCI Express Extended         |
| Capability            | Ver = |  Capability ID = 000Bh        |
| Offset                | 1h    |                               |
+-----------------------+-------+-------------------------------+
|                       | VSEC  |                               |
| VSEC Length = 10h     | REV = |  VSEC ID = 0041h              |
|                       | 1h    |  (SDSi Capability No.)        |
+---------------+-------+-------+-------------------------------+
| Entry Size    | Number of     |                               |
| = 4 (MMIO size| Entries = 1   |  Reserved                     |
| in DWORDS)    |               |                               |
+---------------+---------------+-------------------------+-----+
|                                                         |     |
| Offset                                                  | TBIR|
+---------------------------------------------------------+-----+

SDSi Discovery Structure
------------------------

The SDSi Discovery structure provides the location of the SDSi MMIO Layout where
the hardware registers are located.

+---------------+---------------+---------------+---------------+
|    Byte 3     |    Byte 2     |    Byte 1     |    Byte 0     |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| RSVD  |       Size in DWORDS          |   Type = 3    | Access|
|       |                               |               | Type  |
+-------+-------------------------------+---------------+-------+
|                            GUID                               |
+---------------------------------------------------------+-----+
|                                                         |     |
| BAR Offset from TBIR                                    | TBIR|
+---------------------------------------------------------+-----+

Access Type = 2: MMIO space is at BAR[TBIR] + Offset

Access Type = 3: MMIO space is at OFFSET + 10h from the from the address of the SDSi Discover Structure

GUID: An ID that identifies the layout of the registers in the SDSI MMIO space.


SDSi MMIO Layout for GUID = 006DD191h
-------------------------------------

+-------------------+-----+---------+----------+---------------------------------------+
| Name              | R/W | Size    | Offset   | Description                           |
|                   |     | (Bytes) | (DWORDS) |                                       |
+===================+=====+=========+==========+=======================================+
| Control Structure | R/W | 8       |  0       | Control for mailbox (see below)       |
+-------------------+-----+---------+----------+---------------------------------------+
| Mailbox           | R/W | 1024    |  2       | Mailbox buffer (see below)            |
+-------------------+-----+---------+----------+---------------------------------------+
| PPIN              | R   | 8       |  258     | Protected Processor Inventory Number  |
+-------------------+-----+---------+----------+---------------------------------------+
| SDSi Registers    | R   | 48      |  260     | SDSi registers (see below)            |
+-------------------+-----+---------+----------+---------------------------------------+
| RESERVED          |     | 8       |  272     |                                       |
+-------------------+-----+---------+----------+---------------------------------------+
| Socket ID         | R   | 8       |  274     | Socket ID is bits 3:0                 |
+-------------------+-----+---------+----------+---------------------------------------+


SDSI MMIO details
-----------------

Control Structure
+++++++++++++++++

+---------------+---------------+---------------+-------------------------------+
|    Byte 3     |    Byte 2     |    Byte 1     |             Byte 0            |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+===+===+===+===+===+===+===+===+
|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0| 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---+---+---+---+---+---+---+---+
| Packet Size                   | Status        |RDY|CPL| Owner |EOM|SOM|R/W|RUN|
|                               | Code          |   |   |       |   |   |   |BSY|
|                               |               |   |   |       |   |   |   |   |
|                               |               |   |   |       |   |   |   |   |
|                               |               |   |   |       |   |   |   |   |
|                               |               |   |   |       |   |   |   |   |
|                               |               |   |   |       |   |   |   |   |
|                               |               |   |   |       |   |   |   |   |
+-------------------------------+---------------+---+---+-------+---+---+---+---+
| Message Size                  | Reserved                                      |
+-------------------------------+-----------------------------------------------+

CONTROL FIELDS

+-------+----------+---------------+-----+---------------------------------------------------------+
| Bits  | Field    | Default value | R/W |                    Description                          |
+=======+==========+===============+=====+=========================================================+
| 0     | RUN/BUSY | 0             | R/W | Flag is set by requester to initiate data transmission. |
|       |          |               |     | SDSi firmware clears this flag when the transmission    |
|       |          |               |     | has ended.                                              |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 1     | Read/    | 0             | R/W | Determines whether read from or write to mailbox shall  |
|       | Write    |               |     | be performed:                                           |
|       | Command  |               |     |                                                         |
|       |          |               |     | = 0 for read                                            |
|       |          |               |     |                                                         |
|       |          |               |     | = 1 for write                                           |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 2     | SOM      | 0             | R/W | When data is ready to be read, this bit is set by       |
|       |          |               |     | firmware to indicate that the data in the mailbox is    |
|       |          |               |     | the first packet.                                       |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 3     | EOM      | 0             | R/W | When data is ready to be read, this bit is set by       |
|       |          |               |     | firmware to indicate that the data in the mailbox is    |
|       |          |               |     | the last packet.                                        |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 5:4   | Owner    | 0             | R   | Mailbox ownership – read only:                          |
|       |          |               |     |                                                         |
|       |          |               |     | 2’b00 – None – Mailbox is free to use                   |
|       |          |               |     |                                                         |
|       |          |               |     | 2’b01 – In-band agent                                   |
|       |          |               |     |                                                         |
|       |          |               |     | 2’b10 – Out-of-band agent                               |
|       |          |               |     |                                                         |
|       |          |               |     | 2’b11 – Reserved                                        |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 6     | CPL      | 0             | R/W | Complete bit.                                           |
|       |          |               |     |                                                         |
|       |          |               |     | At the end of data transmission, used to indicate to    |
|       |          |               |     | firmware that processing is complete and ownership of   |
|       |          |               |     | the mailbox is relinquished.                            |
|       |          |               |     |                                                         |
|       |          |               |     | Read Command: For reads greater than 1024B, setting     |
|       |          |               |     | this bit also adjusts the buffer read position forward  |
|       |          |               |     | by 1024B.                                               |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 7     | RDY      | 0             | R   | Read Command Only. This bit is set by firmware when a   |
|       |          |               |     | packet is ready to be read.                             |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 15:8  | Status   | 0             | R   | Status of mailbox operation filled by firmware at the   |
|       | Code     |               |     | end of a read or write operation.                       |
|       |          |               |     |                                                         |
|       |          |               |     | = 0x40 – Success                                        |
|       |          |               |     |                                                         |
|       |          |               |     | = 0x80 – Timeout                                        |
|       |          |               |     |                                                         |
|       |          |               |     | = 0x90 – Failure                                        |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 31:16 | Packet   | 0             | R/W | Mailbox packet size in bytes. Written by the requester  |
|       | Size     |               |     | at start of transmission. Written by the firmware when  |
|       |          |               |     | data is ready to be read.                               |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 47:32 | Reserved |               |     |                                                         |
+-------+----------+---------------+-----+---------------------------------------------------------+
| 63:48 | Message  | 0             | R   | Read Command Only. Total message size in bytes. Set by  |
|       | Size     |               |     | firmware when data is greater than 1024B. Size is QWORD |
|       |          |               |     | aligned.                                                |
+-------+----------+---------------+-----+---------------------------------------------------------+

Mailbox
+++++++

+---------------+---------------+---------------+---------------+-------+   
|    Byte 3     |    Byte 2     |    Byte 1     |    Byte 0     | DWORD |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=======+
|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|7|6|5|4|3|2|1|0|       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-------+
|                       (63:0) Data 0                           |   0   |
|                                                               +-------+
|                                                               |   1   |
+---------------------------------------------------------------+-------+
|                          ...                                          |
+---------------------------------------------------------------+-------+
|                  (8191:8128) Data 127                         |  254  |
|                                                               +-------+
|                                                               |  255  |
+---------------------------------------------------------------+-------+

MAILBOX COMMANDS

+------------------+------------+---------------------------------------------------------+
| Command Name     | Command ID | Description                                             |
+==================+============+=========================================================+
| PROVISION_AKC    | 0x04       | Write the Authentication Key Certificate (AKC) in the   |
|                  |            | mailbox to SDSi hardware.                               |
+------------------+------------+---------------------------------------------------------+
| PROVISION_CAP    | 0x08       | Write the Capability Activation Payload (CAP) in the    |
|                  |            | mailbox to SDSi hardware.                               |
+------------------+------------+---------------------------------------------------------+
| READ_SDSI_STATE  | 0x10       | Read the State Certificate from the SDSi hardware to    |
|                  |            | mailbox.                                                |
+------------------+------------+---------------------------------------------------------+

Mailbox commands are written to the Mailbox buffer in the last QWORD following a
payload, if applicable.

SDSi Registers
++++++++++++++

+--------+---------+---------------------------------+---------------------------------+
| Offset | Size    | Name                            | Description                     |
|        | (bytes) |                                 |                                 |
+========+=========+=================================+=================================+
|  0x00  |  8      | Reserved                        |                                 |
+--------+---------+---------------------------------+---------------------------------+
|  0x08  |  8      | ENABLED_FEATURES                | Enabled Features (see below)    |
+--------+---------+---------------------------------+---------------------------------+
|  0x10  |  8      | Reserved                        |                                 |
+--------+---------+---------------------------------+---------------------------------+
|  0x18  |  8      | PROVISIONING_AUTH_FAILURE_COUNT | Failure counts (see below)      |
+--------+---------+---------------------------------+---------------------------------+
|  0x20  |  8      | PROVISIONING_AVAILABILITY       | Provisioning availability (see  |
|        |         |                                 | below)                          |
+--------+---------+---------------------------------+---------------------------------+
|  0x28  |  8      | Reserved                        |                                 |
+--------+---------+---------------------------------+---------------------------------+

ENABLED_FEATURES

+--------+-------+------------------------------------+--------------------------------+
| Bit    | Bit   | Name                               | Description                    |
| Offset | Width |                                    |                                |
+========+=======+====================================+================================+
|  63:4  |  60   | RESERVED                           |                                |
+--------+-------+------------------------------------+--------------------------------+
|  3     |  1    | SDSi                               | SDSi is enabled                |
+--------+-------+------------------------------------+--------------------------------+
|  2:0   |  3    | RESERVED                           |                                |
+--------+-------+------------------------------------+--------------------------------+

PROVISIONING_AUTH_FAILURE_COUNT

+--------+-------+------------------------------------+--------------------------------+
| Bit    | Bit   | Name                               | Description                    |
| Offset | Width |                                    |                                |
+========+=======+====================================+================================+
|  63:12 |  60   | RESERVED                           |                                |
+--------+-------+------------------------------------+--------------------------------+
|  9:11  |  3    | LICENSE_AUTH_FAILURE_THRESHOLD     | Capability Activation Payload  |
|        |       |                                    | provisioning failure threshold |
|        |       |                                    | between power cycles           |
+--------+-------+------------------------------------+--------------------------------+
|  6:8   |  3    | LICENSE_AUTH_FAILURE_COUNT         | Number of times Capability     |
|        |       |                                    | Activation Payload provisioning|
|        |       |                                    | failed in a power cycle        |
+--------+-------+------------------------------------+--------------------------------+
|  3:5   |  3    | LICENSE_KEY_AUTH_FAILURE_THRESHOLD | Authentication Key Certificate |
|        |       |                                    | provisioning failure threshold |
|        |       |                                    | between power cycles           |
+--------+-------+------------------------------------+--------------------------------+
|  2:0   |  3    | LICENSE_KEY_AUTH_FAILURE_COUNT     | Number of times Authentication |
|        |       |                                    | Key Certificate provisioning   |
|        |       |                                    | failed in a power cycle        |
+--------+-------+------------------------------------+--------------------------------+

PROVISIONING_AVAILABILITY

+--------+-------+------------------------------------+--------------------------------+
| Bit    | Bit   | Name                               | Description                    |
| Offset | Width |                                    |                                |
+========+=======+====================================+================================+
|  53:51 |  3    | UPDATES_THRESHOLD                  | Maximum number of provision    |
|        |       |                                    | operations allowed between     |
|        |       |                                    | power cycles                   |
+--------+-------+------------------------------------+--------------------------------+
|  50:48 |  3    | UPDATES_AVAILABLE                  | Number of provision operations |
|        |       |                                    | left before power cycle        |
|        |       |                                    | required                       |
+--------+-------+------------------------------------+--------------------------------+
|  47:0  |  48   | RESERVED                           |                                |
+--------+-------+------------------------------------+--------------------------------+
