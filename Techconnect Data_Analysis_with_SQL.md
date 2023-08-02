# Data Analysis with SQL

![source: The Rise of SQL - IEEE Spectrum](https://assets.rbl.ms/31141774/origin.jpg)

# Intro

In this project, we are going to analyze data for a telecom company called techconnect. At the end of this project, you will have a fair understanding of how to approach data analysis when working with a large database.


## Why Use SQL?

You can use python alone to do almost anything when it comes to data analysis and data science. From data preparation, visualization and even machine learning, python is a very reliable tool for analyzing thousands of rows of data, but when you have data containing millions and millions of rows is where you start running into problems. Let me explain.

With more data involved, python libraries become less and less efficient for data analysis. Have you ever run code in jupyter notebook and it takes like 3minutes to execute? This kind of inefficiency is what I’m talking about. So that’s where SQL comes in. SQL was created to analyze millions and trillions of rows of data in the most efficient way possible. It can be used to get insights from a large database in seconds! 


## The database

For this project, we will use [**Microsoft SQL Server**](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) to run all our SQL code. Connect to SQL Server using the following information:

- **Server Name:** dap-projects-database.database.windows.net
- **Username:** hackathon
- **Password:** A$uB1SQ!H@ck@f3$t2023
- **Database Name:** dapDB

You will find a database named **dapDB** with 3 tables named, **customers**, **location**, and **current_offer_balance**.


# Case Study

Allow me to introduce my company called techconnect. TechConnect is a provider of mobile services, offering voice, data, SMS, and various value-added services to its vast customer base.
Over the past 5 years, the company has garnered a reputation for its reliable network coverage and innovative service offerings, attracting customers from all walks of life.

To maintain our position in the highly competitive market, TechConnect focuses on data-driven decision-making and customer-centric strategies. This commitment led to the creation of an extensive database to capture and analyze crucial information about our customers and operations.


# Dataset Description
## Table: dbo.current_officers_balance 

The "current_officers_balance" table was the backbone of TechConnect's data infrastructure. It stored detailed information about each customer's account and usage patterns. The table held records of customer IDs, call durations, data consumption, messaging activities, revenue details, and various other metrics. This data allowed TechConnect to gain insights into customer behavior, identify potential upselling opportunities, and provide personalized services.


## Table: dbo.customers 

The "customers" table contained a comprehensive list of all TechConnect's customers. It included activation dates, tenure information, and spending habits of each customer. This data helped the company understand the customer lifecycle, loyalty trends, and tailor marketing strategies based on customer preferences and behavior.


## Table: dbo.location 

The "location" table held vital information about the geographical distribution of customers and the network infrastructure. It stored site IDs, regions, and cell types associated with specific cellular towers and base stations. 

This data will enable TechConnect to optimize network coverage, identify potential network expansion areas, and assess regional performance.
TechConnect's data analysts and business intelligence teams (You😉) will work diligently to transform raw data into actionable insights. The insights generated from the database will play a pivotal role in shaping the company's decisions, product offerings, and marketing campaigns.


# Data Analysis

The company is still in it's early stages, so even though we are making money and we have achieved product-market-fit (creating a product or service that people are willing to pay for), we must ask important questions that will ensure that the company stays afloat. 

The single most important question we need to answer is: Why exactly are people paying for our services, what exactly are they paying for, and who are our customers exactly? In order to reduce cost and make more money, we must pinpoint exactly what our customers need the most from us. This is the goal of any business analysis. Let's dive into it!


# Questions
## Customer spending behaviour

**1.0 How do customers spend their money on different services such as voice, data, SMS, and other offerings. Identify the most popular services among customers.**

```sql
    SELECT
        'Voice' AS Service,
        SUM(VOICE_SPENT) AS TotalSpent
    FROM
        dbo.customers
    UNION ALL
    SELECT
        'Data' AS Service,
        SUM(DATA_SPENT) AS TotalSpent
    FROM
        dbo.customers
    UNION ALL
    SELECT
        'SMS' AS Service,
        SUM(SMS_SPENT) AS TotalSpent
    FROM
        dbo.customers
    UNION ALL
    SELECT
        'Other Services' AS Service,
        SUM(OTHER_SPENT) AS TotalSpent
    FROM
        dbo.customers
    ORDER BY TotalSpent DESC;
 ```   

**1.1 Result**
Voice is the most popular service that our customers spend money on. This means our customers mostly make calls, followed by sending text messages, followed by buying data.

![Result of the query](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690800814376_image.png)


**You only have read permissions, however, right-click on the results to save the results as a CSV file for your visualizations.**


![Column chart of spending Vs service](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690801010136_image.png)


Since voice is the most popular service, lets take a closer look at the customers that make calls.

**2.0 What is the device_type of customers that spend money to make calls?**

```sql
     --Voice Spending Variation Based on Customer's Device Type
    
    -- Query to get voice spending for smartphone and non-smartphone users
    SELECT 
        CASE WHEN dbo.location.cell_type = 2 THEN 'Smartphone' --assuming 2 is smartphone
                     WHEN dbo.location.cell_type = 3 THEN 'Non-Smartphone'
                     ELSE 'Other-Devices' --null values as other-devices
                     END AS device_type,
        SUM(VOICE_SPENT) AS sum_voice_spent
            --AVG(VOICE_SPENT) AS average_voice_spent we can't use mean cuz of outliers
    FROM dbo.customers
    LEFT JOIN dbo.location 
    ON dbo.customers.Cell_ID = dbo.location.CELL_ID
    GROUP BY dbo.location.cell_type;
```
**2.1 Result**
Majority of our voice customers are smartphone users.

![Result of the query](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690801899882_image.png)

![Column chart of Voice_spent Vs Device_type](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690802153801_image.png)


**3.0 Top recharge method used by customers**

```sql
    --top recharge method used by customers
    SELECT 
           SUM(RECHARGE_EVD_AMOUNT) AS evd,
           SUM(RECHARGE_MOMO_AMOUNT) AS momo,
           SUM(RECHARGE_CARD_AMOUNT) AS recharge_card
    FROM dbo.customers;
```
**3.1 Result**
Most customers use recharge card as their recharge method

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690804128234_image.png)


