schema for the document DB collections

Collection: audit_event

```json
{
  "_id": "xxxxx",
  "timestamp": "ISODate('2025-11-15T21:46:53Z')",
  "event": "LINK_CREATED",
  "user": {
    "id": 123, // ID of event *owner* from SQL "user" table
    "is_verified": true
  },
  "target": {
    "type": "shortlink", // type will determine body
    "id": 456,
    "body": {
      // any structure, for link creation could be below but could also be any other shape
      "destination": "xxxxx",
      "short": "xxxx"
    }
  },
  "detail": "User created a new link for https://..."
}
```

Collection: link_visit

```json
{
  "_id": "ObjectId('...another_unique_id...')",
  "timestamp": "ISODate('2025-11-15T21:50:00Z')",
  "link_id": 456, // ID from SQL "shortlink" table
  "user_id": 123, // ID of *owner* from SQL "user" table
  "ip_hash": "a_hashed_ip_address_for_privacy",
  "user_agent": "Mozilla/5.0 (Windows NT 10.0; ...)",
  "referrer": "https://t.co/",
  "location": {
    "country": "GB",
    "city": "London"
  }
}
```
