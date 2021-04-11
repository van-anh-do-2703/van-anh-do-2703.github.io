## Udacity's SQL Chinook Music Database

The SQL queries utilized in the making of this project contains multiple aggregation and sub-queries. The four main questions that were being explored and the SQL queries for each respective question can be found below:
1. Who spent on the most-earning artist?
```sql
WITH rich_artist AS (
    SELECT
        a.Name artists,
        COUNT(*) products,
        il.UnitPrice unit_price,
        COUNT(*) * il.UnitPrice amount_spent
    FROM
        Artist a
        JOIN Album al ON a.ArtistId = al.ArtistId
        JOIN Track t ON al.AlbumId = t.AlbumId
        JOIN InvoiceLine il ON t.TrackId = il.TrackId
    GROUP BY
        a.Name
    ORDER BY
        COUNT(*) * il.UnitPrice DESC
    LIMIT
        1
)
SELECT
    c.CustomerId customer_id,
    a.name artists,
    c.FirstName || ' ' || c.LastName full_name,
    COUNT(*) * il.UnitPrice amount_spent
FROM
    Customer c
    JOIN Invoice i ON c.CustomerId = i.CustomerId
    JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
    JOIN Track t ON il.TrackId = t.TrackId
    JOIN Album al ON al.AlbumId = t.AlbumId
    JOIN Artist a ON a.ArtistId = al.ArtistId
WHERE
    a.name = (
        SELECT
            artists
        FROM
            rich_artist
    )
GROUP BY
    c.CustomerId,
    c.FirstName,
    c.LastName
ORDER BY
    amount_spent DESC
LIMIT 10;
```
2. What are the most music genre for each country?
```sql
WITH t1 AS (
    SELECT
        c.Country country,
        g.Name genres,
        COUNT(*) purchases
    FROM
        Customer c
        JOIN Invoice i ON c.CustomerId = i.CustomerId
        JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
        JOIN Track t ON t.TrackId = il.TrackId
        JOIN Genre g ON t.GenreId = g.GenreId
    GROUP BY
        g.Name,
        c.Country
    ORDER BY
        c.Country
),
t2 AS (
    SELECT
        country,
        genres,
        MAX(purchases) purchases
    FROM
        t1
    GROUP BY
        country
    ORDER BY
        country
)
SELECT
    t1.country country,
    t1.genres genres,
    t2.purchases purchases
FROM
    t1
    JOIN t2 ON t1.country = t2.country
WHERE
    t1.purchases = t2.purchases
GROUP BY
    t1.country,
    t1.genres;
```
3. What are the numbers of orders per month?
```sql
SELECT
    STRFTIME('%Y-%m', i.InvoiceDate) invoice_date,
    SUM(il.Quantity)
FROM
    Invoice i
    JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
GROUP BY
    STRFTIME('%Y-%m', i.InvoiceDate);
```
4. What are the number of invoices and money spent in each country?
```sql
SELECT
    i.BillingCountry,
    COUNT(*) Invoices,
	SUM(il.UnitPrice * il.Quantity) Total_Price
FROM
    Invoice i
JOIN 
	InvoiceLine il
ON 
	i.InvoiceId = il.InvoiceId
GROUP BY
    1
ORDER BY
    3 DESC;
```