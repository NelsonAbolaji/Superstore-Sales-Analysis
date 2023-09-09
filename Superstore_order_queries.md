# Business Questions

1. What is the range of the numeric values in the dataset?

2. Which category has the most profit value outliers?

3. How many days on average does it take for the store to send an order for shipping?

4. How many days on average does it take to send an order for shipping for each ship mode?

5. Which states have an average time before shipping that is greater than 4 days ?

6. What is the percentage of late orders in each business year?

7. How many day late were our late shipment?

8. Over the years of operation, how many percent of our sales profit did we lose after deducting our losses ?

9. How many percent of our profits do we lose every year to sales losses ?

10. How much revenue has the store made from each region since it began operation?

11. How much revenue has the store made from each state since it began operation?

12. Which states recorded profits in the negative(losses)?

13. Compare the profit and loss of the 10 states who recorded more losses than profits

14. What are the top 3 cities that generated the most revenue for the store?

15. What segment placed the most orders and the most profitable orders?

16. Which of our product category did we receive the most orders for and make the most sales and profit from?

17. Why did we record low profits from the Furniture Category ?

18. What are the top 5 products that generate the most sales and profit for the store?

19. What are the store's best performing sub categories?

20. Who are our top 3 customers?

21. Which year did we receive the most orders?

22. Compare total yearly sales and profits with their performance from the previous year

23. Is there a noticeable trend in our monthly generation of revenue over the years of operation?

24. Which day(s) do we receive the most sales and orders?

25. Which quarter of the year do we receive the most orders and make the most sales?

##### The queries and outputs below are answers to business questions that will offer the stakeholders insight into the company's operation.

## Solution

1. **What is the range of the numeric values in the dataset?**

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

|                    |                      |                      |                         |
| ------------------ | -------------------- | -------------------- | ----------------------- |
| max_and_min_sales  | max_and_min_quantity | max_and_min_discount | max_and_min_profit      |
| \$22,638.48/ $0.44 | 14/1                 | \$ 0.80/ \$ 0.00     | \$ 8,399.98/-$ 6,599.98 |

*The highest and lowest sales value, quantity, discount and profit from a single order in the whole dataset*

2. **Which category has the most profit value outliers?**

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

*The Technology category has the most natural outliers i.e. the most unusual profit values*

3. **How many days on average does it take for the store to send an order for shipping?**

```sql
SELECT CONCAT(ROUND (AVG(ship_date - order_date)), ' ', 'days') AS avg_time_before_shipping
FROM superstore_order ;
```

**Output**

|                          |
|:------------------------ |
| avg_time_before_shipping |
| 4 days                   |

