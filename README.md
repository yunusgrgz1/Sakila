### 📌 Sakila Film Rental Reporting and Automated Recommendation System  

<img src="https://raw.githubusercontent.com/yunusgrgz1/Sakila/refs/heads/main/films.jpg" alt="Image" width="850" height="520">


- In this project, we will use **MySQL** to perform data analysis, examine **customer behavior** with advanced queries, conduct **performance optimizations**, and develop an **automated recommendation system**.


## **🔹 The Project Steps**  
- The dataset we will use is the **Sakila dataset**, which is open-source data representing a fictional movie rental company. We will carry out this project using this dataset.

<img src="https://raw.githubusercontent.com/yunusgrgz1/Sakila/refs/heads/main/sakila.png" alt="Image" width="850" height="520">


### **1️⃣ Customer Analysis and Segmentation ** 
- In this scenario, our marketing team requested a list of the top 10 customers who have spent the most. These customers will be categorized as premium customers and offered exclusive opportunities.

```sql
SELECT DISTINCT c.customer_id, c.first_name, c.last_name, SUM(p.amount) AS total_amount
FROM customer c 
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
order by total_amount desc
LIMIT 10;
```

- Answer
  
|customer_id|first_name|last_name|total_amount|
|-----------|----------|---------|------------|
|526        |KARL      |SEAL     |221.55      |
|148        |ELEANOR   |HUNT     |216.54      |
|144        |CLARA     |SHAW     |195.58      |
|137        |RHONDA    |KENNEDY  |194.61      |
|178        |MARION    |SNYDER   |194.61      |
|459        |TOMMY     |COLLAZO  |186.62      |
|469        |WESLEY    |BULL     |177.6       |
|468        |TIM       |CARY     |175.61      |
|236        |MARCIA    |DEAN     |175.58      |
|181        |ANA       |BRADLEY  |174.66      |

- Our sales team need the information of customers who are either below or above average in terms of spending. They need this data to reach out to the low-spending customers via email and SMS.

```sql
WITH customer_spending AS (
    SELECT 
        p.customer_id,
        SUM(p.amount) AS total_spent,
        COUNT(p.amount) AS payment_count,
        SUM(p.amount) / COUNT(p.amount) AS avg_spent
    FROM payment p
    GROUP BY p.customer_id
),
average_spending AS (
    SELECT AVG(total_spent) AS overall_avg_spent FROM customer_spending
)
select distinct
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,  
    cs.customer_id, 
    cs.total_spent,
    cs.avg_spent,
    c.email,   -- Müşteri e-postası
    ad.phone,  -- Müşteri telefon numarası
    (SELECT overall_avg_spent FROM average_spending) AS overall_avg_spent, 
    CASE
        WHEN cs.avg_spent > (SELECT overall_avg_spent FROM average_spending) THEN 'Above'
        WHEN cs.avg_spent < (SELECT overall_avg_spent FROM average_spending) THEN 'Below'
        ELSE 'Average'
    END AS spending_category
FROM customer_spending cs
JOIN customer c ON cs.customer_id = c.customer_id  
JOIN address ad ON c.address_id = ad.address_id 
JOIN payment p ON cs.customer_id = p.customer_id;  
```
- Answer

