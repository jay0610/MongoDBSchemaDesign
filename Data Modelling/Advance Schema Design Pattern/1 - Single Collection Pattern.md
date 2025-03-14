
When you are accessing the database let’s say for Online Book store application, many entitied will be queried frequently, like reviews, user information and releted books. To optimize overall performance, we want to use as few queries as possible and avoid $lookup operations. Embedding these entities in a single document is not an option since will either result in unbounded array or exceeding the 16MB document size limit. However, storing these entities in different collections can impact performance, since we’ll need to use expensive lookup operations or multiple queries to retrieve the data. So in such case Single Collection Pattern is a solution to consider.

The **Single Collection Pattern** in MongoDB is a schema design pattern where all related data is stored in a single collection rather than being split across multiple collections. This approach is beneficial when you want to simplify queries and reduce the number of joins or lookups, which can improve read performance.

### Example: E-commerce Order System

Imagine you have an e-commerce system where users place orders, and you need to store both user information and the details of their orders. Instead of creating separate collections for `users` and `orders`, you can use a single collection to store all this information.

#### Document Structure:

Here’s how the documents in the single collection might look:

```json
 {
    "user_id": "U12345",
    "name": "Jay Rathod",
    "email": "jay@example.com",
    "orders": [
        {
            "order_id": "O98765",
            "order_date": "2025-03-11",
            "items": [
                { "item_id": "I54321", "name": "Smartphone", "quantity": 1, "price": 699.99 },
                { "item_id": "I67890", "name": "Wireless Charger", "quantity": 2, "price": 19.99 }
            ]
        },
        {
            "order_id": "O56789",
            "order_date": "2025-02-25",
            "items": [
                { "item_id": "I12345", "name": "Headphones", "quantity": 1, "price": 49.99 }
            ]
        }
    ]
}
```

#### Benefits:

1. **Simplified Queries**: To fetch all orders for a user, you only need a single query.

```javascript
db.orders.find({ user_id: "U12345" });
```

2. **Performance**: No need for `$lookup` or joins, which saves time.
3. **Atomicity**: Updates to user details and orders happen in a single operation, ensuring consistency.

#### Considerations:

- **Document Size**: MongoDB documents have a maximum size limit (16 MB). If there’s too much data, the document can become too large to store.

- **Query Efficiency**: While retrieval is simpler, updates to deeply nested data (e.g., modifying an item in an order) can become complex.

The Single Collection Pattern is ideal for scenarios where related data is always accessed together. For example, blog posts with comments or product reviews with ratings are also good candidates.