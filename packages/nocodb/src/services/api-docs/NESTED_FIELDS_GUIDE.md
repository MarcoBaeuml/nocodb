# NocoDB Nested Fields Guide

This guide explains how to get actual data from linked/nested fields instead of just counts in NocoDB API responses.

## Problem
By default, both API v2 and v3 return only **counts** for linked (LTAR) fields:

```json
{
  "records": [{
    "id": 1,
    "fields": {
      "Title": "post 1",
      "post_interactions": 1  // Count, not actual data
    }
  }]
}
```

## Solution: Use nested[columnName][fields] Parameter

### Basic Syntax

```bash
nested[columnName][fields]=*           # Get all fields
nested[columnName][fields]=field1,field2  # Get specific fields
```

### API v2 Example

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=name,status,budget
```

### API v3 Example

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[post_interactions][fields]=*
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=name,status
```

### With URL Encoding

Browsers and some HTTP clients require URL encoding for brackets:

```bash
# [ becomes %5B and ] becomes %5D
GET /api/v3/data/{baseId}/{tableId}/records?nested%5Bpost_interactions%5D%5Bfields%5D=*
```

## Using Column ID Instead of Title

If the column title doesn't work (e.g., special characters, spaces), use the column ID:

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[cl_abc123][fields]=*
```

To find the column ID:
1. Use the table metadata API: `GET /api/v2/meta/tables/{tableId}`
2. Look for your column in the `columns` array
3. Use the `id` field

## Response Format

### With nested[columnName][fields]=*

```json
{
  "records": [{
    "id": 1,
    "fields": {
      "Title": "post 1",
      "post_interactions": [
        {
          "id": "rec1",
          "fields": {
            "type": "like",
            "user": "John",
            "timestamp": "2025-12-09"
          }
        },
        {
          "id": "rec2",
          "fields": {
            "type": "comment",
            "user": "Jane",
            "timestamp": "2025-12-09"
          }
        }
      ]
    }
  }]
}
```

## Additional Parameters

### Control Quantity (nestedLimit)

```bash
# Limit to 50 linked records per field (v3 only)
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nestedLimit=50
```

### Filter Nested Records (nested[columnName][where])

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][where]=(status,eq,active)
```

### Sort Nested Records (nested[columnName][sort])

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][sort]=name
```

### Pagination (v2 only - per column)

```bash
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][limit]=25&nested[Projects][offset]=0
```

## Multiple Linked Fields

Get data from multiple link fields in one request:

```bash
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nested[Customers][fields]=name,email
```

## Complete Example

```bash
# Get posts with full interaction data, sorted by timestamp, limited to 10 per post
GET /api/v3/data/base123/posts/records?nested[interactions][fields]=*&nested[interactions][sort]=-timestamp&nestedLimit=10
```

Response:
```json
{
  "records": [
    {
      "id": "post1",
      "fields": {
        "Title": "My First Post",
        "interactions": [
          {
            "id": "int10",
            "fields": {
              "type": "comment",
              "user": "Alice",
              "timestamp": "2025-12-09 15:30:00"
            }
          },
          {
            "id": "int9",
            "fields": {
              "type": "like",
              "user": "Bob",
              "timestamp": "2025-12-09 14:20:00"
            }
          }
          // ... up to 10 interactions
        ]
      }
    }
  ]
}
```

## Comparison: v2 vs v3

| Feature | API v2 | API v3 |
|---------|--------|--------|
| Get nested data | `nested[col][fields]=*` | `nested[col][fields]=*` |
| Column ID support | ✅ Yes | ✅ Yes |
| Per-column limit | `nested[col][limit]` | Global `nestedLimit` |
| Per-column offset | `nested[col][offset]` | ❌ Not supported |
| Filtering | `nested[col][where]` | `nested[col][where]` |
| Sorting | `nested[col][sort]` | `nested[col][sort]` |
| Default limit | 25 per column | 1000 global |

## Troubleshooting

### Still Getting Counts?

1. **Check column name**: Use exact column title or column ID
2. **Try URL encoding**: Encode brackets as `%5B` and `%5D`
3. **Use column ID**: If title has special characters, use the column ID
4. **Check API version**: Ensure you're using the correct endpoint

### Common Mistakes

❌ **Wrong:** `nested[Projects]=*` (missing `[fields]`)
✅ **Correct:** `nested[Projects][fields]=*`

❌ **Wrong:** `nested[post interactions][fields]=*` (space in URL without encoding)
✅ **Correct:** `nested[post_interactions][fields]=*` or use column ID

## Alternative: Dedicated Link Endpoints

For getting ONLY linked records (without the parent record):

```bash
# API v3
GET /api/v3/data/{baseId}/{tableId}/links/{linkFieldId}/{recordId}

# API v2  
GET /api/v2/tables/{tableId}/links/{linkFieldId}/{rowId}
```

## Best Practices

1. **Request specific fields** when possible to reduce response size:
   ```bash
   nested[Projects][fields]=name,status  # Better than *
   ```

2. **Set appropriate limits** to avoid large payloads:
   ```bash
   nestedLimit=25  # v3
   nested[Projects][limit]=25  # v2
   ```

3. **Use pagination** for large datasets (v2 only)

4. **Combine with main query filters**:
   ```bash
   where=(status,eq,active)&nested[Projects][fields]=*
   ```

## Environment Variables

### Global Nested Limit (v3)

Configure the default limit for nested records:

```bash
DB_QUERY_LIMIT_LTAR_V3_LIMIT=1000  # Default
```

## Summary

✅ **Use** `nested[columnName][fields]=*` to get actual nested data instead of counts
✅ **Works** with both column title and column ID  
✅ **Supports** filtering, sorting, and pagination
✅ **Available** in both API v2 and v3
✅ **URL encode** brackets in actual HTTP requests
