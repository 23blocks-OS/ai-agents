---
name: forms-validation-api
description: Configure and debug 23blocks form validation rules. Use when setting up field validators, custom validation, or troubleshooting validation errors.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Forms Validation API

Complete reference for 23blocks form validation system.

## Built-in Validators

### String Validators

#### required
Field must have a non-empty value.
```json
{ "required": true }
```
Error: `This field is required`

#### minLength / maxLength
String length constraints.
```json
{
  "minLength": 2,
  "maxLength": 100
}
```
Error: `Must be at least 2 characters` / `Must be at most 100 characters`

#### pattern
Regex pattern matching.
```json
{
  "pattern": "^[A-Z]{2}[0-9]{4}$"
}
```
Error: `Invalid format`

#### format
Predefined format validators.
```json
{ "format": "email" }    // RFC 5322 email
{ "format": "url" }      // Valid URL
{ "format": "phone" }    // E.164 phone number
{ "format": "uuid" }     // UUID v4
```

### Numeric Validators

#### min / max
Numeric range constraints.
```json
{
  "min": 0,
  "max": 100
}
```
Error: `Must be at least 0` / `Must be at most 100`

### Selection Validators

#### oneOf
Value must be in allowed list.
```json
{
  "oneOf": ["option1", "option2", "option3"]
}
```
Error: `Invalid selection`

### Async Validators

#### unique
Value must be unique in database.
```json
{
  "unique": true,
  "uniqueScope": "organization_id"  // optional scope
}
```
Error: `This value is already taken`

---

## Custom Validators

### Register Custom Validator

```bash
curl -X POST "$API_URL/forms/validators" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "strong_password",
    "description": "Requires uppercase, lowercase, number, and special char",
    "pattern": "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$",
    "error_message": "Password must include uppercase, lowercase, number, and special character"
  }'
```

### Use Custom Validator

```json
{
  "name": "password",
  "type": "password",
  "validation": {
    "custom": "strong_password"
  }
}
```

### Webhook Validator

For complex validation requiring server logic:

```bash
curl -X POST "$API_URL/forms/validators" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "verify_coupon",
    "type": "webhook",
    "webhook_url": "https://your-api.com/validate-coupon",
    "timeout_ms": 5000,
    "error_message": "Invalid or expired coupon code"
  }'
```

Webhook receives:
```json
{
  "field": "coupon_code",
  "value": "SAVE20",
  "form_data": { "email": "user@example.com" }
}
```

Webhook responds:
```json
{ "valid": true }
// or
{ "valid": false, "error": "Coupon expired on 2024-01-01" }
```

---

## Cross-Field Validation

Validate fields based on other field values.

### Conditional Required

```json
{
  "name": "company_name",
  "type": "text",
  "validation": {
    "requiredIf": {
      "field": "account_type",
      "equals": "business"
    }
  }
}
```

### Field Comparison

```json
{
  "name": "confirm_password",
  "type": "password",
  "validation": {
    "equals": {
      "field": "password",
      "error": "Passwords must match"
    }
  }
}
```

### Date Range

```json
{
  "name": "end_date",
  "type": "date",
  "validation": {
    "after": {
      "field": "start_date",
      "error": "End date must be after start date"
    }
  }
}
```

---

## Validation API Endpoints

### POST /forms/:schema_id/validate

Validate data without submitting.

**Request:**
```bash
curl -X POST "$API_URL/forms/frm_abc123/validate" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "email": "invalid-email",
      "message": "Hi"
    }
  }'
```

**Response 200 (Valid):**
```json
{
  "valid": true,
  "errors": {}
}
```

**Response 200 (Invalid):**
```json
{
  "valid": false,
  "errors": {
    "email": "Invalid email format",
    "message": "Must be at least 10 characters"
  }
}
```

### GET /forms/validators

List available validators.

**Request:**
```bash
curl -X GET "$API_URL/forms/validators" \
  -H "Authorization: Bearer $TOKEN"
```

**Response 200:**
```json
{
  "builtin": [
    { "name": "required", "description": "Field must have a value" },
    { "name": "email", "description": "Valid email format" },
    { "name": "minLength", "description": "Minimum string length" }
  ],
  "custom": [
    { "name": "strong_password", "description": "Complex password requirements" }
  ]
}
```

---

## Error Messages

### Customize Error Messages

Per-field custom messages:

```json
{
  "name": "age",
  "type": "number",
  "validation": {
    "min": 18,
    "max": 120,
    "messages": {
      "min": "You must be at least 18 years old",
      "max": "Please enter a valid age"
    }
  }
}
```

### Localized Messages

```json
{
  "name": "email",
  "type": "email",
  "validation": {
    "format": "email",
    "messages": {
      "format": {
        "en": "Please enter a valid email",
        "es": "Por favor ingrese un correo electr\u00f3nico v\u00e1lido",
        "fr": "Veuillez entrer une adresse email valide"
      }
    }
  }
}
```

---

## Validation Timing

Configure when validation runs:

```json
{
  "settings": {
    "validateOn": {
      "change": true,      // Validate as user types
      "blur": true,        // Validate when field loses focus
      "submit": true       // Validate on form submit (always true)
    },
    "debounceMs": 300,     // Debounce for change validation
    "asyncTimeout": 5000   // Timeout for async validators
  }
}
```

---

## Debugging Validation

### Enable Debug Mode

```bash
curl -X POST "$API_URL/forms/frm_abc123/validate?debug=true" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"data": {"email": "test"}}'
```

**Response with debug info:**
```json
{
  "valid": false,
  "errors": {
    "email": "Invalid email format"
  },
  "debug": {
    "validators_run": [
      { "field": "email", "validator": "required", "passed": true, "ms": 0 },
      { "field": "email", "validator": "format:email", "passed": false, "ms": 1 }
    ],
    "total_ms": 1
  }
}
```

### Common Validation Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Validation not triggering | Field name mismatch | Check field names match schema |
| Async validation timeout | Slow webhook | Increase `asyncTimeout` or optimize webhook |
| Pattern not matching | Regex escaping | Double-escape backslashes in JSON |
| Cross-field validation fails | Field order | Ensure dependent field exists in schema |
