# Deep Dive into Eloquent: 40 Rarely Used Eloquent ORM Methods Every Laravel Developer MUST Know

## 1. tap()

**Why**: To apply changes to a model and return the model itself for method chaining.

**When**: Use it when you want to modify an object and immediately use it in another operation.

```
User::find(1)->tap(function ($user) {
    $user->name = 'Updated Name';
})->save();
```

## 2. firstOrFail()

**Why**: To ensure you get a valid result or handle the case where no result is found.

**When**: Use it when you need to fetch a specific record and want to throw an error if it doesn’t exist.

```
$user = User::where('email', 'example@example.com')->firstOrFail();
// Process user details
```

## 3. updateOrCreate()

Why: To avoid duplicate entries by updating an existing record or creating a new one.

When: Use it when you want to ensure a record is created if it doesn’t exist or updated if it does.

```
User::updateOrCreate(
    ['email' => 'example@example.com'],
    ['name' => 'John Doe']
);
```

## 4. increment() / decrement()

I love this one so much. It’s just straightforward and beautiful. When do you use it?
When: Use it to increase or decrease a numerical column by one or more.

Why: To efficiently update a numerical column’s value.

```
User::where('id', 1)->increment('points'); // if points is 7, it will now be 8
User::where('id', 1)->decrement('points', 5); // if 7, it will become 2
```

## 5. withTrashed() / onlyTrashed() / restore()

This group of methods are used to manage Soft Deletes feature in Laravel.
I wrote a very elaborate article about everything in Soft Deletes

Why: To manage soft deleted records.
When: Use these methods to include, only include, or restore soft deleted records.

```
$users = User::withTrashed()->get();
$trashedUsers = User::onlyTrashed()->get();
User::withTrashed()->where('id', 1)->restore();
```

## 6. withoutEvents()

Why: To prevent event listeners from firing.
When: Use it when performing actions that shouldn’t trigger events, such as batch imports.

Suppose you are importing a large number of users from an external system and you don’t want to trigger the UserCreated event for each imported user to avoid sending welcome emails or logging each creation.
```
User::withoutEvents(function () {
    User::create([
      'name' => 'John Doe', 
      'email' => 'john@example.com'
    ]);
    User::create([
      'name' => 'Jane Doe', 
      'email' => 'jane@example.com'
    ]);
});
```

## 7. withoutGlobalScopes()

Why: To bypass global query constraints.
When: Use it to fetch all records, ignoring global scopes like is_published.

Consider a scenario where your application has a Post model with a global scope that only includes posts that are published. An admin might need to see all posts, including drafts and unpublished posts, to manage the content effectively.

Fetching All Posts Ignoring Global Scopes:
```
$allPosts = Post::withoutGlobalScopes()->get();
foreach ($allPosts as $post) {
    echo $post->title . ($post->is_published ? ' (Published)' : ' (Draft)') . "\n";
}
```

In this example, withoutGlobalScopes() allows the admin to view all posts, bypassing the global scope that filters out unpublished posts.

Using withoutGlobalScopes() is particularly useful in administrative tasks where comprehensive access to data is required, or during debugging and testing when you need to ensure that global constraints are not affecting your queries.

## 10. is() / isNot()

I love this one too. Very easy and useful for comparisons and conditions.

Why: To compare two model instances.
When: Use it to check if two models are the same instance.
```
$user1 = User::find(1);
$user2 = User::find(2);

if ($user1->is($user2)) {
    // Same user
}

if ($user1->isNot($user2)) {
    // Not the same user
}
```

## 11. loadMissing()

Example: Suppose you have a User model that has a posts relationship. You want to load the user along with their posts, but you're not sure if the posts relationship has already been loaded.

Why: To conditionally eager load relationships that are not already loaded, optimizing database queries and avoiding the N+1 query problem.

