## Tasks

### Problem 1
Please show any calculations. In the following we do not expect exact numerical answers,

but we do expect clear and correct reasoning. Aside from calculations, please keep answers< 500 words.

1. You are asked to design a migration strategy for a telematics provider. The objective is to grab data from the provider API and insert it into a local table or a table in the cloud.

2.  Data becomes available on the API in real time at at rate of 1000 - 10000
packets / second with a mean of 200 / packets / second in any one hour
period. 
-
```TEXT
so max= 3600 * 200 = 720,000 packets /hour.
```
3. The API will not allow more than 100 calls / minute, and serves out a maximum of 250 packets / call.

```
6000 calls x 250 = 1,500,000 / hour. 
```
4. The migration will make repeated APIcalls on a schedule to transport packets that have recently become available.

If cost is no object, design a simple migration schedule to handle the load.

What if it costs 0.1 cent / call (even if it returns empty) - how would the design be modified to control costs? 
___

### Solution


**Strategy with Cost Constraints (0.1 cent per call - Cost-Effective Approach):**

1.  **Batching Calls:**
    
    -   Instead of making individual API calls for each packet, batch multiple packets into a single API call.
    -   Calculate the batch size to maximize the 100-call per minute limit. For example, if each call can fetch 250 packets, batch 4 packets into each call (4 x 250 = 1000 packets per call).
2.  **Scheduled Batching:**
    
    -   Schedule API calls at regular intervals, say every 5 seconds, to stay within budget.
    -   Each call fetches a batch of data (1000 packets per call) to optimize the cost per call.
3.  **Parallelism:**
    
    -   Implement parallel processing to fetch and process multiple batches simultaneously.
    -   Ensure that parallelism does not exceed the API rate limit of 100 calls per minute.
4.  **Data Buffering:**
    
    -   Maintain a buffer to temporarily store fetched data until it reaches a reasonable batch size or the scheduled call time.
    -   Process and insert the data from the buffer into the local or cloud table.

This strategy allows you to minimize the number of API calls, reducing costs, while still efficiently handling data retrieval. It balances cost control with data processing efficiency by optimizing the batch size and call frequency.

**Strategy with Cost Constraints (0.1 cent per call):**

-   To control costs, schedule API calls every 5 seconds, ensuring you stay within budget (100 calls per minute).
-   Query data efficiently to minimize empty returns.
-   Implement a call frequency that matches the mean data rate:
    -   Calculation: 1000 packets per second (min rate) / 200 packets per second (mean) = 5 calls per second.
-   Define the latency as the time from data availability to transport, and calculate the worst-case latency:
    -   In the worst-case scenario (all 720,000 packets available at 10,000 packets per second), it takes 72 seconds to transport the hourly bandwidth.
    
These strategies ensure efficient data retrieval and cost control, aligning with operational and financial goals.

---
**(ii)** Define the latency as the time from when a packet becomes available to the time it is transported by an API call. Then the worst case latency is what happens when all 720000 packets become available at 10000 packets / second ie. in 72 seconds, maxing out the hourly bandwidth.

1.  Number of API calls needed to transport all packets:
    
    -   720,000 packets / 1,000 packets per call = 720 calls
2.  Time taken to make 720 calls (seconds):
    
    -   720 calls * 5 seconds per call = 3,600 seconds
3.  Convert the time to minutes (since there are 60 seconds in a minute):
    
    -   3,600 seconds / 60 seconds per minute = 60 minutes

So, with this strategy, the best worst-case latency that can be achieved is 60 minutes, even in a scenario where all 720,000 packets become available in 72 seconds at the maximum data rate.

---
Suppose the maximum rate of availability on the API is less than 500 packets / second in 90 percent of any random 5 minute periods. Estimate the worst case latency that could be achieved 90 percent of the time?
1.  Determine the worst-case data rate:
    
    -   During the 10 percent of the time when the data rate is at its highest, it is less than 500 packets per second. We'll assume it's 500 packets per second to calculate the worst-case latency.
2.  Number of API calls needed to transport all packets:
    
    -   In the worst case, the hourly bandwidth is 500 packets per second * 3600 seconds per hour = 1,800,000 packets.
        
    -   Number of API calls needed: 1,800,000 packets / 1,000 packets per call = 1,800 calls.
        
3.  Time taken to make 1,800 calls (seconds):
    
    -   1,800 calls * 5 seconds per call = 9,000 seconds.
