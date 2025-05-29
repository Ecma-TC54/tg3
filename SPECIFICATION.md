# Common Lifecycle Enumeration (CLE) Specification v1.0.0

!! THIS IS A WORK IN PROGRESS AND NOT READY FOR USE.

## Overview

The Common Lifecycle Enumeration (CLE) provides a standardized format for communicating software component lifecycle events in a machine-readable format. This specification defines the JSON schema and requirements for CLE documents.

## Requirements Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Types of Work

The following types of work are commonly associated with software projects regardless of whether they are open source or not.

- `Marketing`: Promoting and advertising a software project to potential users.
- `Substantial Modifications`: Making substantial changes to a software project that are not considered bug fixes or security fixes, such as adding new features or functionality.
- `Bug Fixes`: Addressing and resolving issues or defects in a software project.
- `Security Fixes`: A distinct type of bug fix focused on security vulnerabilities that is useful to differentiate from other types of bug fixes.
- `Distribution`: The process of making a software project available for use by others.
- `Documentation`: Writing and updating documentation for a software project to help users understand how to use it.

## Schema Definition

### JSON Schema

TODO: Add JSON Schema reference here.

### Top-level Fields

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `$schema` | string | URI identifying the [JSON Schema](https://json-schema.org/) document that describes the version of the CLE schema to use (e.g., "https://TODO/cle.v1.0.0.json"). CLE is built on JSON Schema draft 2020-12. |
| `events` | array | Ordered array of Event objects representing the component's lifecycle events. MUST be chronologically ordered by effective date, descending. |

**Additional Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `versions` | array[object] | Array of version objects. Each object must contain either a `version` field with a simple string or a `range` field with a VERS-formatted version range string. |
| `definitions` | object | Container for reusable policy definitions that can be referenced throughout the document. |

### Definitions Object

The definitions object allows specification of reusable policies and calculations that can be referenced by events.

**Support**

This is a list of support policies provided for a specific version or version range of a component. Note this SHOULD NOT include third party support policies or options.

| Field | Type | Description |
|-------|------|-------------|
| `support` | array[object] | List of support policies. |

Each support policy object has the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | **Required.** Unique identifier for the support policy |
| `description` | string | **Required.** Human readable description of the policy |
| `url` | string | **Optional.** URL to detailed documentation about this support policy |

Example support policy definition:
```json
{
  "definitions": {
    "support": [
      {
        "id": "standard",
        "description": "Standard product support policy",
        "url": "https://example.com/support/standard"
      },
      {
        "id": "extended",
        "description": "Extended support with priority response times",
        "url": "https://example.com/support/extended"
      },
      {
        "id": "premium",
        "description": "Premium support with 24/7 coverage and dedicated team",
      }
    ]
  }
}
```

### Event Object

The base object that represents a discrete lifecycle event. All events share these common fields.

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | The type of lifecycle event. MUST be one of the defined Event Types. |
| `effective` | string | The time when the event takes effect, as an ISO 8601 formatted timestamp in UTC (ending in "Z"). |
| `published` | string | The time when the event was first published, as an ISO 8601 formatted timestamp in UTC (ending in "Z"). |
| `modified` | string | The time when the event was last modified, as an ISO 8601 formatted timestamp in UTC (ending in "Z"). |

### Event Types

#### released
*Category: Version Event*

Indicates when a component version is released and available for use.

**Additional Required Fields:**

- version

**Additional Optional Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `license` | string | License identifier that summarizes the license as declared in the component's metadata. |

Example:
```json
{
  "type": "released",
  "effective": "2024-01-01T00:00:00Z",
  "version": "1.0.0",
  "license": "MIT",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
}
```

#### endOfDevelopment
*Category: Version Event*

The manufacturer or maintainer ceases work on `Substantial Modifications` for a specific version or version range of a component or service. `Security Fixes` and `Bug Fixes` will continue to be provided for this specific version or version range until `endOfSupport` is declared, but no new features or enhancements will be added.

> TL;DR: Ceasing Substantial Modifications.

**Additional Required Fields:**

- versions
- supportId

Example:
```json
{
  "type": "endOfDevelopment",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "version": "1.0.0"
    },
    {
      "range": "vers:npm/>=1.0.0|<2.0.0"
    }
  ],
  "supportId": "standard",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
}
```

#### endOfSupport
*Category: Version Event*

The manufacturer or maintainer ceases providing `Security Fixes` and `Bug Fixes` for a specific version or version range of a component or service.

The `supportId` field MUST be included and used to specify which support policy is ending, referencing a support policy defined in the definitions section.

An `endOfSupport` event should only exist when a support definition object exists.

> TL;DR: Ceasing Security Fixes and Bug Fixes.

**Additional Required Fields:**

- versions
- supportId

Example:
```json
{
  "type": "endOfSupport",
  "effective": "2024-01-01T00:00:00Z",
  "versions": ["1.0.0"],
  "supportId": "guaranteed",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z"
}
```

#### endOfLife
*Category: Version Event*

The manufacturer or maintainer formally ceases all work (including `Distribution`, `Substantial Modifications`, `Bug Fixes`, `Security Fixes`, `Documentation`, and `Maintenance`) for a specific version or version range of a component. No further updates, support, or distribution will be provided for this specific version or version range. The component is considered retired.

> TL;DR: Ceasing all work.

**Additional Required Fields:**

- versions

Example:
```json
{
  "type": "endOfLife",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "range": "vers:npm/>=1.0.0|<2.0.0"
    }
  ],
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
}
```

### endOfDistribution
*Category: Version Event*

The manufacturer or maintainer ceases distribution of a specific version or version range of a component or service. This should only be used when the manufacturer has control over the distribution of the component or service.

**Additional Required Fields:**

- versions

Example:
```json
{
  "type": "endOfDistribution",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "version": "1.0.0"
    }
  ],
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
}
```

#### endOfMarketing
*Category: Version Event*

The manufacturer or maintainer ceases marketing and promotion of a specific version or version range of a component or service. The component or service may still be available, and existing support policies may remain in effect, but the manufacturer will no longer seek new customers or promote its use.

> TL;DR: Ceasing marketing and promotion.

**Additional Required Fields:**

- versions

Example:
```json
{
  "type": "endOfMarketing",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "range": "vers:npm/>=1.0.0|<2.0.0"
    }
  ],
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
}
```

#### supersededBy
*Category: Version Event*

Indicates when a version of a component is superseded by another version of a component. This should only exist for components in which version progression is not implicit, for example, a component is not using semantic versioning.

**Additional Required Fields:**
- `supersededByVersion`: Plain version string that supersedes it (e.g., "2.0.0")

**Additional Optional Fields:**

- versions

Example:
```json
{
  "type": "supersededBy",
  "effective": "2024-01-01T00:00:00Z",
  "versions": [
    {
      "version": "1.0.0"
    }
  ],
  "supersededByVersion": "2.0.0",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
}
```

#### componentRenamed
*Category: Component Event*

Indicates when a component is renamed.

**Additional Required Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `identifiers` | array[object] | Array of identifier objects specifying the new identifiers for the component. Each object must have `type` field set to "PURL" and `value` field containing a valid Package URL (PURL) string. |

**Additional Optional Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Human-readable description of the event. SHOULD be clear and concise, and SHOULD explain why the event is occurring. |
| `references` | array[string] | List of URLs to supporting documentation. SHOULD point to authoritative sources. |

Example:
```json
{
  "type": "componentRenamed",
  "effective": "2024-01-01T00:00:00Z",
  "published": "2023-06-01T00:00:00Z",
  "modified": "2023-06-01T00:00:00Z",
  "description": "The component has been renamed to 'new-component', because the product has been acquired by a new company",
  "references": [
    "https://example.com/component-renamed"
  ],
  "identifiers": [
      {
      "type": "PURL",
      "value": "pkg:npm/new-component"
    }
  ]
}
```

## Event Categories

The CLE specification supports two distinct categories of events: `Version Events` and `Component Events`.

The distinction between these categories is important as it affects how tools and systems should process and interpret the events. `Version Events` are typically used for lifecycle management and support planning, while `Component Events` are crucial for maintaining proper component identification and traceability over time.

### Version Events

Events that affect specific versions or ranges of versions of a component. These events use either the `version` or `range` field to specify which versions are affected. Version Events include:

- `released`
- `endOfDevelopment`
- `endOfSupport`
- `endOfLife`
- `supersededBy`

### Component Events

Events that affect the component itself and may impact how the component is identified or referenced. These events often affect all versions of a component from the effective date forward. Component Events include:

- `componentRenamed`: Changes how the component is identified

## Distribution

TODO: think about the possible distribution methods around GitHub, TEA, etc.

## Example

```json
{
  "$schema": "https://TODO/cle.v1.0.0.json",
  "events": [
    {
      "type": "released",
      "effective": "2019-01-01T00:00:00Z",
      "versions": [
        {
          "version": "1.0.0"
        }
      ],
      "license": "MIT",
      "published": "2019-01-01T00:00:00Z",
      "modified": "2019-01-01T00:00:00Z"
    },
    {
      "type": "endOfSupport",
      "effective": "2020-01-01T00:00:00Z",
      "versions": [
        {
          "range": "vers:npm/>=1.0.0|<2.0.0"
        }
      ],
      "published": "2020-01-01T00:00:00Z",
      "modified": "2020-01-01T00:00:00Z"
    },
    {
      "type": "componentRenamed",
      "effective": "2020-01-01T00:00:00Z",
      "description": "The component has been renamed to 'new-component', because the product has been acquired by a new company",
      "identifiers": [
        {
          "type": "PURL",
          "value": "pkg:npm/new-component"
        }
      ],
      "references": [
        "https://example.com/component-renamed"
      ],
      "published": "2020-01-01T00:00:00Z",
      "modified": "2020-01-01T00:00:00Z"
    }
  ]
}
```

## Use Cases

The CLE specification aims to address several common use cases in software component lifecycle management. The following use cases are supported in version 1.0.0:

### v1.0.0 Supported Use Cases:
- General Availability: Track when new versions of a component are released and available for use
- End of Support/End of Life: Communicate when versions will no longer receive updates or support
- Component Renaming: Handle cases where a component's identifiers change (e.g., package rename)

### Future Use Cases:
- Complex License Changes: Handling license changes for previously released versions. A maintainer may change the license for a previously released version, we may use a `licenseChange` event with a range of versions to handle this use case.
- Component Bundling/Unbundling: Track when components are bundled into or extracted from larger packages
- Component Acquisition: Handle cases where components change ownership
- Extended Support: Support for third-party extended support offerings
- Third Party Claims: Handling CLE from a third party perspective making claims about a component which they have no operational control over.
- Component Forking: Track when components are forked into new projects
- Export Restrictions: Handle cases where components become restricted in certain regions
- Security Status Changes: Track when components are marked as compromised or unsafe

These use cases will be addressed in future versions of the specification based on community feedback and requirements.

## References

- Package URL (PURL) Specification: https://github.com/package-url/purl-spec
- Version Range (VERS) Specification: https://github.com/package-url/purl-spec/blob/master/VERSION-RANGE-SPEC.rst