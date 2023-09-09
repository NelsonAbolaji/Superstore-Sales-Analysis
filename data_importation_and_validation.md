## Data Importation

The dataset that will be analyzed is a superstore order data, that contains detailed information about products customers buy from the store. The dataset, which is a csv file can be accessed **here:** [**SuperStore Time Series Dataset | Kaggle**](https://www.kaggle.com/blurredmachine/superstore-time-series-dataset)



---

The SQL statement below was used to create the table where the superstore orders dataset will be loaded into

```sql
CREATE TABLE sample_superstore_orders(
row_id INTEGER,
order_id CHAR(14),
order_date VARCHAR,
ship_date VARCHAR,
ship_mode VARCHAR,
customer_id CHAR(8),
customer_name VARCHAR,
segment VARCHAR,
country VARCHAR,
city VARCHAR,
state VARCHAR,
postal_code VARCHAR(6),
region VARCHAR,
product_id CHAR(15),
category VARCHAR,
sub_category VARCHAR,
product_name VARCHAR,
sales NUMERIC,
quantity NUMERIC,
discount NUMERIC,
profit NUMERIC
 );
```



The csv file 'superstore_train' has been loaded into its designated table. It can be accessed with the code below

```sql
SELECT *
FROM sample_superstore_orders ;
```



The name of the table is lengthy and needs to be shortened

```sql
ALTER TABLE sample_superstore_orders
RENAME TO superstore_order ;
```



The table can now be viewed with this query

```sql
SELECT *
FROM superstore_order
LIMIT 5;
```

**Output**

|        |                |            |            |                |             |                 |           |               |                 |            |             |        |                 |                 |              |                                                             |          |          |          |          |
| ------ | -------------- | ---------- | ---------- | -------------- | ----------- | --------------- | --------- | ------------- | --------------- | ---------- | ----------- | ------ | --------------- | --------------- | ------------ | ----------------------------------------------------------- | -------- | -------- | -------- | -------- |
| row_id | order_id       | order_date | ship_date  | ship_mode      | customer_id | customer_name   | segment   | country       | city            | state      | postal_code | region | product_id      | category        | sub_category | product_name                                                | sales    | quantity | discount | profit   |
| 1      | CA-2016-152156 | 11/8/2016  | 11/11/2016 | Second Class   | CG-12520    | Claire Gute     | Consumer  | United States | Henderson       | Kentucky   | 42420       | South  | FUR-BO-10001798 | Furniture       | Bookcases    | Bush Somerset Collection Bookcase                           | 261.96   | 2        | 0        | 41.9136  |
| 2      | CA-2016-152156 | 11/8/2016  | 11/11/2016 | Second Class   | CG-12520    | Claire Gute     | Consumer  | United States | Henderson       | Kentucky   | 42420       | South  | FUR-CH-10000454 | Furniture       | Chairs       | Hon Deluxe Fabric Upholstered Stacking Chairs, Rounded Back | 731.94   | 3        | 0        | 219.582  |
| 3      | CA-2016-138688 | 6/12/2016  | 6/16/2016  | Second Class   | DV-13045    | Darrin Van Huff | Corporate | United States | Los Angeles     | California | 90036       | West   | OFF-LA-10000240 | Office Supplies | Labels       | Self-Adhesive Address Labels for Typewriters by Universal   | 14.62    | 2        | 0        | 6.8714   |
| 4      | US-2015-108966 | 10/11/2015 | 10/18/2015 | Standard Class | SO-20335    | Sean O'Donnell  | Consumer  | United States | Fort Lauderdale | Florida    | 33311       | South  | FUR-TA-10000577 | Furniture       | Tables       | Bretford CR4500 Series Slim Rectangular Table               | 957.5775 | 5        | 0.45     | -383.031 |
| 5      | US-2015-108966 | 10/11/2015 | 10/18/2015 | Standard Class | SO-20335    | Sean O'Donnell  | Consumer  | United States | Fort Lauderdale | Florida    | 33311       | South  | OFF-ST-10000760 | Office Supplies | Storage      | Eldon Fold 'N Roll Cart System                              | 22.368   | 2        | 0.2      | 2.5164   |



The datatype of some columns will be changed to reflect their appropriate datatypes

```sql
ALTER TABLE superstore_order
ALTER COLUMN order_date TYPE date
USING order_date::date, 
ALTER COLUMN ship_date TYPE date
USING ship_date::date,
ALTER COLUMN postal_code TYPE INTEGER
USING postal_code::INTEGER
```

**Messages**

*ALTER TABLE
Query returned successfully in 321 msec.*



## Data Validation

The dataset has been validated in Microsoft Excel before it was loaded into PostgreSQL's database. The steps remaining to confirm if our dataset is usable is a check for null values and outliers.



#### Null Values

Checking the dataset for null values in the relevant columns

```sql
SELECT *
FROM superstore_order
WHERE row_id IS NULL OR 
order_id IS NULL OR
 order_date IS NULL OR
 ship_date IS NULL OR
 customer_id IS NULL OR
 country IS NULL OR
 city IS NULL OR
 state IS NULL OR
 sales IS NULL OR
 quantity IS NULL OR
 discount IS NULL OR
 profit IS NULL ;
```

**Messages**

*Successfully run. Total query runtime: 280 msec.
0 rows affected.*



Sales should have a quantity, at least 1 attached to it. This query validates that

```sql
SELECT *
FROM superstore_order 
WHERE quantity=0 ;
```

**Messages**

*Successfully run. Total query runtime: 349 msec.
0 rows affected.*



#### Range

**What is the range of the numeric values in the dataset?**

```sql
SELECT CONCAT(to_char(ROUND(MAX(sales),2), 'MIL999,999,990.99'), '/', 
              to_char(ROUND(MIN(sales),2), 'MIL999,999,990.99')) AS max_and_min_sales,
        CONCAT (MAX(quantity), '/', MIN(quantity)) AS max_and_min_quantity,
        CONCAT(to_char(MAX(discount), 'MIL999,999,990.99'), '/', 
               to_char(MIN(discount), 'MIL999,999,990.99')) AS max_and_min_discount,
        CONCAT(to_char(ROUND(MAX(profit),2), 'MIL999,999,990.99'), '/', 
               to_char(ROUND(MIN(profit),2), 'MIL999,999,990.99')) AS max_and_min_profit
FROM superstore_order ;
```

**Output**

|                                   |                      |                                  |                                   |
| --------------------------------- | -------------------- | -------------------------------- | --------------------------------- |
| max_and_min_sales                 | max_and_min_quantity | max_and_min_discount             | max_and_min_profit                |
| \$     22,638.48/ $          0.44 | 14/1                 | $          0.80/ $          0.00 | \$      8,399.98/-$      6,599.98 |

A profit of "-$6599.98" is a possible outlier



#### Checking for outliers

**Calculate the z_score() for the profit column** (This query is being used to check the profit column for more natural outliers.  Z score values greater than 3 or less than -3 are being assumed to be outliers)

```sql
SELECT to_char(profit, 'MIL999,999,990.99') AS profit,
        z_score
FROM 
        (SELECT profit,
                ROUND((profit - AVG(profit) OVER())/(stddev_samp(profit) OVER ()),2) AS z_score
        FROM superstore_order) AS z_score_table
WHERE z_score >3 OR z_score<-3
ORDER BY z_score DESC ; 
```

**Output**

|            |         |
| ---------- | ------- |
| profit     | z_score |
| $8,399.98  | 36.83   |
| $5,039.99  | 22.05   |
| $4,630.48  | 20.25   |
| $3,919.99  | 17.12   |
| $3,177.48  | 13.85   |
| $2,591.96  | 11.28   |
| $2,504.22  | 10.89   |
| $2,400.97  | 10.44   |
| $2,365.98  | 10.28   |
| $2,239.99  | 9.73    |
| $1,995.99  | 8.66    |
| $1,906.49  | 8.26    |
| $1,668.21  | 7.21    |
| $1,644.29  | 7.11    |
| $1,480.47  | 6.39    |
| $1,459.20  | 6.3     |
| $1,439.98  | 6.21    |
| $1,416.80  | 6.11    |
| $1,415.43  | 6.1     |
| $1,379.98  | 5.95    |
| $1,371.98  | 5.91    |
| $1,351.99  | 5.82    |
| $1,276.49  | 5.49    |
| $1,270.99  | 5.47    |
| $1,264.76  | 5.44    |
| $1,228.18  | 5.28    |
| $1,159.99  | 4.98    |
| $1,120.00  | 4.8     |
| $1,114.51  | 4.78    |
| $1,061.57  | 4.55    |
| $1,049.99  | 4.5     |
| $1,014.98  | 4.34    |
| $1,007.98  | 4.31    |
| $944.99    | 4.03    |
| $942.82    | 4.02    |
| $909.98    | 3.88    |
| $899.98    | 3.84    |
| $884.06    | 3.77    |
| $874.99    | 3.73    |
| $843.17    | 3.59    |
| $843.17    | 3.59    |
| $839.99    | 3.57    |
| $829.38    | 3.52    |
| $792.27    | 3.36    |
| $770.35    | 3.26    |
| $767.20    | 3.25    |
| $762.18    | 3.23    |
| $757.41    | 3.21    |
| $751.96    | 3.18    |
| $746.41    | 3.16    |
| $743.99    | 3.15    |
| $742.63    | 3.14    |
| $742.63    | 3.14    |
| $735.03    | 3.11    |
| $726.56    | 3.07    |
| -$729.91   | -3.33   |
| -$734.53   | -3.35   |
| -$760.98   | -3.47   |
| -$766.01   | -3.49   |
| -$786.74   | -3.58   |
| -$786.01   | -3.58   |
| -$814.48   | -3.71   |
| -$935.96   | -4.24   |
| -$938.28   | -4.25   |
| -$944.99   | -4.28   |
| -$950.40   | -4.3    |
| -$1,002.78 | -4.53   |
| -$1,031.54 | -4.66   |
| -$1,049.34 | -4.74   |
| -$1,065.37 | -4.81   |
| -$1,143.89 | -5.16   |
| -$1,181.28 | -5.32   |
| -$1,237.85 | -5.57   |
| -$1,306.55 | -5.87   |
| -$1,359.99 | -6.11   |
| -$1,480.03 | -6.63   |
| -$1,665.05 | -7.45   |
| -$1,811.08 | -8.09   |
| -$1,850.95 | -8.27   |
| -$2,287.78 | -10.19  |
| -$2,639.99 | -11.74  |
| -$2,929.48 | -13.01  |
| -$3,399.98 | -15.08  |
| -$3,839.99 | -17.02  |
| -$6,599.98 | -29.16  |



**Which category has the most profit value outliers?**

```sql
SELECT DISTINCT category,
        COUNT(*) AS number_of_outliers
FROM superstore_order
WHERE profit IN     
    (SELECT profit
        FROM (
              SELECT profit,
                    ROUND((profit - AVG(profit) OVER())/(stddev_samp(profit) OVER ()),2) AS z_score
                FROM superstore_order) AS z_score_table
                WHERE z_score >3 OR z_score<-3
                )
GROUP BY category
ORDER BY number_of_outliers DESC; 
```

**Output**

|                 |                    |
| --------------- | ------------------ |
| category        | number_of_outliers |
| Technology      | 45                 |
| Office Supplies | 33                 |
| Furniture       | 7                  |

*The Technology category has the most outliers*

*While these 85 records do not resemble the rest of the dataset, they will be analyzed with the data so as to prevent bias and also to prevent losing important data that can inform sales.*