|customer_name   |customer_id|total_spent|avg_spent|email                              |phone       |overall_avg_spent|spending_category|
|----------------|-----------|-----------|---------|-----------------------------------|------------|-----------------|-----------------|
|LORI WOOD       |78         |141.69     |4.570645 |LORI.WOOD@sakilacustomer.org       |875756771675|112.53182        |Below            |
|RACHEL BARNES   |79         |84.78      |3.853636 |RACHEL.BARNES@sakilacustomer.org   |18581624103 |112.53182        |Below            |
|MARILYN ROSS    |80         |137.7      |4.59     |MARILYN.ROSS@sakilacustomer.org    |701457319790|112.53182        |Below            |
|ANDREA HENDERSON|81         |93.78      |4.262727 |ANDREA.HENDERSON@sakilacustomer.org|223664661973|112.53182        |Below            |
|KATHRYN COLEMAN |82         |130.74     |5.028462 |KATHRYN.COLEMAN@sakilacustomer.org |821972242086|112.53182        |Below            |
|LOUISE JENKINS  |83         |101.75     |4.07     |LOUISE.JENKINS@sakilacustomer.org  |800716535041|112.53182        |Below            |
|SARA PERRY      |84         |141.67     |4.29303  |SARA.PERRY@sakilacustomer.org      |48417642933 |112.53182        |Below            |
|ANNE POWELL     |85         |87.77      |3.816087 |ANNE.POWELL@sakilacustomer.org     |720998247660|112.53182        |Below            |
|JACQUELINE LONG |86         |148.67     |4.505152 |JACQUELINE.LONG@sakilacustomer.org |135117278909|112.53182        |Below            |
|WANDA PATTERSON |87         |145.7      |4.856667 |WANDA.PATTERSON@sakilacustomer.org |198123170793|112.53182        |Below            |
|BONNIE HUGHES   |88         |87.79      |4.180476 |BONNIE.HUGHES@sakilacustomer.org   |978987363654|112.53182        |Below            |

- We want to rank the most active customers in the last 6 months based on their spending..

```sql
SELECT 
    p.customer_id,
    concat (c.first_name, ' ', c.last_name) as customer_name,
    SUM(p.amount) AS total_spent 
FROM payment p
join customer c
on p.customer_id = c.customer_id
WHERE p.payment_date > '2005-06-01'
GROUP BY p.customer_id
ORDER BY total_spent desc
```
- Answer

|customer_id|customer_name |total_spent|
|-----------|--------------|-----------|
|148        |ELEANOR HUNT  |211.55     |
|526        |KARL SEAL     |208.58     |
|178        |MARION SNYDER |194.61     |
|137        |RHONDA KENNEDY|191.62     |
|144        |CLARA SHAW    |189.6      |
|459        |TOMMY COLLAZO |183.63     |
|181        |ANA BRADLEY   |167.67     |
|410        |CURTIS IRBY   |167.62     |
|236        |MARCIA DEAN   |166.61     |
|403        |MIKE WAY      |162.67     |

- Our Sales Manager needs to identify the employee with the highest sales and declare them Employee of the Month.

```sql
select s.staff_id, concat (s.first_name, ' ', s.last_name), sum(p.amount) as total_sales
from staff s
join payment p on s.staff_id = p.staff_id
group by s.staff_id
order by total_sales desc
limit 1;
```
- Answer

|staff_id|concat (s.first_name, ' ', s.last_name)|total_sales|
|--------|---------------------------------------|-----------|
|2       |Jon Stephens                           |33,924.06  |


- Our marketing team needs to obtain a list of customers who are eligible for the $30 gift card campaign for purchases over $150.

```sql
SELECT 
    p.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    SUM(p.amount) AS total_spent,
    CASE 
        WHEN SUM(p.amount) > 150 THEN 'Eligible'
        ELSE 'Not Eligible'
    END AS reward_status
FROM 
    payment p
JOIN 
    customer c ON p.customer_id = c.customer_id
WHERE 
    p.payment_date BETWEEN '2005-01-01' AND '2005-12-31'
GROUP BY 
    p.customer_id
HAVING 
    reward_status = 'Eligible'
ORDER BY 
    total_spent DESC;
```
- Answer

|customer_id|customer_name |total_spent|reward_status|
|-----------|--------------|-----------|-------------|
|526        |KARL SEAL     |221.55     |Eligible     |
|148        |ELEANOR HUNT  |216.54     |Eligible     |
|144        |CLARA SHAW    |195.58     |Eligible     |
|137        |RHONDA KENNEDY|194.61     |Eligible     |
|178        |MARION SNYDER |189.62     |Eligible     |
|459        |TOMMY COLLAZO |186.62     |Eligible     |
|469        |WESLEY BULL   |177.6      |Eligible     |
|468        |TIM CARY      |175.61     |Eligible     |
|236        |MARCIA DEAN   |174.59     |Eligible     |
|176        |JUNE CARROLL  |173.63     |Eligible     |
|181        |ANA BRADLEY   |171.67     |Eligible     |
|50         |DIANE COLLINS |169.65     |Eligible     |


