---
layout: post
title: "Austin Animal Shleter"
subtitle: "Animal shelter analysis in SQL server"
background: '/img/posts/austin_shelter.jpg'
---
### About
The Austin Animal Center in Austin, Texas is the largest no-kill animal shelter in the United States. Every year, they provide care and shelter for over 18000 animals. Their goal is to place all adoptable animals in forever homes.

**What is the average time(days) an animal stays in shelter?**
```sql
SELECT AVG(cast(datediff(d,intak.datetime,ou.datetime) as bigint)) AS Avg_Time_at_shelter FROM aac_intakes$ AS intak
LEFT JOIN aac_outcomes$ AS ou
	ON intak.animal_id = ou.animal_id;
```
**Average time(days) spent in shleter by each animal types?**
```sql
SELECT intak.animal_type, AVG(CAST(datediff(d,intak.datetime,ou.datetime) as bigint)) AS Avg_Time_at_shelter FROM aac_intakes$ AS intak
LEFT JOIN aac_outcomes$ AS ou
	ON intak.animal_id = ou.animal_id
GROUP BY intak.animal_type;
```
**Average time in shelter by age and animal type?**
```sql
SELECT intak.animal_type,intak.age_upon_intake, AVG(CAST(datediff(d,intak.datetime,ou.datetime) as bigint)) AS Avg_Time_at_shelter FROM aac_intakes$ AS intak
LEFT JOIN aac_outcomes$ AS ou
	ON intak.animal_id = ou.animal_id
GROUP BY intak.age_upon_intake,intak.animal_type;
```
**Percentage of animals recieved by animal type?**
```sql
SELECT int.animal_type,
		CAST(COUNT(*)*100.0/sum(count(*)) over() AS decimal(18,2)) AS Percentage_Intake
FROM aac_intakes$ AS int
GROUP BY int.animal_type;
```
**Count of each Outcome type of animals?**
```sql
SELECT out.outcome_type, COUNT(*) AS total_count FROM aac_outcomes$ AS out
GROUP BY out.outcome_type
ORDER BY total_count DESC ;
```
**Which month has the most intakes?**
```sql
SELECT DATENAME(MONTH, datetime) AS MONTH, COUNT(*) AS No_of_Intakes
FROM aac_intakes$ 
GROUP BY  DATENAME(MONTH, datetime)
ORDER BY No_of_Intakes DESC;
```
**Which month has the most outcomes?**
```sql
SELECT DATENAME(MONTH, datetime) AS MONTH, COUNT(*) AS No_of_outcomes
FROM aac_outcomes$
GROUP BY DATENAME(MONTH, datetime)
ORDER BY No_of_outcomes DESC;
```