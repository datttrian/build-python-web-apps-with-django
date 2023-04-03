**Course Description**

Often in data science, you’ll encounter fascinating data that is spread
across multiple tables. This course will teach you the skills you’ll
need to join multiple tables together to analyze them in combination.
You’ll practice your skills using a fun dataset about LEGOs from the
Rebrickable website. The dataset contains information about the sets,
parts, themes, and colors of LEGOs, but is spread across many tables.
You’ll work with the data throughout the course as you learn a total of
six different joins! You’ll learn four mutating joins: inner join, left
join, right join, and full join, and two filtering joins: semi join and
anti join. In the final chapter, you’ll apply your new skills to Stack
Overflow data, containing each of the almost 300,000 Stack Oveflow
questions that are tagged with R, including information about their
answers, the date they were asked, and their score. Get ready to take
your dplyr skills to the next level!

# Joining Tables

Get started with your first joining verb: inner-join! You’ll learn to
join tables together to answer questions about the LEGO dataset, which
contains information across many tables about the sets, parts, themes,
and colors of LEGOs over time.

## The inner_join verb

### What columns would you join on?

You’ll be joining together the `parts` and `part_categories` tables. You
can first inspect them in the console. To join these two tables together
using the `inner_join` verb, what columns would you join from each
table?

Since you’ll be working with dplyr throughout the course, the package
will be preloaded for you in each exercise.

- [ ] `c("id" = "part_num")`
- [x] `c("part_cat_id" = "id")`
- [ ] `c("part_num" = "part_cat_id")`
- [ ] `c("part_material_id" = "part_categories")`

### Joining parts and part categories

The `inner_join` is the key to bring tables together. To use it, you
need to provide the two tables that must be joined and the columns on
which they should be joined.

In this exercise, you’ll join a list of LEGO parts, available as
`parts`, with these parts’ corresponding categories, available as
`part_categories`. For example, the part
`Sticker Sheet 1 for Set 1650-1` is from the `Stickers` part category.
You can join these tables to see all parts’ categories!

- Add the correct joining verb, the name of the second table, and the
  joining column for the second table.
- Now, use the `suffix` argument to add `"_part"` and `"_category"`
  suffixes to replace the `name.x` and `name.y` fields.

``` r
# edited/added
library(tidyverse)
parts <- readRDS('parts.rds')
part_categories <- readRDS('part_categories.rds')

# Add the correct verb, table, and joining column
parts %>% 
  inner_join(part_categories, by = c("part_cat_id" = "id"))
  
# Use the suffix argument to replace .x and .y suffixes
parts %>% 
  inner_join(part_categories, by = c("part_cat_id" = "id"), suffix = c("_part", "_category"))
```

## Joining with a one-to-many relationship

### Joining parts and inventories

The LEGO data has many tables that can be joined together. Often times,
some of the things you care about may be a few tables away (we’ll get to
that later in the course). For now, we know that `parts` is a list of
all LEGO parts, and a new table, `inventory_parts`, has some additional
information about those parts, such as the `color_id` of each part you
would find in a specific LEGO kit.

Let’s join these two tables together to observe how joining `parts` with
`inventory_parts` increases the size of your table because of the
one-to-many relationship that exists between these two tables.

- Connect the `parts` and `inventory_parts` tables by their **part
  numbers** using an inner join.

``` r
# edited/added
inventory_parts = readRDS('inventory_parts.rds')

# Combine the parts and inventory_parts tables
parts %>%
  inner_join(inventory_parts, by = "part_num")
```

### Joining in either direction

An `inner_join` works the same way with either table in either position.
The table that is specified first is arbitrary, since you will end up
with the same information in the resulting table either way.

Let’s prove this by joining the same two tables from the last exercise
in the opposite order!

- Connect the `inventory_parts` and `parts` tables by their **part
  numbers** using an inner join.

``` r
# Combine the parts and inventory_parts tables
inventory_parts %>%
  inner_join(parts, by = "part_num")
```

## Joining three or more tables

### Joining three tables

You can string together multiple joins with `inner_join` and the pipe
(`%>%`), both with which you are already very familiar!

We’ll now connect `sets`, a table that tells us about each LEGO kit,
with `inventories`, a table that tells us the specific version of a
given set, and finally to `inventory_parts`, a table which tells us how
many of each part is available in each LEGO kit.

So if you were building a Batman LEGO set, `sets` would tell you the
name of the set, `inventories` would give you IDs for each of the
versions of the set, and `inventory_parts` would tell you how many of
each part would be in each version.

- Combine the `inventories` table with the `sets` table.
- Next, join the `inventory_parts` table to the table you created in the
  previous join by the inventory IDs.

``` r
# edited/added
sets <- readRDS('sets.rds')
inventories <- readRDS('inventories.rds')

sets %>%
  # Add inventories using an inner join 
  inner_join(inventories, by = "set_num") %>%
  # Add inventory_parts using an inner join 
  inner_join(inventory_parts, by = c("id" = "inventory_id"))
```

### What’s the most common color?

Now let’s join an additional table, `colors`, which will tell us the
color of each part in each set, so that we can answer the question,
“what is the most common color of a LEGO piece?”

