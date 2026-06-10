---
tags: [concept, backend, database]
---

# MongoDB Notes

Document database: instead of tables/rows, you store **JSON-like documents** in **collections**. No fixed schema — every document can have different fields (which is why the frontend's `types/index.ts` is the closest thing this project has to a schema).

## SQL → Mongo translation
| SQL | MongoDB |
|---|---|
| table | collection |
| row | document (BSON/JSON) |
| column | field |
| `WHERE x = 1` | `find({"x": 1})` |
| `JOIN` | usually avoided — embed data or do two queries |
| auto-increment id | `_id` (ObjectId) |

## How this project uses it
- **Hosted on MongoDB Atlas** (cloud). Connection string in `DB_CONNECTION_STRING` env var.
- **Async driver: Motor** (`motor.motor_asyncio.AsyncIOMotorClient`) because the backend is [[Quart and Async Python|async]].
- DB selected by environment: `KnowGap` (prod) vs `KnowGap_Dev` (`ENVIRONMENT=development`).
- All collection names are constants in `config.py` — full list and meanings in [[Database Collections]].

```python
# typical patterns from the services
doc = await collection.find_one({"course_id": course_id})
await collection.update_one({"_id": x}, {"$set": {"video": v}}, upsert=True)  # upsert = insert if missing
async for user in users.find({"role": "instructor"}):
    ...
await quizzes_collection.create_index("course_id")   # indexes speed up frequent queries (app.py)
```

## Gotchas for the team
- **Upserts everywhere**: much of the sync logic uses `upsert=True`, so re-running syncs is idempotent-ish.
- **Schemaless cuts both ways**: a typo'd field name fails silently — you just get documents with two spellings. Grep before renaming anything.
- Use **MongoDB Compass** (GUI) with the Atlas connection string to browse real data — fastest way to learn the actual document shapes.

Related: [[Database Collections]] · [[Flow - Background Canvas Sync]]
