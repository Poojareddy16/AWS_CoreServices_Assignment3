# 🧾 AWS Core Services Assignment – ITCS 6190  
**Name:** Sai Pooja Buttamgari  
**Course:** Big Data Analytics (ITCS 6190)  
**University:** UNC Charlotte  
**Semester:** Fall 2025  

---

## 🧠 Objective  
The goal of this project was to design and implement an end-to-end AWS data pipeline using **Amazon S3**, **AWS Lambda**, **AWS Glue**, **Amazon Athena**, and **EC2**, to automate ETL (Extract-Transform-Load) processing and visualize Athena query results through a webpage hosted on an EC2 instance.  

---

## ⚙️ Workflow Overview  

1. **Amazon S3** – stores raw, processed, and enriched datasets.  
2. **AWS Lambda** – triggered automatically when a file is uploaded to the `raw/` folder; processes and cleans data, then outputs to `processed/`.  
3. **AWS Glue Crawler** – catalogs the processed data to make it queryable in Athena.  
4. **Amazon Athena** – executes SQL queries on cataloged data and saves results in `enriched/`.  
5. **Amazon EC2 (Apache)** – hosts the final webpage displaying query results.  

---

## 🪣 1️⃣ Amazon S3 Bucket Structure  
**Bucket Name:** `amz3bucket`
## 🗂️ S3 Bucket Structure
```bash
amz3bucket/
├── raw/           ← Orders.csv uploaded here (triggers Lambda)
├── processed/     ← Lambda output files stored here
└── enriched/      ← Athena query CSV results exported here
```

<img width="959" height="442" alt="S3_bucketStructure" src="https://github.com/user-attachments/assets/4dfc6971-a34a-4e77-bcb3-06945ec7a175" />


---

## 🧾 2️⃣ IAM Roles Created  
| Role Name | Attached Policies | Purpose |
|------------|-------------------|----------|
| `lambda-s3-processing-role` | AmazonS3FullAccess, AWSLambdaBasicExecutionRole | Allows Lambda to read/write S3 and log to CloudWatch. |
| `Glue-S3-Crawler-Role` | AmazonS3FullAccess, AWSGlueServiceRole, AWSGlueConsoleFullAccess | Enables Glue Crawler to read S3 and update Data Catalog. |


<img width="952" height="527" alt="IAM roles" src="https://github.com/user-attachments/assets/f8ed9875-23f6-414e-972c-2da15d86a5d2" />


---

## ⚡ 3️⃣ AWS Lambda Function – FilterAndProcessOrders  
- **Function Name:** `FilterAndProcessOrders`  
- **Runtime:** Python 3.9  
- **Trigger:** S3 (raw folder upload event)  
- **Execution Role:** `lambda-s3-processing-role`

<img width="958" height="427" alt="lambda_function" src="https://github.com/user-attachments/assets/ff97f392-b09d-4655-a0b7-a483a29fa272" />


---

## 🔔 4️⃣ Configured S3 Trigger  
- S3 (raw folder) event triggers Lambda automatically.
- Lambda processes data and saves it in `processed/`.

<img width="751" height="362" alt="Configure_trigger" src="https://github.com/user-attachments/assets/6ba67be1-0c46-4a68-aab3-ea75a87119ce" />

---

## 🧮 5️⃣ Processed File in S3 (processed/)  
- After uploading `Orders.csv` into `raw/`, Lambda processed and saved the output to `processed/`.

<img width="1906" height="995" alt="image" src="https://github.com/user-attachments/assets/00b5805d-e680-4fc0-9523-64a434cff802" />


---

## 🔍 6️⃣ AWS Glue Crawler Logs  
- **Crawler Name:** `orders_processed_crawler`  
- Successfully created table `processed` inside database `orders_db`.

<img width="1890" height="984" alt="image" src="https://github.com/user-attachments/assets/eebd0835-48b1-4c16-965e-a849c6191562" />

---

<img width="943" height="469" alt="crawler_cloudwatch" src="https://github.com/user-attachments/assets/1ae80872-6cca-4fcd-bf72-adb8d8767a75" />

---

## 📊 7️⃣ Amazon Athena Query Results  

**Database:** `orders_db`  
**Table:** `processed`  
**Columns:** `orderid`, `customer`, `amount`, `status`, `orderdate`  

---

### 🧩 Query 1 – Total Sales by Customer
```sql
SELECT 
    customer,
    SUM(amount) AS total_sales
FROM processed
GROUP BY customer
ORDER BY total_sales DESC;
```