- Inner join the `colors` table using the `color_id` column from the
  previous join and the `id` column from `colors`; use the suffixes
  `"_set"` and `"_color"`.
- Count the `name_color` column and sort the results so the most
  prominent colors appear first.

``` r
# edited/added
colors = readRDS('colors.rds')

# Add an inner join for the colors table
sets %>%
  inner_join(inventories, by = "set_num") %>%
  inner_join(inventory_parts, by = c("id" = "inventory_id")) %>%
  inner_join(colors, by = c("color_id" = "id"), suffix = c("_set", "_color"))
  
# Count the number of colors and sort
sets %>%
  inner_join(inventories, by = "set_num") %>%
  inner_join(inventory_parts, by = c("id" = "inventory_id")) %>%
  inner_join(colors, by = c("color_id" = "id"), suffix = c("_set", "_color")) %>%
  count(name_color, sort = TRUE)
```

# Left and Right Joins

Learn two more mutating joins, the left and right join, which are mirror
images of each other! You’ll learn use cases for each type of join as
you explore parts and colors of LEGO themes. Then, you’ll explore how to
join tables to themselves to understand the hierarchy of LEGO themes in
the data.

## The left_join verb

### Left joining two sets by part and color

In the video, you learned how to left join two LEGO sets. Now you’ll
practice your ability to do this looking at two new sets: the Millennium
Falcon and Star Destroyer sets. We’ve created these for you and they
have been preloaded for you:

    millennium_falcon <- inventory_parts_joined %>%
      filter(set_num == "7965-1")

    star_destroyer <- inventory_parts_joined %>%
      filter(set_num == "75190-1")

- Left join the `star_destroyer` and `millennium_falcon` tables on the
  `part_num` and `color_id` columns with the suffixes `_falcon` and
  `_star_destroyer`.

``` r
# edited/added
inventory_parts_joined <- sets %>%
  inner_join(inventories, by = "set_num") %>%
  inner_join(inventory_parts, by = c("id" = "inventory_id"))
millennium_falcon <- inventory_parts_joined %>%
  filter(set_num == "7965-1")
star_destroyer <- inventory_parts_joined %>%
  filter(set_num == "75190-1")
    
# Combine the star_destroyer and millennium_falcon tables
millennium_falcon %>%
  left_join(star_destroyer, by = c("part_num", "color_id"), suffix = c("_falcon", "_star_destroyer"))
```

### Left joining two sets by color

In the videos and the last exercise, you joined two sets based on their
part and color. What if you joined the datasets by color alone? As with
the last exercise, the Millennium Falcon and Star Destroyer sets have
been created and preloaded for you:

    millennium_falcon <- inventory_parts_joined %>%
      filter(set_num == "7965-1")

    star_destroyer <- inventory_parts_joined %>%
      filter(set_num == "75190-1")

- Sum the `quantity` column by `color_id` in the Millennium Falcon
  dataset.
- Now, sum the `quantity` column by `color_id` in the Star Destroyer
  dataset.
- Left join the two datasets, `millennium_falcon_colors` and
  `star_destroyer_colors`, using the `color_id` column and the `_falcon`
  and `_star_destroyer` suffixes.

``` r
# Aggregate Millennium Falcon for the total quantity in each part
millennium_falcon_colors <- millennium_falcon %>%
  group_by(color_id) %>%
  summarize(total_quantity = sum(quantity))
  
# Aggregate Millennium Falcon for the total quantity in each part
millennium_falcon_colors <- millennium_falcon %>%
  group_by(color_id) %>%
  summarize(total_quantity = sum(quantity))

# Aggregate Star Destroyer for the total quantity in each part
star_destroyer_colors <- star_destroyer %>%
  group_by(color_id) %>%
  summarize(total_quantity = sum(quantity))
  
# Aggregate Millennium Falcon for the total quantity in each part
millennium_falcon_colors <- millennium_falcon %>%
  group_by(color_id) %>%
  summarize(total_quantity = sum(quantity))

# Aggregate Star Destroyer for the total quantity in each part
star_destroyer_colors <- star_destroyer %>%
  group_by(color_id) %>%
  summarize(total_quantity = sum(quantity))

# Left join the Millennium Falcon colors to the Star Destroyer colors
millennium_falcon_colors %>%
  left_join(star_destroyer_colors, by = "color_id", suffix = c("_falcon", "_star_destroyer"))
```

### Finding an observation that doesn’t have a match

Left joins are really great for testing your assumptions about a data
set and ensuring your data has integrity.

For example, the `inventories` table has a `version` column, for when a
LEGO kit gets some kind of change or upgrade. It would be fair to assume
that all `sets` (which joins well with `inventories`) would have at
least a version 1. But let’s test this assumption out in the following
exercise.

- Use a `left_join` to join together `sets` and `inventory_version_1`
  using their common column.
- `filter` for where the `version` column is `NA` using `is.na`.

``` r
inventory_version_1 <- inventories %>%
    filter(version == 1)

# Join versions to sets
sets %>%
  left_join(inventory_version_1, by = "set_num") %>%
  # Filter for where version is na
  filter(is.na(version))
```

## The right_join verb

### Which join is best?

Now that you’ve learned three different types of joins, you will be able
to identify situations in which each join should be used.

