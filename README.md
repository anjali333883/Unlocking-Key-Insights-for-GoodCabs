### 𝗣𝗿𝗼𝘃𝗶𝗱𝗲 𝗜𝗻𝘀𝗶𝗴𝗵𝘁𝘀 𝘁𝗼 𝗖𝗵𝗶𝗲𝗳 𝗼𝗳 𝗢𝗽𝗲𝗿𝗮𝘁𝗶𝗼𝗻𝘀 𝗶𝗻 𝗧𝗿𝗮𝗻𝘀𝗽𝗼𝗿𝘁𝗮𝘁𝗶𝗼𝗻 𝗗𝗼𝗺𝗮𝗶𝗻 


#### 🤔 Problem Statement:
GoodCabs, a rapidly growing cab service provider focused on tier-2 cities in India, seeks to assess its performance for 2024.<br>
The Chief of Operations requires insights into key metrics like trip volume, passenger satisfaction, repeat passenger rate, and trip distribution. 

#### ✅ Task:
As a data analyst, my task was to analyze datasets and answer key business questions, design an interactive visuals, and present actionable insights to the Chief of Operations. 
Additionally, generate SQL-based reports and explore additional research questions to support business decisions.

#### 🔍 Key Insights:
➡️ Generated ₹108.2 Million revenue from 425K trips, with an average fare of ₹254.02 per trip and ₹13.28 per km.  
➡️ 74% of passengers were new, yet repeat trips contributed to 58.4% of total trips.  
➡️ Average trip distance was 19.13 km, perfect for tier-2 city operations.  
➡️ Driver and passenger ratings averaged 7.83 and 7.66, reflecting positive experiences with room for improvement.  
➡️ Jaipur led weekend trips (44,397), while Lucknow topped weekday trips (49,617).  

#### 🔧 Tools Used: 
Power BI<br>
MySQL<br>

#### 🛢️ Ad Hoc Requests:
Business Request - 1: City-Level Fare and Trip Summary Report<br>
Generate a report that displays the total trips, average fare per km, average fare per trip, and the percentage contribution of each city’s trips to the overall trips. This report will help in assessing trip volume, pricing efficiency, and each city’s contribution to the overall trip count.

Fields: city_name ,total_trips ,avg_fare_per_km ,avg_fare_per_trip,%_contribution_to_total_trips
``` 
WITH cte1 AS(
SELECT city_name,
       COUNT(DISTINCT trip_id) AS total_trips,
       CONCAT("Rs ",ROUND(SUM(fare_amount)/SUM(distance_travelled_km),2)) AS avg_fare_per_km,
       CONCAT("Rs ",ROUND(SUM(fare_amount)/COUNT(DISTINCT trip_id),2)) AS avg_fare_per_trip
FROM fact_trips t 
JOIN dim_city c ON c.city_id=t.city_id
GROUP BY 1),cte2 AS(
SELECT COUNT(DISTINCT trip_id) AS overall_trips
FROM fact_trips)
SELECT city_name,
       total_trips,
       avg_fare_per_km,
       avg_fare_per_trip,
	   ROUND((total_trips/overall_trips)*100,2) AS "%_contribution_to_total_trips"
FROM cte1
JOIN cte2;
```

Business Request - 2: Monthly City-Level Trips Target Performance Report <br>
Generate a report that evaluates the target performance for trips at the monthly and city level. For each city and month, compare the actual total trips with the target trips and categorise the performance as follows:<br>
If actual trips are greater than target trips, mark it as "Above Target".<br>
If actual trips are less than or equal to target trips, mark it as "Below Target".<br>
Additionally, calculate the % difference between actual and target trips to quantify the performance gap.<br>

Fields: City_name ,month_name ,actual_trips ,target_trips ,performance_status ,%_difference

```
WITH cte1 AS(
SELECT city_name,
       month_name,
       COUNT(trip_id) AS actual_trips
FROM dim_city c
JOIN fact_trips t ON c.city_id=t.city_id
JOIN dim_date d ON d.date=t.date
GROUP BY 1,2)
SELECT c.city_name,
       c.month_name,
	   actual_trips,
       target_trips,
       CASE WHEN actual_trips > target_trips THEN "Above Target"
            WHEN actual_trips <= target_trips THEN "Below Target"
	   END AS performance_status,
       ROUND((actual_trips-target_trips)/target_trips*100,2) AS "%_difference"
FROM cte1 c 
JOIN (SELECT  city_name,
              MONTHNAME(month) AS month_name,
              total_target_trips AS target_trips
	 FROM targets_db.monthly_target_trips t 
	 JOIN dim_city c ON t.city_id=c.city_id) x ON c.city_name=x.city_name AND c.month_name=x.month_name;									  
```

Business Request - 3: City-Level Repeat Passenger Trip Frequency Report <br>
Generate a report that shows the percentage distribution of repeat passengers by the number of trips they have taken in each city. Calculate the percentage of repeat passengers who took 2 trips, 3 trips, and so on, up to 10 trips.<br>
Each column should represent a trip count category, displaying the percentage of repeat passengers who fall into that category out of the total repeat passengers for that city.<br>
This report will help identify cities with high repeat trip frequency, which can indicate strong customer loyalty or frequent usage patterns.<br>

