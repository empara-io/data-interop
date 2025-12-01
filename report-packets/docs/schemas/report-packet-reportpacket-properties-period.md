# Period Schema

```txt
https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period
```

The publishing period of the report packet.

| Abstract            | Extensible | Status         | Identifiable | Custom Properties | Additional Properties | Access Restrictions | Defined In                                                                             |
| :------------------ | :--------- | :------------- | :----------- | :---------------- | :-------------------- | :------------------ | :------------------------------------------------------------------------------------- |
| Can be instantiated | No         | Unknown status | No           | Forbidden         | Allowed               | none                | [report-packet.schema.json\*](../out/report-packet.schema.json "open original schema") |

## period Type

`object` ([Period](report-packet-reportpacket-properties-period.md))

# period Properties

| Property        | Type     | Required | Nullable       | Defined by                                                                                                                                                                                                          |
| :-------------- | :------- | :------- | :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [start](#start) | `string` | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-period-properties-start.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period/properties/start") |
| [end](#end)     | `string` | Optional | cannot be null | [ReportPacketList](report-packet-reportpacket-properties-period-properties-end.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period/properties/end")     |

## start

The start of the period.

`start`

* is optional

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-period-properties-start.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period/properties/start")

### start Type

`string`

### start Constraints

**date time**: the string must be a date time string, according to [RFC 3339, section 5.6](https://tools.ietf.org/html/rfc3339 "check the specification")

## end

The end of the period.

`end`

* is optional

* Type: `string`

* cannot be null

* defined in: [ReportPacketList](report-packet-reportpacket-properties-period-properties-end.md "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json#/items/properties/period/properties/end")

### end Type

`string`

### end Constraints

**date time**: the string must be a date time string, according to [RFC 3339, section 5.6](https://tools.ietf.org/html/rfc3339 "check the specification")