### Output:
<img width="953" height="478" alt="s31" src="https://github.com/user-attachments/assets/7e249565-956d-4f85-97e7-f992504c4027" />



### 📅 Query 2 – Monthly Order Volume and Revenue
```sql
SELECT 
    date_format(date_parse(orderdate, '%Y-%m-%d'), '%Y-%m') AS month,
    COUNT(orderid) AS total_orders,
    SUM(amount) AS total_revenue
FROM processed
GROUP BY date_format(date_parse(orderdate, '%Y-%m-%d'), '%Y-%m')
ORDER BY month;
```

### Output:
<img width="947" height="479" alt="s32" src="https://github.com/user-attachments/assets/ac425784-fa11-4730-aa1a-a1c3eb79f60b" />



### 🚚 Query 3 – Order Status Dashboard

```sql
SELECT 
    status,
    COUNT(orderid) AS total_orders,
    SUM(amount) AS total_sales
FROM processed
GROUP BY status
ORDER BY total_orders DESC;
```

### Output:

<img width="945" height="485" alt="s3_3" src="https://github.com/user-attachments/assets/14fcaf6a-4c16-499d-bbe8-983c2976d062" />



### 💵 Query 4 – Average Order Value (AOV) per Customer

```sql
SELECT 
    customer,
    AVG(amount) AS avg_order_value
FROM processed
GROUP BY customer
ORDER BY avg_order_value DESC;
```

### Output:

<img width="941" height="464" alt="s3_4" src="https://github.com/user-attachments/assets/8a97f6d2-6a6e-481d-94fd-bca9bca1e9e3" />


### 🏆 Query 5 – Top 10 Largest Orders in February 2025

```sql
SELECT orderid, customer, amount, orderdate
FROM processed
WHERE date_format(date_parse(orderdate, '%Y-%m-%d'), '%Y-%m') = '2025-02'
ORDER BY amount DESC
LIMIT 10;

```
<img width="1879" height="985" alt="image" src="https://github.com/user-attachments/assets/3093f741-c5f9-4f16-a431-77ce490db23a" />



## 💾 8️⃣ Athena Query Results Saved to S3 (enriched/)

Athena outputs are automatically saved to the S3 `enriched/` folder as CSV files after each query execution.
These files contain the query results from Amazon Athena, ready for further analysis or dashboard integration.

<img width="1919" height="994" alt="image" src="https://github.com/user-attachments/assets/8afdb9d4-3a43-4a00-8908-e0bd7193ddf2" />



---

## 💻 9️⃣ EC2 Instance Connection (SSH)

- **Instance Type:** t2.micro  
- **AMI:** Amazon Linux 2023  
- **Public IPv4:** 18.217.38.52  

Connected successfully to the EC2 instance using the command below:

```bash
ssh -i "dashboard.pem" ec2-user@18.217.38.52
```
After connecting, the EC2 terminal displays:

<img width="637" height="372" alt="dashboard_ec2" src="https://github.com/user-attachments/assets/67202666-46bc-4e05-9620-97c468890fab" />

## 🌐 🔟 Final Webpage Hosted on EC2

After connecting to the EC2 instance, the Apache web server was installed and started successfully.

### 🧩 Commands Used:
```bash
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```
### 📝 HTML File Created:

```bash
echo "<html>
<head><title>Welcome to My AWS Project</title></head>
<body style='font-family:Arial; text-align:center; background-color:#006241; color:white; padding:50px;'>
<h1>🌟 Hi, I'm Sai Pooja!</h1>
<h2>Welcome to My Cloud Adventure 🚀</h2>
<p>This webpage is proudly hosted on an <b>AWS EC2 instance</b> using the <b>Apache Web Server</b>.</p>
<p>Behind the scenes, I used <b>Amazon S3</b>, <b>Lambda</b>, <b>Glue</b>, and <b>Athena</b> to process and analyze my dataset.</p>
<hr style='margin:40px 0; border:1px solid #ffffff;'>
<p><i>Created by Sai Pooja Buttamgari<br>ITCS 6190 – Big Data Analytics</i></p>
</body>
</html>" | sudo tee /var/www/html/index.html
```

### 🌈 Output:
Access the EC2 public IP in your browser:
http://18.217.38.52/

<img width="958" height="527" alt="dashboard-final" src="https://github.com/user-attachments/assets/60118579-02c0-49fb-b1db-2c2ece0cd29d" />

