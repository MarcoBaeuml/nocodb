# NocoDB Nested Fields Guide

This guide explains how to control nested (linked) field data in NocoDB API responses for both v2 and v3 APIs.

## Overview

When your tables have relationships (Links/LinkToAnotherRecord fields), API responses can include data from linked records. **By default, both v2 and v3 APIs return only counts or primary keys for nested fields**. This guide explains how to get actual field data.

## Common Behavior (Both v2 and v3)

### Default Behavior

Without any nested parameters, linked fields return only:
- **Count** (number of linked records)
- **Primary keys** (IDs of linked records)

**They do NOT return the actual field data from linked records.**

### Getting Actual Data

To get actual field data from linked records, you must use the `nested[columnName][fields]` parameter:

```bash
# Get all fields from linked records
nested[columnName][fields]=*

# Get specific fields only
nested[columnName][fields]=field1,field2,field3
```

## API v2 

### Basic Syntax

```bash
GET /api/v2/tables/{tableId}/records?nested[<columnName>][fields]=*
```

### Examples

#### Get all fields from nested records

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*
```

Returns all fields from linked `Projects` records instead of just counts.

#### Get specific fields from nested records

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=name,status,budget
```

Returns only the `name`, `status`, and `budget` fields from linked `Projects` records.

#### Multiple nested relations

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Customers][fields]=name,email
```

### Additional Nested Parameters (v2)

- `nested[<columnName>][where]` - Filter nested records
- `nested[<columnName>][sort]` - Sort nested records
- `nested[<columnName>][limit]` - Limit number of nested records (default: 25)
- `nested[<columnName>][offset]` - Pagination offset for nested records

**Example with limit:**
```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][limit]=50&nested[Projects][sort]=name
```

## API v3

### Important Note

**API v3 uses the SAME nested parameter structure as v2.** The primary difference is in the response format and global limit parameter.

### Basic Syntax

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[<columnName>][fields]=*
```

### Examples

#### Get all fields from nested records

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*
```

#### Get specific fields

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=name,status,budget
```

#### Multiple nested relations

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nested[Customers][fields]=name,email
```

### Additional Parameters (v3)

#### `nestedLimit` (Global Limit)

Controls the maximum number of linked records returned for ALL link fields.

- **Default**: 1000 records per link field
- **Configurable via**: `DB_QUERY_LIMIT_LTAR_V3_LIMIT` environment variable
- **Applies to**: All link/relation fields in the response (when fields are requested)

**Example:**
```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nestedLimit=50
```

This gets all fields from Projects but limits to 50 records maximum.

#### `nestedPage` 

Controls pagination of nested records.

- **Default**: 1 (first page)
- **Use with**: `nestedLimit` to navigate through pages

**Example:**
```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nestedLimit=50&nestedPage=2
```

### Response Structure (v3)

```json
{
  "records": [
    {
      "id": "rec001",
      "fields": {
        "Name": "John Doe",
        "Projects": [
          {
            "id": "proj001",
            "fields": {
              "ProjectName": "Website Redesign",
              "Status": "Active",
              "Budget": 50000
            }
          }
        ]
      }
    }
  ],
  "next": "...",
  "prev": null,
  "nestedNext": "...",
  "nestedPrev": null
}
```

## Comparison Table

| Feature | API v2 | API v3 |
|---------|--------|--------|
| Default nested data | Counts/Primary keys only | Counts/Primary keys only |
| Get full data | `nested[col][fields]=*` | `nested[col][fields]=*` |
| Per-field limit | `nested[col][limit]` | Global `nestedLimit` |
| Default limit | 25 per column | 1000 (global) |
| Per-field filtering | ✅ `nested[col][where]` | ✅ `nested[col][where]` |
| Per-field sorting | ✅ `nested[col][sort]` | ✅ `nested[col][sort]` |
| Response format | Flat objects | Nested `{id, fields}` structure |

## Common Mistake

**Incorrect assumption**: "API v3 returns full nested data by default"

**Reality**: Both v2 and v3 return only counts or primary keys by default. You must explicitly use `nested[columnName][fields]=*` to get actual field data.

## Migration from v2 to v3

The nested parameter structure is the **same** between v2 and v3:

**Before (v2):**
```bash
GET /api/v2/tables/tbl123/records?nested[Projects][fields]=*&nested[Projects][limit]=25
```

**After (v3):**
```bash
GET /api/v3/data/base123/tbl123/records?nested[Projects][fields]=*&nestedLimit=25
```

Main changes:
1. URL structure: `/api/v2/tables/` → `/api/v3/data/{baseId}/`
2. Per-column limits → Global `nestedLimit` parameter
3. Response structure with nested `{id, fields}` format

## Best Practices

1. **Always specify fields**: Use `nested[columnName][fields]=*` to get data instead of counts
2. **Use specific fields when possible**: Request only needed fields to reduce response size
3. **Set appropriate limits**: Use `nestedLimit` to prevent large payloads
4. **Monitor pagination**: Check `nestedNext` in responses for truncated data

## Examples by Use Case

### Get employee data with all project details

```bash
# API v3
GET /api/v3/data/{baseId}/employees/records?nested[Projects][fields]=*&nestedLimit=100
```

### Get orders with customer name and email only

```bash
# API v3
GET /api/v3/data/{baseId}/orders/records?nested[Customer][fields]=name,email
```

### Get posts with limited interactions

```bash
# API v3
GET /api/v3/data/{baseId}/posts/records?nested[post_interactions][fields]=*&nestedLimit=10
```

## Environment Configuration

### v3 Default Nested Limit

Configure the default `nestedLimit` value server-wide:

```bash
DB_QUERY_LIMIT_LTAR_V3_LIMIT=500
```

This sets the default to 500 records per link field instead of 1000.

## Common Issues

### Issue: Getting only counts (numbers) in responses
**Solution**: Add `nested[columnName][fields]=*` to your query

### Issue: Too many nested records
**Solution**: Use `nestedLimit` parameter to reduce the number

### Issue: Missing nested data
**Solution**: Check permissions and verify the relationship exists

## Additional Resources

- [NocoDB API Documentation](https://docs.nocodb.com/)
- [REST API Reference](https://docs.nocodb.com/developer-resources/rest-apis)
- [Swagger/OpenAPI Documentation](http://<your-nocodb-instance>/api/v3/docs)