4.  Convert the time to minutes (since there are 60 seconds in a minute):
    
    -   9,000 seconds / 60 seconds per minute = 150 minutes.

So, in the worst-case scenario during the 10 percent of the time when the data rate is at its highest, the worst-case latency that could be achieved is approximately 150 minutes. This estimate accounts for the variation in data rate as specified.

---
**(iii)** Suppose the migration pauses for 15 minutes but data continues to arrive. How long will
it take to clear the backlog on the average? (based on the migration schedule designed
above)

**Step 1: Calculate the number of API calls during the 15-minute pause:**

-   There are 60 seconds in a minute, so 15 minutes = 15 * 60 = 900 seconds.
-   API calls are made every 5 seconds, so during the pause, 900 seconds / 5 seconds per call = 180 API calls would have been scheduled.

**Step 2: Calculate the number of packets that would have arrived during the pause:**

-   Given the data rate of 200 packets per second, during 15 minutes, 200 packets/second * 60 seconds/minute * 15 minutes = 180,000 packets would have arrived.

**Step 3: Calculate how many of these packets can be handled by the 180 API calls:**

-   Each API call fetches a batch of 1,000 packets.
-   So, the number of packets that can be handled by 180 API calls is 180 * 1,000 = 180,000 packets.

**Step 4: Calculate the remaining backlog after the 15-minute pause:**

-   Subtract the number of handled packets from the arrived packets: 180,000 packets - 180,000 packets = 0 packets remaining in the backlog.

**Step 5: Calculate the time to clear the remaining backlog using the regular schedule:**

-   We can calculate the time to clear 0 packets using the regular schedule, which is one batch every 5 seconds.
-   Since there are no packets remaining, it will take 0 time to clear the backlog using the regular schedule.

So, after the 15-minute pause, it will take virtually no time to clear the backlog on average using the designed migration schedule, as the backlog will be efficiently handled by the regular schedule of API calls.

---
 Data is ingested at a mean rate of 5 mb / minute (over a day) and stored in a buffer with maximum capacity X mb. The peak ingestion rate is 20 mb /minute in any hour. The rate of consumption from the buffer is 12 mb / minute. Estimate the minimum X can be to avoid buffer overflow. Give quantitative reasoning.

1.  **Calculate the Daily Ingested Data:**
    
    -   Over a day, the mean ingestion rate is 5 mb/minute.
    -   In one hour, there are 60 minutes.
    -   So, daily ingestion = 5 mb/minute * 60 minutes/hour * 24 hours/day = 7,200 mb (or 7.2 GB) of data ingested.
2.  **Determine the Buffer's Purpose:**
    
    -   The buffer is used to accommodate bursts in data ingestion when the rate peaks at 20 mb/minute. Its purpose is to prevent data loss during these bursts.
3.  **Calculate the Maximum Buffer Fill During a Burst:**
    
    -   During a burst hour, 20 mb/minute * 60 minutes/hour = 1,200 mb (or 1.2 GB) of data can be ingested in one hour.
4.  **Ensure the Buffer Can Accommodate the Peak Hour Data:**
    
    -   To avoid overflow during the peak ingestion hour, the buffer capacity must be at least equal to the maximum data that can be ingested during that hour.
    -   So, the minimum buffer capacity (X) required to handle the peak hour without overflow is 1,200 mb (or 1.2 GB).

In conclusion, to avoid buffer overflow while handling bursts of data ingestion at a peak rate of 20 mb/minute in any hour, the minimum buffer capacity (X) should be at least 1,200 mb (or 1.2 GB). This ensures that the buffer can store the data during the peak hour without data loss.

---

### Problem 2

**Database Structure:**

-   Columns: `rtc_time` (integer), `created_at` (timestamp), `car_id` (integer), `data` (json).
-   Data Format: The `data` column contains JSON objects which can have various combinations of fields. Examples include:
    -   `{'x': n1, 'y': n2, 'z': n3}`
    -   `{'w': m1, 'v': m2, 'u': m3}`
    -   `{'x': n1, 'y': n2, 'u': m3}`
    -   Other combinations are possible, indicating variability in data fields.

**Issue Identification:**

