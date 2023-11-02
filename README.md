# SQL

## Connection

```bash
psql -U postgres -h 127.0.0.1 -p 5432 -d
```

## Basic Commands

1. Show Databases
```plain
dvdrental=# \l

                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 dvdrental | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
(4 rows)
```

2. Show tables
```plain
dvdrental=# \dt

             List of relations
 Schema |     Name      | Type  |  Owner   
--------+---------------+-------+----------
 public | actor         | table | postgres
 public | address       | table | postgres
 public | category      | table | postgres
 public | city          | table | postgres
 public | country       | table | postgres
 public | customer      | table | postgres
 public | film          | table | postgres
 public | film_actor    | table | postgres
 public | film_category | table | postgres
 public | inventory     | table | postgres
 public | language      | table | postgres
 public | payment       | table | postgres
 public | rental        | table | postgres
 public | staff         | table | postgres
 public | store         | table | postgres
(15 rows)
```

## Challenges

### Do we have actors in the actor table that share the full name and if yes display those shared names.

```SQL
SELECT  a1.actor_id,a1.first_name,a1.last_name from
actor as a1 JOIN actor as a2
ON
  a1.first_name = a2.first_name and a1.last_name = a2.last_name and a1.actor_id <> a2.actor_id;
```

### Display the customer names that share the same address (e.g. husband and wife).

```sql
select c1.first_name,c1.last_name,c2.first_name,c2.last_name FROM
customer c1 JOIN customer c2
ON
  c1.customer_id <> c2.customer_id and c1.address_id = c2.address_id
```

### Display the total amount payed by all customers in the payment table.

```sql
SELECT first_name, last_name, sum(amount) as Total
FROM
customer c join payment p
on
  c.customer_id = p.customer_id
GROUP BY c.customer_id
ORDER BY Total DESC
```

### What is the movie(s) that was rented the most

Tripple join method (less efficient)

```sql
SELECT f.title,count(*) as times_rented FROM
rental r join inventory i
ON
  r.inventory_id = i.inventory_id
join film f
ON
  i.film_id = f.film_id
GROUP BY f.film_id
ORDER BY times_rented DESC
LIMIT 1
```

Sub command method

```sql
SELECT title FROM film
WHERE film_id IN
  (SELECT film_id FROM
  rental r join inventory i
  ON
    r.inventory_id = i.inventory_id
  GROUP BY film_id
  HAVING count(film_id) IN
    (SELECT count(*) FROM
    rental r join inventory i
    ON
      r.inventory_id = i.inventory_id
    GROUP BY film_id
    ORDER BY count(*) DESC limit 1))
```

### Which movies have not been rented so far

```sql
SELECT title from film
WHERE film_id NOT IN 
  (SELECT DISTINCT film_id
  from rental r
  join inventory i
  ON r.inventory_id = i.inventory_id)

```

### Show the number of movies each actor acted in

```sql
SELECT concat(a.first_name,' ', a.last_name) as actor, count(film_id) as films_acted_in FROM
actor a LEFT JOIN film_actor fa
ON
  a.actor_id = fa.actor_id
GROUP BY a.actor_id
ORDER BY a.first_name
```

### Display the names of the actors that acted in more than 20 movies

```sql
SELECT concat(a.first_name,' ', a.last_name) as actor, count(film_id) as films_acted_in FROM
actor a LEFT JOIN film_actor fa
ON
  a.actor_id = fa.actor_id
GROUP BY a.actor_id
HAVING count(film_id) > 20
ORDER BY a.first_name
```

### How many actors have 8 letters only in their first_names

```sql
SELECT count(*) FROM actor
WHERE length(first_name) = 8
```

### For all the movies rated “PG” show me the movie and the number of times it got rented

```sql

SELECT f.title as title, count(f.film_id) as Count, rating FROM
inventory i JOIN rental r
ON i.inventory_id = r.inventory_id
JOIN film f
ON i.film_id = f.film_id
WHERE f.rating = 'PG'
GROUP BY f.film_id
```