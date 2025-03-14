
The **Inheritance Design Pattern** in MongoDB schema design is inspired by **Polymorphism**, where different forms of similar or shared entities coexist within the same collection.

**Parent Entity:** Contains shared fields that are common across all subtypes (or child entities).

**Child Entity:** Extends the parent entity with unique fields specific to the subtype.

### **Use Case:**

This pattern is ideal when there are **more similarities than differences** in the schema structure.

Example: Online Books Store, a product collection for books. In this collection, there are three types of books.

- Printed Books
- E-Books
- Audio Books

Here, Books act as the parent entity, and the specific book types are the children. All three share common fields like -

- title
- author
- publisher
- Product_type ( values: “printed“, “ebook“, “audiobook“ )

The value of **product_type** determines the shape of the document, as each subtype can have unique fields:

- For Printed Books: number_of_pages (unique to printed books)
- For E-Books: media_format (e.g., PDF, EPUB)
- For E-Books: download_url (unique to e-books)
- For Audio Books: audio_length (e.g., in hours or minutes)

Above example of Online Book Store for books can be visually represented as below -

![alt text](https://github.com/jay0610/MongoDBSchemaDesign/blob/main/.attachments/Books-Collection.png)

The idea of **Data that is accessed together should be stored together** aligns well with its document-based architecture. Storing different types of books (printed, audio and ebooks) in the same collection can indeed leverage polymorphism effectively.

Above visual can be represented as below in collection as follows -

**audiobook**

```javascript
{
 "title": "Practical MongoDB Aggregations",
 "author": "Paul Done",
 ... 
 "genres": ["Programming"],
 "price": 0.00,
 "publisher": "MongoDB Press",
 ...
 "product_type": "audiobook",
 "download_url": "https://dl.mdb.com/dune.m4a",
 "narrators": ["Paul Done"],
 "duration": "21 hours 8 minutes",
 "time_by_chapter": [{
 "chapter": 1, "start": "00:00:00",
 "end": "01:00:00" 
 ｝，
 ...
 ]
}
```

**printed book**

```javascript
{
  ...
  "title": "Practical MongoDB Aggregations",
  "author": "Paul Done",
  "authorId": "author0034",
  "rating": 4.8,
  "genres": ["Programming"],
  "pages": 338,
  "price": 0.00,
  "publisher": "MongoDB Press",
  ...
  "product_type": "printed_book",
  "stockLevel": 132,
  "deliveryTime": "2",
  "number_of_pages": 438,
}
```

**ebook**

```javascript
{
  ...
  "title": "Practical MongoDB Aggregations",
  "author": "Paul Done"
  "authorId": "author0034",
  "rating": 4.8,
  "genres": ["Programming"],
  "pages": 338,
  "price": 0.00,
  "publisher": "MongoDB Press",
  ...
  "product_type": "ebook",
  "download_url": "https://mdb.com/dune.ebpub"
 }
```

Observe the product_type unique values are changing accroding to the value of product_type. When product_type is “ebook”, the “download_url” is required, while product_type is “printed_book” then “stock level” or “deliveryTime” or “number_of_pages” these properties are activated and so on.

To apply the Inheritance Pattern to documents in a collection, you can use the aggregation framework.

In above mentioned book types, we can use two pipelines to add product type and product ID to all the documents in books collection, by making some shared fields consistent.

```javascript
var apply_inheritance_pattern_to_books_pipeline = [
  {
    $project: {
      _id: "$_id",
      product_id: "$product_id",
      product_type: {
        $ifNull: ["$product_type", "Unspecified"],
      },
      description: {
        $ifNull: [
          "$desc",
          "$description",
          "$details",
          "Unspecified",
        ],
      },
      authors: {
        $ifNull: [
          "$authors",
          ["$author"],
          "Unspecified",
        ],
      },
      publisher: "$publisher",
      language: "$language",
      pages: "$pages",
      catalogues: "$catalogues",
      eformats: "$eformats",
      isbn10: "$isbn10",
      isbn13: "$isbn13",
      narrator: "$narrator",
      length_minutes: "$length_minutes",
    },
  },
  {
    $merge: {
      into: "books",
      on: "_id",
      whenMatched: "replace",
      whenNotMatched: "discard",
    },
  },
]

db.books.aggregate(apply_inheritance_pattern_to_books_pipeline) 
```

Using Pipeline in audiobook example:

```javascript
var cleanup_audiobook_entries_in_book_pipeline = [
{
  $match: {
  $and: [ { product_type: "Unspecified" },  { length_minutes: { $gte: 0 }  }],
  },
  {
    $set: { product_type: "audiobook" },
  },
    $merge: {
    into: "books", 
    on: " _id",
    whenMatched: "replace", 
    whenNotMatched: "discard",
   },
 },
] ;
```

If product_type is unspecified and length_minutes is greater than or equal to zero then set the product_type to “audiobook“ and add it into the books collection then save the changes.

Let’s go for printed books example:

```javascript
var cleanup_printedbook_entries_in_book_pipeline = [
 {
   $match: {
     $and: [{ product_type : "Unspecified"},{number_of_pages: {$gte : 0}, {stock_level: {$gte: 0}} }],
   },
   {
    $set: {product_type: “printed_book"}, 
   },
 { 
   $merge: {
    into: “books",
    on: "_id",
    whenMatched: "replace",
    whenNotMatched: "discard",
   },
 },
];      
```

and so on.
