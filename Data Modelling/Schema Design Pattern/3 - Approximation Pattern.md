
In case of population of the earth, one cannot provide the exact number as this number keeps on changing every second, so the cost spent in getting the perfect number is much more than the it’s worth - There comes the idea of approximation pattern schema design pattern. This pattern generated the stastically valid approximate number that is not exact.

The Approximation Pattern in MongoDB is a schema design approach used when precise values aren't necessary, and approximate data is sufficient. This pattern is particularly useful for reducing the frequency of database writes and improving performance in scenarios where exact numbers aren't critical.

Circumstances under which Approximation Pattern is used :

- Data is difficult or expensive to calculate.
- Getting exact number is not critical.
- Working with big data.

**Approximation Pattern logic is implemented on Application Side.**

### Example: City Population Tracking

Imagine you're tracking the population of a city with approximately 40,000 residents. The population changes frequently due to births, deaths, and migrations. Instead of updating the database every time there's a change, you can use the Approximation Pattern to update the population in increments of 100.

#### Implementation:

1. **Initial Data**: Start with a document in your collection

```json
{
    "city": "New Perth",
    "population": 40000,
    "date": "2022-09-15"
}
```

2. **Update Logic**: Instead of updating the population for every change, use application logic to update the database only when the population changes by 100.

   For example:

   ```javascript
   let population = 40000;
   
   function updateStoredPopulation(curr_population, new_population) {
       let population_change = Math.abs(curr_population - new_population);
       if (population_change >= 100) {
           db.population.insertOne({
               city: "New Perth",
               population: new_population,
               date: new Date()
           });
           population = new_population;
       }
   }
   ```

3. **Resulting Data**: Over time, your collection might look like this:

   ```json
   [
       { "city": "New Perth", "population": 40100, "date": "2024-09-20" },
       { "city": "New Perth", "population": 40200, "date": "2024-10-01" },
       { "city": "New Perth", "population": 40300, "date": "2024-10-09" }
   ]
   ```

### Benefits:

- **Performance**: Reduces the number of write operations, saving resources.

- **Scalability**: Handles large datasets or high-frequency changes efficiently.

- **Simplicity**: Provides a high-level trend without the need for exact precision.

This pattern is also applicable to scenarios like website visits or airline passenger counts, where approximate values are sufficient for decision-making.