- Match each definition with the type of join that is described.

##### inner

You want to keep only observations that match perfectly between tables.

##### left

You want to keep all observations in the first table, including matching
observations in the second table.

##### right

You want to keep all observations in the second table, including
matching observations in the first table.

### Counting part colors

Sometimes you’ll want to do some processing *before* you do a join,
*and* prioritize keeping the second (right) table’s rows instead. In
this case, a right join is for you.

In this exercise, we’ll count the `part_cat_id` from `parts`, before
using a `right_join` to join with `part_categories`. The reason we do
this is because we don’t only want to know the count of `part_cat_id` in
`parts`, but we also want to know if there are any `part_cat_id`s *not*
present in `parts`.

- Use the `count` verb to count each `part_cat_id` in the `parts` table.
- Use a `right_join` to join `part_categories`. You’ll need to use the
  `part_cat_id` from the count and the `id` column from
  `part_categories`.
- `filter` for where the column `n` is NA.

``` r
parts %>%
  # Count the part_cat_id
  count(part_cat_id) %>%
  # Right join part_categories
  right_join(part_categories, by = c("part_cat_id" = "id"))
  
parts %>%
  count(part_cat_id) %>%
  right_join(part_categories, by = c("part_cat_id" = "id")) %>%
  # Filter for NA
  filter(is.na(n))
```

### Cleaning up your count

In both left and right joins, there is the opportunity for there to be
NA values in the resulting table. Fortunately, the `replace_na` function
can turn those NAs into meaningful values.

In the last exercise, we saw that the `n` column had NAs after the
`right_join`. Let’s use the `replace_na` column, which takes a `list` of
column names and the values with which NAs should be replaced, to clean
up our table.

- Use `replace_na` to replace NAs in the `n` column with the value `0`.

``` r
parts %>%
  count(part_cat_id) %>%
  right_join(part_categories, by = c("part_cat_id" = "id")) %>%
  # Use replace_na to replace missing values in the n column
  replace_na(list(n = 0))
```

## Joining tables to themselves

### Joining themes to their children

Tables can be joined to themselves!

In the `themes` table, which is available for you to inspect in the
console, you’ll notice there is both an `id` column and a `parent_id`
column. Keeping that in mind, you can join the `themes` table to itself
to determine the parent-child relationships that exist for different
themes.

In the videos, you saw themes joined to their own parents. In this
exercise, you’ll try a similar approach of joining themes to their own
*children*, which is similar but reversed. Let’s try this out to
discover what children the theme `"Harry Potter"` has.

- Inner join `themes` to their own children, resulting in the suffixes
  `"_parent"` and `"_child"`, respectively.
- Filter this table to find the children of the `"Harry Potter"` theme.

``` r
# edited/added
themes <- readRDS('themes.rds')

themes %>% 
  # Inner join the themes table
  inner_join(themes, by = c("id" = "parent_id"), suffix = c("_parent", "_child")) %>% 
  # Filter for the "Harry Potter" parent name 
  filter(name_parent == "Harry Potter")
```

### Joining themes to their grandchildren

We can go a step further than looking at themes and their children. Some
themes actually have *grandchildren*: their children’s children.

Here, we can inner join `themes` to a filtered version of itself *again*
to establish a connection between our last join’s children and *their*
children.

- Use another inner join to combine `themes` again with itself.
  - Be sure to use the suffixes `"_parent"` and `"_grandchild"` so the
    columns in the resulting table are clear.
  - Update the `by` argument to specify the correct columns to join on.
    If you’re unsure of what columns to join on, it might help to look
    at the result of the first join to get a feel for it.

``` r
# Join themes to itself again to find the grandchild relationships
themes %>% 
  inner_join(themes, by = c("id" = "parent_id"), suffix = c("_parent", "_child")) %>%
  inner_join(themes, by = c("id_child" = "parent_id"), suffix = c("_parent", "_grandchild"))
```

### Left joining a table to itself

So far, you’ve been inner joining a table to itself in order to find the
children of themes like `"Harry Potter"` or `"The Lord of the Rings"`.

But some themes might not have any children at all, which means they
won’t be included in the inner join. As you’ve learned in this chapter,
you can identify those with a `left_join` and a `filter()`.

- Left join the themes table to its own children, with the suffixes
  `_parent` and `_child` respectively.
- Filter the result of the join to find themes that have no children.

``` r
themes %>% 
  # Left join the themes table to its own children
  left_join(themes, by = c("id" = "parent_id"), suffix = c("_parent", "_child")) %>%
  # Filter for themes that have no child themes
  filter(is.na(name_child))
```

# Full, Semi, and Anti Joins

In this chapter, you’ll cover three more joining verbs: full-join,
semi-join, and anti-join. You’ll then use these verbs to answer
questions about the similarities and differences between a variety of
LEGO sets.

## The full_join verb

### Differences between Batman and Star Wars

In the video, you compared two sets. Now, you’ll compare two themes,
each of which is made up of many sets.

First, you’ll need to join in the `themes`. Recall that doing so
requires going through the `sets` first. You’ll use the
`inventory_parts_joined` table from the video, which is already
available to you in the console.

    inventory_parts_joined <- inventories %>%
      inner_join(inventory_parts, by = c("id" = "inventory_id")) %>%
      arrange(desc(quantity)) %>%
      select(-id, -version)

