# Quick Answer: Get Nested Field Data Instead of Counts

## Problem
Linked fields return counts instead of actual data.

## Solution

Use the `nested[columnName][fields]` parameter:

```bash
# Get all fields from linked records
nested[columnName][fields]=*

# Get specific fields only
nested[columnName][fields]=field1,field2,field3
```

## Examples

### API v3
```bash
# Basic - get all nested data
GET /api/v3/data/{baseId}/{tableId}/records?nested[post_interactions][fields]=*

# URL encoded (for browsers)
GET /api/v3/data/{baseId}/{tableId}/records?nested%5Bpost_interactions%5D%5Bfields%5D=*

# Specific fields only
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=name,status,budget

# With limit
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nestedLimit=50

# Multiple link fields
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nested[Customers][fields]=name,email
```

### API v2
```bash
# Basic
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*

# With filters and sorting
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][where]=(status,eq,active)&nested[Projects][sort]=name

# With pagination
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*&nested[Projects][limit]=25&nested[Projects][offset]=0
```

## Response Comparison

### Without Parameter (Default)
```json
{
  "records": [{
    "id": 1,
    "fields": {
      "Title": "post 1",
      "post_interactions": 5  // Just a count
    }
  }]
}
```

### With nested[post_interactions][fields]=*
```json
{
  "records": [{
    "id": 1,
    "fields": {
      "Title": "post 1",
      "post_interactions": [  // Actual data!
        {
          "id": "rec1",
          "fields": {
            "type": "like",
            "user": "John"
          }
        },
        {
          "id": "rec2",
          "fields": {
            "type": "comment",
            "user": "Jane"
          }
        }
      ]
    }
  }]
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Still getting counts | Try URL encoding: `%5B` for `[`, `%5D` for `]` |
| Column not found | Use column ID instead of title (get from `GET /api/v3/meta/tables/{tableId}`) |
| Too much data | Add `&nestedLimit=25` (v3) or `&nested[col][limit]=25` (v2) |
| Special characters in name | Use column ID from table metadata API |

## Key Points

✅ Works in both v2 and v3
✅ Supports both column title and column ID
✅ Can request specific fields or all fields with `*`
✅ Can filter, sort, and paginate nested data
✅ Remember to URL encode brackets in actual HTTP requests

For more details, see [NESTED_FIELDS_GUIDE.md](./NESTED_FIELDS_GUIDE.md)
