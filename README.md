# MongoDB-Operations-and-Aggregations

This project demonstrates various MongoDB operations, including importing data, running queries, and performing advanced aggregations. It is based on tasks outlined in the Homework for COMP306 Database Management Systems.

## Features
- Importing JSON files into a MongoDB collection.
- Running find queries with filters, sorting, and limiting results.
- Using aggregation pipelines for advanced data analysis.
- Setting up schema validation for a collection.
- Exporting data to JSON files.

## Instructions

### 1. Importing Data
The dataset `zips.json` is imported into a MongoDB database using the following command:
```bash
mongoimport --db hw4 --collection zipcodes --file zips.json
```

### 2. Example Queries
#### Query 1: Find Documents in California
Retrieve all documents in California where the population exceeds 50,000 and latitude is greater than 35. Results are sorted by population in descending order:
```javascript
db.zipcodes.find({
  state: "CA",
  pop: { $gt: 50000 },
  "loc.1": { $gt: 35 }
}).sort({ pop: -1 }).limit(5).pretty();
```

#### Query 2: Exclude California and Apply Conditions
Find documents where the state is not California, and either longitude is less than -120 or latitude is less than 40:
```javascript
db.zipcodes.find({
  state: { $ne: "CA" },
  $or: [
    { "loc.0": { $lt: -120 } },
    { "loc.1": { $lt: 40 } }
  ]
}).limit(5).pretty();
```

#### Query 3: Aggregation to Find Most Populated Cities
Find the top 5 most populated cities with their populations:
```javascript
db.zipcodes.aggregate([
  { $group: { _id: "$city", totalPopulation: { $sum: "$pop" } } },
  { $sort: { totalPopulation: -1 } },
  { $limit: 5 }
]);
```

### 3. Schema Validation for Customers Collection
A schema validator is set up for the `customers` collection to ensure data consistency:
```javascript
db.createCollection("customers", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "zipcode", "avg_rating"],
      properties: {
        name: { bsonType: "string" },
        zipcode: { bsonType: "string" },
        avg_rating: { bsonType: "double", minimum: 0.0, maximum: 10.0 },
        last_order: {
          bsonType: "object",
          required: ["year"],
          properties: {
            year: { bsonType: "int" },
            tags: { bsonType: "array", items: { bsonType: "string" } }
          }
        }
      }
    }
  }
});
```

### 4. Data Insertion
Example documents inserted into the `customers` collection:
```javascript
db.customers.insertMany([
  { name: "Sarp Cagan Kelleci", zipcode: "99503", avg_rating: 8.3 },
  { name: "Andrei T.", zipcode: "90025", avg_rating: 3.5, last_order: { year: 2009 } },
  { name: "Bela T.", zipcode: "33126", avg_rating: 4.9, last_order: { year: 2019, tags: ["art", "melancholy"] } },
  { name: "Nuri Bilge C.", zipcode: "90010", avg_rating: 6.5, last_order: { year: 3005 } }
]);
```

### 5. Data Export
The `customers` collection is exported to a JSON file using the following command:
```bash
mongoexport --db hw4 --collection customers --out ~/customers.json --jsonArray
```

### 6. Additional Tasks
- **Deletion**: Remove documents where `last_order.year` is greater than 2025:
  ```javascript
  db.customers.deleteMany({ "last_order.year": { $gt: 2025 } });
  ```
- **Aggregation with `$lookup`**: Join `customers` with `zipcodes` to display additional details:
  ```javascript
  db.customers.aggregate([
    { $lookup: { from: "zipcodes", localField: "zipcode", foreignField: "_id", as: "zip_details" } },
    { $unwind: "$zip_details" },
    { $project: { name: 1, "zip_details.city": 1, "zip_details.state": 1, zipcode: 1 } },
    { $sort: { zipcode: 1 } }
  ]);
  ```
