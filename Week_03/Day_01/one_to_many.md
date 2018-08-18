## Modeling a one to many relationship

Let's create a table to store the lightsabers of our characters.

Lightsabers will have

* hilt_metal
* colour

We will worry about the owner later

> Ask the students to determine what type these columns will be.

Let's add a new table at the top of our file and comment out the queries we wrote earlier.

```sql
-- star_wars.sql
DROP TABLE lightsabers; -- Above DROP TABLE characters
-- Just below CREATE TABLE characters
CREATE TABLE lightsabers (
  id SERIAL8,
  hilt_metal VARCHAR(255),
  colour VARCHAR(255)
);
```

```zsh
# terminal
psql -d star_wars -f star_wars.sql;
```

Aside: You will often see 255 used because it's the largest number of characters that can be counted with an 8-bit number. It maximizes the use of the 8-bit count, without frivolously requiring another whole byte to count the characters above 255. We'll come back to binary another time.

```sql
-- star_wars.sql
INSERT INTO lightsabers (colour, hilt_metal) VALUES ('green', 'palladium');
INSERT INTO lightsabers (colour, hilt_metal) VALUES ('red', 'gold');

SELECT * FROM lightsabers;
```

```zsh
# terminal
psql -d star_wars -f star_wars.sql
```

```sql
# psql terminal
SELECT * FROM lightsabers;
```

[Task:] Add a lightsaber

# Constraints

We can add "constraints" to our table definition, which will validate the data we try to enter against some basic rules.

* A lightsaber must have a colour and a hilt metal

```sql
-- star_wars.sql
CREATE TABLE lightsabers (
  id SERIAL8,
  hilt_metal VARCHAR(255) NOT NULL,
  colour VARCHAR(255) NOT NULL
);
```

```zsh
# terminal
psql -d star_wars -f star_wars.sql
```

Let's try to insert some invalid data.

```sql
# star_wars.sql
INSERT INTO lightsabers (colour) VALUES ('red');
```

```zsh
# terminal
psql -d star_wars -f star_wars.sql
```

## Primary Keys

We already discussed associating lightsabers and characters by adding the owner's name to the lightsaber table. We came to the conclusion that using the owner's ID is the better solution.

If we want to use an ID, it's important that we make sure that every row has an ID. Currently, we could set the ID field of our lightsabers to be null or a duplicate value.

```sql
-- jedi.sql
UPDATE lightsabers SET ID = 1;
```

> Ask if anybody can remember what we mentioned earlier that can solve this

The way we can make sure that his will never happen is to ask SQL to set our ID column to be the table's PRIMARY KEY.

A primary key is a column that uniquely defines a record. A primary key column cannot contain a NULL value. A table can have only one primary key. So we are explicitly saying that we want our ID field to be our main identifier for the rows in the table.

```sql
-- star_wars.sql
CREATE TABLE lightsabers (
  id SERIAL8 PRIMARY KEY, -- UPDATED
  colour VARCHAR(255) NOT NULL,
  hilt_metal VARCHAR(255) NOT NULL
);

CREATE TABLE characters (
  id SERIAL8 PRIMARY KEY, -- UPDATED
  name VARCHAR(255),
  darkside BOOLEAN,
  age INT
);
```

```zsh
# terminal
psql -d star_wars -f star_wars.sql
```

Now we can't alter it like we just did.

```sql
UPDATE lightsabers SET ID = 1;
```

## Foreign Keys

The last thing we want to do is to reflect the relationship between our characters and our lightsaber!

We can now use this primary key as an identifier in another table. When we do this we refer to it as a 'foreign key'. It's simply a primary key from another table.

> draw this on the board (one to many)

```sql
-- star_wars.sql
CREATE TABLE lightsabers (
  id SERIAL8 PRIMARY KEY,
  colour VARCHAR(255) NOT NULL,
  character_id INT8 REFERENCES characters(id), -- UPDATED
  hilt_metal VARCHAR(255) NOT NULL
);
```

We can see that the characters table now has a serial id and the lightsabers table now has a "references character(id)" statement. Our character_id is a reference to the primary key in the characters table.

Foreign keys are generally named according to the convention "table_name_singular_id", unless another name makes more 'sense' (but it would always have `_id` to indicate it's a foreign key).

Now, before we do anything else - what happens if we change the order of the drops and run this again? Because lightsabers now depends on characters, if we want to delete the character table we must remove any tables that depend on it's primary keys.

Otherwise we'd end up with a whole bunch of zombie references to it. Let's fix that up and put it back in the correct sequence.

If we inspect our newly created rows, we can see the ids of the characters. Let's use these to modify the creation of the lightsabers.

```sql
-- star_wars.sql
SELECT * FROM characters; --find the ids - depending on who got deleted 1 should be gone...

-- Now update the 2 previous lightsabers, by adding the character_id to them

INSERT INTO lightsabers (colour, character_id, hilt_metal) VALUES ('green', 2, 'palladium');

INSERT INTO lightsabers (colour, character_id, hilt_metal) VALUES ('red', 3, 'gold');
```

```zsh
# terminal
psql -d star_wars -f star_wars.sql
```

What happens if we try to add a lightsaber with a character id that doesn't exist?

```sql
-- star_wars.sql
INSERT INTO lightsabers (colour, character_id, hilt_metal) VALUES ('red', 1138, 'gold');
```

We get an error, as we might expect:

```
psql:star_wars.sql:24: ERROR:  insert or update on table "lightsabers" violates foreign key constraint "lightsabers_character_id_fkey"
```

PSQL expects that the `character_id` field of `lightsabers` will contain an `INT` which `REFERENCES` the primary key of another table - the `id` column of `characters`. This `character_id` column therefore contains a **foreign key**.

When we try to `INSERT INTO lightsabers` some data, including a `INT` that _doesn't_ correspond to any `id` field in `characters`, we get an error. The constraint of a foreign key is that its value must also exist somewhere in the column of the other table, and `1138` is not an `id` of any character. We have violated the foreign key constraint, which is what the error tells us.

## Conclusion

This is what we call a One to Many relationship. Each lightsaber has ONE owner (`character`). A Character can have MANY `lightsabers`, as different rows in the lightsaber table can have the same `character_id`.

As a final step, lets add one more lightsaber to Anakin/Darth Vader, then lets find all lightsabers that he has!

```sql
INSERT INTO lightsabers (colour, character_id, hilt_metal) VALUES ('red', 2, 'titanium');

SELECT * FROM lightsabers WHERE character_id = 2;
```
