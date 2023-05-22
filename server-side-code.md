# Express, Node, and MongoDb -- Building REST API

## Handling the HTTP requests (CRUD)

(REST API routes to perform Create, Read, Update, and Delete (CRUD) operations)

### READ ROUTE - GET DATA FROM DB

---

The Read route will be returning 50 of the articles when there is a get request on the /posts route. Here, we start by defining our collection object, then do a .find operation. We limit the results and convert the response data to an array. We can then send data to the client using res.send. We can also specify the status code with the status method. Code 200 stands for "OK, " meaning that the operation was successful.

```
// Get a list of 50 posts
app.get("/", async (req, res) => {
  let collection = await db.collection("posts"); //check!
  let results = await collection.find({})
    .limit(50)
    .toArray();

  res.send(results).status(200);
});
```

### READ ROUTE (ADVANCED) - GET

---

(more complex route using aggregation pipelines)
Add a route that will return the three most recent articles in the collection. The endpoint will catch all the get requests to /post/latest. We then use an aggregation pipeline to sort the collection in descending order of date and limit the results to three. This leverages the database to do the heavy lifting with our data.

```
// Fetches the latest posts
app.get("/latest", async (req, res) => {
  let collection = await db.collection("posts"); //check!
  let results = await collection.aggregate([
    {"$project": {"author": 1, "title": 1, "tags": 1, "date": 1}},
    {"$sort": {"date": -1}},
    {"$limit": 3}
  ]).toArray();
  res.send(results).status(200);
});
```

### READ SINGLE RESULT - GET

---

use parametrized routes to return filtered results or, in this case, a single object. Notice in the following code snippet that the route takes an :id parameter.

```
// Get a single post
app.get("/:id", async (req, res) => {
  let collection = await db.collection("posts"); //check!
  let query = {_id: ObjectId(req.params.id)};
  let result = await collection.findOne(query);

  if (!result) res.send("Not found").status(404);
  else res.send(result).status(200);
});

```

### CREATE ROUTE - POST

---

Based on the REST conventions, adding new items should be done with a POST method.

```
// Add a new document to the collection
app.post("/", async (req, res) => {
  let collection = await db.collection("posts"); //check!
  let newDocument = req.body;
  newDocument.date = new Date();
  let result = await collection.insertOne(newDocument);
  res.send(result).status(204);
});
```

### UPDATE ROUTE - PATCH

---

Best practices in REST API design state that we should use a PATCH request for updates. Sometimes, a PUT request might also be used.

```
// Update the post with a new comment
app.patch("/comment/:id", async (req, res) => {
  const query = { _id: ObjectId(req.params.id) };
  const updates = {
    $push: { comments: req.body }
  };

  let collection = await db.collection("posts"); //check!
  let result = await collection.updateOne(query, updates);

  res.send(result).status(200);
});
```

### DELETE ROUTE - DELETE

---

```
// Delete an entry
app.delete("/:id", async (req, res) => {
  const query = { _id: ObjectId(req.params.id) }; //check!

  const collection = db.collection("posts");
  let result = await collection.deleteOne(query);

  res.send(result).status(200);
});
```

### SEARCH FROM MONGODB WITH SORTING --

---

> URL: "{page-link}/page?search={search-text}&sort={ascending/descending}"

```
app.get('/page', async(req, res) => {
  const sort = req.query.sort;
  const search = req.query.search;

  const query = {
    title: {
      $regex: search, // FOR SEARCH DATA
      $options: 'i, // FOR CASE INSENSITIVE
    }
  };

  const options = {
    sort: {
      "price": sort === 'ascending' ? 1 : -1,  // SORTING DATA BY PRICE
    }
  };

  const cursor = pageCollection.find(query, options);
  const result = await cursor.toArray();

  res.send(result);
});

```
