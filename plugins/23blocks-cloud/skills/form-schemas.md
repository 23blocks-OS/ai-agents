---
name: form-schemas
description: Create and manage 23blocks form schemas with validation rules
---

# 23blocks Form Schemas

Create dynamic form schemas for the 23blocks platform.

## Schema Structure

```json
{
  "name": "user_profile",
  "version": "1.0",
  "fields": [
    {
      "name": "email",
      "type": "email",
      "required": true,
      "validation": {
        "format": "email",
        "unique": true
      }
    },
    {
      "name": "name",
      "type": "text",
      "required": true,
      "validation": {
        "minLength": 2,
        "maxLength": 100
      }
    }
  ]
}
```

## Field Types

- `text`: Single line text
- `textarea`: Multi-line text
- `email`: Email with validation
- `number`: Numeric input
- `select`: Dropdown selection
- `multiselect`: Multiple selection
- `checkbox`: Boolean toggle
- `date`: Date picker
- `datetime`: Date and time
- `file`: File upload
- `image`: Image upload with preview

## Validation Rules

- `required`: Field must have a value
- `minLength` / `maxLength`: Text length limits
- `min` / `max`: Numeric range
- `pattern`: Regex validation
- `format`: Predefined formats (email, url, phone)
- `unique`: Value must be unique in database
- `custom`: Custom validation function

## Usage

When asked to create a form schema:
1. Identify all required fields
2. Choose appropriate field types
3. Add validation rules
4. Consider UX (field order, grouping)
5. Generate the JSON schema
