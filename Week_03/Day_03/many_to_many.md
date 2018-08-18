# Many to many associations

## Objectives

* Have an awareness of the different types of relationships that exist in a relational database
* Describe and provide an example of a many to many relationship and when we would use one
* Draw and 'model' many to many relationship

## Recap

Until now we have learned 2 associations:

* one to one; for example a Person and NI number
* one to many; Team has many players; Bank has many accounts...

There is one other relationship we need to learn about.

## Many to many

When modelling data there will be occasions when one or more rows in a table are associated with one or more rows in another table.

For example:

Let's say we are going to build a review site for travellers to show locations that they have visited and include reviews. A location can be visited by many users, and a user can visit many locations. I could visit Paris, London and New York, and Paris could be visited by me, you and someone else.

User:

* name

Location:

* name
* category (e.g. restaurants, hotels, attractions, museums)

When we have a many to many association like this we require a third table in the mix. We call this a join table. A join table will have two foreign keys; one for each model. It will also have its own ID, and can contain extra fields.

So we need a third model. General convention is that if there is no better name for it, we give it the name table1_table2. For our example, 'visits' is probably a good name, but we could call it 'users_locations'.

Visit:

* id
* user_id
* location_id
* review

Here's the twist.. we're going to make this a bit more fun and imagine this review site is for fantasy characters, and it's going to be called QuestAdvisor (get it..).

## Models and our database

Let the students have a read through the code for 5 minutes then talk through the structure.

What's missing?

The SQL to create our visits table. It needs TWO foreign keys, let's write it:

```sql
-- quest_advisor.sql

CREATE TABLE visits (
  id SERIAL4 PRIMARY KEY,
  user_id INT4 REFERENCES users(id) ON DELETE CASCADE,
  location_id INT4 REFERENCES locations(id) ON DELETE CASCADE,
  review TEXT
);

-- at top
DROP TABLE visits;
```

What is this ON DELETE CASCADE doing? It means that if an entry in the linked table which matches the id of this foreign key is deleted, all of its corresponding records are deleted. E.g. if a user is deleted, all of the visits with their user_id are deleted too.

Now let's start by creating our database:

```bash
#terminal

dropdb quest_advisor
createdb quest_advisor
psql -d quest_advisor -f db/quest_advisor.sql
```

## In-depth with the join model

Now that we have nice and DRY code with some nifty mapping methods, let's look at how to improve our join model!

Right now, if you check a visit in pry, you can't really see any useful information about them, only 2 ids. It would be really cool if we could call a method on a join model instance, and check a visit's user and the location itself.

This is similar to what we did yesterday. You can actually think of the join table as having two one-to-many relationships - each visit has one user, but a user can have many visits, and same with locations.

In visit.rb:

```ruby
#visit.rb

def location()
  sql = "SELECT * FROM locations
  WHERE id = $1"
  values = [@location_id]
  location = SqlRunner.run(sql, values).first
  return Location.new(location)
end
```

This will return us a location instance that is associated with our visit.

Let's do the same but this time with a user!

```ruby
#visit.rb

def user()
  sql = "SELECT * FROM users
  WHERE id = $1"
  values = [@user_id]
  user = SqlRunner.run(sql, values).first
  return User.new(user)
end
```

## Mapping our join

Ok, we now have a relational link setup. It would be cool if our models could request data from one another though:

* user1.locations would return all the locations that person has visited
* location1.users would return all users who have visited there

Let's first write the SQL in the postgres terminal and work it out.

First, we need a valid user id.

```ruby
#pry

User.all()
```

Take a note of an id and slot it in here.

```bash
#terminal
psql -d quest_advisor
```

```sql
SELECT locations.*
FROM locations
INNER JOIN visits
ON visits.location_id = locations.id
WHERE user_id = 1
```

Cool now we need to teleport this into our model.

```ruby
# user.rb

  def locations()
    sql = "SELECT locations.*
    FROM locations
    INNER JOIN visits
    ON visits.location_id = locations.id
    WHERE user_id = $1"
    values = [@id]
   	locations = SqlRunner.run(sql, values)
  	result = locations.map { |location| Location.new( location ) }
  	return result
  end
```

Now in the terminal we can see a user's visited locations.

```ruby
#pry

user1.locations
```

> TASK: Add the users() method to Location.

```ruby
# location.rb
  def users()
    sql = "SELECT users.*
    FROM users
    INNER JOIN visits
    ON visits.user_id = users.id
    WHERE location_id = $1"
    values = [@id]
    users = SqlRunner.run(sql, values)
  	result = users.map { |user| User.new( user ) }
  	return result
  end
```

## Refactoring

Before we finish, let's check if there is anything that can get tedious. What about mapping? Every time we want to return more than one entry, we'll have a whole lotta data to map through. Writing that every time can get a bit tiring and not too DRY. So let's make it dry! How about creating a method that'll return us the collection we need! We do not want to pass in SQL, if we do, we can delete the entire database through the console. Let's just pass in what the sqlRunner is returning to us

```ruby
#location.rb

def self.map_items(location_data)
  result = location_data.map { |location| Location.new( location ) }
  return result
end
```

### Task

1.  Create the same method for the User and Visit classes
2.  Use this in our 'all' methods.
