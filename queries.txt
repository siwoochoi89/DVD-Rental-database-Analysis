/*Query 1 - How many non-family-friendly rentals happen after 22:00, and how many happen before? What about family-friendly rentals ?*/

SELECT rental_day_part.day_part,
COUNT(rental_day_part.non_family_friendly) AS non_family_friendly,
COUNT(rental_day_part.family_friendly) AS family_friendly
FROM(SELECT CASE
	 WHEN category.name NOT IN ('Animation', 'Children', 'Classics', 'Comedy','Family', 'Music')
	 THEN 'Non-Family-Friendly'
	 END AS non_family_friendly,
	 CASE
	 WHEN category.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
	 THEN 'Family-Friendly'
	 END AS family_friendly,
	 CASE
	 WHEN DATE_PART('hour', rental.rental_date) < 22 THEN 'Early'
	 ELSE 'Late'
	 END AS day_part, rental.rental_id AS rentals
	 
FROM category
JOIN film_category
ON film_category.category_id = category.category_id
JOIN inventory
ON film_category.film_id = inventory.film_id
JOIN rental
ON inventory.inventory_id = rental.inventory_id) AS rental_day_part
GROUP BY 1
ORDER BY 1;


/*Query2 - What are the top categories that brought in the long rental average?*/

SELECT  c.name AS category_name , AVG(f.rental_duration) AS avg_rental
FROM category c
JOIN film_category fc
ON fc.category_id = c.category_id
JOIN film f
ON f.film_id = fc.film_id
JOIN inventory i
ON f.film_id = i.film_id
JOIN rental r
ON i.inventory_id = r.inventory_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;


/*Query 3 - Which category of non-family-friendly films are the most popular?*/

WITH t1 AS (SELECT f.title title, c.name category_name, r.rental_date AS rent_date
	FROM film f
	JOIN film_category fc
	ON f.film_id = fc.film_id
	JOIN inventory i
	ON f.film_id = i.film_id
	JOIN category c
	ON fc.category_id = c.category_id
	JOIN rental r
	ON i.inventory_id = r.inventory_id
	GROUP BY 1,2,3
	ORDER BY 1,2)
SELECT CASE WHEN category_name = 'Action' THEN 'Action'
WHEN category_name = 'Documentary' THEN 'Documentary'
WHEN category_name = 'Drama' THEN 'Drama'
WHEN category_name = 'Sci-Fi' THEN 'Sci-Fi'
WHEN category_name = 'Sports' THEN 'Sports'
ELSE 'Music' END AS category_name,
COUNT (*) AS rental_count
FROM t1
GROUP BY 1
ORDER BY 2 DESC


/*Query 4 - What are the most rental categories of the family movies?*/

WITH t1 AS (SELECT f.title AS film_title, c.name AS category_name, COUNT(r.rental_id) AS rental_count
	FROM category c
	JOIN film_category fc
	ON c.category_id = fc.category_id
	JOIN film f
	ON f.film_id = fc.film_id
	JOIN inventory i
	ON f.film_id = i.film_id
	JOIN rental r
	ON i.inventory_id = r.inventory_id
			
WHERE c.name LIKE 'Animation'
			OR c.name LIKE 'Children'
			OR c.name LIKE 'Classics'
			OR c.name LIKE 'Comedy'
			OR c.name LIKE 'Family'
			OR c.name LIKE 'Music'
			GROUP BY 1, 2)

SELECT Film_Title, category_Name, rental_count
FROM t1
GROUP BY category_Name, Film_Title, t1.rental_count
ORDER BY 2,1;
