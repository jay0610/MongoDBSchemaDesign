
Often it’s required to store the result of computations in the collection for operation.

Example: Whenever a document is inserted into a book collection, average of the book’s rating field needs to be calculated. There computation is required to change the value of the field reviews; computed pattern design is required in such scenarios. Computed patterns, run operations when data changes and stores results for quick access. Commonly used computed patterns are mathematical and roll-up. For mathematical operations, the goal is to pre-compute result when data in written instead of running the calculations every time. In case of BookStore application, everytime user gives a review, even if no review is added application does the average review computation, causing billions of reviews computed.

Let’s see example of roll-up operation - We want to display summary document of the each book and summary includes the number of books and average number of authors for each product type. The `$sum` operator increments the count of books for each type. The `$size` operator gets the number of authors in the `authors` array which is then averaged for the type using the `$avg` operator.

```javascript
var roll_up_product_type_and_number_of_authors_pipeline = [
  {
    $group: {
      _id: "$product_type",
      count: {
        $sum: 1,
      },
      averageNumberOfAuthors: {
        $avg: {
          $size: "$authors",
        },
      },
    },
  },
]
```

We run the aggregation by using the aggregate method on the books collection, passing in the roll up pipeline as an argument.

```javascript
db.books.aggregate(roll_up_product_type_and_number_of_authors_pipeline)
```

The resulting documents look like this:

```javascript
[
  { _id: 'audiobook', count: 1, averageNumberOfAuthors: 1 },
  { _id: 'ebook', count: 1, averageNumberOfAuthors: 3 },
  { _id: 'book', count: 1, averageNumberOfAuthors: 3 }
]
```

To summarize the computed pattern, pre-compute data before the read operations. This lets you run operarions when the data changes and store reaults in document for quick access.