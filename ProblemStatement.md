# Django: Paginated Catalog Caching

In this challenge, your task is to implement a simple caching layer for a paginated catalog API. The goal is to provide fast read performance while ensuring that data stays consistent when items are added, updated, or deleted.

The application exposes a list of items that should be returned in pages, ordered by `updated_at` in descending order. To improve performance, each paginated response must be cached. When data changes, affected pages must be invalidated so that fresh data is returned.

Your implementation must handle the following requirements:

1. **Pagination Response**
   The endpoint `GET /api/items` accepts `page` and `page_size` query parameters.  
   - Default: `page=1`, `page_size=20`  
   - Maximum allowed `page_size=100`  
   The response must include:  
   - `page`  
   - `page_size`  
   - `total` number of items  
   - `items` for that page  

2. **Caching**
   Each page (page + page_size combination) should be cached.  
   - Cache TTL: **2 minutes**  
   - Cached pages must be returned instantly when available.  
   - Cache keys must uniquely represent page and page_size.

3. **ETag & 304 Support**
   - When returning a page, include an ETag in the response.  
   - If the client sends `If-None-Match` with a matching ETag, return **304 Not Modified** with no body.

4. **Cache Invalidation**
   When an item is created, updated, or deleted:
   - Any cached pages that may include that item must be invalidated.  
   - After invalidation, the next page request should generate a fresh cache entry.

5. **Preload Next Page (Optional)**
   If the client sends the header:  
   `X-Preload-Next: true`  
   then the next page (if it exists) should be preloaded into cache.

6. **Validation**
   - `title`: Required, non-empty  
   - `price`: Must be positive  
   - Pagination params must be valid integers  

7. **Memory Management**
   Cached entries must not grow without limit. Old or expired pages should be cleaned up.

Do not modify the `Item` model or the provided controller. Your code should be placed in the service layer as indicated in the project structure.