### 2️⃣ Film Popülerlik ve Kiralama Eğilimleri

- We need to know which films or categories are more popular.
- We are listing the most rented films by category.

```sql
select f.film_id, f.title as film_name, c.name as category, count(r.inventory_id) as number_of_rentals
from film f
join film_category fc
on f.film_id = fc.film_id
join category c
on c.category_id = fc.category_id
join inventory i
on f.film_id = i.film_id
join rental r
on i.inventory_id = r.inventory_id 
group by f.title , f.film_id, c.name
order by  number_of_rentals desc
```
- Answer

|film_id|film_name          |category   |number_of_rentals|
|-------|-------------------|-----------|-----------------|
|103    |BUCKET BROTHERHOOD |Travel     |34               |
|738    |ROCKETEER MOTHER   |Foreign    |33               |
|382    |GRIT CLOCKWORK     |Games      |32               |
|767    |SCALAWAG DUCK      |Music      |32               |
|489    |JUGGLER HARDLY     |Animation  |32               |
|730    |RIDGEMONT SUBMARINE|New        |32               |
|331    |FORWARD TEMPLE     |Games      |32               |
|891    |TIMBERLAND SKY     |Classics   |31               |
|621    |NETWORK PEAK       |Family     |31               |
|418    |HOBBIT ALIEN       |Drama      |31               |
|973    |WIFE TURN          |Documentary|31               |
|753    |RUSH GOODFELLAS    |Family     |31               |


- We are analyzing the most popular film genres.
  
```sql

SELECT 
    c.name AS category, 
    COUNT(r.rental_id) AS rental_count
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
GROUP BY c.name
ORDER BY rental_count DESC;
```
- Answer


  
|category   |rental_count|
|-----------|------------|
|Sports     |1,179       |
|Animation  |1,166       |
|Action     |1,112       |
|Sci-Fi     |1,101       |
|Family     |1,096       |
|Drama      |1,060       |
|Documentary|1,050       |
|Foreign    |1,033       |
|Games      |969         |
|Children   |945         |
|Comedy     |941         |
|New        |940         |
|Classics   |939         |
|Horror     |846         |
|Travel     |837         |
|Music      |830         |


- Is there a correlation between customers' rental frequency and their spending habits?
- Do customers who rent more frequently also spend more, or do customers who rent less still make high-value payments?


```sql
SELECT 
    c.customer_id, 
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(r.rental_id) AS rental_count, 
    SUM(p.amount) AS total_spent, 
    ROUND(SUM(p.amount) / COUNT(r.rental_id), 2) AS avg_spent_per_rental  
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, customer_name
ORDER BY rental_count DESC;  
```
- Answer

|customer_id|customer_name  |rental_count|total_spent|avg_spent_per_rental|
|-----------|---------------|------------|-----------|--------------------|
|148        |ELEANOR HUNT   |2,116       |9,960.84   |4.71                |
|526        |KARL SEAL      |2,025       |9,969.75   |4.92                |
|236        |MARCIA DEAN    |1,764       |7,374.36   |4.18                |
|144        |CLARA SHAW     |1,764       |8,214.36   |4.66                |
|75         |TAMMY SANDERS  |1,681       |6,379.19   |3.79                |
|469        |WESLEY BULL    |1,600       |7,104      |4.44                |
|197        |SUE PETERS     |1,600       |6,184      |3.87                |
|137        |RHONDA KENNEDY |1,521       |7,589.79   |4.99                |
|468        |TIM CARY       |1,521       |6,848.79   |4.5                 |
|178        |MARION SNYDER  |1,521       |7,589.79   |4.99                |
|5          |ELIZABETH BROWN|1,444       |5,495.56   |3.81                |
|459        |TOMMY COLLAZO  |1,444       |7,091.56   |4.91                |

### **3️⃣ Developing a Recommendation System **

- We are providing 5 recommended films from each of the top three categories most rented by a customer, ensuring that these films have not been rented by the customer before.

