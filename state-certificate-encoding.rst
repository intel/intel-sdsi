==========================
State Certificate Encoding
==========================

License Region NVRAM Format
---------------------------

+-----------------+---------------------+----------+-------------------------------------------------+
| Name            | Region Offset       | Bytes    | Description                                     |
+=================+=====================+==========+=================================================+
| Content type    | 0x00                |   4      | Specific to what is stored in this area.        |
|                 |                     |          |                                                 |
|                 |                     |          | 0xD: License key encoding                       |
|                 |                     |          |                                                 |
|                 |                     |          | 0xE: License key + blob encoding                |
+-----------------+---------------------+----------+-------------------------------------------------+
| Region rev ID   | 0x04                |   4      | Region revision ID                              |
+-----------------+---------------------+----------+-------------------------------------------------+
| Header size     | 0x08                |   4      | Size of the entire header in DWORDS             |
+-----------------+---------------------+----------+-------------------------------------------------+
| Total size      | 0x0C                |   4      | Size of the license region                      |
+-----------------+---------------------+----------+-------------------------------------------------+
| Key size        | 0x10                |   4      | Size of the OEM key in DWORDS                   |
+-----------------+---------------------+----------+-------------------------------------------------+
| NUM_OF_LICENSES | 0x14                |   4      | Number of licenses. Increments from last known  |
|                 |                     |          | value - always goes up                          |
+-----------------+---------------------+----------+-------------------------------------------------+
| License 0       | 0x14 + 1*4          |   4      | Size of the license blob in DWORDS (This is the |
| blob bize       |                     |          | size of license blob derived from the license   |
|                 |                     |          | blob provision. All 0s indicates that there is  |
|                 |                     |          | no license blob content provisioned.            |
|                 |                     |          | Bit[31] is always dedicated to License Valid    |
+-----------------+---------------------+----------+-------------------------------------------------+
| License 1       | 0x14 + 2*4          |   4      | Size of the license blob in DWORDS (This is the |
| blob size       |                     |          | size of license blob derived from the license   |
|                 |                     |          | blob provision. All 0s indicates that there is  |
|                 |                     |          | no license blob content provisioned.            |
|                 |                     |          | Bit[31] is always dedicated to License Valid    |
+-----------------+---------------------+----------+-------------------------------------------------+
| License N       | 0x14                |   4      | Size of the license blob in DWORDS (This is the |
| blob size       | NUM_LICENSES * 4    |          | size of license blob derived from the license   |
|                 |                     |          | blob provision. All 0s indicates that there is  |
|                 |                     |          | no license blob content provisioned.            |
|                 |                     |          | Bit[31] is always dedicated to License Valid    |
+-----------------+---------------------+----------+-------------------------------------------------+
| Body            | 0x14                | variable | License key revision ID - 4 bytes               |
|                 | + NUM_OF_LICENSES*4 |          |                                                 |
|                 | + 4                 |          | License key image's content which is a 384 bit  |
|                 |                     |          | hash of the OEM's ECDSA384 key that comes as a  |
|                 |                     |          | part of license blob provision image            |
|                 +---------------------+          +-------------------------------------------------+
|                 | 0x14                |          | License blob content                            |
|                 | + NUM_OF_LICENSES*4 |          |                                                 |
|                 | + 4 + Key size      |          |                                                 |
|                 | + Sum of all        |          |                                                 |
|                 | license blob sizes) |          |                                                 |
+-----------------+---------------------+----------+-------------------------------------------------+
| Padding         |                     | variable |                                                 |
|                 |                     | up to 15 |                                                 |
+-----------------+---------------------+----------+-------------------------------------------------+

License Blob Format
-------------------

+-------------------+-------------------+----------+-------------------------------------------------+
| Name              | Region Offset     | Bytes    | Description                                     |
|                   |                   |          |                                                 |
+===================+===================+==========+=================================================+
| License blob      | START POINTER     | 4        | License blob type                               |
| type              | from Header       |          |                                                 |
|                   |                   |          | 0x1: One time upgrade license blob              |
|                   |                   |          |                                                 |
|                   |                   |          | 0x2: Metered upgrade license blob               |
+-------------------+-------------------+----------+-------------------------------------------------+
| License blob ID   |                   | 8        | 64 bit GUID for this license. This is how a     |
|                   |                   |          | license is tracked and can potentially migrate  |
|                   |                   |          | from one CPU to another.                        |
+-------------------+-------------------+----------+-------------------------------------------------+
| PPIN              |                   | 8        | Protected Processor Inventory Number            |
+-------------------+-------------------+----------+-------------------------------------------------+
| RESERVED          |                   | 8        |                                                 |
+-------------------+-------------------+----------+-------------------------------------------------+
| License blob      |                   | 4        |                                                 |
| revision ID       |                   |          |                                                 |
|                   |                   |          |                                                 |
+-------------------+-------------------+----------+-------------------------------------------------+
| Number of         |                   | 4        | Number of encodings in the license blob         |
| feature bundles   |                   |          |                                                 |
+-------------------+-------------------+----------+-------------------------------------------------+
| FUSE BUNDLE       |                   |          |                                                 |
| ENCODING 0        |                   | 4        |                                                 |
+-------------------+-------------------+----------+-------------------------------------------------+
| FUSE BUNDLES      |                   | 28       | License blob may or may not come with any       |
| ENCODING 0 RSVD   |                   |          | content in here but it is part of the           |
|                   |                   |          | signature. Intel On Demand firmware ignores     |
|                   |                   |          | this field.                                     |
+-------------------+-------------------+----------+-------------------------------------------------+
| FUSE BUNDLE       |                   |          |                                                 |
| ENCODING N        |                   | 4        |                                                 |
+-------------------+-------------------+----------+-------------------------------------------------+
| FUSE BUNDLES      |                   | 28       | License blob may or may not come with any       |
| ENCODING N RSVD   |                   |          | content in here but it is part of the           |
|                   |                   |          | signature. Intel On Demand firmware ignores     |
|                   |                   |          | this field.                                     |
+-------------------+-------------------+----------+-------------------------------------------------+
