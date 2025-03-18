### **📌 Sakila Film Kiralama İçin  Raporlama ve Otomatik Öneri Sistemi **  

<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">


- Bu projede **MySQL** kullanarak veri analizi yapacak, gelişmiş sorgular ile **müşteri davranışlarını inceleyecek**, **performans optimizasyonları** gerçekleştirecek ve **otomatik öneri sistemi** geliştireceksin.

## **🔹 Proje Adımları**  

### **1️⃣ Müşteri Analizi ve Segmentasyonu** 
- Bu senaryoda marketing ekibimiz en çok harcama yapan 10 müşteriyi belirleyerek onları preimum müşteriler kategorisine alarak onlara özel fırsatlar sunmak için bu müşterilerin listesini istedi.

```sql
SELECT DISTINCT c.customer_id, c.first_name, c.last_name, SUM(p.amount) AS total_amount
FROM customer c 
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
order by total_amount desc
LIMIT 10;
```
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

Satış ekibimiz müşterilerin ortalama altında mı yoksa üstünde mi satın alım yaptığını buna göre düşük alışveriş yapan müşterilere mail ve sms aracılığıyla erişmek için bu kişilerin bilgilerini istemektedir

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
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,  -- Müşteri ismi birleştirildi
    cs.customer_id, 
    cs.total_spent,
    cs.avg_spent,
    c.email,   -- Müşteri e-postası
    ad.phone,  -- Müşteri telefon numarası
    (SELECT overall_avg_spent FROM average_spending) AS overall_avg_spent, -- Genel ortalama harcama
    CASE
        WHEN cs.avg_spent > (SELECT overall_avg_spent FROM average_spending) THEN 'Above'
        WHEN cs.avg_spent < (SELECT overall_avg_spent FROM average_spending) THEN 'Below'
        ELSE 'Average'
    END AS spending_category
FROM customer_spending cs
JOIN customer c ON cs.customer_id = c.customer_id  -- customer tablosu ile JOIN
JOIN address ad ON c.address_id = ad.address_id  -- address tablosu ile JOIN
JOIN payment p ON cs.customer_id = p.customer_id;  -- payment tablosu ile JOIN
```

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

- Son 6 ayda en aktif müşterileri harcamalarına göre sıralamak istiyoruz.

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

- Satış Müdürümüz en çok satış yapan elemanı bilmek ve onu ayın elemanı ilan etmek istiyor.

```sql
select s.staff_id, concat (s.first_name, ' ', s.last_name), sum(p.amount) as total_sales
from staff s
join payment p on s.staff_id = p.staff_id
group by s.staff_id
order by total_sales desc
limit 1;
```

|staff_id|concat (s.first_name, ' ', s.last_name)|total_sales|
|--------|---------------------------------------|-----------|
|2       |Jon Stephens                           |33,924.06  |


- Pazarlama ekibimiz daha önce yapılan 150 $ dan fazla alışverişe 30 $ hediye çeki kampanyasında hangi müşterilerin eligible olduğunu bir liste olarak almak istiyor.

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


### **2️⃣ Film Popülerlik ve Kiralama Eğilimleri**  

- Hangi filmlerin ya da kategorilerin daha popüler olduğunu bilmeye ihtiyacımız var.
- En çok kiralanan filmleri kategori bazlı listeliyoruz.

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

En popüler film türlerini analiz ediyoruz
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


- Müşterilerin kiralama sıklığı, harcama alışkanlıklarıyla ilişkili mi?
- Daha sık kiralama yapan müşteriler daha fazla mı harcıyor? Yoksa az kiralayanlar da yüksek meblağlarda ödeme yapıyor mu?


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

### **3️⃣ Öneri Sistemi Geliştirme**

- Bir müşterinin en çok kiraladığı üç kategoriye göre, o kategorilerde daha önce kiralanmamış ve her kategoriden 5 önerilen film sunuyoruz.

```sql
WITH top_categories AS (
    -- Müşterinin en çok kiraladığı 3 kategori
    SELECT 
        c.name AS category,
        COUNT(r.rental_id) AS rental_count
    FROM rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    WHERE r.customer_id = 1  -- Örneğin, müşteri 1'in kiralamalarını al
    GROUP BY c.name
    ORDER BY rental_count DESC
    LIMIT 3  -- En çok kiralanan 3 kategori
),
-- 2. Her kategoriden 5 film öner
recommended_films AS (
    SELECT 
        f.title AS recommended_film,
        c.name AS category,
        ROW_NUMBER() OVER (PARTITION BY c.name ORDER BY f.title) AS row_num  -- Her kategori için sıralama
    FROM film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    WHERE c.name IN (SELECT category FROM top_categories)
    AND f.film_id NOT IN (
        SELECT f.film_id
        FROM rental r
        JOIN inventory i ON r.inventory_id = i.inventory_id
        JOIN film f ON i.film_id = f.film_id
        WHERE r.customer_id = 1  -- Müşteri 1'in daha önce kiraladığı filmleri hariç tut
    )
)
-- 3. Sonuçları filtreleyerek her kategoriden sadece 5 film öner
SELECT recommended_film, category
FROM recommended_films
WHERE row_num <= 5  -- Her kategoriden 5 film seç
ORDER BY category, row_num;


-- 3. Sonuçları filtreleyerek her kategoriden sadece 5 film öner
SELECT recommended_film, category
FROM recommended_films
WHERE row_num <= 5  -- Her kategoriden 5 film seç
ORDER BY category, row_num;
```

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


### **4️⃣ Stored Procedure ve Trigger Oluşturma** 

- Birçok müşteri için sıklıkla en popüler kategoriyi öğrenmeye ihtiyacımız olduğunu fark ettik bu sebeple buna özel **Stored Procedure** fonksiyonu oluşturuyoruz.

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
|category|
|--------|
|Classics|

- Her müşteri yıllık maksimum 10 film kiralayabilir bu sebeple bir trigger oluşturup bu sınırı aşan müşterileri tespit etmek istiyoruz

```sql
DELIMITER $$

CREATE TRIGGER CheckRentalLimit
AFTER INSERT ON rental
FOR EACH ROW
BEGIN
    DECLARE rental_count INT;
    DECLARE rental_limit INT DEFAULT 10; -- Kiralama limiti

    -- Kullanıcının belirli bir tarih aralığında yaptığı kiralamaları sayıyoruz
    SELECT COUNT(*) INTO rental_count
    FROM rental
    WHERE customer_id = NEW.customer_id
      AND rental_date BETWEEN '2025-01-01' AND '2025-12-31'; -- 2025 yılı için örnek tarih aralığı

    -- Eğer kiralama sayısı limitin üstündeyse, rental_history tablosuna kayıt ekliyoruz
    IF rental_count > rental_limit THEN
        INSERT INTO rental_history (customer_id, rental_id, message)
        VALUES (NEW.customer_id, NEW.rental_id, 'Rental limit exceeded!');
    END IF;
END$$

DELIMITER ;
```








