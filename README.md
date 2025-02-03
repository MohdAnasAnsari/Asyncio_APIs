# Asyncio_APIs

✅ Uses `asyncio` and `aiohttp` to fetch data concurrently.  
✅ Implements **error handling** (timeouts, retries, HTTP errors).  
✅ Is structured to be **AWS Lambda-compatible**.  

```python
import asyncio
import aiohttp
import json

# API endpoints to fetch data from
API_ENDPOINTS = [
    "https://api.example.com/data1",
    "https://api.example.com/data2",
    "https://api.example.com/data3",
]

# Max retries and timeout settings
MAX_RETRIES = 3
TIMEOUT = 5  # seconds

async def fetch(session, url, retries=MAX_RETRIES):
    """Async function to fetch data from a single API with error handling."""
    for attempt in range(retries):
        try:
            async with session.get(url, timeout=TIMEOUT) as response:
                if response.status == 200:
                    return await response.json()
                elif response.status in {500, 502, 503, 504}:  # Retry on server errors
                    await asyncio.sleep(2 ** attempt)  # Exponential backoff
                else:
                    return {"error": f"Request failed with status {response.status}"}
        except (aiohttp.ClientError, asyncio.TimeoutError) as e:
            if attempt == retries - 1:
                return {"error": f"Request failed after retries: {str(e)}"}
            await asyncio.sleep(2 ** attempt)  # Retry delay

async def fetch_all():
    """Fetch data concurrently from multiple APIs."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in API_ENDPOINTS]
        results = await asyncio.gather(*tasks)
    return results

def lambda_handler(event, context):
    """AWS Lambda entry point."""
    loop = asyncio.get_event_loop()
    data = loop.run_until_complete(fetch_all())
    return {
        "statusCode": 200,
        "body": json.dumps(data),
    }
```

### **Key Features:**
- **Concurrency:** Uses `asyncio.gather()` for parallel execution.  
- **Error Handling:** Retries on transient failures, returns meaningful errors.  
- **Timeout Handling:** Ensures slow APIs don’t block execution.  
- **AWS Lambda-Ready:** Compatible with `lambda_handler`.  