- In order to join in the themes, you’ll first need to combine the
  `inventory_parts_joined` and `sets` tables.
- Then, combine the first join with the `themes` table, using the suffix
  argument to clarify which table each `name` came from (`"_set"` or
  `"_theme"`).

``` r
# edited/added
inventory_parts_joined <- inventories %>%
  inner_join(inventory_parts, by = c("id" = "inventory_id")) %>%
  arrange(desc(quantity)) %>%
  select(-id, -version)

# Start with inventory_parts_joined table
inventory_parts_joined %>%
  # Combine with the sets table 
  inner_join(sets, by = "set_num") %>%
  # Combine with the themes table
  inner_join(themes, by = c("theme_id" = "id"), suffix = c("_set", "_theme"))
```

### Aggregating each theme

Previously, you combined tables to compare themes. Before doing this
comparison, you’ll want to aggregate the data to learn more about the
pieces that are a part of each theme, as well as the colors of those
pieces.

The table you created previously has been preloaded for you as
`inventory_sets_themes`. It was filtered for each theme, and the objects
have been saved as `batman` and `star_wars`.

    inventory_sets_themes <- inventory_parts_joined %>%
      inner_join(sets, by = "set_num") %>%
      inner_join(themes, by = c("theme_id" = "id"), suffix = c("_set", "_theme"))

    batman <- inventory_sets_themes %>%
      filter(name_theme == "Batman")

    star_wars <- inventory_sets_themes %>%
      filter(name_theme == "Star Wars")

- Count the part number and color id for the parts in Batman and Star
  Wars, weighted by quantity.

``` r
# edited/added
inventory_sets_themes <- inventory_parts_joined %>%
  inner_join(sets, by = "set_num") %>%
  inner_join(themes, by = c("theme_id" = "id"), suffix = c("_set", "_theme"))
batman <- inventory_sets_themes %>%
  filter(name_theme == "Batman")
star_wars <- inventory_sets_themes %>%
  filter(name_theme == "Star Wars")
    
# Count the part number and color id, weight by quantity
batman %>%
  count(part_num, color_id, wt = quantity)

star_wars %>%
  count(part_num, color_id, wt = quantity)
```

### Full joining Batman and Star Wars LEGO parts

Now that you’ve got separate tables for the pieces in the `batman` and
`star_wars` themes, you’ll want to be able to combine them to see any
similarities or differences between the two themes. The aggregating from
the last exercise has been saved as `batman_parts` and
`star_wars_parts`, and is preloaded for you.

    batman_parts <- batman %>%
      count(part_num, color_id, wt = quantity)

    star_wars_parts <- star_wars %>%
      count(part_num, color_id, wt = quantity)

- Combine the `star_wars_parts` table with the `batman_parts` table; use
  the suffix argument to include the `"_batman"` and `"_star_wars"`
  suffixes.
- Replace all the NA values in the `n_batman` and `n_star_wars` columns
  with 0s.

``` r
# edited/added
batman_parts <- batman %>%
  count(part_num, color_id, wt = quantity)
star_wars_parts <- star_wars %>%
  count(part_num, color_id, wt = quantity)
    
batman_parts %>%
  # Combine the star_wars_parts table 
  full_join(star_wars_parts, by = c("part_num", "color_id"), suffix = c("_batman", "_star_wars")) %>%
  # Replace NAs with 0s in the n_batman and n_star_wars columns 
  replace_na(list(n_batman = 0, n_star_wars = 0))
```

### Comparing Batman and Star Wars LEGO parts

The table you created in the last exercise includes the part number of
each piece, the color id, and the number of each piece in the Star Wars
and Batman themes. However, we have more information about each of these
parts that we can gain by combining this table with some of the
information we have in other tables. Before we compare the themes, let’s
ensure that we have enough information to make our findings more
interpretable. The table from the last exercise has been saved as
`parts_joined` and is preloaded for you.

    parts_joined <- batman_parts %>%
      full_join(star_wars_parts, by = c("part_num", "color_id"), suffix = c("_batman", "_star_wars")) %>%
      replace_na(list(n_batman = 0, n_star_wars = 0))

- Sort the number of star wars pieces in the `parts_joined` table in
  descending order.
- Inner join the `colors` table to the `parts_joined` table.
- Combine the `parts` table to the previous join using an inner join;
  add `"_color"` and `"_part"` suffixes to specify whether or not the
  information came from the `colors` table or the `parts` table.

``` r
# edited/added
parts_joined <- batman_parts %>%
  full_join(star_wars_parts, by = c("part_num", "color_id"), suffix = c("_batman", "_star_wars")) %>%
  replace_na(list(n_batman = 0, n_star_wars = 0))

parts_joined %>%
  # Sort the number of star wars pieces in descending order 
  arrange(desc(n_star_wars)) %>%
  # Join the colors table to the parts_joined table
  inner_join(colors, by = c("color_id" = "id")) %>%
  # Join the parts table to the previous join 
  inner_join(parts, by = "part_num", suffix = c("_color", "_part"))
```

## The semi_join and anti_join verbs

### Select the join