```sql
WITH top_categories AS (
    -- The 3 categories most rented by the customer
    SELECT 
        c.name AS category,
        COUNT(r.rental_id) AS rental_count
    FROM rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    WHERE r.customer_id = 1  
    GROUP BY c.name
    ORDER BY rental_count DESC
    LIMIT 3  -- En çok kiralanan 3 kategori
),
-- Recommend 5 films from each category
recommended_films AS (
    SELECT 
        f.title AS recommended_film,
        c.name AS category,
        ROW_NUMBER() OVER (PARTITION BY c.name ORDER BY f.title) AS row_num  -- Sorting for each category
    FROM film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    WHERE c.name IN (SELECT category FROM top_categories)
    AND f.film_id NOT IN (
        SELECT f.film_id
        FROM rental r
        JOIN inventory i ON r.inventory_id = i.inventory_id
        JOIN film f ON i.film_id = f.film_id
        WHERE r.customer_id = 1  -- Exclude the films that Customer 1 has previously rented.
    )
)
-- 3. Filter the results and recommend only 5 films from each category.
SELECT recommended_film, category
FROM recommended_films
WHERE row_num <= 5  -- Her kategoriden 5 film seç
ORDER BY category, row_num;

```
- Answer

|recommended_film   |category|
|-------------------|--------|
|ALICE FANTASIA     |Classics|
|ARIZONA BANG       |Classics|
|BEAST HUNCHBACK    |Classics|
|BOUND CHEAPER      |Classics|
|CANDIDATE PERDITION|Classics|
|AIRPLANE SIERRA    |Comedy  |
|ANTHEM LUKE        |Comedy  |
|BRINGING HYSTERICAL|Comedy  |
|CAPER MOTIONS      |Comedy  |
|CAT CONEHEADS      |Comedy  |
|APOLLO TEEN        |Drama   |
|BEAUTY GREASE      |Drama   |
|BEETHOVEN EXORCIST |Drama   |
|BLADE POLISH       |Drama   |
|BRIGHT ENCOUNTERS  |Drama   |


### **4️⃣ Creating Stored Procedure ve Trigger ** 

- We realized that we frequently need to learn the most popular category for many customers, so we are creating a **Stored Procedure** function specifically for this purpose

```sql
DELIMITER $$

CREATE PROCEDURE GetMostPopularCategory (IN p_customer_id INT)
BEGIN
    SELECT c.name AS category
    FROM category c
    JOIN film_category fc ON c.category_id = fc.category_id
    JOIN film f ON f.film_id = fc.film_id
    JOIN inventory i ON f.film_id = i.film_id
    JOIN rental r ON i.inventory_id = r.inventory_id
    WHERE r.customer_id = p_customer_id
    GROUP BY c.name
    ORDER BY COUNT(r.rental_id) DESC
    LIMIT 1;
END$$
```

- Şimdi bu prosedürü 1 numralı müşteri için çağırıyoruz
  
```sql
CALL GetMostPopularCategory(1);
```
- Answer

|category|
|--------|
|Classics|

- Each customer can rent a maximum of 10 films per year, so we want to create a trigger to detect customers who exceed this limit.

```sql
DELIMITER $$

CREATE TRIGGER CheckRentalLimit
AFTER INSERT ON rental
FOR EACH ROW
BEGIN
    DECLARE rental_count INT;
    DECLARE rental_limit INT DEFAULT 10; -- Limit

    -- We count the rentals made by the user within a specific date range.
    SELECT COUNT(*) INTO rental_count
    FROM rental
    WHERE customer_id = NEW.customer_id
      AND rental_date BETWEEN '2025-01-01' AND '2025-12-31'; -- Example date range for the year 2005. 

    -- If the rental count exceeds the limit, we add a record to the rental_history table.
    IF rental_count > rental_limit THEN
        INSERT INTO rental_history (customer_id, rental_id, message)
        VALUES (NEW.customer_id, NEW.rental_id, 'Rental limit exceeded!');
    END IF;
END$$

DELIMITER ;
```


- The End





