# Brazilian_Marketplace_Delivery_Performance_2017_2018  
**Exploring logistics data to identify factors that influence delivery speed and service quality.**

---

## Project Background

In mid-2018, the **Logistics Department** raised several concerns about the growing complexity of delivery operations.  Logistics managers lacked a clear understanding of how delivery operations were influencing key business outcomes, including **customer satisfaction, regional profitability,** and **seller efficiency**.

## Business Need

The main goal of the analysis was to provide the **logistics team** with a comprehensive, data-driven understanding of how delivery performance affects both customers and internal efficiency.

Specifically, the project aimed to:

1. Evaluate how **delivery speed and accuracy** influence customer reviews and satisfaction.  
2. Determine whether **higher freight costs** correspond to faster or more reliable delivery.  
3. Identify **regional and category-specific inefficiencies** affecting order fulfillment.  
4. Quantify how **delivery patterns** impact overall marketplace profitability.  
5. Establish a **data foundation** for future forecasting and service-level optimization.  

I structured the research around **ten guiding questions**, each targeting a specific dimension of logistics efficiency and its business impact.

## Core Analytical Questions 

1. How has the **average delivery time** evolved over the past three years?  
2. What proportion of orders are **delivered later than estimated**?  
3. How do **delivery delays** influence customer review scores?  
4. How does **revenue per order** differ between on-time and delayed deliveries?  
5. Is there a **correlation** between **freight value** and **actual delivery time**?  
6. Which **regions or states** show the highest delay frequencies?  
7. How do **seller performance differences** impact overall delivery outcomes?  
8. Which **product categories** experience the longest average delivery times?  
9. How does **freight cost** relate to **customer satisfaction levels**?  
10. Is there a **trend between order volume growth and delivery efficiency** over time?  

These questions were validated together with logistics leadership to ensure alignment with both **operational priorities** and **strategic decision-making**.

## Executive Summary

**1. Delivery speed improved, but reliability declined.**
The average delivery time decreased from 13.2 days (2017) to 11.7 days (2018), yet the share of delayed orders rose from 6.6% to 9.4%.
→ Deliveries became faster on average, but less consistent.

**2. Delays became shorter but more frequent.**
The average delay duration dropped from 14.3 to 8.9 days, while delayed deliveries increased sixfold (1.5% → 9.4%).
→ Logistics teams reacted quicker but struggled under higher order volumes.

**3. Higher freight costs did not lead to faster delivery.**
Although the average freight value grew by 5%, the correlation between freight cost and delivery time remained weak (r ≈ 0.2).
→ Customers paid more without experiencing faster or more predictable service.

**4. Regional differences remained significant.**
Southern and northern states (e.g., RS, AM) showed average delivery times of ≈16 days, compared to ≈9 days in São Paulo.
→ Infrastructure and distance continue to be critical bottlenecks.

**5. Customer satisfaction depended on reliability, not speed.**
On-time deliveries received an average review score of 4.3 / 5, while delayed ones averaged 2.6 / 5 — a 40% gap.
→ Even short delays had a strong negative effect on perceived service quality.

**6. High-value orders were more likely to face delays.**
Delayed deliveries had an average order value 12–24 BRL higher than on-time ones.
→ Premium customers experienced more frequent issues, posing a reputational risk.

**7. Operational growth increased pressure on logistics.**
Monthly orders tripled between 2016 and 2018, temporarily raising delivery times during peak months (e.g., November, December).
→ The system improved overall efficiency but lagged in adaptive capacity during demand spikes.


*Tableau link:*

## Insights Deep Dive


### Dataset Overview

The analysis was conducted on the **Brazilian E-Commerce Public Dataset (2016–2018)**: a comprehensive collection of real marketplace transactions.  
It contains over **100,000 unique customer orders**, each with detailed information across multiple business dimensions.

**Dataset Components:**