Earlier in the course, you distinguished between inner, left, and right
joins. Since then, you’ve learned three new types of joins! Can you
match these joins to the type of join described here?

- Match each definition with the type of join that is described.

##### full

Keep all observations from both tables.

##### semi

Filter the first table for observations that match the second.

##### anti

Filter the first table for observations that don’t match the second.

### Something within one set but not another

In the videos, you learned how to filter using the semi- and anti join
verbs to answer questions you have about your data. Let’s focus on the
`batwing` dataset, and use our skills to determine which parts are in
both the `batwing` and `batmobile` sets, and which sets are in one, but
not the other. While answering these questions, we’ll also be
determining whether or not the parts we’re looking at in both sets also
have the same color in common.

The `batmobile` and `batwing` datasets have been preloaded for you.

    batmobile <- inventory_parts_joined %>%
      filter(set_num == "7784-1") %>%
      select(-set_num)

    batwing <- inventory_parts_joined %>%
      filter(set_num == "70916-1") %>%
      select(-set_num)

- Filter the `batwing` set for parts that are **also** in the
  `batmobile`, whether or not they have the same color.
- Filter the `batwing` set for parts that **aren’t** also in the
  `batmobile`, whether or not they have the same color.

``` r
# edited/added
batmobile <- inventory_parts_joined %>%
  filter(set_num == "7784-1") %>%
  select(-set_num)
batwing <- inventory_parts_joined %>%
  filter(set_num == "70916-1") %>%
  select(-set_num)

# Filter the batwing set for parts that are also in the batmobile set
batwing %>%
  semi_join(batmobile, by = c("part_num"))

# Filter the batwing set for parts that aren't in the batmobile set
batwing %>%
  anti_join(batmobile, by = c("part_num"))
```

### What colors are included in at least one set?

Besides comparing two sets directly, you could also use a filtering join
like `semi_join` to find out which colors ever appear in any inventory
part. Some of the colors could be optional, meaning they aren’t included
in any sets.

The `inventory_parts` and `colors` tables have been preloaded for you.

- Use the `inventory_parts` table to find the colors that are included
  in at least one set.

``` r
# Use inventory_parts to find colors included in at least one set
colors %>%
  semi_join(inventory_parts, by = c("id" = "color_id")) 
```

### Which set is missing version 1?

Each set included in the LEGO data has an associated version number. We
want to understand the version we are looking at to learn more about the
parts that are included. Before doing that, we should confirm that there
aren’t any sets that are missing a particular version.

Let’s start by looking at the first version of each set to see if there
are any sets that don’t include a first version.

- Use `filter()` to extract `version` `1` from the `inventories` table;
  save the filter to `version_1_inventories`.
- Use `anti_join` to combine `version_1_inventories` with `sets` to
  determine which set is missing a version 1.

``` r
# Use filter() to extract version 1 
version_1_inventories <- inventories %>%
  filter(version == 1)

# Use anti_join() to find which set is missing a version 1
sets %>%
  anti_join(version_1_inventories, by = "set_num")
```

## Visualizing set differences

### Aggregating sets to look at their differences

To compare two individual sets, and the kinds of LEGO pieces that
comprise them, we’ll need to aggregate the data into separate themes.
Additionally, as we saw in the video, we’ll want to add a column so that
we can understand the fractions of specific pieces that are part of each
set, rather than looking at the numbers of pieces alone.

The `inventory_parts_themes` table has been preloaded for you.

    inventory_parts_themes <- inventories %>%
      inner_join(inventory_parts, by = c("id" = "inventory_id")) %>%
      arrange(desc(quantity)) %>%
      select(-id, -version) %>%
      inner_join(sets, by = "set_num") %>%
      inner_join(themes, by = c("theme_id" = "id"), suffix = c("_set", "_theme"))

- Add a filter for the `"Batman"` theme to create the `batman_colors`
  object.
- Add a `fraction` column to `batman_colors` that displays the total
  divided by the sum of the total.
- Repeat the steps to filter and aggregate the `"Star Wars"` set data to
  create the `star_wars_colors` object.
- Add a `fraction` column to `star_wars_colors` to display the fraction
  of the total.

``` r
# edited/added
inventory_parts_themes <- inventories %>%
  inner_join(inventory_parts, by = c("id" = "inventory_id")) %>%
  arrange(desc(quantity)) %>%
  select(-id, -version) %>%
  inner_join(sets, by = "set_num") %>%
  inner_join(themes, by = c("theme_id" = "id"), suffix = c("_set", "_theme"))

batman_colors <- inventory_parts_themes %>%
  # Filter the inventory_parts_themes table for the Batman theme
  filter(name_theme == "Batman") %>%
  group_by(color_id) %>%
  summarize(total = sum(quantity)) %>%
  # Add a fraction column of the total divided by the sum of the total 
  mutate(fraction = total / sum(total))

# Filter and aggregate the Star Wars set data; add a fraction column
star_wars_colors <- inventory_parts_themes %>%
  filter(name_theme == "Star Wars") %>%
  group_by(color_id) %>%
  summarize(total = sum(quantity)) %>%
  mutate(fraction = total / sum(total))
```

### Combining sets