-   Duplicate Data: Occasional duplicates are sent with identical `car_id`, `rtc_time`, and `data` values within a short time window (e.g., 30 minutes).
-   Uniqueness Challenge: Identical `car_id` and `rtc_time` can contain multiple distinct `data` values. This complicates identifying duplicates.
-   Dictionary Order Variability: Due to indeterminate dictionary order, simple text-based indexing is ineffective.
-   Performance Concern: Brute force comparisons may hinder performance, especially as the table size increases.

**Objective:**

-   Develop an efficient method to process only distinct data rows on readout.
-   The solution may include table modifications, indexing, software strategies, or a combination of these.

**Constraints and Assumptions:**

-   Distinct data snippets for the same vehicle and timestamp do not have overlapping fields.
-   Need to account for performance efficiency as the database grows.

---
### Solution

### Table Modification:

1.  **Introduce a Unique Identifier Column:**
    -   Add a new column to the table, let's call it `data_hash`, to store a unique identifier for each data snippet.
    -   The `data_hash` can be generated by hashing the JSON data in the `data` column using a cryptographic hash function (e.g., SHA-256). This ensures a unique hash for each distinct data snippet.

### Indexes:

2.  **Create an Index on `data_hash`:**
    -   Create a database index on the `data_hash` column to speed up searches based on the hash values.
    -   Indexes provide efficient lookup for distinct values, improving query performance.

### Software Strategy:

3.  **Data Ingestion and Duplication Handling:**
    -   During data ingestion, calculate and store the `data_hash` for each data snippet.
    -   Before inserting a new row into the table, check if a row with the same `car_id`, `rtc_time`, and `data_hash` already exists.
    -   If a duplicate is detected within a short time window (dT=30 min), discard the duplicate data snippet.
    -   Ensure that the timestamp `rtc_time` and `created_at` are accurate and represent the actual data capture time, not the processing time.

### Performance Considerations:

4.  **Partitioning and Archiving:**
    
    -   As the table grows, consider partitioning it based on a time range (e.g., monthly partitions) to improve query performance.
    -   Implement data archiving strategies to move older data to archival storage to keep the active table size manageable.
5.  **Regular Maintenance:**
    
    -   Periodically run maintenance routines to optimize table indexes and remove outdated data that is no longer needed for analysis.
6.  **Scaling Resources:**
    
    -   If performance issues persist, consider scaling the database resources (e.g., adding more CPU or memory) or using database sharding to distribute the data load across multiple servers.

By implementing this scheme, you ensure that only distinct data rows are processed on readout while efficiently handling duplicates and considering the performance implications as the table size increases. The use of hashed unique identifiers and proper indexing can significantly improve query performance, even with large datasets.

---
### Problem 3

-   Checking data for vehicles in a shop using the Vehicle Identification Number (VIN).
-   Database Structure:
    -   `car` table contains `id` and `vin`.
    -   `car_shop` table contains `id_shop` and `id_car`.

**Method 1: Direct PostgreSQL Queries**

-   Step 1: Retrieve `id` and `vin` from `car` table.

       ```SQL
    SELECT id, vin FROM car JOIN car_shop ON car.id = car_shop.id_car WHERE id_shop = 123
    ```  
-   Step 2: For each `vin`, query `scanner_data_trip`.
    
    
    ```SQL
    SELECT * FROM scanner_data_trip WHERE vin = vin_value
    ``` 
    

**Method 2: Using Python (or another programming language)**

-   Step 1: Query `car_shop` table for `id_shop=123`.
    
    ``` python
    q = 'SELECT * FROM car_shop WHERE id_shop = 123'
    res = db.query(q)
    idlist = [r['id_car'] for r in res]
    ```
