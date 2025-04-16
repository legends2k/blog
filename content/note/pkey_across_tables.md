+++
title = "Primary Key across Tables"
description = "Unique IDs across different tables"
date = "2023-06-14T17:34:00+05:30"
tags = ["tech", "database"]
toc = true
+++

# Problem

At times we might need uniqueness constraint for a common column across tables.  Let’s say we’ve a bunch of products a retailer sells e.g. books, toys and dresses.  Each item-kind needs its own table as they’ll have different inherent properties e.g. {author, pages} for books, {weight, colour} for toys and {material, fit} for dress.

We’ve two requirements business-wise:

1. A unique item code is needed to identify every product
2. All of these items show up on the same invoice

If we adhere to [third normal form][3nf], we’ll have tables like

| Books  |
|--------|
| ID     |
| Name   |
| Author |

| Toys   |
|--------|
| ID     |
| Weight |
| Colour |

| Dresses  |
|----------|
| ID       |
| Material |
| Fit      |

Here each `ID` field is a primary key of the respective table.

This has two problems:

1. Non-unique IDs for items
2. Proliferation of tables

(1) because both a book and a toy can be of same ID as primary key is per table.  I’ll show you can example to understand (2)

To capture details of an invoice, we’ll have one table for the invoice itself:

| Invoice     |
|-------------|
| ID          |
| Timestamp   |
| Customer ID |

Additionally, we need _n_ (mapping) tables for _n_ kinds of products just to store one invoice:

| Invoice_Books |
|---------------|
| Invoice.ID    |
| Books.ID      |

| Invoice_Toys |
|--------------|
| Invoice.ID   |
| Toys.ID      |

| Invoice_Dresses |
|-----------------|
| Invoice.ID      |
| Dresses.ID      |

Here `Books.ID`, `Toys.ID`, etc. are foreign keys referencing respective tables’ primary key.  This isn’t a mistake, but this is what you end up with if you follow [3NF][][^1].

It’d be ideal if we can just have 2 tables: one for the invoice itself and another mapping an invoice and an item.

| Invoice_Items |
|---------------|
| Invoice.ID    |
| Item.ID       |

where `Item.ID` is a foreign key referencing a unique product ID irrespective of the product kind.

# Trial 1: Table Inheritance

[PostgreSQL supports inheritance][1].  The idea is to have a base table with primary key constraint and have child tables.

``` sql
CREATE TABLE items (
  id bigint PRIMARY KEY
);

CREATE TABLE books (
  LIKE items INCLUDING INDEXES,
  author VARCHAR(32) NOT NULL
) INHERITS (items);

CREATE TABLE toys (
  LIKE items INCLUDING INDEXES,
  colour VARCHAR(16) DEFAULT 'TRANSPARENT' NOT NULL
) INHERITS (items);

INSERT INTO books VALUES (1, 'Kalki');
INSERT INTO toys VALUES (2, 'Saffron');

SELECT tableoid::regclass table, * FROM items;
-- Alternative command:
SELECT p.relname table, * FROM items i JOIN pg_class p ON i.tableoid = p.oid;

 table | id 
-------+----
 books |  1
 toys  |  2

```

Before we declare victory, let’s try one more thing:

``` sql

INSERT INTO books VALUES (2, 'JKR');

SELECT tableoid::regclass, * FROM items;

 tableoid | id 
----------+----
 books    |  1
 books    |  2
 toys     |  2
```

Yikes!  We’ve a book and a toy with same id `2`.  In fact, this is called out in the [manual][2] (thanks to [this SO post][3]):

> A serious limitation of the inheritance feature is that indexes (including unique constraints) and foreign key constraints only apply to single tables, not to their inheritance children. This is true on both the referencing and referenced sides of a foreign key constraint.

In other words, the primary key constraint on `items`, `books` and `toys` are completely disjoint.

# Trial 2: Common table for IDs

Let’s create a table just for IDs with primary key constraint.  All product-kind tables will have an ID column referencing this ID (foreign key).

``` sql
CREATE TABLE items (
  id bigint PRIMARY KEY
);

CREATE TABLE books (
  id bigint PRIMARY KEY REFERENCES items (id),
  author VARCHAR(32) NOT NULL
);

CREATE TABLE toys (
  id bigint PRIMARY KEY REFERENCES items (id),
  colour VARCHAR(16) DEFAULT 'TRANSPARENT' NOT NULL
);

INSERT INTO items VALUES (1);
INSERT INTO items VALUES (2);

INSERT INTO books VALUES (1, 'Kalki');
INSERT INTO toys VALUES (2, 'Saffron')
INSERT INTO books VALUES (2, 'DMR');
```

OK, this didn’t help as expected.  The foreign key constraint, naturally, doesn’t stop us from reusing a product ID of a different kind.

# Trial 3: Common table for IDs with subtype

