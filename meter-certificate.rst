==========================
Meter Certificate Encoding
==========================

+-------------------+----------------+--------+-------------------------------------------------+
| Name              | Offset         | Byte   | Description                                     |
|                   |                | Length |                                                 |
+===================+================+========+=================================================+
| Meter Data Block  | 0x00           | 4      | The meter data block signature. ASCII encoded   |
| Signature         |                |        | 'MTDB'                                          |
+-------------------+----------------+--------+-------------------------------------------------+
| Meter Data Block  | 0x04           | 4      | Version used to distinguish different set of    |
| Version           |                |        | counters.                                       |
+-------------------+----------------+--------+-------------------------------------------------+
| PPIN              | 0x08           | 8      | Protected Processor Inventory Number            |
+-------------------+----------------+--------+-------------------------------------------------+
| Meter Counter     | 0x10           | 4      | Counter tick units in milliseconds. Each unit   |
| Unit              |                |        | of the counter in the counter data below        |
|                   |                |        | corresponds to the milli seconds value          |
|                   |                |        | identified here.                                |
|                   |                |        | Default value is 1000 ms                        |
+-------------------+----------------+--------+-------------------------------------------------+
| Feature Bundle    | 0x14           | 4      | Length of the encodings and meter data in this  |
| Length            |                |        | data block in bytes.                            |
+-------------------+----------------+--------+-------------------------------------------------+
| Reserved          | 0x18           | 8      | Reserved                                        |
+-------------------+----------------+--------+-------------------------------------------------+
| MASTER REFERENCE  | 0x20           | 4      | Master meter reference counter. This is an ever |
| METER ENCODING    |                |        | incrementing running counter identifier that    |
|                   |                |        | counts units of CPU power up time. ASCII        |
|                   |                |        | encoded ‘MMRC’.                                 |
+-------------------+----------------+--------+-------------------------------------------------+
| MASTER REFERENCE  | 0x24           | 4      | Actual master meter reference counter running   |
| METER COUNTER     |                |        | value                                           |
+-------------------+----------------+--------+-------------------------------------------------+
| FUSE BUNDLE       | 0x28           | 4      | 32 bit ASCII encoding for the feature being     |
| ENCODING 0        |                |        | counted. (For example “CC02”)                   |
+-------------------+----------------+--------+-------------------------------------------------+
| FUSE BUNDLE       | 0x2C           | 4      | Encoding 0 counter data                         |
| ENCODING 0        |                |        |                                                 |
| COUNTER           |                |        |                                                 |
+-------------------+----------------+--------+-------------------------------------------------+
| …                 | …              | …      | …                                               |
+-------------------+----------------+--------+-------------------------------------------------+
| FUSE BUNDLE       | [Bundle length | 4      | 32 bit ASCII encoding for the feature being     |
| ENCODING N        | \- 0x8 bytes]  |        | counted.                                        |
|                   |                |        |                                                 |
+-------------------+----------------+--------+-------------------------------------------------+
| FUSE BUNDLE       | [Bundle length | 4      | Encoding N counter data                         |
| ENCODING N        | \- 0x4 bytes]  |        |                                                 |
| COUNTER           |                |        |                                                 |
+-------------------+----------------+--------+-------------------------------------------------+


