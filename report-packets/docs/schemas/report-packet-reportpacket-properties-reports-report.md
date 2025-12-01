# Report Schema

```txt
https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items
```



| Abstract            | Extensible | Status         | Identifiable | Custom Properties | Additional Properties | Access Restrictions | Defined In                                                                             |
| :------------------ | :--------- | :------------- | :----------- | :---------------- | :-------------------- | :------------------ | :------------------------------------------------------------------------------------- |
| Can be instantiated | No         | Unknown status | No           | Forbidden         | Allowed               | none                | [report-packet.schema.json\*](../out/report-packet.schema.json "open original schema") |

## items Type

`object` ([Report](report-packet-reportpacket-properties-reports-report.md))

# items Properties

| Property                    | Type          | Required | Nullable       | Defined by                                                                                                                                                                                                                                     |
| :-------------------------- | :------------ | :------- | :------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [fileName](#filename)       | `string`      | Required | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-filename.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/fileName")       |
| [title](#title)             | `string`      | Required | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-title.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/title")             |
| [description](#description) | `string`      | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-description.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/description") |
| [sortOrder](#sortorder)     | `integer`     | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-sortorder.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/sortOrder")     |

## fileName

The name of the file uploaded.

`fileName`

* is optional

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-filename.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/fileName")

### fileName Type

`string`

## title

The title of the report

`title`

* is optional

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-title.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/title")

### title Type

`string`

## description

The description of the report

`description`

* is optional

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-description.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/description")

### description Type

`string`

## sortOrder

The (relative) display position of this report.

`sortOrder`

* is optional

* Type: `integer`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-sortorder.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/sortOrder")

### sortOrder Type

`integer`

### sortOrder Default Value

The default value is:

```json
0
```

## required



`required`

* is optional

* Type: unknown

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-reports-report-properties-required.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports/items/properties/required")

### required Type

unknown
