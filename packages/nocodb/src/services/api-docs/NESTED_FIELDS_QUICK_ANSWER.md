# Quick Answer: Nested Fields Parameters

## Problem
You want nested fields to contain actual data instead of just counts in NocoDB API responses.

## Solution

### For API v2

Use the `nested[columnName][fields]=*` parameter to get full data from linked records:

```bash
# Example: Get all fields from nested "Projects" column
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=*

# Example: Get specific fields
GET /api/v2/tables/{tableId}/records?nested[Projects][fields]=name,status,budget
```

**Why?** In API v2, nested fields only return primary keys by default. You must explicitly request fields.

### For API v3

No special parameter needed! Nested fields already contain actual data by default.

```bash
# Example: This returns full nested data automatically
GET /api/v3/data/{baseId}/{tableId}/records

# Optional: Control how many nested records per field (default: 1000)
GET /api/v3/data/{baseId}/{tableId}/records?nestedLimit=50
```

**Why?** API v3 returns full nested data automatically. Use `nestedLimit` to control the quantity.

## Parameters Reference

| Parameter | API Version | Purpose | Default |
|-----------|-------------|---------|---------|
| `nested[column][fields]` | v2 | Specify which fields to return from nested data | Primary keys only |
| `nestedLimit` | v3 | Limit number of nested records per field | 1000 |
| `nestedPage` | v3 | Paginate through nested records | 1 |

## Examples

### API v2 - Get all nested data
```bash
GET /api/v2/tables/tbl_employees/records?nested[Department][fields]=*&nested[Projects][fields]=*
```

### API v3 - Get all nested data (default behavior)
```bash
GET /api/v3/data/ws_abc123/tbl_employees/records
```

### API v3 - Limit nested records
```bash
GET /api/v3/data/ws_abc123/tbl_employees/records?nestedLimit=25
```

For more details, see [NESTED_FIELDS_GUIDE.md](./NESTED_FIELDS_GUIDE.md)
