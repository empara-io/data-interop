# Report Packet EDI Specification

**Inbound | Semi-Structured Format**  
Transmission from Partner Organization to Empara

## Revision History

| Version | Last Updated |
|---------|--------------|
| 1.0     | 2025-11-20   |

## Overview

The report packet file is a manifest file containing metadata describing report files which have been uploaded to our SFTP server. This document provides details on the required file structure and data schema.

## Data Exchange

All files must be exchanged via the secure file transfer protocol (SFTP). It is recommended to use an SFTP service hosted by Empara as the exchange point, but connections to a partner-hosted SFTP server can be accommodated if necessary.

### SFTP Path Structure

Below is an example of the SFTP site's structure:

```text
.
├── fromEmpara/
│   ├── ...
└── toEmpara/
    ├── reportPackets/
    │   ├── [YYYYMMDD]/
    │   ├── [prefix]_report_packets_[YYYYMMDD].json
    │   └── ...
    └── ...
```

The directory relevant to this specification is `toEmpara/reportPackets`.

### Naming Conventions

The file naming convention is `[prefix]_[file_type]_[YYYYMMDD].json` The prefix is a unique identifier assigned to the trading partner by Empara during implementation. The [YYYYMMDD] portion MUST correspond to the date the file is transmitted.

For example, for a company named "Contoso Benefit Administrators (CBA)" with an assigned ID of `contoso`, the file names would be `contoso_report_packets_20250401.json` and `contoso_report_packets_20250401.json` These files should include data for all relevant member groups for the implementation.

If a trading partner has multiple clients with separate data environments, it is recommended to send separate files for each client to isolate client-specific data characteristics. In this case, the prefixes would be client-specific, such as `datasolutions_acme` and `datasolutions_vandelay`.

The report files referenced in the report packet manifest should be placed in the `[YYYYMMDD]/` directory.

### Miscellaneous Rules

Once a daily file has been transmitted, it MUST NOT be overwritten. As the pipeline's daily scan concludes upon file detection, any subsequent modifications to a transmitted file on the same day will be ignored. To correct or enhance a submitted file, an ad-hoc file MUST be provided as outlined in the section "Ad-hoc File Exchange".

The report files being uploaded MUST be sent before the report packet manifest.

## File Format

### File Encoding

Files should be JSON formatted, in accordance to the schemas given.

Files should use a UTF-8 character encoding without a the order mark (BOM) and should use Unix line endings (line feeds only). Any system limitations preventing adherence to this format must be communicated during implementation.

### Top-level schemas

* [ReportPacketList](./schemas/report-packet.md "A list of report packets to create or update") – `https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json`

### Raw JSON Schema

```json
{
    "$id": "https://empara-io.github.io/data-interop/schemas/v1/report-packet-list.schema.json",
    "$schema": "https://json-schema.org/draft/2020-12/schema",

    "title": "ReportPacketList",
    "description": "A list of report packets to create or update.",

    "type": "array",
    "items": {
        "title": "ReportPacket",
        "description": "Report Packet details.",

        "type": "object",
        "properties": {
            "memberGroupId": {
                "description": "Identifier of the member group the packet is for.",
                "type": "string"
            },
            "title": {
                "description": "The title of the report packet.",
                "type": "string"
            },
            "description": {
                "description": "A description of the report packet.",
                "type": "string"
            },
            "period": {
                "title": "Period",
                "description": "The publishing period of the report packet.",
                "type": "object",
                "properties": {
                    "start": {
                        "description": "The start of the period.",
                        "type": "string",
                        "format": "date-time"
                    },
                    "end": {
                        "description": "The end of the period.",
                        "type": "string",
                        "format": "date-time"
                    }
                }
            },
            "containsProtectedHealthInfo": {
                "description": "Indicates the report packet contains PHI.",
                "type": "boolean",
                "default": false
            },
            "containsSensitiveFinancialInfo": {
                "description": "Indicates the report packet contains sensitive financial information.",
                "type": "boolean",
                "default": false
            },
            "reports": {
                "title": "ReportList",
                "description": "The list of reports included in the packet.",
                "type": "array",
                "items": {
                    "title": "Report",
                    "type": "object",
                    "properties": {
                        "fileName": {
                            "description": "The name of the file uploaded.",
                            "type": "string"
                        },
                        "title": {
                            "description": "The title of the report",
                            "type": "string"
                        },
                        "description": {
                            "description": "The description of the report",
                            "type": "string"
                        },
                        "sortOrder": {
                            "description": "The (relative) display position of this report.",
                            "type": "integer",
                            "default": 0
                        }
                    },
                    "required": ["fileName", "title"]
                }
            }
        },
        "required": ["memberGroupId", "title"]
    }
}
```

### Version Note

The schemas linked above follow the JSON Schema Spec version: `https://json-schema.org/draft/2020-12/schema`
