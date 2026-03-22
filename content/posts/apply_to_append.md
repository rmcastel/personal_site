---
title: "Using Apply To Append Columns"
date: 2026-03-20
draft: false
description: "A trick I came across to reduce redundant coding in SQL."
tags: ["general", "programming", "sql"]
showAuthor: true
---

Recently I was working with a complex SQL query that had been in production for some time. It was a situation where a `CASE` statement was being reused and called multiple times. As a result, with each call the `CASE` statement had to be rewritten. So the end output looked something like this:

```sql
SELECT pk.*
      ,fm.name as move_name
      ,fm.power
      ,fm.accuracy
      -- SOME MORE COMPLEX SQL CASE STATEMENT
      ,CASE WHEN fm.move_type IN ('normal', 'grass')
            THEN IFNULL(fm.power, 0) / (IFNULL(fm.accuracy, 1) * 1.0)
            WHEN fm.move_type = 'flying'
            THEN IFNULL(fm.power, 0) * 2 / (IFNULL(fm.accuracy, 1) * 1.0)
            ELSE IFNULL(fm.power, 0) * 3 / (IFNULL(fm.accuracy, 1) * 1.0) END AS new_var
      -- SOME MORE COMPLEX SQL CASE STATEMENT, AGAIN...
      ,SQRT(
       CASE WHEN fm.move_type IN ('normal', 'grass')
            THEN IFNULL(fm.power, 0) / (IFNULL(fm.accuracy, 1) * 1.0)
            WHEN fm.move_type = 'flying'             
            THEN IFNULL(fm.power, 0) * 2 / (IFNULL(fm.accuracy, 1) * 1.0)
            ELSE IFNULL(fm.power, 0) * 3 / (IFNULL(fm.accuracy, 1) * 1.0) END 
       ) AS new_var_sqrt
       -- ....
       -- ....
       -- MORE VERSIONS OF THE SAME CASE STATEMENT
FROM   pokemon.fact_pokemon pk
JOIN   pokemon.dim_moves mv
ON     pk.id = mv.id
AND    mv.version_group = 'diamond-pearl'
JOIN   pokemon.fact_moves fm
ON     mv.move = fm.name
AND    fm.version = 'diamond-pearl'
```

This is a bit of an oversimplification (especially using a simple [pokemon database](https://github.com/rmcastel/poke_db)), but one can see how repeating a `CASE` statement can become unwieldy and annoying to handle. One could argue the code shouldn't have been written this way to begin with, but changing or rewriting production code can be time-consuming and risky. This led me down the path of using a cross apply, so we only had to select the `CASE` statement once and then we could just call the column name at any time. So the updated code would be something like this:

```sql
SELECT pk.*
      ,fm.name as move_name
      ,fm.power
      ,fm.accuracy
      ,nw.new_col
      ,sqrt(nw.new_col) as new_col_sqrt
      -- ....
      -- ....
      -- MORE VERSIONS OF THE new_col 
FROM   pokemon.fact_pokemon pk
JOIN   pokemon.dim_moves mv
ON     pk.id = mv.id
AND    mv.version_group = 'diamond-pearl'
JOIN   pokemon.fact_moves fm
ON     mv.move = fm.name
AND    fm.version = 'diamond-pearl'
CROSS APPLY (
       SELECT CASE WHEN fm.move_type IN ('normal', 'grass')
                   THEN IFNULL(fm.power, 0) / (IFNULL(fm.accuracy, 1) * 1.0)
                   WHEN fm.move_type = 'flying'
                   THEN IFNULL(fm.power, 0) * 2 / (IFNULL(fm.accuracy, 1) * 1.0)
                   ELSE IFNULL(fm.power, 0) * 3 / (IFNULL(fm.accuracy, 1) * 1.0) END  AS new_col
) as nw
```

We now only have one `CASE` statement to update and the code becomes a lot easier to understand. Of course, with SQL we are always limited by the functions available to us with the SQL flavor we are using. Nonetheless, remembering something like this can help clean up code and prevent missing `END` keywords in `CASE` statements down the road.