The data you aggregated in the last exercise has been preloaded for you
as `batman_colors` and `star_wars_colors`. Prior to visualizing the
data, you’ll want to combine these tables to be able to directly compare
the themes’ colors.

    batman_colors <- inventory_parts_themes %>%
      filter(name_theme == "Batman") %>%
      group_by(color_id) %>%
      summarize(total = sum(quantity)) %>%
      mutate(fraction = total / sum(total))

    star_wars_colors <- inventory_parts_themes %>%
      filter(name_theme == "Star Wars") %>%
      group_by(color_id) %>%
      summarize(total = sum(quantity)) %>%
      mutate(fraction = total / sum(total))

- Join the `batman_colors` and `star_wars_colors` tables; be sure to
  include all observations from both tables.
- Replace the NAs in the `total_batman` and `total_star_wars` columns.
- Add a `difference` column which is the difference between
  `fraction_batman` and `fraction_star_wars`, and a `total` column,
  which is the sum of `total_batman` and `total_star_wars.`
- Add a `filter` to select observations where `total` is at least 200.

``` r
batman_colors %>%
  # Join the Batman and Star Wars colors
  full_join(star_wars_colors, by = "color_id", suffix = c("_batman", "_star_wars")) %>%
  # Replace NAs in the total_batman and total_star_wars columns
  replace_na(list(total_batman = 0, total_star_wars = 0)) %>%
  inner_join(colors, by = c("color_id" = "id"))
  
batman_colors %>%
  full_join(star_wars_colors, by = "color_id", suffix = c("_batman", "_star_wars")) %>%
  replace_na(list(total_batman = 0, total_star_wars = 0)) %>%
  inner_join(colors, by = c("color_id" = "id")) %>%
  # Create the difference and total columns
  mutate(difference = fraction_batman - fraction_star_wars,
         total = total_batman + total_star_wars) %>%
  # Filter for totals greater than 200
  filter(total >= 200)
```

### Visualizing the difference: Batman and Star Wars

In the last exercise, you created `colors_joined`. Now you’ll create a
bar plot with one bar for each color (`name`), showing the difference in
fractions.

Because factors and visualization are beyond the scope of this course,
we’ve done some processing for you: here is the code that created the
`colors_joined` table that will be used in the video.

    colors_joined <- batman_colors %>%
      full_join(star_wars_colors, by = "color_id", suffix = c("_batman", "_star_wars")) %>%
      replace_na(list(total_batman = 0, total_star_wars = 0)) %>%
      inner_join(colors, by = c("color_id" = "id")) %>%
      mutate(difference = fraction_batman - fraction_star_wars,
             total = total_batman + total_star_wars) %>%
      filter(total >= 200) %>%
      mutate(name = fct_reorder(name, difference)) 

- Create a bar plot using the `colors_joined` table to display the most
  prominent colors in the Batman and Star Wars themes, with the bars
  colored by their `name`.

``` r
# edited/added
colors_joined <- batman_colors %>%
  full_join(star_wars_colors, by = "color_id", suffix = c("_batman", "_star_wars")) %>%
  replace_na(list(total_batman = 0, total_star_wars = 0)) %>%
  inner_join(colors, by = c("color_id" = "id")) %>%
  mutate(difference = fraction_batman - fraction_star_wars,
         total = total_batman + total_star_wars) %>%
  filter(total >= 200) %>% 
  drop_na() %>% # edited/added
  mutate(name = fct_reorder(name, difference))
color_palette <- setNames(colors_joined$rgb, colors_joined$name)

# Create a bar plot using colors_joined and the name and difference columns
ggplot(colors_joined, aes(name, difference, fill = name)) +
  geom_col() +
  coord_flip() +
  scale_fill_manual(values = color_palette, guide = "none") +
  labs(y = "Difference: Batman - Star Wars")
```

# Case Study: Joins on Stack Overflow Data

Put together all the types of join you learned in this course to analyze
a new dataset: Stack Overflow questions, answers, and tags. This
includes calculating and visualizing trends for some notable tags like
dplyr and ggplot2. You’ll also master one more method for combining
tables, the bind_rows verb, which stacks tables on top of each other.

## Stack Overflow questions

### Left joining questions and tags

Three of the Stack Overflow survey datasets are `questions`,
`question_tags`, and `tags`:

- `questions`: an ID and the score, or how many times the question has
  been upvoted; the data only includes R-based questions
- `question_tags`: a tag ID for each question and the question’s id
- `tags`: a tag id and the tag’s name, which can be used to identify the
  subject of each question, such as ggplot2 or dplyr

In this exercise, we’ll be stitching together these datasets and
replacing NAs in important fields.

Note that we’ll be using `left_join`s in this exercise to ensure we keep
all questions, even those without a corresponding tag. However, since we
know the `questions` data is all R data, we’ll want to manually tag
these as R questions with `replace_na`.

- Join together `questions` and `question_tags` using the `id` and
  `question_id` columns, respectively.
- Use another join to add in the `tags` table.
- Use `replace_na` to change the `NA`s in the `tag_name` column to
  `"only-r"`.