**4.0 Can existing customers be upsold to higher-value data plans?** 

```sql
    SELECT TOP 10
        Customer_ID,
        SUM(Data_Vol_MB) AS Total_Data_MB,
        SUM(TOTAL_RECHARGE_AMOUNT) AS TOTAL_RECHARGE_AMOUNT
    FROM dbo.customers
    GROUP BY Customer_ID
    HAVING SUM(Data_Vol_MB) = 0
    ORDER BY TOTAL_RECHARGE_AMOUNT DESC;
```
**4.1 Result**
Yes, some customers have high recharge amount but no data at all, so they can be convinced to buy data.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690905679619_image.png)


It's good to know what customers want, but where exactly is the money coming from🤔
Now let's look at the MONEY.🤑


## Main sources of revenue

**1.0 Who are our best customers?**

```sql
    --Top 5 customers by total revenue:
    SELECT TOP 5
        cob.Customer_ID AS top_customers_by_revenue,
        SUM(cob.tot_rev) AS total_revenue,
            SUM(cust.VOICE_SPENT + cust.DATA_SPENT + cust.SMS_SPENT + cust.OTHER_SPENT) AS total_spending
    FROM dbo.current_offers_balance AS cob
    JOIN dbo.customers cust ON cob.Customer_ID = cust.Customer_ID
    GROUP BY cob.Customer_ID
    ORDER BY total_revenue DESC;
    
    --Top 5 customers by spending
    SELECT TOP 5
        cob.Customer_ID AS top_customers_by_spending,
        SUM(cob.tot_rev) AS total_revenue,
        SUM(cust.VOICE_SPENT + cust.DATA_SPENT + cust.SMS_SPENT + cust.OTHER_SPENT) AS total_spending
    FROM dbo.current_offers_balance AS cob
    JOIN dbo.customers cust ON cob.Customer_ID = cust.Customer_ID
    GROUP BY cob.Customer_ID
    ORDER BY total_spending DESC;
```
**1.1 Result**
From the tables below, our top spenders don’t make us the most money, but customers who make us the most money (total_revenue) may not be the best spenders. This goes to show that revenue does not correlate with spending.

![top customers by revenue and spending](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690804850211_image.png)


**2.0 What are the main sources of revenue for the company.**

```sql
    --Revenue breakdown by revenue types
    SELECT
        SUM(cob.rev_data_total) AS data_revenue,
        SUM(cob.rev_voice_int) AS voice_revenue,
        SUM(cob.rev_other_vas) AS vas_revenue,
        SUM(cob.rev_rentals) AS rentals_revenue
    FROM dbo.current_offers_balance cob;
```
**2.1 Result**
Voice is the major revenue source of the company. However, we are loosing a lot of money from rentals. Meaning our customers borrow data, voice and value added services from us, this borrowing of services is causing us to loose potential revenue.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690806251053_image.png)


**3.0 Do we make more money from incoming calls and texts sent to our customers, or otherwise?**

```sql
    --percentage of total revenue from services coming in Vs out
    SELECT
            (SUM(tot_rev_in) / SUM(tot_rev)) * 100 AS total_rev_in_percentage,
            (SUM(tot_rev_out) / SUM(tot_rev)) * 100 AS total_rev_out_percentage
    FROM dbo.current_offers_balance;
```
**3.1 Result**
We make more money from outgoing services; when our customers send text messages to outside networks, or when our customers make calls to other networks.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690808573921_image.png)


