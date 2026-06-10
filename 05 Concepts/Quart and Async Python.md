---
tags: [concept, backend]
---

# Quart and Async Python

**Quart** is an async re-implementation of Flask: same decorators and patterns, but every handler is `async def` and runs on Python's `asyncio` event loop.

## Why async here?
The backend spends most of its time **waiting on I/O** — Canvas API, OpenAI, YouTube, MongoDB. Async lets one process serve other requests while waiting, and lets the [[Flow - Background Canvas Sync|sync loop]] run *inside* the web process as a background task.

## Patterns you'll see in this repo
```python
# A route (Flask-style, but async)
@skill_bp.route('/achieveup/skills/assign', methods=['POST'])
async def assign_skills():
    data = await request.get_json()        # note the await
    result = await skill_service.assign(...)
    return jsonify(result), 200

# Blueprints — group routes, registered in app.py
skill_bp = Blueprint('skill', __name__)
app.register_blueprint(skill_bp)

# Lifecycle hooks
@app.before_serving        # runs once at startup → starts the sync loop
@app.before_request        # runs before every request → logging
@app.after_request         # runs after every request → CORS headers

# Background task that outlives any request
asyncio.create_task(schedule_updates())

# Async Mongo iteration (Motor driver)
async for token in token_collection.find():
    ...
await collection.create_index("course_id")
```

## Rules of thumb
- Inside `async def`, any I/O call must be `await`ed; forgetting `await` gives you a coroutine object instead of a result (a classic bug).
- **Never block** the event loop: `time.sleep()` freezes the whole server; use `await asyncio.sleep()` (the sync loop does this for rate limiting).
- Two route-registration styles coexist here: legacy `init_*_routes(app)` functions vs AchieveUp Blueprints — see [[Backend Overview]].

## Serving
- Local: `python app.py` → Quart dev server on :5001.
- Prod: `Procfile` runs **hypercorn** (an ASGI server — async equivalent of gunicorn).

Related: [[MongoDB Notes]] (Motor is the async driver) · [[Backend File Guide]]