``` r
# edited/added
questions <- readRDS("questions.rds")
question_tags <- readRDS("question_tags.rds")
tags <- readRDS("tags.rds")

# Join the questions and question_tags tables
questions %>%
  left_join(question_tags, by = c("id" = "question_id"))
  
# Join in the tags table
questions %>%
  left_join(question_tags, by = c("id" = "question_id")) %>%
  left_join(tags, by = c("tag_id" = "id"))
  
# Replace the NAs in the tag_name column
questions %>%
  left_join(question_tags, by = c("id" = "question_id")) %>%
  left_join(tags, by = c("tag_id" = "id")) %>%
  replace_na(list(tag_name = "only-r"))
```

### Comparing scores across tags

The complete dataset you created in the last exercise is available to
you as `questions_with_tags`. Let’s do a quick bit of analysis on it!
You’ll use familiar dplyr verbs like `group_by`, `summarize`, `arrange`,
and `n` to find out the average score of the most asked questions.

- Aggregate by the `tag_name`.
- Summarize to get the mean score for each question, `score`, as well as
  the total number of questions, `num_questions`.
- Arrange `num_questions` in descending order to sort the answers by the
  most asked questions.

``` r
# edited/added
questions_with_tags <- questions %>%
  left_join(question_tags, by = c("id" = "question_id")) %>%
  left_join(tags, by = c("tag_id" = "id")) %>%
  replace_na(list(tag_name = "only-r"))

questions_with_tags %>%
  # Group by tag_name
  group_by(tag_name) %>%
  # Get mean score and num_questions
  summarize(score = mean(score),
            num_questions = n()) %>%
  # Sort num_questions in descending order
  arrange(desc(num_questions))
```

### What tags never appear on R questions?

The `tags` table includes all Stack Overflow tags, but some have nothing
to do with `R`. How could you filter for just the tags that never appear
on an R question? The `tags` and `question_tags` tables have been
preloaded for you.

- Use a join to determine which tags never appear on an R question.

``` r
# Using a join, filter for tags that are never on an R question
tags %>%
  anti_join(question_tags, by = c("id" = "tag_id"))
```

## Joining questions and answers

### Finding gaps between questions and answers

Now we’ll join together `questions` with `answers` so we can measure the
time between questions and answers.

Make sure to explore the tables and columns in the console before
starting the exercise. Can you tell how are questions identified in the
`questions` table? How can you identify which answer corresponds to
which question using the `answers` table?

- Use an inner join to combine the `questions` and `answers` tables
  using the suffixes `"_question"` and `"_answer"`, respectively.
- Subtract `creation_date_question` from `creation_date_answer` within
  the `as.integer()` function to create the `gap` column.

``` r
# edited/added
answers <- readRDS("answers.rds")

questions %>%
  # Inner join questions and answers with proper suffixes
  inner_join(answers, by = c("id" = "question_id"), suffix = c("_question", "_answer")) %>%
  # Subtract creation_date_question from creation_date_answer to create gap
  mutate(gap = as.integer(creation_date_answer - creation_date_question)) 
```

### Joining question and answer counts

We can also determine how many questions actually yield answers. If we
count the number of answers for each question, we can then join the
answers counts with the `questions` table.

- Count and sort the `question_id` column in the `answers` table to
  create the `answer_counts` table.
- Join the `questions` table with the `answer_counts` table and include
  all observations from the `questions` table.
- Replace the NA values in the `n` column with 0s.

``` r
# Count and sort the question id column in the answers table
answer_counts <- answers %>%
  count(question_id, sort = TRUE)

# Combine the answer_counts and questions tables
questions %>%
  left_join(answer_counts, by = c("id" = "question_id")) %>%
  # Replace the NAs in the n column
  replace_na(list(n = 0))
```

### Joining questions, answers, and tags

Let’s build on the last exercise by adding the `tags` table to our
previous joins. This will allow us to do a better job of identifying
which R topics get the most traction on Stack Overflow. The tables you
created in the last exercise have been preloaded for you as
`answer_counts` and `question_answer_counts`.

    answer_counts <- answers %>%
        count(question_id, sort = TRUE)

    question_answer_counts <- questions %>%
        left_join(answer_counts, by = c("id" = "question_id")) %>%
        replace_na(list(n = 0))

- Combine the `question_tags` table with `question_answer_counts` using
  an `inner_join`.
- Now, use another `inner_join` to add the `tags` table.

``` r
# edited/added
answer_counts <- answers %>%
  count(question_id, sort = TRUE)
question_answer_counts <- questions %>%
  left_join(answer_counts, by = c("id" = "question_id")) %>%
  replace_na(list(n = 0))
    
question_answer_counts %>%
  # Join the question_tags tables
  inner_join(question_tags, by = c("id" = "question_id")) %>%
  # Join the tags table
  inner_join(tags, by = c("tag_id" = "id"))
```

### Average answers by question

The table you created in the last exercise has been preloaded for you as
`tagged_answers`. You can use this table to determine, on average, how
many answers each questions gets.

    tagged_answers <- question_answer_counts %>%
        inner_join(question_tags, by = c("id" = "question_id")) %>%
        inner_join(tags, by = c("tag_id" = "id"))

Some of the important variables from this table include: `n`, the number
of answers for each question, and `tag_name`, the name of each tag
associated with each question.

Let’s use some of our favorite dplyr verbs to find out how many answers
each question gets on average.

