# Proposal: Template Properties CLI Support

## Overview

Extend the `sf mobilesdk` CLI to support template-specific properties that can be injected during template cloning. This enables templates to require additional configuration values beyond the standard `appname`, `packagename`, and `organization` parameters.

## Problem Statement

Some templates require additional configuration properties that must be injected at clone time. For example, the `AgentforceDemo` template requires enhanced in-app chat deployment configuration values:
- ES developer name
- Organization ID  
- API URL

Currently, these values must be manually edited in the generated project files after cloning, which is error-prone and not user-friendly.

## Proposed Solution

Add support for template properties via CLI flags using the pattern `--template-property-<propertyName>`.

### Syntax

```bash
sf mobilesdk <platform> createwithtemplate \
  --template="<templateName>" \
  --appname="<appName>" \
  --packagename="<packageName>" \
  --organization="<organization>" \
  --template-property-<property1>="<value1>" \
  --template-property-<property2>="<value2>" \
  ...
```

### Example Usage

```bash
sf mobilesdk ios createwithtemplate \
  --template="AgentforceDemo" \
  --appname="MyAgentApp" \
  --packagename="com.example.myagentapp" \
  --organization="MyOrg" \
  --template-property-developerName="My_Support" \
  --template-property-organizationId="00D1234567890ABC" \
  --template-property-apiURL="https://example.my.salesforce-scrt.com"
```

## Implementation Details

### CLI Changes

1. **Flag Parsing**: Parse all flags matching the pattern `--template-property-*`
2. **Property Extraction**: Extract property names by removing the `--template-property-` prefix
3. **Config Object Construction**: Build a `templateProperties` object and attach it to the config passed to `template.js`:

```javascript
const templateProperties = {};
for (const [flag, value] of Object.entries(parsedFlags)) {
  if (flag.startsWith('template-property-')) {
    const propertyName = flag.replace('template-property-', '');
    templateProperties[propertyName] = value;
  }
}

const config = {
  appname: parsedFlags.appname,
  packagename: parsedFlags.packagename,
  organization: parsedFlags.organization,
  templateProperties: templateProperties  // Add this
};
```

### Template Compatibility

- Templates that don't use `templateProperties` will continue to work unchanged (backward compatible)
- Templates can access properties via `config.templateProperties.<propertyName>` in their `template.js` files
- The template system already supports this structure (see `AgentforceDemo/template.js`)

### Template Schema

Templates can declare required properties in their `template.json`:

```json
{
  "templatePrerequisites": {
    "templateProperties": {
      "developerName": {
        "value": "",
        "required": true,
        "description": "ES developer name from chat deployment configuration"
      },
      "organizationId": {
        "value": "",
        "required": true,
        "description": "Organization ID from chat deployment configuration"
      }
    }
  }
}
```

## Validation Considerations

The CLI could optionally:
- Read `template.json` to identify required `templateProperties`
- Validate that all required properties are provided before cloning
- Provide helpful error messages if required properties are missing

## Questions for Discussion

1. Should we validate required properties before cloning?
2. Should we add a `--list-template-properties` flag to show required properties for a template?
3. Should we support reading from a JSON file as an alternative to flags?

