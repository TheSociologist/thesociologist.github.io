---
layout: post
title:  "Ordered list Representation in Postgres"
date:   2022-1-20 3:48:25 -0500
categories: article
---

Representing ordered lists in relational databases is surprisingly non-trivial. Relational stores are intended to store and process set data which has no inherent ordering. This makes user-defined ordering somewhat difficult (e.g. drag and drop lists). 

The most naive approach is to add an index integer column. Queries are relatively straightforward, simply order on the column. But updates become nightmarish as you have to update every row's index that comes after the location of an element. In the worst case (when a user moves an item to the beginning of a list) this is every item in the list. 

There are some performance enhancements that can help here. For example, if you increase the interval between items (0, 100, 200, ...) you create new indices that you can insert into without having to update extra entries. For example, if you'd like to move an item into the second index (between items with indices 0 and 100) all you have to do is find the midpoint and round (0 + 100) / 2 = 50. As you can probably guess, this will eventually fail and you'll start getting collisions. At this point, you have two options, enter the wonderful world of floating point or rebalance the entire representation. The latter is not ideal and the former is...messy. And it's not even really a solution since you will exhaust your data store's precision at some point leading back to the rebalancing requirement. There are some interesting approaches with rational numbers and fractional representations outlined in this [post](https://begriffs.com/posts/2018-03-20-user-defined-order.html). However, adding an extension is not always possible and there's still something offputting about storing increasingly smaller fractions. 

On the very far end of this approach is LexoRank developed by Atlassian for Jira. Here's a [video](https://www.youtube.com/watch?v=OjQv9xMoFbg) and [article](https://medium.com/whisperarts/lexorank-what-are-they-and-how-to-use-them-for-efficient-list-sorting-a48fc4e7849f) detailing this approach. There's a couple implementations out there for languages including [Typescript](https://github.com/kvandake/lexorank-ts), [Go](https://github.com/xissy/lexorank), [C#](https://github.com/kvandake/lexorank-dotnet), and [Ruby](https://github.com/DevStarSJ/LexoRank), and even [PL/pgSQL](https://gist.github.com/lukeramsden/de956a2bf2c9c8bb9d091e6ffeb38dd0). If you're dead set on using the most normalized ordering mechanism this is likely the best you can do. It still requires occasionally rebalancing but its not remotely as bad as our original naive approach. 

There's also an approach involving linked lists but this requires recursive CTE queries which are not particularly performant at scale. 

But what if you don't care about normalization? I've been experimenting with using materialized integer arrays to store positions and it seems to work relatively well. While this is far from best practices when it comes to database design this approach scales somewhat well, requires no rebalancing, and queries are pretty fast too (given some constraints). You will lose out on referential integrity, so you need to pay very close attention to making sure all table storing positional information in arrays are up to date. 

Here's a high-level implementation of this solution. Let's assume we're developing a system like [Airtable](https://airtable.com). We have a table that contains a bunch of rows. This table also has a bunch of views that store different orderings of those rows. That's three Postgres tables, one for the table, one for the view, and one for the row. 

```
create table tables (
    id serial primary key,
    name text
);

create table rows (
    id serial primary key,
    table_id int references tables on delete cascade
    /* pretend there are more columns here */
);

create table views (
    id serial primary key,
    name text,
    table_id int references tables on delete cascade,
    row_ids int[]
);
```

Here are the basic actions:

## Insert
Assume we've created the row with the correct table_id and we just need to add it to each of the views.
### At the start:
```
update 
views
set row_ids = <new_row_id> || row_ids
where
table_id = <table_id>;
``` 
### At the end:
```
update 
views
set row_ids = row_ids || <new_row_id>
where
table_id = <table_id>;
``` 
### At a particular index:
```
update 
views
set row_ids = row_ids[:<index> - 1] || <new_row_id> || row_ids[<index>:]
where
table_id = <table_id>;
``` 

## Delete
Assume we've already deleted the entry and we just need to remove it from the views.

```
update
views
set row_ids = array_remove(row_ids, <deleted_row_id>)
where
table_id = <table_id>;
```

## Reorder
Okay, now onto the actual purpose of this post. There's probably a way to do this in a single query but I'll just use a delete and insert within a transaction for simplicity. 

```
BEGIN;

update
views
set row_ids = array_remove(row_ids, <deleted_row_id>)
where
table_id = <table_id>;

update 
views
set row_ids = row_ids[:<new_index> - 1] || <moved_row_id> || row_ids[<new_index>:]
where
table_id = <table_id>;

COMMIT;
```

## Read
This is tougher and somewhat slower than with the order column approach. We can leverage *with ordinality* to unnest the elements in the row_ids array for a given view, associate the row_ids with the rows and then order by the ordinal index. 
```
select
*
from
rows
inner join (
    select 
	a.elem as row_id, 
	a.nr
	from 
	views v, 
	unnest(v.row_ids) WITH ORDINALITY a(elem, nr)
	where v.id = <view_id>
) view on view.row_id = rows.id;
```
Based on my **extremely** limited testing with 50,000 rows in a table (the maximum allowed in an Airtable Base), this doubles the query time compared to regular select on the rows with an order column. However the difference is negligble when selecting slices of the list. If you're paginating this list whatsoever the query times are brought down to almost the same level as the vanilla queries. I'd consider that to be a relatively solid tradeoff between read and write speeds. 

*Side Note:* I personally use GraphQL, which as you might know uses cursor-based pagination i.e. select the next 100 after the item with this id instead of select the next 100 items after this offset number. The latter is congruent with the pagination approach I mentioned above. For cursor based pagination, you need to lookup the index of the item in the list (which is trivial assuming you've installed the intarray extension and your row ids are integers) and use this as the start index for the array slice. Beware: idx returns 0 if the item is no longer in the list which means you could end up selecting the first 100 entries in your list again. 
```
select
*
from
rows
inner join (
    select 
	a.elem as row_id, 
	a.nr
	from 
	views v, 
	unnest(v.row_ids[idx(<row_id>):idx(<row_id>) + (<count> - 1)]) WITH ORDINALITY a(elem, nr)
	where v.id = <view_id>
) view on view.row_id = rows.id;
```