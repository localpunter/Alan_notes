# Joining data together

> Setup for this lesson

```bash
mkdir joins_one_to_many
cd joins_one_to_many
cp ~/Downloads/star_wars.sql
dropdb star_wars
createdb star_wars
psql -d star_wars -f star_wars.sql
touch queries.sql
```

In our Star Wars example, we linked the Jedi to a lightsaber through the `character_id`, but we didn't have an easy way to see that information all together.

Say for example, we wanted to view a list of Jedi names with the relevant information about their lightsaber.

We could do it, but we'd have to use multiple queries.

It is possible to do in one query, we just need to add a new tool to our toolbelt to make this happen - joins.

## Joins

When we join data, there's a few different ways we can approach it but the one thing that really matters is that there has to be some commonality between the tables. In our case, we have the Jedi id linking the two tables and we can use this to do some interesting things.

With joins, we think in terms of a 'right' column and a 'left' column. These can be any table you like, but it's worth being clear in your head which one you are talking about.

Let's say the `characters` table is the 'left' table and the `lightsabers` table is the 'right table'.

You can think of a join query as giving us back a sort of 'middle' view that takes data from both sides and combines it.

We might want to:
- find a match between the columns in both tables using a common key => INNER JOIN
- return all rows from the left table (`characters`), with the matching rows in the right table (`lightsabers`). The result is NULL in the right side when there is no match. => LEFT JOIN
- return all rows from the right table (`lightsabers`), with the matching rows in the left table (`characters`). The result is NULL in the left side when there is no match. => RIGHT JOIN

The first option is the most common and that's what we are going to look at just now.

## Explicitly Naming Tables

Sometimes when writing our queries, we want to make it clear which tables we're working with. Let's look at a really simple example:

```sql
-- psql
SELECT colour FROM lightsabers;
```

We can explicitly name the table like so:

```sql
SELECT lightsabers.colour FROM lightsabers;
```

This helps when we have tables with the same property e.g. id or name etc. We can make it very clear which table we want to use.

This will be incredibly useful when we make joins between tables, as it will help
us keep track of which tables have which properties.

## INNER JOIN

INNER JOIN syntax is pretty weird, but hang in there and we'll be okay.

Let's see if we can list all of the Jedi with their relevant lightsaber data.

STEP ONE: Select the columns from the left hand table, using an alias.

```sql
SELECT characters.* FROM characters;
```

STEP TWO: Introduce the right hand table you want to bring into the join and the common property you want to match on

```sql
SELECT characters.* FROM characters
INNER JOIN lightsabers
ON characters.id = lightsabers.character_id;
```

Notice that it's only matched Luke and Kylo Ren, there is no sign of Rey. This is because there is no entry for her in the `lightsabers` table.

There's also no sign of the columns from the `lightsabers` table! How do you think we can fix this?

STEP THREE: Bring in the columns from the right hand table

```sql
SELECT characters.*, lightsabers.* FROM characters
  INNER JOIN lightsabers
  ON characters.id = lightsabers.character_id;
```

There is our data!

[TASK]: See if you can return only the character name and lightsaber colour from the join.

```sql
SELECT characters.name, lightsabers.colour FROM characters
  INNER JOIN lightsabers
  ON characters.id = lightsabers.character_id;
```

## Left (Outer) Join

I mentioned earlier that we might want to do a query where we return all the rows from the left table (`characters`), with the matching rows in the right table (`lightsabers`). In this case, any Jedi with no matching lightsaber will have values shown as null for those columns.

```sql
SELECT characters.*, lightsabers.* FROM characters
  LEFT JOIN lightsabers
  ON characters.id = lightsabers.character_id;
```

Right joins are exactly the same, but with the tables swapped around. It doesn't matter for our lightsabers, since every single lightsaber is owned by a Jedi so there is never a case where a null would be shown.
