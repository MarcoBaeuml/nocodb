# Quick Answer: Nested Fields Parameters

## Problem
You want nested fields to contain actual data instead of just counts in NocoDB API responses.

## Solution

**Both API v2 and v3 use the same approach** - you must explicitly request nested field data using the `nested[columnName][fields]` parameter.

### For API v2 AND v3

Use the `nested[columnName][fields]=*` parameter to get full data from linked records:

```bash
# API v2 Example
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*

# API v3 Example  
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*

# Get specific fields only
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=name,status,budget
```

**Why?** Both v2 and v3 return only counts or primary keys by default for nested fields. You must explicitly request fields to get actual data.

## Parameters Reference

| Parameter | API Version | Purpose | Default |
|-----------|-------------|---------|---------|
| `nested[column][fields]` | v2, v3 | Specify which fields to return from nested data | Counts/primary keys only |
| `nestedLimit` | v2, v3 | Limit number of nested records per field | 1000 (v3), 25 (v2) |
| `nestedPage` | v3 | Paginate through nested records | 1 |

## Examples

### Get all nested data
```bash
# API v2
GET /api/v2/tables/tbl_employees/records?nested[Department][fields]=*&nested[Projects][fields]=*

# API v3 (same approach)
GET /api/v3/data/ws_abc123/tbl_employees/records?nested[Department][fields]=*&nested[Projects][fields]=*
```

### Limit nested records
```bash
# API v3 - Get nested data with limit
GET /api/v3/data/ws_abc123/tbl_employees/records?nested[Projects][fields]=*&nestedLimit=25
```

### Multiple link fields
```bash
# Get data from multiple linked columns
GET /api/v3/data/{baseId}/{tableId}/records?nested[Projects][fields]=*&nested[Customers][fields]=name,email
```

For more details, see [NESTED_FIELDS_GUIDE.md](./NESTED_FIELDS_GUIDE.md)