Fields: city_name, 2-Trips, 3-Trips, 4-Trips, 5-Trips, 6-Trips, 7-Trips, 8-Trips, 9-Trips, 10-Trips

```
SELECT c.city_name, 
       CONCAT(ROUND(SUM(CASE WHEN trip_count="2-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "2-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="3-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "3-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="4-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "4-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="5-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "5-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="6-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "6-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="7-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "7-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="8-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "8-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="9-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "9-Trips",
       CONCAT(ROUND(SUM(CASE WHEN trip_count="10-Trips" THEN repeat_passenger_count ELSE 0 END)/SUM(repeat_passenger_count)*100,2),"%") AS  "10-Trips"
FROM dim_repeat_trip_distribution r 
JOIN dim_city c ON c.city_id=r.city_id
GROUP BY 1;
```

Business Request - 4: Identify Cities with Highest and Lowest Total New Passengers <br>
Generate a report that calculates the total new passengers for each city and ranks them based on this value. Identify the top 3 cities with the highest number of new passengers as well as the bottom 3 cities with the lowest number of new passengers, categorising them as "Top 3" or "Bottom 3" accordingly.<br>

Fields: city_name ,total_new_passengers ,city_category ("Top 3" or "Bottom 3")

```
WITH cte AS(
SELECT city_name,
       SUM(new_passengers) AS total_new_passengers
FROM fact_passenger_summary p 
JOIN dim_city c ON c.city_id=p.city_id
GROUP BY 1),cte_top AS (
SELECT *,
	  DENSE_RANK() OVER(ORDER BY total_new_passengers DESC) AS rnk
FROM cte
),cte_bottom AS(
SELECT *,DENSE_RANK() OVER(ORDER BY total_new_passengers ASC) AS rnk
FROM cte
)
SELECT city_name,
       total_new_passengers,
       "Top 3" AS city_category
FROM cte_top
WHERE rnk<=3
UNION ALL
SELECT city_name,
       total_new_passengers,
       "Bottom 3" AS city_category
FROM cte_bottom
WHERE rnk<=3;
```

Business Request - 5: Identify Month with Highest Revenue for Each City <br>
Generate a report that identifies the month with the highest revenue for each city. For each city, display the month_name, the revenue amount for that month, and the percentage contribution of that month’s revenue to the city’s total revenue.<br>

Fields: city_name, highest_revenue_month, revenue, percentage_contribution (%)

```
WITH cte AS(
SELECT c.city_name,
       MONTHNAME(date) AS month_name,
       SUM(fare_amount) AS revenue,
       DENSE_RANK() OVER(PARTITION BY city_name ORDER BY SUM(fare_amount) DESC) AS rnk
FROM fact_trips t 
JOIN dim_city c ON c.city_id=t.city_id
GROUP BY 1,2)
SELECT c.city_name,
	   month_name AS highest_revenue_month,
       revenue,
       ROUND((revenue/total_revenue_per_city)*100,2) AS "%_contribution"
FROM cte c 
JOIN (SELECT c.city_name,
             SUM(fare_amount) AS total_revenue_per_city
FROM fact_trips t 
JOIN dim_city c ON c.city_id=t.city_id
GROUP BY 1) x ON c.city_name=x.city_name
WHERE rnk=1;
```

Business Request - 6: Repeat Passenger Rate Analysis <br>
Generate a report that calculates two metrics:<br>
1. Monthly Repeat Passenger Rate: Calculate the repeat passenger rate for each city and month by comparing the number of repeat passengers to the total passengers.<br>
2. City-wise Repeat Passenger Rate: Calculate the overall repeat passenger rate for each city, considering all passengers across months. These metrics will provide insights into monthly repeat trends as well as the overall repeat behaviour for each city.<br>

Fields: city_name, month, total_passengers, repeat_passengers, monthly_repeat_passenger_rate (%): Repeat passenger rate at the city and month level, city_repeat_passenger_rate (%): Overall repeat passenger rate for each city,aggregated across months

```
WITH cte AS(
SELECT c.city_name,
       MONTHNAME(month) AS month,
       SUM(total_passengers) total_passengers,
	   SUM(repeat_passengers) AS repeat_passengers,
       ROUND((SUM(repeat_passengers)/SUM(total_passengers))*100,2) AS monthly_repeat_passenger_rate
FROM fact_passenger_summary ps
JOIN dim_city c ON ps.city_id=c.city_id
GROUP BY 1,2)
SELECT c.city_name,
       month,
       c.total_passengers,
       c.repeat_passengers,
       monthly_repeat_passenger_rate,
       city_repeat_passenger_rate
FROM cte c 
JOIN (SELECT c.city_name,
			 SUM(total_passengers) total_passengers,
			 SUM(repeat_passengers) AS repeat_passengers,
			 ROUND((SUM(repeat_passengers)/SUM(total_passengers))*100,2) AS city_repeat_passenger_rate
	  FROM fact_passenger_summary ps
	  JOIN dim_city c ON ps.city_id=c.city_id
	  GROUP BY 1) x ON c.city_name=x.city_name;
```