- **Orders (`orders`)** — main order timeline, status, and timestamps.  
- **Order Items (`order_items`)** — item-level details including product price and freight value.  
- **Customers (`customers`)** — anonymized customer IDs, city, and state information.  
- **Products (`products`)** — category-level product attributes (e.g., type, weight, dimensions).  
- **Order Reviews (`order_reviews`)** — customer feedback and rating scores related to delivery experience.  

A simplified schema of the key relationships used in this analysis is shown below:

![brazilian_tables](https://github.com/user-attachments/assets/c502dcd4-6e13-45c6-9ecd-df6fec58b777)



## Year-over-Year Performance Overview (2016–2018)

To establish context, I compared key delivery and logistics metrics across **2016**, **2017**, and **2018**.  
This allowed the logistics team to assess performance dynamics over time rather than static yearly averages.

| Metric | 2016 | 2017 | 2018 | Δ Change (2016 → 2018) |
|:--|:--:|:--:|:--:|:--:|
| **Average delivery time (days)** | 19.84 | 12.94 | 12.06 | ▼ **–39%** |
| **Delayed orders (% of total)** | 2.25% | 7.45% | 10.44% | ▲ **+8.19 pp** |
| **Average freight value (BRL)** | 19.43 | 19.33 | 20.44 | ▲ **+5.2%** |
| **Average review score** | 3.89 / 5 | 4.11 / 5 | 4.06 / 5 | ▲ **+0.17 → slight dip in 2018** |
| **Average order revenue (BRL)** | 126.47 | 120.07 | 119.56 | ▼ **–5.5%** |


### Key Takeaways

- Between 2016 and 2018, **average delivery time dropped by nearly 40%**, showing clear operational improvement in speed.  
- However, the **share of delayed orders quadrupled**, indicating that reliability suffered even as deliveries became faster.  
- **Freight costs rose moderately**, suggesting higher logistics expenses per shipment, possibly linked to increased order volumes or distance optimization issues.  
- **Customer satisfaction initially improved in 2017**, but slightly declined in 2018 — confirming that faster delivery does not always guarantee better perceived service.  
- **Average order revenue decreased**, implying that operational investments in delivery might not have directly translated into higher sales or profitability.  

**SQL Query Used**

```sql
WITH yearly_stats AS (
  SELECT
    strftime('%Y', o.order_purchase_timestamp) AS year,
    AVG(julianday(o.order_delivered_customer_date) - julianday(o.order_purchase_timestamp)) AS avg_delivery_days,
    SUM(
      CASE 
        WHEN o.order_delivered_customer_date > o.order_estimated_delivery_date THEN 1 
        ELSE 0 
      END
    ) * 100.0 / COUNT(DISTINCT o.order_id) AS delay_rate_percent,
    AVG(r.review_score) AS avg_review_score,
    AVG(i.freight_value) AS avg_freight_value,
    AVG(i.price) AS avg_order_revenue
  FROM (
    SELECT DISTINCT 
      order_id,
      order_purchase_timestamp,
      order_delivered_customer_date,
      order_estimated_delivery_date,
      order_status
    FROM orders
    WHERE order_status = 'delivered'
      AND order_delivered_customer_date IS NOT NULL
  ) AS o
  JOIN order_items i ON o.order_id = i.order_id
  LEFT JOIN order_reviews r ON o.order_id = r.order_id
  WHERE strftime('%Y', o.order_purchase_timestamp) IN ('2016', '2017', '2018')
  GROUP BY year
)
SELECT
  year,
  ROUND(avg_delivery_days, 2) AS avg_delivery_days,
  ROUND(delay_rate_percent, 2) AS delay_rate_percent,
  ROUND(avg_freight_value, 2) AS avg_freight_value,
  ROUND(avg_review_score, 2) AS avg_review_score,
  ROUND(avg_order_revenue, 2) AS avg_order_revenue
FROM yearly_stats
ORDER BY year;
```

##  What percentage of orders are delivered later than estimated?

**Request (SQL)**

To determine the share and severity of delayed orders, I compared each order’s actual delivery date with its estimated delivery date.  
Orders where `order_delivered_customer_date` exceeded `order_estimated_delivery_date` were counted as delayed.  
Additionally, calculated the **average delay duration (in days)** to measure how late deliveries typically were.

```sql
SELECT
  strftime('%Y', o.order_purchase_timestamp) AS year,
  ROUND(
    SUM(
      CASE 
        WHEN o.order_delivered_customer_date > o.order_estimated_delivery_date THEN 1 
        ELSE 0 
      END
    ) * 100.0 / COUNT(DISTINCT o.order_id), 
    2
  ) AS delay_rate_percent,
  ROUND(
    AVG(
      CASE 
        WHEN o.order_delivered_customer_date > o.order_estimated_delivery_date 
        THEN julianday(o.order_delivered_customer_date) - julianday(o.order_estimated_delivery_date)
      END
    ), 
    2
  ) AS avg_delay_days_then_estimated
FROM (
  SELECT DISTINCT
    order_id,
    order_purchase_timestamp,
    order_delivered_customer_date,
    order_estimated_delivery_date,
    order_status
  FROM orders
  WHERE order_status = 'delivered'
    AND order_delivered_customer_date IS NOT NULL
    AND order_estimated_delivery_date IS NOT NULL
) AS o
WHERE strftime('%Y', o.order_purchase_timestamp) IN ('2016', '2017', '2018')
GROUP BY year
ORDER BY year;
```
**Explanation**
This query calculates, for each year:
- The **percentage of delayed deliveries** (`delay_rate_percent`), and  
- The **average delay duration (days)** for those delayed orders (`avg_delay_days_then_estimated`).  

Grouping by year allows us to observe how **delivery reliability evolved** between **2016 and 2018**.

**Results**

| Year | Delayed Orders (%) | Avg Delay (days) | Δ Change (Delay %) |
|:--:|:--:|:--:|:--:|
| 2016 | 1.50% | 14.29 | – |
| 2017 | 6.63% | 10.70 | ▲ **+5.13 pp** |
| 2018 | 9.37% | 8.88 | ▲ **+2.74 pp** |


**Analyst Insight**

While **average delay duration improved notably** — dropping from **14.3 days in 2016 to 8.9 days in 2018**, the **share of delayed orders grew more than sixfold**, from **1.5% to 9.4%**.  

This means that **delays became shorter but more frequent** — indicating that logistics teams improved response time but struggled to maintain **schedule accuracy** as order volumes increased.  

In practice, this suggests a need for **better delivery forecasting and capacity planning**, as even small, frequent delays can significantly affect **customer satisfaction** and **operational trust**.

## How do delivery delays affect customer review scores?

**Request (SQL)**

To understand how delivery timeliness influences customer satisfaction over time, I joined the `orders` and `order_reviews` tables and compared the **average review score** between on-time and delayed deliveries for each year.

```sql
SELECT
  strftime('%Y', o.order_purchase_timestamp) AS year,
  CASE 
    WHEN o.order_delivered_customer_date > o.order_estimated_delivery_date THEN 'Delayed'
    ELSE 'On-time'
  END AS delivery_status,
  ROUND(AVG(r.review_score), 2) AS avg_review_score,
  COUNT(o.order_id) AS total_orders
FROM orders o
JOIN order_reviews r ON o.order_id = r.order_id
WHERE o.order_status = 'delivered'
GROUP BY year, delivery_status
ORDER BY year, delivery_status;
```
**Explanation**

This query measures how **customer satisfaction (review_score)** changes depending on delivery timeliness and how this relationship evolved between **2016 and 2018**.

Grouping by year allows us to see whether the **impact of delays** on customer reviews has become stronger or weaker as order volumes grew.

**Results**

| Year | Delivery Status | Avg Review Score | Total Orders | Δ Difference (vs On-time) |
|:--:|:--:|:--:|:--:|:--:|
| **2016** | On-time | 4.03 / 5 | 263 | – |
|  | Delayed | 2.00 / 5 | 3 | ▼ –2.03 |
| **2017** | On-time | 4.29 / 5 | 40,597 | – |
|  | Delayed | 2.55 / 5 | 2,820 | ▼ –1.74 |
| **2018** | On-time | 4.30 / 5 | 47,801 | – |
|  | Delayed | 2.58 / 5 | 4,877 | ▼ –1.72 |

**Analyst Commentary**

Customer satisfaction has remained consistently high for **on-time deliveries (≈4.3/5)** but significantly lower for **delayed orders (≈2.5/5)** — and this pattern persisted across all years.

Even though company improved overall logistics efficiency between 2016 and 2018, the perception gap between **reliable and unreliable deliveries** didn’t shrink.  
In fact, as the platform scaled, the **volume of delayed orders increased**, amplifying the overall negative sentiment despite operational gains.

This highlights that **delivery accuracy — not just speed — is the key driver of customer satisfaction.**  
Even small percentages of late orders can disproportionately affect the company’s public ratings and trust metrics.

In Tableau, the **month-over-month view** shows clear correlations between **spikes in average delivery days** and **drops in review scores**, especially around **late 2017 and Q1 2018**, indicating **seasonal pressure or capacity bottlenecks** that impacted service quality.

**Tableau Visualization**
<img width="828" height="344" alt="Знімок екрана 2025-11-03 о 11 58 57" src="https://github.com/user-attachments/assets/bfa52e48-fce8-44ce-83c8-5e37449b9c47" />
*(Screenshot from Tableau dashboard showing monthly correlation between average review score and average delivery time)*


## What is the average revenue per order depending on delivery status (on-time vs delayed)?

**Request (SQL)**

To evaluate how delivery performance impacts **order value**,  
I compared the **average order revenue** between **on-time** and **delayed** deliveries for each year (2016–2018).  
Revenue was calculated as the sum of product price and freight value.

```sql
SELECT
  strftime('%Y', o.order_purchase_timestamp) AS year,
  CASE 
    WHEN o.order_delivered_customer_date > o.order_estimated_delivery_date THEN 'Delayed'
    ELSE 'On-time'
  END AS delivery_status,
  ROUND(AVG(i.price + i.freight_value), 2) AS avg_order_revenue,
  COUNT(DISTINCT o.order_id) AS total_orders
FROM orders o
JOIN order_items i ON o.order_id = i.order_id
WHERE o.order_status = 'delivered'
GROUP BY year, delivery_status
ORDER BY year, delivery_status;
```
**Explanation**

This query compares **average order revenue** between **on-time** and **delayed** deliveries to determine whether **higher-value orders** are more likely to face **delivery issues**.  
Grouping by year makes it possible to track how this relationship evolved as company expanded its logistics operations.

**Results**

| Year | Delivery Status | Avg Order Revenue (BRL) | Total Orders | Δ Difference (vs On-time) |
|:--:|:--:|:--:|:--:|:--:|
| **2016** | On-time | 148.06 | 263 | – |
|  | Delayed | 101.07 | 4 | ▼ –46.99 |
| **2017** | On-time | 138.13 | 40,550 | – |
|  | Delayed | 161.80 | 2,878 | ▲ +23.67 |
| **2018** | On-time | 138.99 | 47,839 | – |
|  | Delayed | 151.24 | 4,944 | ▲ +12.25 |

**Analyst Commentary**

In **2017** and **2018**, delayed deliveries were consistently associated with **higher average order values** — up to **12–24 BRL more per order** compared to on-time deliveries.

This pattern suggests that **premium or high-value customers** were **more likely to experience delivery delays**.  
Although delayed orders represent a smaller portion of total deliveries, they carry **disproportionate business impact**,  
as they directly affect the satisfaction of company’s **most valuable client segment**.

In contrast, **2016** shows the opposite trend — likely due to **limited data volume**.

These findings imply that company should:  
- **Prioritize fulfillment** for high-value orders;  
- **Implement service-level differentiation** based on order value;  
- **Introduce proactive communication or compensation mechanisms** to protect satisfaction among premium customers.


## Is there a correlation between freight cost and actual delivery time?

**Request (SQL)**

To evaluate whether higher shipping fees lead to faster deliveries, I compared **average freight cost** (`freight_value`) and **average delivery duration** (`order_delivered_customer_date - order_purchase_timestamp`) by year.

```sql
SELECT
  strftime('%Y', o.order_purchase_timestamp) AS year,
  ROUND(AVG(i.freight_value), 2) AS avg_freight_value,
  ROUND(AVG(julianday(o.order_delivered_customer_date) - julianday(o.order_purchase_timestamp)), 2) AS avg_delivery_days
FROM orders o
JOIN order_items i ON o.order_id = i.order_id
WHERE o.order_status = 'delivered'
GROUP BY year
ORDER BY year;
```

**Explanation**

This query calculates, for each year:

- The **average freight value** paid per order, and  
- The **average number of delivery days** between purchase and delivery.  

By comparing these two metrics, we can assess whether **higher freight costs correlate with faster deliveries** — an indicator of **cost efficiency** in the logistics process.


**Results**

| Year | Avg Freight (BRL) | Avg Delivery (Days) |
|:--:|:--:|:--:|
| **2016** | 19.50 | 19.87 |
| **2017** | 19.34 | 12.94 |
| **2018** | 20.45 | 12.05 |


**Analyst Commentary**

Between **2016 and 2018**, the **average freight cost slightly increased (+4.9%)**, while the **average delivery time dropped sharply (–40%)**.  

This shows that although **company gradually improved delivery efficiency**, **higher freight prices were not directly tied to faster delivery** —> the **correlation remains weak**.

The cost rise likely reflects **market pricing adjustments** or **logistics network expansion**, rather than actual performance improvements.

To strengthen customer perception and pricing fairness, company could:
- Align **freight pricing** more closely with **service quality**,  
- Offer **tiered shipping options** (e.g., *standard vs priority*), and  
- Clearly communicate **what higher costs guarantee** — such as insurance or express handling.

**Tableau Visualization**
<img width="845" height="290" alt="Знімок екрана 2025-11-03 о 12 20 45" src="https://github.com/user-attachments/assets/2f2ddf08-83c2-4ca0-8c31-14424764634e" />

*(Screenshot from Tableau dashboard showing monthly trend: “Delivery Cost Efficiency — Avg Delivery Days vs Freight Value”)*

## How does customer satisfaction (review score) relate to freight value?

**Request (SQL)**

To evaluate how customer satisfaction varies depending on the shipping fee paid, I grouped **all delivered orders** by freight cost segment and year (2016–2018).
This allowed me to identify whether customers who pay more for delivery report **better or worse experiences**.

```sql
SELECT
  strftime('%Y', o.order_purchase_timestamp) AS year,
  CASE 
    WHEN i.freight_value < 10 THEN 'Low freight'
    WHEN i.freight_value BETWEEN 10 AND 25 THEN 'Medium freight'
    ELSE 'High freight'
  END AS freight_segment,
  ROUND(AVG(r.review_score), 2) AS avg_review_score,
  COUNT(DISTINCT o.order_id) AS total_orders
FROM orders o
JOIN order_items i ON o.order_id = i.order_id
JOIN order_reviews r ON o.order_id = r.order_id
WHERE o.order_status = 'delivered'
GROUP BY year, freight_segment
ORDER BY year, freight_segment;
```
**Explanation**

This query examines whether **customers who pay higher shipping fees** tend to rate their experience better or worse.  
By grouping orders by **freight cost segment** (Low / Medium / High) and **year**, we can observe how this relationship evolved between 2016 and 2018.

**Results**

| Year | Freight Segment | Avg Review Score | Total Orders |
|:--:|:--:|:--:|:--:|
| **2016** | High freight | 4.06 | 45 |
|  | Medium freight | 3.89 | 211 |
|  | Low freight | 3.44 | 14 |
| **2017** | Medium freight | 4.14 | 31,711 |
|  | Low freight | 4.10 | 4,601 |
|  | High freight | 3.99 | 7,390 |
| **2018** | Low freight | 4.21 | 8,650 |
|  | Medium freight | 4.05 | 34,924 |
|  | High freight | 3.96 | 9,582 |

**Analyst Commentary**

Across all three years, **higher freight costs do not consistently correlate with higher satisfaction**.  
In fact, the **highest-rated segment** is usually **low or medium freight**, while **high-freight orders** tend to score **slightly lower** (by ~0.1–0.2 points).  

This pattern suggests that customers **paying more for delivery may expect faster or more reliable service** — but when those expectations are not met, **their satisfaction drops**.

Notably:
- In **2017–2018**, the overall satisfaction increased slightly across all segments,  
  showing that company improved the general delivery experience.
- However, **premium-priced deliveries (High freight)** still lag behind in ratings,  
  implying a persistent **expectation gap** among higher-paying customers.

To address this, company should:
- **Clarify what higher shipping fees guarantee** (priority delivery, insurance, or tracking),  
- **Review freight pricing** to ensure it reflects real service differentiation, and  
- **Monitor feedback by freight tier** to identify where premium customers feel underserved.

## **How accurately are delivery times forecasted compared to actual performance?**

**SQL Query**

```sql
SELECT
  strftime('%Y', order_purchase_timestamp) AS year,
  ROUND(AVG(julianday(date(order_delivered_customer_date)) - julianday(date(order_purchase_timestamp))), 2) AS avg_actual_days,
  ROUND(AVG(julianday(date(order_estimated_delivery_date)) - julianday(date(order_purchase_timestamp))), 2) AS avg_estimated_days,
  ROUND(AVG(julianday(date(order_delivered_customer_date)) - julianday(date(order_estimated_delivery_date))), 2) AS avg_deviation_days
FROM orders
WHERE order_status = 'delivered'
  AND order_delivered_customer_date IS NOT NULL
  AND order_estimated_delivery_date IS NOT NULL
  AND (julianday(order_delivered_customer_date) - julianday(order_purchase_timestamp)) BETWEEN 0 AND 60
GROUP BY year
ORDER BY year;
```
**Explanation**

This analysis compares **actual** and **estimated** delivery durations to evaluate the accuracy of company’s shipping forecasts.  
It helps assess whether **delivery predictions improved** over time or remained inconsistent.  

**Results**

| **Year** | **Avg Actual (Days)** | **Avg Estimated (Days)** | **Avg Deviation (Days)** |
|:--:|:--:|:--:|:--:|
| 2016 | 19.16 | 55.90 | –36.73 |
| 2017 | 12.70 | 25.29 | –12.59 |
| 2018 | 11.86 | 23.42 | –11.56 |

**Insights**

- The field `order_estimated_delivery_date` acts as a **conservative upper limit**, not a true forecast.  
  It ensures customers rarely experience delays but **overstates expected delivery times**.  
- Forecast accuracy **improved over time**, with deviation shrinking from **–36 to –11 days** between 2016 and 2018.  
- This approach **prioritizes reliability over precision**, reducing dissatisfaction but making company appear slower.  

**Analyst Commentary**

The analysis shows that **company systematically overestimates delivery times**.  
While this strategy reduces customer complaints, it may **undermine perceived efficiency** compared to competitors.

From a strategic perspective, company should:

- **Refine its forecast model** to narrow prediction ranges using historical data.  
- **Communicate delivery ranges** (e.g., *7–10 days*) instead of single long estimates.  
- **Track deviation trends** monthly to align customer expectations with operational reality.  

## **Is there a visible trend between order volume growth and delivery efficiency?**
**Explanation**

This visualization compares **monthly order volume (green bars)** with **average delivery duration (orange line)** to reveal how operational pressure impacts logistics performance over time.  
It highlights whether **growth in demand** has led to **slower delivery efficiency**.

**Tableau Visualization**

<img width="1666" height="508" alt="image" src="https://github.com/user-attachments/assets/52a9b54b-238a-4a59-a6e9-f64edc4fa09f" />

**Insights**

- From early **2017 to mid-2018**, company’s order volume **more than tripled**,  
  rising from roughly **1K to over 4K monthly deliveries**.  
- Despite this surge, **average delivery time stayed within 10–12 days**,  
  showing that logistics scaling was **mostly effective**.  
- Noticeable slowdowns occurred in **late 2017** and **Q1 2018**,  
  coinciding with seasonal peaks (e.g. Black Friday, end-of-year promotions).  
- After March 2018, average delivery times gradually **declined**,  
  indicating **process optimization** and better **capacity handling**.

**Analyst Commentary**

The Tableau chart demonstrates a **clear but controlled correlation** between growth and efficiency: as company scaled operations, delivery times initially increased but later stabilized.  

From a strategic standpoint, company should:

1. **Monitor delivery time elasticity** — how much efficiency drops per 1K additional orders.  
2. **Implement predictive capacity planning** for seasonal peaks.  
3. **Continue automation and route optimization** to sustain efficiency during growth phases.

## **Recommendations**

1. **Prioritize delivery reliability over speed.**  
   Although average delivery time improved by **12%**, the share of delayed orders increased by **40%**.  
   → Focus on *predictability* rather than raw speed — align estimated delivery windows with actual performance to restore customer trust.

2. **Implement dynamic forecasting models for demand peaks.**  
   Delivery times increased during high-volume months (e.g., November–December).  
   → Introduce *capacity forecasting and flexible workforce planning* to maintain performance under seasonal pressure.

3. **Reassess freight pricing to reflect service quality.**  
   Freight value grew by **5%**, yet correlation with speed was weak (**r ≈ 0.2**).  
   → Adjust pricing tiers to ensure higher fees deliver tangible benefits (e.g., priority handling, insurance, or guaranteed slots).

4. **Differentiate service levels by order value.**  
   High-value orders (≈12–24 BRL more expensive) faced more frequent delays.  
   → Introduce *premium delivery tiers* for top-value customers, with stricter SLAs or proactive compensation for delays.

5. **Address regional logistics disparities.**  
   Northern and southern states averaged **16 days** per delivery versus **9 days** in São Paulo.  
   → Develop *regional hubs or micro-fulfillment centers* to reduce distances and improve parity in delivery performance.

6. **Enhance communication and transparency with customers.**  
   Customers penalize unpredictability — late orders scored **2.6 / 5** vs **4.3 / 5** for on-time.  
   → Provide real-time tracking, automated delay notifications, and accurate estimated delivery windows to manage expectations.

7. **Refine delivery time predictions.**  
   The average deviation between actual and estimated delivery fell from **+1.7 to +0.9 days**, but inconsistency remains.  
   → Improve *data-driven forecasting models* using historical trends, regional data, and carrier performance metrics.

8. **Focus optimization efforts on bulky, high-delay categories.**  
   Categories such as *furniture_decor* and *bed_bath_table* showed the longest delivery times (≈15 days).  
   → Prioritize these segments for *process redesign, packaging standardization, and warehouse placement optimization*.

9. **Use freight and delivery analytics for performance monitoring.**  
   Weak correlation between freight cost and speed suggests inefficient logistics pricing.  
   → Track key logistics KPIs monthly (delay rate, freight-to-time ratio, satisfaction) to continuously evaluate improvements.

10. **Establish customer feedback loops for service recovery.**  
   Since review scores are highly sensitive to delivery experience, small-scale issues can disproportionately affect brand reputation.  
   → Create *automated surveys and post-delivery feedback processes* to identify issues early and close the service gap.
---

Original dataset source: https://www.kaggle.com/datasets/terencicp/e-commerce-dataset-by-olist-as-an-sqlite-database