-   Step 2: Retrieve `vin` from `car` table using `idlist`.
    ``` python
    q1 = 'SELECT vin FROM car WHERE id IN (insert_id_list)'
    res1 = db.query(q1)` 
    ```

-   Step 3: Query `scanner_data_trip` for list of `vins`.
    ``` python
    q2 = 'SELECT * FROM scanner_data_trip WHERE vin IN (insert_list_of_vins)'
    res2 = db.query(q2)
    ```
-   Step 4: Filter `trip_list_this_vin` for each `vin`.
    ``` python
    trip_list_this_vin = [r['trip_data'] for r in res2 if r['vin'] = vin_value]
    ```
    
---
### Solution
1.  **Sequential SQL Queries Approach:**
    
    -   Pros:
        -   Simple and straightforward.
        -   Minimal memory usage per query.
    -   Cons:
        -   Requires multiple round trips to the database, leading to increased latency.
        -   Inefficient for large datasets or many vehicles.
        -   Increased network overhead due to multiple queries.
2.  **Bulk SQL Queries with Python Approach:**
    
    -   Pros:
        -   Reduces the number of round trips to the database, improving query efficiency.
        -   Reduces network overhead as you make fewer database calls.
        -   More efficient for large datasets or when dealing with many vehicles.
    -   Cons:
        -   Potentially higher memory usage as you fetch all results at once, especially with a large number of vehicles.
3.  **Performance Considerations:**
    
    -   On a local machine, differences may not be as pronounced for small to medium datasets.
    -   In the cloud, Approach 2 becomes more advantageous due to potential network latency.
    -   For clusters making multiple calls, Approach 2 is generally more efficient, reducing database load and network traffic.
    -   For many large fleets, Approach 2 remains more efficient as it scales better, leveraging database optimizations and reducing round trips.

In most scenarios, especially in the cloud or with multiple calls and large datasets, Approach 2 (Bulk SQL Queries with Python) offers better performance, effectively balancing memory and network considerations while reducing database load.

---
### Problem

For the most part, fleets provide their telematics provider credentials to us to faciliate migration onto our platforms. Sometimes, due to 3rd party changes these credentials might suddenly stop working. The symptom is a cessation of data flow that might not be noticed for hours or days. Of course, data stoppages might also occur because of new bugs in our code, version changes in modules and because vehicles do not drive on national holidays. 

What approach would you take to quickly detect and remediate this kind of data interruption? 

We need to 
- (i) notice something is wrong 
-  (ii) determine the root cause 
- (iii) deploy a fix or communicate with the customer Alarms and logs are useful, but demand constant vigilance. Discuss

---

### Solution

To quickly detect and remediate data interruptions in telematics data flow due to credential issues or other factors, you can implement a proactive monitoring and alerting system. While alarms and logs are valuable, they do require constant vigilance. Here's an approach to address this challenge:

1.  **Automated Monitoring and Alerts:**
    -   Set up an automated monitoring system that continuously checks the health and flow of data.
    -   Establish alerts for key performance indicators (KPIs) that can signal data interruption. Examples include:
        -   Data volume drops below a certain threshold.
        -   A sudden decrease in the number of active vehicles transmitting data.
        -   A significant increase in error rates or failed data ingestion attempts.
        -   Unsuccessful API requests or authentication failures.
2.  **Real-time Monitoring Dashboard:**
    -   Create a real-time monitoring dashboard that provides visibility into the current state of data flow, system performance, and key metrics.
    -   Include visualizations and charts that highlight any anomalies or deviations from normal data flow patterns.
3.  **Scheduled Health Checks:**
    -   Implement scheduled health checks that periodically verify the validity of credentials and the connectivity of each fleet's telematics provider.
    -   These checks can include validating API endpoints, authentication tokens, and testing data transmission.
4.  **Root Cause Analysis (RCA):**
    -   When an alert is triggered, initiate an automated RCA process to determine the root cause of the data interruption.
    -   RCA can involve checking logs, error messages, and historical data to pinpoint the issue.
    -   Implement AI or machine learning algorithms to identify patterns or anomalies that could be contributing to the interruption.
5.  **Automated Remediation:**
    -   Where possible, automate the remediation process to address common issues without human intervention.
    -   For example, if an authentication failure is detected, attempt to renew or request new credentials automatically.
6.  **Escalation and Communication:**
    -   Define an escalation process to notify the appropriate team members or administrators when a data interruption is detected.
    -   Implement an alerting hierarchy that ensures the right individuals are informed based on the severity of the issue.
    -   Establish clear communication channels to inform customers about the interruption, its root cause, and the expected resolution time.
7.  **Continuous Improvement:**
    -   Regularly review and update the monitoring system, alerts, and RCA procedures based on past incidents and evolving threats.
    -   Use historical data to refine detection algorithms and identify potential issues before they become critical.
8.  **Redundancy and Failover:**
    -   Implement redundancy and failover mechanisms in your data ingestion architecture to minimize the impact of interruptions.
    -   Consider multiple data sources or backup systems to ensure continuous data flow, even if one source experiences issues.

By implementing these proactive measures, you can reduce the reliance on constant vigilance and improve your ability to notice, diagnose, and remediate data interruptions promptly. This approach helps maintain data integrity and customer satisfaction while minimizing downtime.