When: Use loadMissing() when you want to load relationships on a model instance but only if they are not already loaded. This is particularly useful when you have conditional relationships that you want to load dynamically based on certain conditions or when loading relationships in a loop where some may have already been loaded.
```
$user = User::find(1);

// Check if the 'posts' relationship is already loaded
if (!$user->relationLoaded('posts')) {
    // Load the 'posts' relationship only if it's not already loaded
    $user->loadMissing('posts');
}

// Now you can access the 'posts' relationship without worrying about duplicate queries
foreach ($user->posts as $post) {
    echo $post->title . "\n";
}
```

## 12. makeHidden() / makeVisible()

Why: To control the visibility of model attributes.
When: Use it to hide or show attributes temporarily, for example, in API responses.

```
$user = User::find(1);
$user->makeHidden('email');
$user->makeVisible('email');
```

## 13. touch()

Why: To update the updated_at timestamp.
When: Use it to mark a record as updated without changing any other attributes.
```
$user = User::find(1);
$user->touch();
```

## 14. append()

Why: To add custom attributes to the model’s array or JSON form.
When: Use it when you want to include additional, computed attributes in the model’s representation.
```
$user = User::find(1);
$user->append('custom_attribute');
```

I have written a very comprehensive and step-by-step tutorial on how to manage Json Data in Laravel.

## 15. replicate()

Why: To duplicate a model instance.
When: Use it to create a new instance with the same attributes, such as cloning a template.
```
$user = User::find(1);
$newUser = $user->replicate(); // $newUser is matches to $user
$newUser->save();
```

## 16. chunkById()

Imagine dealing with a table with 20,000,000 records and you have to take action on each record. How do you do this without….

Why: To process large datasets in chunks efficiently.
When: Use it for processing large datasets to handle memory efficiently, with better performance on large tables.

Suppose you have a database table with 20,000,000 records, and you need to perform a certain action on each record.

```
use App\Models\YourModel;

YourModel::orderBy('id')->chunkById(1000, function ($records) {
    foreach ($records as $record) {
        // Process each record
    }
});
```

NOTE: There is a similar method to this called chunk(). They perform similar jobs, but there are differences.
Both chunk() and chunkById() are used to process large datasets efficiently in batches, preventing memory exhaustion and optimizing performance. They both allow you to iterate over a large dataset without loading the entire dataset into memory at once. However, they differ in how they determine the batches of data:

**chunk():**

> chunk() divides the dataset into chunks based on the number of records per chunk specified as the first parameter.
> It retrieves records from the database table sequentially without considering any specific ordering.
> The records in each chunk are fetched based on the order in which they were retrieved from the database, which may not necessarily be based on the primary key order.
> This method is useful when the order of processing is not critical, or when you simply need to process the data in smaller, more manageable chunks.

**chunkById():**

> chunkById() divides the dataset into chunks based on the primary key (usually id) order of the records.
> It retrieves records from the database table sequentially based on the order of their primary keys.
> Each chunk contains records with primary keys within a specified range, ensuring that the records are processed in the order of their primary keys.
> This method is particularly useful when the order of processing is important, such as when performing data migrations or updates that require sequential processing based on primary key order.

## 17. existsOr()

Why: To execute a callback if the model exists, or return a default value.
When: Use it to handle existence checks with custom logic.

```
$exists = User::where('email', 'example@example.com')->existsOr(function () {
    return 'User does not exist';
});
```

## 18. firstOrCreate()

Why: To retrieve or create a record in one step.
When: Use it to avoid duplicate entries by updating or creating records as needed.
```
$user = User::firstOrCreate(['email' => 'example@example.com'], ['name' => 'John Doe']);
```

## 19. firstOrNew()

Why: To retrieve or instantiate a new record without saving.
When: Use it to get an existing record or create a new instance without persisting it.
```
$user = User::firstOrNew(['email' => 'example@example.com'], ['name' => 'John Doe']);
```

## 20. sole()

Why: To ensure only one record is retrieved or throw an exception.
When: Use it when you expect a single unique result and want to handle duplicates as errors.