*It takes an average of 4 days(This value shouldn't be greater than 2. A 4 day average_ time_before_shipping means a sizeable fraction of our orders are sent for shipping late)*

4. **How many days on average does it take to send an order for shipping  for each ship mode?**

```sql
SELECT ship_mode, 
        ROUND (AVG(ship_date -order_date)) AS average_time_before_shipping
FROM superstore_order
GROUP BY ship_mode 
ORDER BY average_time_before_shipping DESC;        
```

**Output**

|                |                              |
| -------------- | ---------------------------- |
| ship_mode      | average_time_before_shipping |
| Standard Class | 5                            |
| Second Class   | 3                            |
| First Class    | 2                            |
| Same Day       | 0                            |

5. **Which states have an average time before shipping that is greater than 4 days ?**

```sql
SELECT state, 
CONCAT(ROUND(AVG(ship_date - order_date)), ' ', 'days') AS avg_time_before_shipping
FROM superstore_order
GROUP BY state
HAVING ROUND(AVG(ship_date - order_date)) >
 (SELECT ROUND(AVG(ship_date - order_date))
 FROM superstore_order) 
ORDER BY avg_time_before_shipping DESC ;
```

**Output**

|                      |                          |
| -------------------- | ------------------------ |
| state                | avg_time_before_shipping |
| District of Columbia | 6 days                   |
| Maine                | 6 days                   |
| New Mexico           | 5 days                   |
| Minnesota            | 5 days                   |
| Wyoming              | 5 days                   |
| Iowa                 | 5 days                   |
| South Dakota         | 5 days                   |

*This is due to the ship mode that is most patronized by the customers from the state*

6. **What is the percentage of late orders in each business year?**

```sql
SELECT year,
 total_shipment,
 late_shipment,
 CONCAT(ROUND((late_shipment::numeric/total_shipment::numeric) *100 ,2), '%')
FROM (
 SELECT year, 
COUNT(*) AS total_shipment,
 (COUNT(*) FILTER(WHERE ship_mode='Same Day' AND days >0)+
 COUNT(*) FILTER(WHERE ship_mode='First Class' AND days >1)+
 COUNT(*) FILTER(WHERE ship_mode='Second Class' AND days >2) +
 COUNT(*) FILTER(WHERE ship_mode='Standard Class' AND days >4)) AS late_shipment
FROM (
 SELECT EXTRACT(year FROM order_date) AS year,
 ship_mode,
 (ship_date - order_date) AS days
 FROM superstore_order  
 ) AS t1
GROUP BY year
ORDER BY year
 ) AS t2 ;
```

**Output**

|      |                |               |                          |
| ---- | -------------- | ------------- | ------------------------ |
| year | total_shipment | late_shipment | percent_of_late_shipment |
| 2014 | 1592           | 938           | 58.92%                   |
| 2015 | 1667           | 994           | 59.63%                   |
| 2016 | 2079           | 1275          | 61.33%                   |
| 2017 | 2662           | 1624          | 61.01%                   |

7. **How many day late were our late shipment?**
   
   ```sql
   SELECT total_orders_to_be_shipped,
          a_day_late_for_shipment,
          CONCAT(ROUND((a_day_late_for_shipment::numeric/total_orders_to_be_shipped::numeric *100), 2), '%') 
                  AS a_day_late_percent,
          two_day_late_for_shipment,
          CONCAT(ROUND((two_day_late_for_shipment::numeric/total_orders_to_be_shipped::numeric *100), 2), '%') 
                  AS two_day_late_percent,
          three_day_late_for_shipment,
          CONCAT(ROUND((three_day_late_for_shipment::numeric/total_orders_to_be_shipped::numeric *100), 2), '%') 
                  AS three_day_late_percent,
          more_than_three_days,
          CONCAT(ROUND((more_than_three_days::numeric/total_orders_to_be_shipped::numeric *100), 2), '%') 
                  AS more_than_three_days_percent
   FROM (
      SELECT 
              COUNT(*) AS total_orders_to_be_shipped,
          COUNT(CASE WHEN ship_mode = 'Same Day' AND days = 1 OR
                ship_mode = 'First Class' AND days = 2 OR
                ship_mode = 'Second Class' AND days = 3 OR 
                ship_mode = 'Standard Class' AND days = 5 THEN 1 END) AS a_day_late_for_shipment,
          COUNT(CASE WHEN ship_mode = 'Same Day' AND days = 2 OR
                ship_mode = 'First Class' AND days = 3 OR
                ship_mode = 'Second Class' AND days = 4 OR 
                ship_mode = 'Standard Class' AND days = 6 THEN 1 END) AS two_day_late_for_shipment,
          COUNT(CASE WHEN ship_mode = 'Same Day' AND days = 3 OR
                ship_mode = 'First Class' AND days = 4 OR
                ship_mode = 'Second Class' AND days = 5 OR 
                ship_mode = 'Standard Class' AND days = 7 THEN 1 END) AS three_day_late_for_shipment,
          COUNT(CASE WHEN ship_mode = 'Same Day' AND days > 4 OR
                ship_mode = 'First Class' AND days > 5 OR
                ship_mode = 'Second Class' AND days > 6 OR 
                ship_mode = 'Standard Class' AND days > 8 THEN 1 END) AS more_than_three_days
      FROM (
           SELECT EXTRACT(year FROM order_date) AS year,
           ship_mode,
           (ship_date - order_date) AS days
           FROM superstore_order  
       ) AS t1
      )AS t2 ; 
   ```

**Output**

|                            |                         |                    |                           |                      |                             |                        |                      |                              |
| -------------------------- | ----------------------- | ------------------ | ------------------------- | -------------------- | --------------------------- | ---------------------- | -------------------- | ---------------------------- |
| total_orders_to_be_shipped | a_day_late_for_shipment | a_day_late_percent | two_day_late_for_shipment | two_day_late_percent | three_day_late_for_shipment | three_day_late_percent | more_than_three_days | more_than_three_days_percent |
| 8000                       | 2180                    | 27.25%             | 1788                      | 22.35%               | 863                         | 10.79%                 | 0                    | 0.00%                        |

8. **Over the years of operation, how many percent of our sales profit did we lose after deducting our losses ?**
   
   ```sql
   SELECT to_char(sales, 'MIL999,999,990.99') AS sales,
           to_char(profit, 'MIL999,999,990.99') AS profit,
           to_char(profit + loss, 'MIL999,999,990.99') AS profit_minus_loss,
           to_char(loss, 'MIL999,999,990.99') AS loss,
           CONCAT(ROUND(loss * -1 /profit *100, 2), '%') AS loss_deduction_percent,
   --The loss deduction percent is the percentage of money deducted from the profit 
           CONCAT(ROUND(((profit+loss)/sales) *100, 2), '%') AS profit_over_sales_percent
   FROM (
       SELECT SUM(sales) AS sales,
               ROUND(SUM(profit) FILTER (WHERE profit > 0), 2) AS profit,
               ROUND(SUM(profit) FILTER (WHERE profit < 0), 2) AS loss
       FROM superstore_order
       ) AS t1 ;
   ```

**Output**

|               |             |                   |              |                        |                           |
| ------------- | ----------- | ----------------- | ------------ | ---------------------- | ------------------------- |
| sales         | profit      | profit_minus_loss | loss         | loss_deduction_percent | profit_over_sales_percent |
| $1,838,587.67 | $351,756.41 | $225,073.86       | -$126,682.55 | 36.01%                 | 12.24%                    |

*We lose 36 percent of our profit after deducting losses and our eventual profit is only 12.24 percent of our sales*

9. **How many percent of our profits do we lose every year to sales losses ?**
   
   ```sql
   SELECT CONCAT(ROUND(AVG(loss_deduction_percent), 2), '%') AS avg_loss_deduction_percent
   FROM (
       SELECT ROUND(loss * -1 /profit *100, 2) AS loss_deduction_percent    
       FROM (
           SELECT EXTRACT(year FROM order_date) AS year,
                   SUM(sales) AS sales,
                   ROUND(SUM(profit) FILTER (WHERE profit > 0), 2) AS profit,
                   ROUND(SUM(profit) FILTER (WHERE profit < 0), 2) AS loss
           FROM superstore_order
           GROUP BY year
           ) AS t1 
       ) AS t2 ;
   ```
   
   **Output**
   
   |                            |
   | -------------------------- |
   | avg_loss_deduction_percent |
   | 35.63%                     |
   
   *We lose an average of 35.63 percent profit each year to losses incurred from orders*

10. **How much revenue has the store made from each region since it began operation?**

```sql
SELECT CONCAT(country, ' ', region) AS region,
        COUNT(*) AS number_of_orders,
        to_char ( ROUND ( SUM(sales) , 2),'MIL999,999,999.99' ) AS total_sales_by_region,
        to_char ( ROUND ( SUM(profit) , 2),'MIL999,999,999.99' ) AS total_profit_by_region
FROM superstore_order
GROUP BY country, region
ORDER BY total_sales_by_region DESC , total_profit_by_region DESC ;
```

**Output**

|                       |                  |                       |                        |
| --------------------- | ---------------- | --------------------- | ---------------------- |
| region                | number_of_orders | total_sales_by_region | total_profit_by_region |
| United States West    | 2605             | $574,871.57           | $79,739.36             |
| United States East    | 2270             | $551,176.30           | $72,473.65             |
| United States Central | 1816             | $394,604.98           | $34,094.49             |
| United States South   | 1309             | $317,934.81           | $38,766.37             |

11. **How much revenue has the store made from each state since it began operation?**

```sql
SELECT state,
        COUNT(*) AS number_of_orders,
        to_char ( ROUND ( SUM(sales) , 2),'MIL999,999,999.99' ) AS total_sales_by_state,
        to_char ( ROUND ( SUM(profit) , 2),'MIL999,999,999.99' ) AS total_profit_by_state
FROM superstore_order
GROUP BY state
ORDER BY total_sales_by_state DESC , total_profit_by_state DESC ;
```

**Output**

|                      |                  |                      |                       |
| -------------------- | ---------------- | -------------------- | --------------------- |
| state                | number_of_orders | total_sales_by_state | total_profit_by_state |
| California           | 1608             | $359,678.19          | $59,671.63            |
| New York             | 912              | $248,234.16          | $58,475.69            |
| Texas                | 734              | $127,608.54          | -$17,096.23           |
| Washington           | 397              | $99,237.30           | $21,411.25            |
| Pennsylvania         | 480              | $93,430.97           | -$12,890.56           |
| Florida              | 296              | $73,679.80           | -$3,535.43            |
| Ohio                 | 361              | $65,354.54           | -$15,753.59           |
| Illinois             | 384              | $64,135.61           | -$10,716.34           |
| Michigan             | 213              | $59,824.55           | $17,742.37            |
| Virginia             | 172              | $52,090.70           | $13,225.07            |
| North Carolina       | 200              | $45,323.34           | -$5,828.23            |
| Indiana              | 122              | $45,155.12           | $16,373.26            |
| Georgia              | 151              | $40,470.86           | $13,742.34            |
| Arizona              | 191              | $31,919.81           | -$2,760.93            |
| Kentucky             | 122              | $31,349.29           | $9,704.90             |
| Colorado             | 165              | $30,430.85           | -$6,592.44            |
| New Jersey           | 95               | $29,622.48           | $8,146.41             |
| Minnesota            | 80               | $27,883.56           | $10,351.40            |
| Wisconsin            | 88               | $25,809.06           | $6,709.13             |
| Delaware             | 74               | $25,169.63           | $9,059.78             |
| Maryland             | 91               | $22,405.37           | $6,728.27             |
| Massachusetts        | 100              | $22,196.85           | $5,162.50             |
| Rhode Island         | 47               | $21,298.19           | $7,264.82             |
| Tennessee            | 132              | $21,047.44           | -$3,468.95            |
| Alabama              | 54               | $18,437.29           | $5,468.24             |
| Nevada               | 31               | $14,951.91           | $3,123.96             |
| Oklahoma             | 52               | $14,400.56           | $3,556.60             |
| Oregon               | 107              | $13,958.89           | -$778.28              |
| Missouri             | 49               | $13,915.83           | $2,922.53             |
| Connecticut          | 66               | $11,643.39           | $2,976.47             |
| Utah                 | 49               | $10,763.34           | $2,383.24             |
| Mississippi          | 49               | $10,130.04           | $2,930.93             |
| Arkansas             | 54               | $9,408.00            | $2,969.60             |
| South Carolina       | 42               | $8,481.71            | $1,769.06             |
| Louisiana            | 37               | $7,516.34            | $1,788.84             |
| Nebraska             | 34               | $6,989.04            | $1,912.84             |
| New Hampshire        | 24               | $6,845.58            | $1,537.37             |
| Montana              | 12               | $5,179.25            | $1,708.38             |
| New Mexico           | 29               | $4,061.18            | $971.75               |
| Iowa                 | 21               | $3,944.76            | $936.89               |
| Idaho                | 15               | $3,087.72            | $500.60               |
| Kansas               | 23               | $2,881.02            | $828.45               |
| District of Columbia | 10               | $2,865.02            | $1,059.59             |
| Wyoming              | 1                | $1,603.14            | $100.20               |
| South Dakota         | 9                | $1,137.42            | $343.43               |
| Vermont              | 2                | $920.23              | $246.46               |
| North Dakota         | 7                | $919.91              | $230.15               |
| Maine                | 5                | $653.41              | $197.56               |
| West Virginia        | 3                | $536.48              | $262.88               |

*There are states where the company recorded losses instead of profits by the thousand. These records should be viewed separately*

12. **Which states recorded profits in the negative(losses)?**

```sql
SELECT state,
        COUNT(*) AS number_of_orders,
        to_char ( ROUND ( SUM(sales) , 2),'MIL999,999,990.99' ) AS total_sales_by_state,
        to_char ( ROUND ( SUM(profit) , 2),'MIL999,999,990.99' ) AS total_profit_by_state
FROM superstore_order
WHERE state IN 
      (SELECT state
       FROM superstore_order 
       GROUP BY state
      HAVING SUM(profit) < 0
     )
GROUP BY state 
ORDER BY total_sales_by_state DESC , total_profit_by_state DESC ;
```

**Output**

|                |                  |                      |                       |
| -------------- | ---------------- | -------------------- | --------------------- |
| state          | number_of_orders | total_sales_by_state | total_profit_by_state |
| Texas          | 734              | $127,608.54          | -$17,096.23           |
| Pennsylvania   | 480              | $93,430.97           | -$12,890.56           |
| Florida        | 296              | $73,679.80           | -$3,535.43            |
| Ohio           | 361              | $65,354.54           | -$15,753.59           |
| Illinois       | 384              | $64,135.61           | -$10,716.34           |
| North Carolina | 200              | $45,323.34           | -$5,828.23            |
| Arizona        | 191              | $31,919.81           | -$2,760.93            |
| Colorado       | 165              | $30,430.85           | -$6,592.44            |
| Tennessee      | 132              | $21,047.44           | -$3,468.95            |
| Oregon         | 107              | $13,958.89           | -$778.28              |

13. **Compare the profit and loss of the 10 states who recorded more losses than profits**

```sql
WITH state_sales AS (
    SELECT state, 
        ROUND (SUM(sales) , 2) AS total_sales_by_state
    FROM superstore_order
    GROUP BY state
                    ),

state_profit AS (
    SELECT state, 
        ROUND (SUM(profit) , 2) AS total_profit_by_state
    FROM superstore_order
    WHERE profit >= 0
    GROUP BY state
                    ),

state_loss AS (
      SELECT state, 
        ROUND (SUM(profit) , 2) AS total_loss_by_state
    FROM superstore_order
    WHERE profit < 0
    GROUP BY state
                ) 

SELECT state_sales.state,
        to_char(total_sales_by_state, 'MIL999,999,999.99') AS total_sales_by_state,
        to_char(total_loss_by_state, 'MIL999,999,999.99') AS total_loss_by_state,
        to_char(total_profit_by_state, 'MIL999,999,999.99')    AS total_profit_by_state    
FROM state_sales
FULL JOIN state_profit
ON state_sales.state = state_profit.state
FULL JOIN state_loss
ON state_sales.state = state_loss.state
GROUP BY state_sales.state, total_sales_by_state, total_profit_by_state, total_loss_by_state                
HAVING (total_loss_by_state * -1) > total_profit_by_state 
ORDER BY total_loss_by_state* -1 DESC;
```

**Output**

|                |                      |                     |                       |
| -------------- | -------------------- | ------------------- | --------------------- |
| state          | total_sales_by_state | total_loss_by_state | total_profit_by_state |
| Texas          | $127,608.54          | -$25,833.93         | $8,737.70             |
| Ohio           | $65,354.54           | -$19,657.81         | $3,904.23             |
| Pennsylvania   | $93,430.97           | -$17,734.00         | $4,843.44             |
| Illinois       | $64,135.61           | -$15,890.38         | $5,174.04             |
| North Carolina | $45,323.34           | -$9,311.05          | $3,482.83             |
| Colorado       | $30,430.85           | -$8,862.66          | $2,270.22             |
| Florida        | $73,679.80           | -$7,524.40          | $3,988.97             |
| Arizona        | $31,919.81           | -$5,740.62          | $2,979.69             |
| Tennessee      | $21,047.44           | -$4,887.43          | $1,418.48             |
| Oregon         | $13,958.89           | -$2,210.79          | $1,432.51             |

*After loss deduction, the store recorded no profit from these states. There is no information provided that can be further investigated to find out the reason for the losses*

14. **What are the top 3 cities that generated the most revenue for the store?**

```sql
SELECT city,
        COUNT(*) AS total_number_of_orders,
        to_char(ROUND(SUM(sales), 2), 'MIL999,999,990.99') AS total_sales,
        to_char(ROUND(SUM(profit), 2), 'MIL999,999,990.99') AS total_profit
FROM superstore_order
GROUP BY city
ORDER BY total_sales DESC
LIMIT 3 ;
```

**Output**

|               |                        |             |              |
| ------------- | ---------------------- | ----------- | ------------ |
| city          | total_number_of_orders | total_sales | total_profit |
| New York City | 743                    | $204,298.53 | $49,037.42   |
| Los Angeles   | 596                    | $136,767.68 | $24,525.88   |
| San Francisco | 417                    | $94,433.98  | $13,855.78   |

15. **What segment placed the most orders and the most profitable orders?**

```sql
SELECT RANK() OVER (ORDER BY ROUND(SUM(sales),2) DESC),
        segment,
        SUM(quantity) AS total_quantity_unit,
        COUNT(*) AS total_orders,
        to_char(ROUND(SUM(sales),2), 'MIL999,999,999.99') AS total_sales_by_segment,
        to_char(ROUND(SUM(profit),2), 'MIL999,999,990.99') AS total_profit_by_segment
FROM superstore_order
GROUP BY segment
ORDER BY ROUND(SUM(sales),2) DESC;
```

**Output**

|      |             |                     |              |                        |                         |
| ---- | ----------- | ------------------- | ------------ | ---------------------- | ----------------------- |
| rank | segment     | total_quantity_unit | total_orders | total_sales_by_segment | total_profit_by_segment |
| 1    | Consumer    | 15802               | 4192         | $906,473.10            | $103,683.14             |
| 2    | Corporate   | 9129                | 2376         | $564,515.34            | $69,434.18              |
| 3    | Home Office | 5364                | 1432         | $367,599.24            | $51,956.55              |

*The 'Consumer' segment recorded the most orders, sales and profits.*

16. **Which of our product category did we receive the most orders for and make the most sales and profit from?**

```sql
SELECT category,
        total_number_of_orders,
        to_char(total_sales_by_category, 'MIL999,999,999.99') AS total_sales_by_category,
        CONCAT(ROUND((total_sales_by_category/total_sales)*100, 2),  '%') AS sales_percentage_by_category,
        to_char(total_profit_by_category, 'MIL999,999,999.99') AS total_profit_by_category,
        CONCAT(ROUND((total_profit_by_category/total_profit)*100, 2), '%') AS profit_percentage_by_category
FROM (SELECT category,
        COUNT(*) AS total_number_of_orders,
        ROUND(SUM(sales), 2) AS total_sales_by_category,
                  (SELECT ROUND(SUM(sales),2) AS total_sales
                 FROM superstore_order),
        ROUND(SUM(profit), 2) AS total_profit_by_category,
                (SELECT ROUND(SUM(profit),2) AS total_profit
                 FROM superstore_order)    
FROM superstore_order
GROUP BY category) AS product_category
ORDER BY sales_percentage_by_category DESC;
```

**Output**

|                 |                        |                         |                              |                          |                               |
| --------------- | ---------------------- | ----------------------- | ---------------------------- | ------------------------ | ----------------------------- |
| category        | total_number_of_orders | total_sales_by_category | sales_percentage_by_category | total_profit_by_category | profit_percentage_by_category |
| Technology      | 1469                   | $679,802.19             | 36.97%                       | $111,435.79              | 49.51%                        |
| Furniture       | 1691                   | $587,825.17             | 31.97%                       | $16,875.24               | 7.50%                         |
| Office Supplies | 4840                   | $570,960.32             | 31.05%                       | $96,762.84               | 42.99%                        |

*There is a 35.49%  difference between the profit percentages of the Furniture and Office Supplies category even though their sales percent is only different by 0.92%*

17. **Why did we record low profits from the Furniture Category ?**
    
    ```sql
    SELECT category,
           total_number_of_orders,
            to_char(total_sales_by_category, 'MIL999,999,990.99') AS total_sales_by_category,
            to_char(total_profit, 'MIL999,999,990.99') AS total_profit,
            to_char(total_profit_by_category, 'MIL999,999,990.99') AS total_profit_by_category,
            to_char(total_loss_by_category, 'MIL999,999,990.99') AS total_loss_by_category
    FROM (SELECT category,
            COUNT(*) AS total_number_of_orders,
            ROUND(SUM(sales), 2) AS total_sales_by_category,
            ROUND(SUM(profit) FILTER(WHERE profit > 0), 2) AS total_profit_by_category,
          ROUND(SUM(profit) FILTER(WHERE profit < 0), 2) AS total_loss_by_category,
                    (SELECT ROUND(SUM(profit),2) AS total_profit
                     FROM superstore_order)    
    FROM superstore_order
    GROUP BY category) AS product_category 
    ORDER BY total_number_of_orders DESC;
    ```

**Output**

|                 |                        |                         |              |                          |                        |
| --------------- | ---------------------- | ----------------------- | ------------ | ------------------------ | ---------------------- |
| category        | total_number_of_orders | total_sales_by_category | total_profit | total_profit_by_category | total_loss_by_category |
| Office Supplies | 4840                   | $570,960.32             | $225,073.86  | $139,511.23              | -$42,748.40            |
| Furniture       | 1691                   | $587,825.17             | $225,073.86  | $64,534.18               | -$47,658.94            |
| Technology      | 1469                   | $679,802.19             | $225,073.86  | $147,711.00              | -$36,275.21            |

*The store does not make a lot of profit from the sale of products in the Furniture Category*

18. **What are the top 5 products that generate the most sales and profit for the store?**

```sql
SELECT RANK() OVER(ORDER BY total_sales DESC, total_profit DESC) AS rank,
        product_name,
        total_quantity,
        to_char(total_sales, 'MIL999,999,990.99') AS total_sales,
        to_char(total_profit, 'MIL999,999,990.99') AS total_profit
FROM (
SELECT DISTINCT product_id,
        SUM(quantity) AS total_quantity,
        SUM(sales) AS total_sales,
        SUM(profit) AS total_profit
FROM superstore_order
GROUP BY product_id
    ) AS t1,
    (SELECT DISTINCT product_id, 
            product_name
     FROM superstore_order
    ) AS t2
WHERE t1.product_id=t2.product_id
ORDER BY total_sales DESC 
LIMIT 5 ;
```

**Output**

|      |                                                       |                |             |              |
| ---- | ----------------------------------------------------- | -------------- | ----------- | ------------ |
| rank | product_name                                          | total_quantity | total_sales | total_profit |
| 1    | Canon imageCLASS 2200 Advanced Copier                 | 16             | $47,599.86  | $18,479.95   |
| 2    | Cisco TelePresence System EX90 Videoconferencing Unit | 6              | $22,638.48  | -$1,811.08   |
| 3    | Hewlett Packard LaserJet 3310 Copier                  | 33             | $17,039.72  | $6,743.89    |
| 4    | High Speed Automatic Electric Letter Opener           | 11             | $17,030.31  | -$262.00     |
| 5    | Lexmark MX611dhe Monochrome Laser Printer             | 18             | $16,829.90  | -$4,589.97   |

19. **What are the store's best performing sub categories?**

```sql
SELECT RANK() OVER(ORDER BY total_sales DESC, total_profit DESC),
        sub_category,
        total_orders,
        to_char(total_sales, 'MIL999,999,990.99') AS total_sales,
        to_char(total_profit, 'MIL999,999,990.99') AS total_profit
FROM (
SELECT DISTINCT sub_category,
        COUNT(*) AS total_orders,
        SUM(sales) AS total_sales,
        SUM(profit) AS total_profit
FROM superstore_order
GROUP BY sub_category
    ) AS sc_t
LIMIT 5 ;
```

**Output**

|      |              |              |             |              |
| ---- | ------------ | ------------ | ----------- | ------------ |
| rank | sub_category | total_orders | total_sales | total_profit |
| 1    | Phones       | 720          | $267,838.33 | $36,534.32   |
| 2    | Chairs       | 502          | $264,689.17 | $22,956.24   |
| 3    | Storage      | 680          | $179,766.53 | $17,622.59   |
| 4    | Tables       | 254          | $162,418.17 | -$13,392.94  |
| 5    | Machines     | 95           | $159,003.28 | -$3,024.86   |

20. **Who are our top 3 customers?**

```sql
SELECT RANK() OVER(ORDER BY total_sales DESC) AS rank,
        customer_name,
        segment,
        number_of_orders,
        to_char(total_sales, 'MIL999,999,999.99') AS total_sales
FROM (
SELECT DISTINCT customer_id,
        segment,
        COUNT(*) AS number_of_orders,
        SUM(sales) AS total_sales
FROM superstore_order
GROUP BY customer_id, segment
    ) AS t1,
    (SELECT DISTINCT customer_id,
            customer_name
    FROM superstore_order) AS t2
WHERE t1.customer_id=t2.customer_id
ORDER BY total_sales DESC
LIMIT 3 ;
```

**Output**

|      |               |             |                  |             |
| ---- | ------------- | ----------- | ---------------- | ----------- |
| rank | customer_name | segment     | number_of_orders | total_sales |
| 1    | Sean Miller   | Home Office | 10               | $24,205.61  |
| 2    | Tamara Chand  | Corporate   | 8                | $18,444.45  |
| 3    | Tom Ashbrook  | Home Office | 9                | $14,588.58  |

21. **Which year did we receive the most orders?**

```sql
SELECT EXTRACT(Year FROM order_date) AS year,
        COUNT(*) AS number_of_order_by_year
FROM superstore_order
GROUP BY year
ORDER BY number_of_order_by_year DESC
LIMIT 1;
```

**Output**

|      |                         |
| ---- | ----------------------- |
| year | number_of_order_by_year |
| 2017 | 2662                    |

*We received the most orders in 2017*

22. **Compare total yearly sales and profits with their performance from the previous year**

```sql
SELECT year, 
        to_char(current_year_sales_total, 'MIL999,999,990.99') AS current_year_sales_total,
        to_char(COALESCE(previous_year_sales_total, 0), 'MIL999,999,990.99') AS previous_year_sales_total,
        CONCAT(COALESCE(ROUND(year_on_year_sales_growth), 0), '%') AS year_on_year_sales_growth_percent,
        to_char(current_year_profit_total, 'MIL999,999,990.99') AS current_year_profit_total,
        to_char(COALESCE(previous_year_profit_total, 0), 'MIL999,999,990.99') AS previous_year_profit_total,
        CONCAT(COALESCE(ROUND(year_on_year_profit_growth), 0), '%') AS year_on_year_profit_growth_percent
FROM(
    SELECT year,
            total_sales AS current_year_sales_total,
            LAG(total_sales) OVER(ORDER BY year) AS previous_year_sales_total,
            (total_sales - LAG(total_sales) OVER(ORDER BY year)) /
                LAG(total_sales) OVER(ORDER BY year) *100 AS year_on_year_sales_growth,
            total_profit AS current_year_profit_total,
            LAG(total_profit) OVER(ORDER BY year) AS previous_year_profit_total,
            (total_profit - LAG(total_profit) OVER(ORDER BY year)) /
                LAG(total_profit) OVER(ORDER BY year) *100 AS year_on_year_profit_growth
/* I left all the necessary coalescing, percentage and currency formatting undone in this subquery 
and did it in the subquery at the beginning in order to make the query clean and clear */
    FROM(
        SELECT  EXTRACT(YEAR FROM order_date) AS year,
            ROUND(SUM(sales), 2) AS total_sales,
            ROUND(SUM(profit), 2) AS total_profit
        FROM superstore_order
        GROUP BY year

        ) AS yearly_sales_and_profit

    ) AS s_and_p_yoy_growth 
ORDER BY year;
```

**Output**

|      |                          |                           |                                   |                           |                            |                                    |
| ---- | ------------------------ | ------------------------- | --------------------------------- | ------------------------- | -------------------------- | ---------------------------------- |
| year | current_year_sales_total | previous_year_sales_total | year_on_year_sales_growth_percent | current_year_profit_total | previous_year_profit_total | year_on_year_profit_growth_percent |
| 2014 | $410,701.40              | $0.00                     | 0%                                | $43,807.45                | $0.00                      | 0%                                 |
| 2015 | $372,506.53              | $410,701.40               | -9%                               | $51,005.25                | $43,807.45                 | 16%                                |
| 2016 | $468,985.32              | $372,506.53               | 26%                               | $59,842.38                | $51,005.25                 | 17%                                |
| 2017 | $586,394.42              | $468,985.32               | 25%                               | $70,418.78                | $59,842.38                 | 18%                                |

*Sales growth dropped after the first year of operation but rose sharply in the third year and dropped by a percentage in the fourth year while profit improved by a percent each year.*

23. **Is there a noticeable trend in our monthly generation of revenue over the years of operation?**

```sql
SELECT to_char(order_date, 'Month') AS month,
        COUNT(*) AS total_orders,
        CAST(SUM(sales) AS money) AS total_sales
FROM superstore_order
GROUP BY month
ORDER BY total_sales DESC;
```

**Output**

|           |              |             |
| --------- | ------------ | ----------- |
| month     | total_orders | total_sales |
| November  | 1178         | $285,587.12 |
| December  | 1148         | $264,613.35 |
| September | 1119         | $256,483.92 |
| October   | 661          | $162,768.39 |
| March     | 537          | $159,354.33 |
| August    | 563          | $128,571.18 |
| July      | 586          | $125,531.71 |
| May       | 573          | $123,847.08 |
| April     | 555          | $117,387.16 |
| June      | 567          | $109,730.26 |
| January   | 292          | $64,486.04  |
| February  | 221          | $40,227.11  |

*We recorded a significantly higher sale in November, December and September than the rest of the months*

24. **Which day(s) do we receive the most sales and orders?**

```sql
SELECT to_char(order_date, 'Dy') AS day_of_week,
        COUNT(*) AS number_of_order,
        CAST(SUM(sales) AS money) AS total_sales
FROM superstore_order
GROUP BY day_of_week
ORDER BY total_sales DESC ;
```

**Output**

|             |                 |             |
| ----------- | --------------- | ----------- |
| day_of_week | number_of_order | total_sales |
| Mon         | 1496            | $348,992.31 |
| Fri         | 1467            | $343,481.76 |
| Sun         | 1393            | $323,945.54 |
| Sat         | 1381            | $297,711.45 |
| Thu         | 1136            | $233,088.31 |
| Tue         | 857             | $225,438.66 |
| Wed         | 270             | $65,929.65  |

25. **Which quarter of the year do we receive the most orders and make the most sales?**

```sql
SELECT to_char(order_date, 'Q') AS quarter_of_the_year,
        COUNT(*) AS orders,
        CAST(SUM(sales) AS money) AS total_sales
FROM superstore_order
GROUP BY quarter_of_the_year
ORDER BY orders DESC, total_sales DESC; 
```

**Output**

|                     |              |             |
| ------------------- | ------------ | ----------- |
| quarter_of_the_year | total_orders | total_sales |
| 4                   | 2987         | $712,968.87 |
| 3                   | 2268         | $510,586.81 |
| 2                   | 1695         | $350,964.51 |
| 1                   | 1050         | $264,067.49 |

*We received the most orders and made the most sales in  the fourth quarter of the year*
