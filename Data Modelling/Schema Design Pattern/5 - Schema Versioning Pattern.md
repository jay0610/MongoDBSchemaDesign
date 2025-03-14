
Applications evolves over time, as application adds up new functionalities or new advancements or patches etc. This change also changes data structure in the database. It is stressful to change structure of the data.

This can be handled using MongoDB’s schema versioning pattern, where documents with different size and shape exists in same collection. Since these documents coexist we have flexibility on how and when we implement schema updates, which helps avoid downtime.

Example Scenario: User Profiles

Initial Schema (Version 1)

```json
{
  "_id": "user1",
  "name": "Jay",
  "email": "jay@example.com",
  "address": "123 Tech Lane"
}
```

Updated Schema (Version 2)

You decide to split the `address` field into `city` and `street`, and add a `phone_number` field. Using the Schema Versioning Pattern:

```json
{
  "_id": "user2",
  "name": "Alex",
  "email": "alex@example.com",
  "city": "Mumbai",
  "street": "456 Innovation Road",
  "phone_number": "+91-9876543210",
  "schema_version": 2
}
```

#### Handling Versioning in the Application

You can now manage the logic for different schema versions in your application:

- For documents with `schema_version: 1`, treat `address` as a single field.

- For documents with `schema_version: 2`, use the `city` and `street` fields and also show the `phone_number`.

#### Query Example

To update old documents to the new schema, you could write a migration script:

```javascript
db.users.find({ schema_version: { $exists: false } }).forEach(user => {
    const [street, city] = user.address.split(", ");
    user.city = city;
    user.street = street;
    user.phone_number = null;  // Initialize phone_number for older records
    user.schema_version = 2;
    db.users.save(user);
});
```

How the existing documents are updated ?

1. Let the application update documents from version 1 to version 2. Application must be updated before documents updation takes place.
2. Have a background task which will perform the updates.

Things needs to be taken into consideration:

- **Backward Compatibility**: This ensures older versions of documents can still be processed by the application.

- **Migration Costs**: Over time, as more versions are added, migrating old documents may become expensive.

- **Code Complexity**: The application needs logic to handle multiple schema versions.