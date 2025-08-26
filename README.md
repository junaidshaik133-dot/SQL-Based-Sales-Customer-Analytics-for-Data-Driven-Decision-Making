#📊 Data-Driven Decision Making using SQL  
This repository contains the SQL scripts used to analyze customer, product, and sales data for AtliQ. The project addresses several business requests focused on sales trends, customer insights, product performance, and channel contribution, helping the company make data-driven strategic decisions.


 #🔎 Introduction  
Data-driven decision making is a critical component in today’s competitive market.  
This project aims to analyze AtliQ’s internal sales database to provide insights into:  

- Yearly and regional sales trends  
- High-performing products and customers  
- Growth of product segments across fiscal years  
- Contribution of sales channels to overall revenue

  By leveraging SQL analytics, the goal is to generate actionable insights and help optimize **sales, resource allocation, and customer engagement strategies.

 #📂 Data Sources  
The analysis is based on data obtained from AtliQ’s internal SQL database.  
The main datasets used include:  

- **fact_sales_monthly** → Monthly sales transactions  
- **fact_gross_price** → Gross product pricing details  
- **fact_manufacturing_cost** → Manufacturing cost per product  
- **fact_pre_invoice_deductions** → Discounts and deductions data

  #📌 Project Overview  
- Analyzed sales and customer data using SQL queries.  
- Addressed **10 business requests** to generate reports and insights.  
- Findings help inform **future strategies, product planning, and sales optimization
- **dim_customer** → Customer details (market, region, channel)  
- **dim_product** → Product details (segment, division, category)

 #🏷️ Business Requests  
     1️⃣ Yearly Report for Croma
       Objective: Generate a yearly sales report for the customer Croma with fiscal year and gross sales amount.
       
               SELECT p.fiscal_year,
               SUM(s.sold_quantity * p.gross_price) AS yearly_gross_sales
               FROM dim_customer c
               JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
               JOIN fact_gross_price p ON s.product_code = p.product_code
               WHERE c.customer = 'Croma'
               GROUP BY p.fiscal_year;

   2️⃣ Unique Products Sold per Fiscal Year  
Objective: Count the number of distinct products sold each fiscal year. 

               SELECT fiscal_year,
                    COUNT(DISTINCT product_code) AS distinct_product_count
                    FROM fact_sales_monthly
                     GROUP BY fiscal_year;
                     
3️⃣ Markets for “Atliq Exclusive” in APAC
         Objective: Identify all markets where Atliq Exclusive operates in the APAC region.

          SELECT DISTINCT market
           FROM dim_customer
           WHERE customer = 'Atliq Exclusive' 
            AND region = 'APAC';


4️⃣ Product Count by Segment  
Objective: List unique product counts for each segment, sorted by highest count.

              SELECT segment,
                 COUNT(DISTINCT product) AS product_count
                 FROM dim_product
                  GROUP BY segment
                   ORDER BY product_count DESC;

  5️⃣ Segment Growth (2020 vs 2021)  
   Objective: Identify the segment with the most increase in unique products between 2020 and 2021.

     WITH segment_counts AS (
    SELECT s.fiscal_year, p.segment,
           COUNT(DISTINCT s.product_code) AS product_count
    FROM fact_sales_monthly s
    JOIN dim_product p ON s.product_code = p.product_code
    GROUP BY s.fiscal_year, p.segment
                                         )
             SELECT segment,
               MAX(CASE WHEN fiscal_year=2020 THEN product_count ELSE 0 END) AS product_count_2020,
              MAX(CASE WHEN fiscal_year=2021 THEN product_count ELSE 0 END) AS product_count_2021,
             (MAX(CASE WHEN fiscal_year=2021 THEN product_count ELSE 0 END) -
              MAX(CASE WHEN fiscal_year=2020 THEN product_count ELSE 0 END)) AS difference
                FROM segment_counts
                  GROUP BY segment
                   ORDER BY difference DESC; 

  6️⃣ Highest & Lowest Manufacturing Costs
      Objective: Get products with the highest and lowest manufacturing costs.

           SELECT p.product, p.product_code, m.manufacturing_cost
                FROM fact_manufacturing_cost m
                 JOIN dim_product p ON m.product_code = p.product_code
                 WHERE manufacturing_cost = (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost)
                 OR manufacturing_cost = (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost);      


   7️⃣ Top 5 Customers with High Discounts (India, 2021)  
      Objective: Find top 5 customers who received a higher-than-average discount percentage in 2021 (India market).

                   WITH discounts AS (
                    SELECT d.customer_code, c.customer, d.pre_invoice_discount_pct
                      FROM fact_pre_invoice_deductions d
                       JOIN dim_customer c ON d.customer_code = c.customer_code
                        WHERE d.fiscal_year = 2021 AND c.market = 'India'
                                                                        )
                          SELECT TOP 5 customer_code, customer, pre_invoice_discount_pct
                           FROM discounts
                            WHERE pre_invoice_discount_pct >= (SELECT AVG(pre_invoice_discount_pct) FROM fact_pre_invoice_deductions)
                             ORDER BY pre_invoice_discount_pct DESC; 
 

   8️⃣ Gross Sales of “Atliq Exclusive” by Year  
      Objective: Generate yearly gross sales for Atliq Exclusive.

                            SELECT s.fiscal_year, c.customer,
                                 CAST(SUM(s.sold_quantity * g.gross_price) AS DECIMAL(10,2)) AS total_sales
                                 FROM dim_customer c
                                   JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
                                    JOIN fact_gross_price g ON s.product_code = g.product_code
                                      WHERE c.customer = 'Atliq Exclusive'
                                        GROUP BY s.fiscal_year, c.customer;

                                        
          

   9️⃣ Top Sales Channel (2021)  
            Objective: Identify the channel contributing the highest sales in 2021 with % contribution.

                         WITH channel_sales AS (
                          SELECT c.channel, SUM(s.sold_quantity * p.gross_price) AS total_sales
                           FROM dim_customer c
                            JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
                             JOIN fact_gross_price p ON s.product_code = p.product_code
                               WHERE s.fiscal_year = 2021
                               GROUP BY c.channel
                                             )
                                    SELECT TOP 1 channel, total_sales,
                                     ROUND(total_sales * 100.0 / SUM(total_sales) OVER(), 2) AS percentage_contribution
                                           FROM channel_sales
                                            ORDER BY total_sales DESC;  


🔟 Top 3 Products per Division (2021)
Objective: Get top 3 products in each division by sold quantity.

                                WITH ranked_products AS (
                                    SELECT p.division, p.product_code, p.product, s.sold_quantity,
                                   ROW_NUMBER() OVER (PARTITION BY division ORDER BY sold_quantity DESC) AS rank
                                    FROM dim_product p
                                     JOIN fact_sales_monthly s ON p.product_code = s.product_code
                                     WHERE s.fiscal_year = 2021
                                                                       )
                                          SELECT division, product_code, product, sold_quantity
                                           FROM ranked_products
                                            WHERE rank <= 3;  
 
                                           
 
