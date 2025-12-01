# ReportPacket Schema

```txt
https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items
```

Report Packet details.

| Abstract            | Extensible | Status         | Identifiable | Custom Properties | Additional Properties | Access Restrictions | Defined In                                                                             |
| :------------------ | :--------- | :------------- | :----------- | :---------------- | :-------------------- | :------------------ | :------------------------------------------------------------------------------------- |
| Can be instantiated | No         | Unknown status | No           | Forbidden         | Allowed               | none                | [report-packet.schema.json\*](../out/report-packet.schema.json "open original schema") |

## items Type

`object` ([ReportPacket](report-packet-reportpacket.md))

# items Properties

| Property                                                          | Type          | Required | Nullable       | Defined by                                                                                                                                                                                                                        |
| :---------------------------------------------------------------- | :------------ | :------- | :------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [memberGroupId](#membergroupid)                                   | `string` | Required | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-membergroupid.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/memberGroupId")                                   |
| [title](#title)                                                   | `string`      | Required | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-title.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/title")                                                   |
| [description](#description)                                       | `string`      | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-description.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/description")                                       |
| [period](#period)                                                 | `object`      | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-period.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period")                                                 |
| [containsProtectedHealthInfo](#containsprotectedhealthinfo)       | `boolean`     | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-containsprotectedhealthinfo.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/containsProtectedHealthInfo")       |
| [containsSensitiveFinancialInfo](#containssensitivefinancialinfo) | `boolean`     | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-containssensitivefinancialinfo.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/containsSensitiveFinancialInfo") |
| [reports](#reports)                                               | `array`       | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-reportlist.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports")                                            |

## memberGroupId

Identifier of the member group the packet is for.

`memberGroupId`

* is required

* Type: unknown

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-membergroupid.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/memberGroupId")

### memberGroupId Type

unknown

## title

The title of the report packet.

`title`

* is required

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-title.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/title")

### title Type

`string`

## description

A description of the report packet.

`description`

* is optional

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-description.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/description")

### description Type

`string`

## period

The publishing period of the report packet.

`period`

* is optional

* Type: `object` ([Period](report-packet-reportpacket-properties-period.md))

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-period.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period")

### period Type

`object` ([Period](report-packet-reportpacket-properties-period.md))

## containsProtectedHealthInfo

Indicates the report packet contains PHI.

`containsProtectedHealthInfo`

* is optional

* Type: `boolean`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-containsprotectedhealthinfo.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/containsProtectedHealthInfo")

### containsProtectedHealthInfo Type

`boolean`

### containsProtectedHealthInfo Default Value

The default value is:

```json
false
```

## containsSensitiveFinancialInfo

Indicates the report packet contains sensitive financial information.

`containsSensitiveFinancialInfo`

* is optional

* Type: `boolean`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-containssensitivefinancialinfo.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/containsSensitiveFinancialInfo")

### containsSensitiveFinancialInfo Type

`boolean`

### containsSensitiveFinancialInfo Default Value

The default value is:

```json
false
```

## reports

The list of reports included in the packet.

`reports`

* is optional

* Type: `object[]` ([Report](report-packet-reportpacket-properties-reportlist-report.md))

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-reportlist.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/reports")

### reports Type

`object[]` ([Report](report-packet-reportpacket-properties-reportlist-report.md))