- Aggregate the `tagged_answers` table by `tag_name`.
- Summarize `tagged_answers` to get the count of `questions` and the
  `average_answers`.
- Sort the resulting `questions` column in descending order.

``` r
# edited/added
tagged_answers <- question_answer_counts %>%
  inner_join(question_tags, by = c("id" = "question_id")) %>%
  inner_join(tags, by = c("tag_id" = "id"))

tagged_answers %>%
  # Aggregate by tag_name
  group_by(tag_name) %>%
  # Summarize questions and average_answers
  summarize(questions = n(),
            average_answers = mean(n)) %>%
  # Sort the questions in descending order
  arrange(desc(questions))
```

## The bind_rows verb

### Joining questions and answers with tags

To learn more about the `questions` and `answers` tables, you’ll want to
use the `question_tags` table to understand the tags associated with
each question that was asked, and each answer that was provided. You’ll
be able to combine these tables using two inner joins on both the
`questions` table and the `answers` table.

- Use two inner joins to combine the `question_tags` and `tags` tables
  with the `questions` table.
- Now, use two inner joins to combine the `question_tags` and `tags`
  tables with the `answers` table.

``` r
# Inner join the question_tags and tags tables with the questions table
questions %>%
  inner_join(question_tags, by = c("id" = "question_id")) %>%
  inner_join(tags, by = c("tag_id" = "id"))

# Inner join the question_tags and tags tables with the answers table
answers %>%
  inner_join(question_tags, by = "question_id") %>%
  inner_join(tags, by = c("tag_id" = "id"))
```

### Binding and counting posts with tags

The tables you created in the previous exercise have been preloaded as
`questions_with_tags` and `answers_with_tags`. First, you’ll want to
combine these tables into a single table called `posts_with_tags`. Once
the information is consolidated into a single table, you can add more
information by creating a date variable using the `lubridate` package,
which has been preloaded for you.

    questions_with_tags <- questions %>%
      inner_join(question_tags, by = c("id" = "question_id")) %>%
      inner_join(tags, by = c("tag_id" = "id"))

    answers_with_tags <- answers %>%
      inner_join(question_tags, by = "question_id") %>%
      inner_join(tags, by = c("tag_id" = "id"))

- Combine the `questions_with_tags` and `answers_with_tags` tables into
  `posts_with_tags`.
- Add a `year` column to the `posts_with_tags` table, then count posts
  by `type`, `year`, and `tag_name`.

``` r
# edited/added
questions_with_tags <- questions %>%
  inner_join(question_tags, by = c("id" = "question_id")) %>%
  inner_join(tags, by = c("tag_id" = "id"))
answers_with_tags <- answers %>%
  inner_join(question_tags, by = "question_id") %>%
  inner_join(tags, by = c("tag_id" = "id"))

# Combine the two tables into posts_with_tags
posts_with_tags <- bind_rows(questions_with_tags %>% mutate(type = "question"),
                              answers_with_tags %>% mutate(type = "answer"))

# Add a year column, then count by type, year, and tag_name
posts_with_tags %>%
  mutate(year = year(creation_date)) %>%
  count(type, year, tag_name)
```

### Visualizing questions and answers in tags

In the last exercise, you modified the `posts_with_tags` table to add a
`year` column, and aggregated by `type`, `year`, and `tag_name`. The
modified table has been preloaded for you as `by_type_year_tag`, and has
one observation for each type (question/answer), year, and tag. Let’s
create a plot to examine the information that the table contains about
questions and answers for the `dplyr` and `ggplot2` tags. The `ggplot2`
package has been preloaded for you.

    by_type_year_tag <- posts_with_tags %>%
      mutate(year = year(creation_date)) %>%
      count(type, year, tag_name)

- Filter the `by_type_year_tag` table for the dplyr and ggplot2 tags.
- Create a line plot with that filtered table that plots the frequency
  (`n`) over time, colored by question/answer and faceted by tag.

``` r
# edited/added
by_type_year_tag <- posts_with_tags %>%
  mutate(year = year(creation_date)) %>%
  count(type, year, tag_name)

# Filter for the dplyr and ggplot2 tag names 
by_type_year_tag_filtered <- by_type_year_tag %>%
  filter(tag_name %in% c("dplyr", "ggplot2"))

# Create a line plot faceted by the tag name 
ggplot(by_type_year_tag_filtered, aes(year, n, color = type)) +
  geom_line() +
  facet_wrap(~ tag_name)
```

## Congratulations!

### Congratulations!

Congratulations on making it through the course!

### The joining verbs

To review, you learned about 6 different joins and how to use classic
dplyr verbs to create meaningful insights from your joined data.

### The mutating joins

You learned about the 4 mutating joins: inner join, which keeps only
observations which match exactly between two tables, left joins, which
keep all observations from the first table in your joins, right joins,
which keep all observations from the second table in your joins, and
full joins, which keep all observations from both tables.

### The filtering joins

You also learned the 2 filtering joins; semi joins, which filter the
first table for observations which also exist in the second table, and
anti joins, which filter the first table for observations that do not
exist in the second table.

### Congratulations!

Pretty neat, huh? I hope you enjoyed the course, and feel confident that
you can use these new tools in your future analyses. And can’t wait for
you to JOIN us in future lessons!