**4.0 How much money have we lost from customers who left (churned).**

```sql
    -- Calculate total revenue for each distinct value of status (pie chart)
    SELECT
        'ACTIVE' AS status, SUM(tot_rev) AS tot_rev
    FROM dbo.current_offers_balance
    WHERE status = 'ACTIVE'
    
    UNION ALL
    
    SELECT
        'DORMANT & INACTIVE' AS status, SUM(tot_rev) AS tot_rev
    FROM dbo.current_offers_balance
    WHERE status IN ('DORMANT', 'INACTIVE')
    
    UNION ALL
    
    SELECT
        'CHURNED' AS status, SUM(tot_rev) AS tot_rev
    FROM dbo.current_offers_balance
    WHERE status IN ('CHURNED', 'CHURN');
```
**4.1 Result**
From the table below, it appears that we didn’t make much money from customers who left or churned; compared to revenue from our active customers.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690811913336_image.png)


**5.0 Do we make more money from international services and transactions?**

```sql
    --international vs domestic voice revenue
    SELECT
        SUM(rev_voice_int) + SUM(rev_voice_roam_incoming) + SUM(rev_voice_roam_outgoing) AS total_international_revenue,
        SUM(tot_rev) - (SUM(rev_voice_int) + SUM(rev_voice_roam_incoming) + SUM(rev_voice_roam_outgoing)) AS total_domestic_revenue
    FROM dbo.current_offers_balance;
```
**5.1 Result**
No, we make more money locally.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690813283095_image.png)


**6.0 Which handset models bring more revenue?**

```sql
    -- Calculate total revenue for each distinct value of hs_model
    SELECT TOP 5 hs_model, SUM(tot_rev) AS total_revenue
    FROM dbo.current_offers_balance
    GROUP BY hs_model
    ORDER BY SUM(tot_rev) DESC;
```
**6.1 Result**
We make the most money from android smartphone users.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690815183689_image.png)



## Revenue and customer engagement performance across different regions. 

**1.0 Compare the spending patterns of customers in different regions and identify which services are more popular in each region.**

```sql
    --Compare the spending patterns of customers in different regions and
    -- Identify which services are more popular in each region
    SELECT TOP 5
        loc.region,
        SUM(cust.VOICE_SPENT) AS total_voice_spent,
        SUM(cust.DATA_SPENT) AS total_data_spent,
        SUM(cust.SMS_SPENT) AS total_sms_spent,
        CASE
            WHEN SUM(cust.VOICE_SPENT) > SUM(cust.DATA_SPENT)
                AND SUM(cust.VOICE_SPENT) > SUM(cust.SMS_SPENT)
                AND SUM(cust.VOICE_SPENT) > SUM(cust.OTHER_SPENT) THEN 'Voice'
            WHEN SUM(cust.DATA_SPENT) > SUM(cust.VOICE_SPENT)
                AND SUM(cust.DATA_SPENT) > SUM(cust.SMS_SPENT)
                AND SUM(cust.DATA_SPENT) > SUM(cust.OTHER_SPENT) THEN 'Data'
            WHEN SUM(cust.SMS_SPENT) > SUM(cust.VOICE_SPENT)
                AND SUM(cust.SMS_SPENT) > SUM(cust.DATA_SPENT)
                AND SUM(cust.SMS_SPENT) > SUM(cust.OTHER_SPENT) THEN 'SMS'
            ELSE 'Other'
        END AS popular_service
    FROM
        dbo.customers AS cust
    JOIN
        dbo.location AS loc ON cust.Cell_ID = loc.CELL_ID
    GROUP BY
        loc.region
    ORDER BY
            total_voice_spent DESC; --we order by voice since voice is the most popular service across all regions;
```
**1.1 Result**
The top regions are the “Greater Accra Region” and the “Ashanti Region” by spending. Voice is the most popular service for all regions. 


![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690817725718_image.png)


Spending rate might not determine revenue, meaning that we might make more money from regions where costumers spent less in total. So let us look at regional revenue.

**2.0 What are the most profitable regions?**

```sql
    --Compare the total_revenue of customers in different regions
    SELECT TOP 5
            l.region AS Region, 
            SUM(cob.tot_rev) AS Total_Revenue
    FROM dbo.current_offers_balance cob
    JOIN dbo.customers cu ON cu.Customer_ID = cob.Customer_ID
    JOIN dbo.location l ON cu.Cell_ID = l.CELL_ID
    GROUP BY l.region
    ORDER BY Total_Revenue DESC;
```
**2.1 Result**
The top regions are the “Greater Accra Region” and the “Western Region” by revenue.


![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690821260388_image.png)



## Analyze customer churn

**1.0 Which regions have the highest churn rate?**

