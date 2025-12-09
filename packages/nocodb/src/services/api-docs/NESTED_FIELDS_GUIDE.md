# NocoDB Nested Fields Guide

This guide explains how to control nested (linked) field data in NocoDB API responses for both v2 and v3 APIs.

## Overview

When your tables have relationships (Links/LinkToAnotherRecord fields), API responses can include data from linked records. This guide explains how to control what nested data is returned.

## API v2 (Legacy)

In API v2, nested fields return **only primary keys by default**. To get actual field data from nested records, you need to explicitly request fields using the `nested` parameter.

### Basic Syntax

```
nested[<columnName>][fields]=<field1>,<field2>,...
```

### Examples

#### Get all fields from nested records

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*
```

This will return all fields from linked `Projects` records instead of just IDs.

#### Get specific fields from nested records

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=name,status,budget
```

This returns only the `name`, `status`, and `budget` fields from linked `Projects` records.

#### Multiple nested relations

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Customers][fields]=name,email
```

### Additional Nested Parameters (v2)

- `nested[<columnName>][where]` - Filter nested records
- `nested[<columnName>][sort]` - Sort nested records
- `nested[<columnName>][limit]` - Limit number of nested records (default: 25)
- `nested[<columnName>][offset]` - Pagination offset for nested records

Example:
```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][limit]=50&nested[Projects][sort]=name
```

## API v3 (Current)

API v3 has a different approach - **nested fields return actual data by default**, not just counts or IDs.

### Key Differences from v2

1. **Full data by default**: Nested records include all their fields automatically
2. **No per-field control**: You cannot specify which nested fields to include (all or nothing)
3. **Global limit**: The `nestedLimit` parameter controls the number of nested records for ALL link fields

### Parameters

#### `nestedLimit` (Query Parameter)

Controls how many linked records are returned for each link field in the response.

- **Default**: 1000 records per link field
- **Configurable via**: `DB_QUERY_LIMIT_LTAR_V3_LIMIT` environment variable
- **Applies to**: All link/relation fields in the response

**Example:**
```bash
GET /api/v3/data/{baseId}/{tableId}/records?nestedLimit=50
```

This limits each link field to return at most 50 linked records with their full data.

#### `nestedPage` (Query Parameter)

Controls pagination of nested records.

- **Default**: 1 (first page)
- **Use with**: `nestedLimit` to navigate through pages of nested data

**Example:**
```bash
GET /api/v3/data/{baseId}/{tableId}/records?nestedLimit=50&nestedPage=2
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
          },
          {
            "id": "proj002",
            "fields": {
              "ProjectName": "Mobile App",
              "Status": "Planning",
              "Budget": 75000
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

### Pagination with Nested Data

When `nestedNext` is present in the response, it means some link fields have more records than the `nestedLimit`. Use `nestedPage` to retrieve additional pages:

```bash
# First page
GET /api/v3/data/{baseId}/{tableId}/records?nestedLimit=50&nestedPage=1

# Second page
GET /api/v3/data/{baseId}/{tableId}/records?nestedLimit=50&nestedPage=2
```

## Comparison Table

| Feature | API v2 | API v3 |
|---------|--------|--------|
| Default nested data | Primary keys only | Full record data |
| Control nested fields | Per-column: `nested[col][fields]` | Global: `nestedLimit` |
| Default limit | 25 per column | 1000 per column |
| Per-field filtering | ✅ Yes | ❌ No |
| Nested sorting | ✅ Yes | ❌ No |
| Nested filtering | ✅ Yes | ❌ No |
| Max nesting depth | Unlimited | 3 levels (configurable) |

## Migration from v2 to v3

If you're migrating from v2 to v3:

1. **Remove `nested[column][fields]` parameters** - v3 returns full data automatically
2. **Replace per-column limits with global `nestedLimit`** - v3 uses a single limit for all fields
3. **Update response parsing** - v3 uses a different structure with `id` and `fields` objects
4. **Adjust pagination logic** - v3 uses `nestedPage` instead of per-column offsets

### Example Migration

**Before (v2):**
```bash
GET /api/v2/tables/tbl123/records?nested[Projects][fields]=*&nested[Projects][limit]=25
```

**After (v3):**
```bash
GET /api/v3/data/base123/tbl123/records?nestedLimit=25
```

## Best Practices

### For v2
- Use `nested[col][fields]=*` to get all nested data
- Be specific with fields to reduce response size: `nested[col][fields]=id,name`
- Use limits to prevent large payloads: `nested[col][limit]=50`

### For v3
- Use `nestedLimit` to control response size
- Monitor `nestedNext` in responses to detect truncated data
- Consider the default 1000-record limit when designing your application
- For very large datasets, implement pagination using `nestedPage`

## Environment Configuration

### v3 Default Nested Limit

You can configure the default `nestedLimit` value server-wide:

```bash
DB_QUERY_LIMIT_LTAR_V3_LIMIT=500
```

This sets the default to 500 records per link field instead of 1000.

### Maximum Nesting Depth

The maximum nesting depth can be configured via code (default is 3 levels deep). This prevents infinite recursion in self-referential relationships.

## Common Issues

### Issue: Getting only IDs in v2 responses
**Solution**: Add `nested[columnName][fields]=*` to your query

### Issue: Too many nested records in v3
**Solution**: Use `nestedLimit` parameter to reduce the number of returned records

### Issue: Missing nested data
**Solution**: Check if the relationship exists and the user has permissions to access linked records

## Additional Resources

- [NocoDB API Documentation](https://docs.nocodb.com/)
- [REST API Reference](https://docs.nocodb.com/developer-resources/rest-apis)
- [Swagger/OpenAPI Documentation](http://<your-nocodb-instance>/api/v3/docs)
