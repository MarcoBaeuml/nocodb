# NocoDB Linked Fields (LTAR) - Current Behavior

## Problem
You want linked/nested fields to contain actual data instead of just counts in NocoDB API responses.

## Current Behavior (v2 and v3)

**Both API v2 and v3 currently return only COUNTS for linked fields in list and read endpoints.**

Example response:
```json
{
  "records": [
    {
      "id": 1,
      "fields": {
        "Title": "post 1",
        "post_interactions": 1  // This is a count, not the actual data
      }
    }
  ]
}
```

## Solution: Use Dedicated Link Endpoints

To get actual linked record data, use the dedicated link endpoints:

### API v3

```bash
# Get linked records for a specific field
GET /api/v3/data/{baseId}/{tableId}/links/{linkFieldId}/{recordId}

# Example:
GET /api/v3/data/pu9a59u6ihj4qe0/mcle87a3eoh7ymi/links/post_interactions_field_id/1
```

This returns the actual linked records with their data.

### API v2

```bash
# Get linked records
GET /api/v2/tables/{tableId}/{linkField}/{rowId}
```

## Why Not nested[columnName][fields]?

The `nested[columnName][fields]` parameter **is not currently functional** in the codebase, despite:
- Being mentioned in swagger documentation
- Having code that appears to support it
- Being documented in API guides

**Evidence:**
- All test cases in `tests/unit/rest/tests/dataApiV3/` expect counts, not nested data
- User testing confirms the parameter has no effect
- No working examples exist in the codebase

## Alternatives

### 1. Make Multiple API Calls

Fetch the main records, then fetch linked data separately:

```bash
# Step 1: Get main records
GET /api/v3/data/{baseId}/{tableId}/records

# Step 2: For each record, get linked data
GET /api/v3/data/{baseId}/{tableId}/links/{linkFieldId}/{recordId}
```

### 2. Use Lookup/Rollup Columns

If you need specific fields from linked records:
1. Create a Lookup column to pull in the field you need
2. The lookup value will be included in the main response

### 3. Future Enhancement

The `nested[columnName][fields]` functionality may be implemented in a future version, but it's not currently working.

## API Comparison

| Feature | API v2 | API v3 | Status |
|---------|--------|--------|--------|
| Linked field in list response | Count only | Count only | Working |
| `nested[col][fields]` parameter | Not functional | Not functional | Not working |
| Dedicated link endpoint | ✅ Available | ✅ Available | Working |
| Lookup columns | ✅ Available | ✅ Available | Working |

## Example Workflow

```bash
# 1. Get posts with interaction counts
GET /api/v3/data/base123/posts/records
# Response: { "records": [{ "id": 1, "fields": { "Title": "Post 1", "interactions": 5 }}]}

# 2. Get actual interaction data for post 1
GET /api/v3/data/base123/posts/links/interactions_field_id/1
# Response: { "records": [{ "id": "int1", "fields": { "type": "like", "user": "John" }}, ...]}
```

## Summary

**To get actual linked data instead of counts:**
- ✅ **Use dedicated link endpoints** (`/links/{fieldId}/{recordId}`)
- ❌ **Don't rely on** `nested[columnName][fields]` parameter (not functional)
- ✅ **Consider** Lookup columns for specific fields
- ✅ **Make** separate API calls for linked data when needed