[Youtube tutorial](https://youtu.be/FWl1P2nd5mw)
```
$user = User::where('email', 'example@example.com')->sole();
```

## 21. findMany()

Why: To retrieve multiple records by their primary keys. When: Use it to fetch multiple records in one query by an array of IDs.
```
$users = User::findMany([1, 2, 3]);
```

## 22. update()

Why: To update multiple records at once.
When: Use it to perform bulk updates efficiently.
```
User::where('status', 'active')->update(['status' => 'inactive']);
```

## 23. forceDelete()

Why: To permanently delete a soft deleted model.
When: Use it to remove a record completely, bypassing soft deletes.
```
$user = User::withTrashed()->find(1);
$user->forceDelete();
```

## 24. getDirty()

I love this. You can use this know everything that have changed in a model before it is persisted into the database.

Why: To get attributes that have been changed.
When: Use it to check which attributes have been modified before saving.
```
$user = User::find(1);
$user->name = 'New Name';
$dirty = $user->getDirty();
```

## 25. getOriginal()

Why: To get the original values of the model’s attributes.
When: Use it to compare current and original values before changes.
```
$user = User::find(1);
$original = $user->getOriginal('name');
```

## 26. setRelation()

Why: To set a specific relationship on the model.
When: Use it to manually define relationships for a model instance.
```
$user = User::find(1);
$user->setRelation('posts', $posts);
```

## 27. without()

Why: To exclude certain relationships from the query.
When: Use it to optimize queries by excluding unnecessary relationships.
```
$user = User::with('posts', 'comments')->without('comments')->find(1);
```

## 28. preventLazyLoading()

Why: To prevent lazy loading of relationships.
When: Use it to catch unintended lazy loading in development.
```
Model::preventLazyLoading(!app()->isProduction());
```

## 29. withoutTimestamps()

Why: To disable updating of the created_at and updated_at timestamps.
When: Use it for actions that shouldn't trigger timestamp updates, like imports.
```
User::withoutTimestamps(function () {
    User::create(['name' => 'John Doe']);
});
```

## 30. withCasts()

Laravel allows you to apply casting rules dynamically to model attributes. This is useful when you need to change how attributes are cast on the fly, depending on certain conditions or runtime scenarios. For example, you can cast an attribute to a different type based on user input or database values, ensuring data consistency and flexibility in your application.
```
Why: To apply casting rules dynamically.
When: Use it to change how attributes are cast on the fly.

$user = User::withCasts(['is_admin' => 'boolean'])->find(1);
```

## 31. upsert()

Why: To insert or update records based on matching criteria.
When: Use it to avoid duplicate entries by performing bulk inserts or updates.

Suppose you have a users table with an email column as the unique identifier. You want to insert a new user if their email doesn't already exist in the table, or update their name if the email already exists.
```
use App\Models\User;

User::upsert([
    ['email' => 'john@example.com', 'name' => 'John Doe'],
    ['email' => 'jane@example.com', 'name' => 'Jane Doe']
], ['email'], ['name']);
```

## 32. scope

Why: To define reusable query scopes.
When: Use it to apply common query constraints across multiple queries.
```
// In User model
public function scopeActive($query)
{
    return $query->where('status', 'active');
}

// Usage
$activeUsers = User::active()->get();
```

## 33. macro()

I love this method so much. You can use it to create your own custom methods to suit whatever you want to do.

Why: To define custom methods on the Eloquent query builder.
When: Use it to extend the query builder with your own methods.

Suppose you frequently need to filter users based on their role in your application. You can define a custom macro called role() on the query builder to simplify this task.
```
use Illuminate\Database\Eloquent\Builder;

// Define the 'role' macro
Builder::macro('role', function ($role) {
    return $this->where('role', $role);
});

// Usage
$admins = User::role('admin')->get();
$customers = User::role('customer')->get();
```

## 34. filter()

Why: To apply dynamic query filters.
When: Use it to apply multiple filters based on request parameters.
```
// In User model
public function scopeFilter($query, $filters)
{
    return $filters->apply($query);
}

// Usage
$filters = new UserFilters(['status' => 'active']);
$filteredUsers = User::filter($filters)->get();
```

In this example, we define a filter() scope on the User model that accepts a set of filters. These filters can be applied to the query using the apply() method of a UserFilters object. This allows you to dynamically filter users based on different criteria specified in the $filters variable.

By using filter(), you can make your database queries more adaptable to changing requirements and user input, resulting in more flexible and dynamic data retrieval in your Laravel applications.

## 35. whereJsonContains()

Why: To query JSON column for specific values.
When: Use it to query JSON columns containing arrays or objects.
```
$users = User::whereJsonContains('options->languages', 'en')->get();
```

Remember that I have an article for everything about JSON in Laravel. You can see it here

## 36. findOr()

Why: To retrieve the model or execute a callback if not found.
When: Use it to handle the absence of a record with custom logic.
```
$user = User::findOr(1, function () {
    return 'User not found';
});
```

## 37. lockForUpdate()

The lockForUpdate() method in Laravel's Eloquent ORM is used to lock database rows for update within a transaction. When you apply this method to a query, it prevents other database transactions from modifying the selected rows until the current transaction is completed. This ensures data consistency and prevents conflicts when multiple transactions try to update the same rows simultaneously.

Why: To apply a “for update” lock to the query.
When: Use it to prevent other transactions from modifying the rows during your transaction.
```
$user = User::where('email', 'example@example.com')->lockForUpdate()->first();
```

## 38. sharedLock()

Why: To apply a “shared lock” to the query.
When: Use it to lock the selected rows for the duration of the transaction.

Suppose you have a financial application where users can view their account balances. You want to ensure that when a user checks their balance, the displayed amount remains consistent even if other transactions are updating the account balance concurrently. You can use sharedLock() to lock the rows corresponding to the user's account during the transaction.
```
use App\Models\Account;

DB::transaction(function () use ($userId) {
    $account = Account::where('user_id', $userId)->sharedLock()->first();
    // Display the user's account balance
});
```

## 39. withSum()

Why: To add a sum of a related model’s attribute to the result.
When: Use it to aggregate data from related models, such as summing order totals.
```
$users = User::withSum('posts', 'views')->get(); // total posts
```

Suppose you have a User model and each user can have multiple orders. You want to retrieve a list of users along with the total sum of their order amounts. You can use withSum() to achieve this.
```
use App\Models\User;

$usersWithTotalOrderAmount = User::withSum('orders', 'amount')->get();

foreach ($usersWithTotalOrderAmount as $user) {
    echo "User: {$user->name}, Total Order Amount: {$user->orders_sum_amount}\n";
}
```

In this example, withSum('orders', 'amount') is used to retrieve the total sum of the amount column from the orders relationship for each user. The aggregated sum is made available as a dynamically generated attribute (orders_sum_amount) on each user object.

By using withSum(), you can efficiently retrieve aggregated data from related models alongside the main query result, simplifying your code and improving performance.

## 40. withCount()

The withCount() method in Laravel's Eloquent ORM is used to retrieve related models along with the count of the related models. This is useful when you want to retrieve the count of related records without needing to perform additional queries or manual counting.

Why: To count the number of related models.
When: Use it to get the count of related records, like the number of posts per user.
```
use App\Models\User;

$usersWithPostCounts = User::withCount('posts')->get();

foreach ($usersWithPostCounts as $user) {
    echo "User: {$user->name}, Post Count: {$user->posts_count}\n";
}
```

In this example, withCount('posts') is used to retrieve the count of posts associated with each user. The count of posts is made available as a dynamically generated attribute (posts_count) on each user object.

By using withCount(), you can efficiently retrieve the count of related records from the database alongside the main query result, simplifying your code and improving performance.