```sql
    --Actionable insight: Use insights from high revenue regions to create personalized offerings and improve customer experiences in other regions.
    --regions with the highest churn rate
    --The Volta Region has the highest churn rate but 3rd highest revenue
    SELECT TOP 5
               l.region,
           AVG(c.tenure) AS avg_tenure,
           SUM(CASE WHEN co.churn_date IS NOT NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(c.customer_id) AS churn_rate_percentage --if there is a date for churn, then true or 1
    FROM dbo.location l
    LEFT JOIN dbo.customers c ON l.cell_id = c.cell_id
    LEFT JOIN dbo.current_offers_balance co ON c.customer_id = co.customer_id
    GROUP BY l.region
    ORDER BY churn_rate_percentage DESC;
```
**1.1 Result**
The Volta Region has the highest churn rate but brings the 3rd highest revenue.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690822551499_image.png)




**2.0 Evaluate the effectiveness of the loyalty program. how customers engage with it.**
The loyalty program was put in place to encourage our active customers to stay with us and continue to pay for our services.

```sql
    SELECT DISTINCT loyalty_points_balance
    FROM dbo.current_offers_balance;
    SELECT DISTINCT loyalty_points_redeemed
    FROM dbo.current_offers_balance;
    SELECT DISTINCT loyalty_points_earned
    FROM dbo.current_offers_balance;
```
**2.1 Result**
Its time to try something other than loyalty points cuz customers have not responding to it. Either that or it was recently introduced and needs some time to kick off.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690845049892_image.png)



## Track the adoption of different mobile network technologies (2G) among customers.

**1.0 Compare the revenue made from customers on 2G networks to the revenue from others.**

```sql
    SELECT '2G' AS nw_2g_ind, SUM(tot_rev) AS Total_Revenue 
    FROM dbo.current_offers_balance WHERE nw_2g_ind = -1
    UNION ALL
    SELECT '3G' AS nw_2g_ind, SUM(tot_rev) AS Total_Revenue 
    FROM dbo.current_offers_balance WHERE nw_2g_ind = 0
    UNION ALL
    SELECT '4G' AS nw_2g_ind, SUM(tot_rev) AS Total_Revenue 
    FROM dbo.current_offers_balance WHERE nw_2g_ind = 1;
```
**1.1 Result**
We should advertise to those using 2G network to upgrade to 3G or 4G.

![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690890809927_image.png)



# Conclusion
## Recommendations

After queries have been written and beautiful dashboards have been built, the single most important thing a data analyst has to give is ***actionable*** insights. So from our analysis, here are some things I recommend:



> **Focus on Voice Services**: Voice is the most popular service among customers, so it is essential to maintain and improve the voice service quality. Additionally, consider introducing voice-based value-added services to further engage customers.
> 
> **Target Smartphone Users**: Majority of the voice customers are smartphone users. This presents an opportunity to create smartphone-specific offers, promotions, and services to cater to this segment.
> 
> **Promote Data Plans to High-Recharge Customers**: Identify customers who have high recharge amounts but do not use data. Target these customers with personalized offers to upsell higher-value data plans.
> 
> **Optimize Recharge Methods**: As most customers use recharge cards, consider promoting mobile money and electronic voucher distribution (EVD) methods to increase convenience and accessibility.
> 
> **Address Churn Rate in Volta Region**: The Volta Region has the highest churn rate despite being a high-revenue region. Investigate the reasons behind the churn and implement strategies to improve customer retention in this area.
> 
> **Reduce Losses from Rentals**: Analyze the rental services and identify ways to reduce losses from customers borrowing services. Encourage customers to opt for regular data and voice plans instead of rentals.
> 
> **Upgrade 2G Customers**: Encourage customers on 2G networks to upgrade to 3G or 4G to capitalize on the revenue potential from these advanced services.


## [Dashboard](https://app.powerbi.com/view?r=eyJrIjoiYjAzNGYxMGMtNTY3Mi00Y2I4LTlhYTYtMTkzNGRlZThjZGI0IiwidCI6ImI4MGJiZGQ4LWFlM2UtNDY1Yy1hNDcwLWMwODgyYTUxYWViNyIsImMiOjh9)
![](https://paper-attachments.dropboxusercontent.com/s_F4F6DBB24B644CD4CDCAB0AB1595055B51F16902764960AF9034E891578F3388_1690959363473_ezgif.com-optimize.gif)



## Resources
- [SQL Tutorial - Full Database Course for Beginners](https://youtu.be/HXV3zeQKqGY)
- [Why learn SQL if you can use python?](https://youtube.com/shorts/DWte9zjGl0I?feature=share)
- [Practice SQL on HackerRank](https://www.hackerrank.com/domains/sql?filters%5Bstatus%5D%5B%5D=unsolved&badge_type=sql)
- [Introduction to Power BI](https://youtu.be/KPc6950u0TE) 
- [Installing Microsoft SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

