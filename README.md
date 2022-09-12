# Amazon_Vine_Analysis

## Overview of the analysis:

The purpose of the analysis is to analyze Amazon reviews written by members of the paid Amazon program. For this to occur a data set from "Amazon Review datasets" will be chosen. To extract the data PySpark was used. The data was than transformed and filter. Than by connectings RDS on AWS to postgres (pgAdmin), the data will be tabulated. From there the final analysis will be performed to determine if a paid Vine review makes a difference in the percentage of 5-star reviews. For this review the use of panda was necessary. This meant exporting the file to be able to perform the analysis.

### Load Amazon Data into Spark DataFrame
```
from pyspark import SparkFiles
url = "https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Wireless_v1_00.tsv.gz"
spark.sparkContext.addFile(url)![table_screenshot](https://user-images.githubusercontent.com/104809098/189554847-b1bcf014-89ce-4017-bb45-202cc2632dd0.png)

df = spark.read.option("encoding", "UTF-8").csv(SparkFiles.get(""), sep="\t", header=True, inferSchema=True)
df.show()
```
### Create DataFrames to match tables

```
# Create the customers_table DataFrame
# customers_df = df.groupby("").agg({""}).withColumnRenamed("", "customer_count")
customers_df = df.groupby("customer_id").agg({"customer_id":"count"}).withColumnRenamed("count(customer_id)", "customer_count")
customers_df.show()

# Create the products_table DataFrame and drop duplicates. 
# products_df = df.select([]).drop_duplicates()
products_df = df.select(['product_id', 'product_title']).drop_duplicates(['product_id'])
products_df.show()

# Create the review_id_table DataFrame. 
# Convert the 'review_date' column to a date datatype with to_date("review_date", 'yyyy-MM-dd').alias("review_date")
# review_id_df = df.select([, to_date("review_date", 'yyyy-MM-dd').alias("review_date")])
review_id_df = df.select(['review_id','customer_id', 'product_id', 'product_parent', to_date("review_date", 'yyyy-MM-dd').alias("review_date")])
review_id_df.show()

# Create the vine_table. DataFrame
# vine_df = df.select([])
vine_df = df.select(['review_id','star_rating','helpful_votes','total_votes','vine','verified_purchase'])
vine_df.show()
```

### Connect to the AWS RDS instance and write each DataFrame to its table

```
# Configure settings for RDS
from getpass import getpass
password = getpass('Enter database password')
mode = "append"
# Configure settings for RDS
mode = "append"
jdbc_url="jdbc:postgresql://dataviz-2.cryag6ji9emi.us-east-1.rds.amazonaws.com:5432/postgres"
config = {"user":"postgres",
          "password": password,
          "driver":"org.postgresql.Driver"}
```          
          
```
# Write review_id_df to table in RDS
review_id_df.write.jdbc(url=jdbc_url, table='review_id_table', mode=mode, properties=config)

# Write products_df to table in RDS
# about 3 min
products_df.write.jdbc(url=jdbc_url, table='products_table', mode=mode, properties=config)

# Write customers_df to table in RDS
# 5 min 14 s
customers_df.write.jdbc(url=jdbc_url, table='customers_table', mode=mode, properties=config)

# Write vine_df to table in RDS
# 11 minutes
vine_df.write.jdbc(url=jdbc_url, table='vine_table', mode=mode, properties=config)
```
![table_screenshot](https://user-images.githubusercontent.com/104809098/189554853-be853656-0c13-47e6-b690-2718ff12785b.png)

## Results: 

* How many Vine reviews and non-Vine reviews were there?

There were 613 Vine reviews and 64968 non-Vine reviews.

* How many Vine reviews were 5 stars? How many non-Vine reviews were 5 stars?

For the Vine reviews there were 200 5-star review. For the non-Vine reviews there were 28,842 5-star reviews.

* What percentage of Vine reviews were 5 stars? What percentage of non-Vine reviews were 5 stars?

The Vine review and non-Vine reviews had percentage can be broken down as 32.6% (Vine) and 44.4%(non-Vine). 

## Summary 

Looking at the count and percentage break down between the Vine and non-Vine review, one can determine that there is no positive bias. Further analysis would need to be performed to verify this finding. A regression analysis using R, could be used to determine how bias being a vine and non-vine user. 