Let’s introduce a subtype column to ensure product ID of a subtype is not reused for a different subtype using a composite primary key.  We’ll use this as foreign key in product-kind tables.  For this example, the subtype is just a one letter code specifying the type; it can be more complex.

``` sql
CREATE TABLE items (
  id bigint,
  -- subtype should be a book or a toy
  subtype VARCHAR(1) NOT NULL CHECK (subtype in ('b', 't')),
  PRIMARY KEY (id, subtype)
);

CREATE TABLE books (
  id bigint PRIMARY REFERENCES items (id),
  subtype VARCHAR(1) NOT NULL DEFAULT 'b' CHECK (subtype = 'b'),
  author VARCHAR(32) NOT NULL,
  FOREIGN KEY (id, subtype) REFERENCES items (id, subtype)
);

CREATE TABLE books (
  id bigint PRIMARY KEY REFERENCES items (id),
  subtype VARCHAR(1) NOT NULL DEFAULT 'b' CHECK (subtype = 'b'),
  colour VARCHAR(16) NOT NULL DEFAULT 'TRANSPARENT',
  FOREIGN KEY (id, subtype) REFERENCES items (id, subtype)
);

INSERT INTO items VALUES (1, 'b');
INSERT INTO items VALUES (2, 't');
INSERT INTO items VALUES (2, 'b');
```

🤦 The same ID can be associated to both a book and a toy, again!  The composite primary key factors all columns to deduce uniqueness and here it isn’t violated.

However, this trial is very close to the solution.  The line of attack on the problem with a foreign key from the product-kind table is a good one; it ensures an ID is bound to a type i.e. it solves the problem trial 2 had.

# Solution: Trial 3 + separate primary key and unique constraints

We’ve two issues to be sorted:

1. Uniqueness of IDs irrespective of product kind
    - Inserting `(1, t)` when there’s a `(1, b)` shouldn’t work
2. Non-reuse of a taken ID of another subtype
    - Using `2` for a book when there’s a `(2, t)` shouldn’t work

We tackle (1) with ID primary key.  When there a row with `ID = 1`, irrespective of the `subtype`, creating a row with same ID will fail; straight forward.

We tackle (2) with two constraints: one on base table, one on product-kind table; let’s see an example:

``` sql
CREATE TABLE items (
  id bigint PRIMARY KEY,
  subtype VARCHAR(1) NOT NULL CHECK (subtype in ('b', 't')),
  UNIQUE (id, subtype)
);

CREATE TABLE books (
  id bigint PRIMARY KEY REFERENCES items(id),
  subtype VARCHAR(1) NOT NULL DEFAULT 'b' CHECK(subtype = 'b'),
  author VARCHAR(32) NOT NULL,
  FOREIGN KEY (id, subtype) REFERENCES items (id, subtype)
);

CREATE TABLE toys (
  id bigint PRIMARY KEY REFERENCES items(id),
  subtype VARCHAR(1) NOT NULL DEFAULT 't' CHECK(subtype = 't'),
  colour VARCHAR(16) NOT NULL DEFAULT 'TRANSPARENT',
  FOREIGN KEY (id, subtype) REFERENCES items (id, subtype)
);

INSERT INTO items VALUES (1, 'b');
INSERT INTO items VALUES (2, 't');

INSERT INTO books (id, author) VALUES (1, 'Kalki');
INSERT INTO books (id, author) VALUES (2, 'DMR');

ERROR:  23503: insert or update on table "books" violates foreign key constraint "books_id_subtype_fkey"
DETAIL:  Key (id, subtype)=(2, b) is not present in table "items".
```

Yes!  Trying to use `(2, b)` won’t work because the same `(2, b)` is expected in `items`, thanks to the foreign key constraint on `books`.  Trying to add `(2, b)` into `items` will be stopped by its primary key constraint.

What use is `items`’s `UNIQUE (id, subtype)` constraint?  It’s needed to exist for `books`’s foreign key constraint to be created.  [Refer here][4] for a good explanation why.

Added advantage of `subtype`: simple kind-based selection

``` sql
SELECT id FROM items WHERE subtype='b';
```

# Table Proliferation Tackled Too

This also also our table proliferation problem.  We don’t need _n_ tables to represent invoices; we just need two: one for the invoice itself and one for its items.

``` sql
CREATE TABLE invoice_items (
  invoice_id bigint NOT NULL REFERENCES invoices (id),
  item_id bigint NOT NULL REFERENCES items (id),
  count smallint NOT NULL DEFAULT 1,
  PRIMARY KEY (invoice_id, item_id)
);
```

If we items of a particular kind in an invoice, it’s just a `SELECT` away:

``` sql
-- Select all book entries in invoice # 124
SELECT t1.item_id FROM invoice_items t1 JOIN items t2 ON t1.item_id = t2.id AND t2.subtype = 'b' WHERE invoice_id = 123;
```

# Other Considerations

[Global serial generators are generally discouraged][5].  This isn’t what we’re after though; we want common serials for objects of different kinds but all belonging to a common, super-class.  This led me to think of inheritance but didn’t get far with it.

