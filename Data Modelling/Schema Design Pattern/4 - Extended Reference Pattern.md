
Data modelling in MongoDB often requires to deal with multiple documents from multiple collections. In RDBMS handling such situation is taken care by using Joins. The equivalent in MongoDB is the $lookup aggregation operator, but using this operator is resolurce intensive operation much like join in RDBMS.

MongoDB can avoid use of joins all together by Extended Reference Pattern to the data.

Extended Reference is a reference that is rich enough to include all that is needed to avoid Join operation. You can create an extended reference by embedding relevant data for multiple documents and different collections into the main document. It’s primary objectives are to reduce the latency of read operations, avoid round trips to the database, and avoid to touch many pieces of data. The result will be faster reads due to a reduced number of joins and lookups.

### **Example: Blogs and Authors**

Authors Collection: 

```json
{
  "_id": "auth1",
  "name": "Jay",
  "bio": "Tech enthusiast and blogger",
  "email": "jay@example.com"
}
```

Blogs Collection using Extended Reference Pattern:

Instead of just storing the `author_id`, you can include additional details (like the author's name) to reduce query overhead:

```json
{
  "_id": "blog1",
  "title": "Understanding MongoDB Data Models",
  "content": "An in-depth look at MongoDB data modeling strategies.",
  "author_id": "auth1",  // Reference to the author
  "author_name": "Jay",  // Extra data from the 'authors' collection
  "published_date": "2025-03-11"
}
```

Here, the `author_name` field is embedded in the `blogs` collection.

> If you need the author's name frequently along with the blog data, this pattern eliminates the need to look up the authors collection every time.

A query to fetch blog details along with the author's name can now be executed in a single operation:

```json
db.blogs.find({ _id: "blog1" })
```

> Without this pattern, you'd need to first query the `blogs` collection to get `author_id` and then query the `authors`collection to fetch the author's name.

Things needs to be taken in consideration: 

- **Duplication**: If the author's name changes, you’ll need to update it in multiple places.
- **Performance**: Useful when the duplicated fields are infrequently updated but frequently read.