I did look at other options to avoid the table proliferation problem alone.  Here’re the ones I rejected

1. Table Per Type (TPT)
    - This is our inheritance solution
    - It solves table proliferation problem but not the _unique ID across tables_ problem
2. Table Per Hierarchy (TPH)
    - A column for every kind with just one filled (rest null)
    - e.g. `invoice_items` will have columns `book_id`, `toy_id`, `dress_id`, etc. with just one filled per row
    - Many redundant columns!  Bad!!
    - What if a new kind of item comes up?  Add more columns!  Eww!!
3. Table Per Concrete (TPC)
    - Solves neither table proliferation nor unique ID across tables problem

Refer [Model Inheritance in a DB][6] and [How can you represent inheritance in a database?][8] StackOverflow posts for details.

# References

1. [Inheritance - PostgreSQL Manual][1]
2. [Inheritance: Caveats - PostgreSQL Manual][2]
3. [Violation of uniqueness in primary key when using inheritance - StackOverflow][3]
4. [Why do composite foreign keys need a separate unique constraint?][4]
5. [Sharing a single primary key sequence across a database?][5]
6. [How do you effectively model inheritance in a database?][6]
7. [Super-type and Sub-types arrangement - StackOverflow][7] - My inspiration but not solution
8. [How can you represent inheritance in a database?][8]
9. [Foreign Key to multiple tables][9]

# Appendix: Tables Definitions

``` bash
# \d items
                      Table "public.items"
 Column  |         Type         | Collation | Nullable | Default 
---------+----------------------+-----------+----------+---------
 id      | bigint               |           | not null | 
 subtype | character varying(1) |           | not null | 
Indexes:
    "items_pkey" PRIMARY KEY, btree (id)
    "items_id_subtype_key" UNIQUE CONSTRAINT, btree (id, subtype)
Check constraints:
    "items_subtype_check" CHECK (subtype::text = ANY (ARRAY['b'::character varying, 't'::character varying]::text[]))
Referenced by:
    TABLE "books" CONSTRAINT "books_id_fkey" FOREIGN KEY (id) REFERENCES items(id)
    TABLE "books" CONSTRAINT "books_id_subtype_fkey" FOREIGN KEY (id, subtype) REFERENCES items(id, subtype)
    TABLE "toys" CONSTRAINT "toys_id_fkey" FOREIGN KEY (id) REFERENCES items(id)
    TABLE "toys" CONSTRAINT "toys_id_subtype_fkey" FOREIGN KEY (id, subtype) REFERENCES items(id, subtype)

# \d books
                              Table "public.books"
 Column  |         Type          | Collation | Nullable |        Default         
---------+-----------------------+-----------+----------+------------------------
 id      | bigint                |           | not null | 
 subtype | character varying(1)  |           | not null | 'b'::character varying
 author  | character varying(32) |           | not null | 
Indexes:
    "books_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "books_subtype_check" CHECK (subtype::text = 'b'::text)
Foreign-key constraints:
    "books_id_fkey" FOREIGN KEY (id) REFERENCES items(id)
    "books_id_subtype_fkey" FOREIGN KEY (id, subtype) REFERENCES items(id, subtype)

# \d toys
                                    Table "public.toys"
 Column  |         Type          | Collation | Nullable |             Default              
---------+-----------------------+-----------+----------+----------------------------------
 id      | bigint                |           | not null | 
 subtype | character varying(1)  |           | not null | 't'::character varying
 colour  | character varying(16) |           | not null | 'TRANSPARENT'::character varying
Indexes:
    "toys_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "toys_subtype_check" CHECK (subtype::text = 't'::text)
Foreign-key constraints:
    "toys_id_fkey" FOREIGN KEY (id) REFERENCES items(id)
    "toys_id_subtype_fkey" FOREIGN KEY (id, subtype) REFERENCES items(id, subtype)
```


[1]: https://www.postgresql.org/docs/15/ddl-inherit.html
[2]: https://stackoverflow.com/q/56637251/183120
[3]: https://www.postgresql.org/docs/15/ddl-inherit.html#DDL-INHERIT-CAVEATS
[4]: https://dba.stackexchange.com/a/153395
[5]: https://dba.stackexchange.com/q/50652/259438
[6]: https://stackoverflow.com/q/190296/183120
[7]: https://stackoverflow.com/a/27415430/183120
[8]: https://stackoverflow.com/q/3579079/183120
[9]: https://stackoverflow.com/q/7844460/183120
[3nf]: https://en.wikipedia.org/wiki/Third_normal_form
[nf-examples]: https://learn.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description

[^1]: 1NF = no list in a cell, 2NF = avoid groups of redundant data by moving them to separate tables and linking them, 3NF = move data not dependent on key to its own table; normal forms beyond these aren’t very practical.  [MSDN article with examples][nf-examples].
