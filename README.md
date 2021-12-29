# Amazon Vine Analysis

![logo](images/module_16_logo.png)

# Overview
This project is to analyize the reviews written by members of the paid Amazon Vine program. This program is a service that allows manufacturers and publishers to receive reviews for their products. Companies like SellBy pay a small fee to Amazon and provide products to Amazon Vine members, who are then required to provide a review.

In this project, I picked a product that was reviewed, from approximately 50 different products, from clothing apparel to wireless products.  

For Deliverable 1, I will use PySpark to perform the ETL process to extract the dataset, transform the data, connect to an AWS RDS instance, and lod the transformed data into pgAdmin.

For Deliverable 2, I will use Pyspark to read in the exported vine_table, in csv format, into a DataFrame, and determine if there is any bias towards favorable reviews from Vine members.

The results of the above, will allow me to provide SellBy the information they can analyize during their decision process to invest into using or not using the Amazon Vine program.

# Resources
* Data Sources: https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Musical_instruments_v1_00.tsv.gz
* Software: Google Colaboratory, Python, PySpark, Spark, Amazon Web Services, PostgreSQL

# GitHub Application Link

<a href="https://jillibus.github.io/Amazon_Vine_Analysis">Amazon Vine Analysis</a>

# Deliverable 1: Perform ETL on Amazon Product Reviews

For my analysis, I chose the product, Musical Instruments. My family, is large, and my parents encouraged us to join clubs, play sports, as well as join the band. All of my siblings, there are 6 kids total, played at least 2 instruments from middle school through high school. I even played during my first year of college.  So this product caught my eye, right away.

* I first read it into a DataFrame, with PySpark.
```
from pyspark import SparkFiles
url = "https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Musical_Instruments_v1_00.tsv.gz"
spark.sparkContext.addFile(url)
df = spark.read.option("encoding", "UTF-8").csv(SparkFiles.get("amazon_reviews_us_Musical_Instruments_v1_00.tsv.gz"), sep="\t", header=True, inferSchema=True)
df.show()
+-----------+-----------+--------------+----------+--------------+--------------------+-------------------+-----------+-------------+-----------+----+-----------------+--------------------+--------------------+-----------+
|marketplace|customer_id|     review_id|product_id|product_parent|       product_title|   product_category|star_rating|helpful_votes|total_votes|vine|verified_purchase|     review_headline|         review_body|review_date|
+-----------+-----------+--------------+----------+--------------+--------------------+-------------------+-----------+-------------+-----------+----+-----------------+--------------------+--------------------+-----------+
|         US|   45610553| RMDCHWD0Y5OZ9|B00HH62VB6|     618218723|AGPtek® 10 Isolat...|Musical Instruments|          3|            0|          1|   N|                N|         Three Stars|Works very good, ...| 2015-08-31|
|         US|   14640079| RZSL0BALIYUNU|B003LRN53I|     986692292|Sennheiser HD203 ...|Musical Instruments|          5|            0|          0|   N|                Y|          Five Stars|Nice headphones a...| 2015-08-31|
```
* Next I extracted each table data I wanted from the DataFrame 'df' and created new "table" DataFrames
```
# Create the customers_table DataFrame
customers_df = df.groupby("customer_id").agg({"customer_id":"count"}).withColumnRenamed("count(customer_id)", "customer_count")
customers_df.show(5)
+-----------+--------------+
|customer_id|customer_count|
+-----------+--------------+
|   27314089|             1|
|    6711334|             1|
|   44525866|             4|
|   47461997|             1|
|   42560427|             1|
+-----------+--------------+
```
This was repeated for the remaining 3 tables, _products_table, review_id_table, and vine_table._ Each have a corresponding DataFrame, products_df, review_id_df and vine_df.

* Lastly, I setup an RDS PostgreSQL Database on the Amazon Web Services and in the PySpark program listed the connection configuration to the database.  I also set up a connection in pgAdmin so I could view the tables locally in my desktop.  I created the 4 tables through the pgAdmin interface so I could load them from the PySpark program.

Once the program was connected, I inserted to each database table the contents of each of the table DataFrames.
```
review_id_df.write.jdbc(url=jdbc_url, table='review_id_table', mode=mode, properties=config)
products_df.write.jdbc(url=jdbc_url, table='products_table', mode=mode, properties=config)
customers_df.write.jdbc(url=jdbc_url, table='customers_table', mode=mode, properties=config)
vine_df.write.jdbc(url=jdbc_url, table='vine_table', mode=mode, properties=config)
```
Below is queries, in pgAdmin showing the data was uploaded into the AWS Database.
<img src="images/review_id_table.png" width=50% height=50% /> 
<img src="images/products_table.png" width=50% height=50% />
<img src="images/customers_table.png" width=50% height=50% />                                                    
<img src="images/vine_table.png" width=50% height=50% />

# Deliverable 2: Determine Bias of Vine Reviews
* For this deliverable, I repeated the steps up to the creating of the DataFrames using Google Colaboratory.  
* From there I created a new DataFrame of the data, removing any row with null values. 
* From the clean_df, I created the vine_df DataFrame, with columns for review_id, star_rating, helpful_votes, total_votes, vine, verified_purchase.
* From the vine_df, I created the total_votes_df DataFrame, by filtering the vine_df with: vine_df.filter(col("total_votes") >= 20).
* From the total_votes_df, I created the percent_votes_df DataFrame, adding a column called percent_votes, by calculating the percent_votes with:
  * total_votes_df.withColumn('percent_votes',col('helpful_votes')/col('total_votes')).alias('percent_votes').filter(col("percent_votes") >= 0.5)
* From the total_votes_df, I created 2 new DataFrames, paid_df and non_paid_df
  * paid_df (those with Vine = 'Y') were determined by:  percent_votes_df.filter(col("vine") == "Y").show()
  * non_paid_df (those with Vine = 'N') were determined by: percent_votes_df.filter(col("vine") == "N").show()

# Results
```
+----+-------------+--------------------+------------------+
|vine|Total_Reviews|Total_5_Star_Reviews| %_5_Star_To_Total|
+----+-------------+--------------------+------------------+
|   Y|           60|                  34|56.666666666666664|
|   N|        14477|                8212| 56.72445948746287|
+----+-------------+--------------------+------------------+
```

* How many Vine reviews and non-Vine reviews were there? **14,537**
* How many Vine reviews were 5 stars? **34**
* How many non-Vine reviews were 5 stars? **8,212**
* What percentage of Vine reviews were 5 stars? **56.7%**
* What percentage of non-Vine reviews were 5 stars? **56.7%**

# Summary

The results show, that despite the lower number of reviewers from the Amazon Vine program, 60, compared to 14,477, the percentage of 5 star reviews were exactly the same as the non-Vine reviewers, at 56.7%.  This tells us that the paid reviewers did not give out more 5 star reviews because they were being paid either in free product or money.

An additional measurment could be done by adding the _verified_purchase_ to the analysis. This gave us a bit of a percentage change for the non-Vine reviewers.
Now we are looking at a 56.7% for Vine Reviewers vs a 57.4% for non-Vine Verified Purchase Reviewers. This shows us a slightly higher rating for the non-Vine Reviews but one that actualy spent their money and came back and reviewed a product.  That to me shows that the non-Paid Reviewers were more honest.
Some questions remain in how valid are the 'non-verified users'?  Do we count them? Do we spend time technically improving determining who a reviewer is to ensure these are not bots? But can we automatically dismiss them? 

These are questions that compaies, like SellBy, need to calculate into their decision.

```
from pyspark.sql.functions import col,when,count,lit
ratings_total_df = percent_votes_df.groupBy("vine","verified_purchase").agg(
    count(col("vine")).alias("Total_Reviews"),
    count(when(col("star_rating") == 5, True)).alias("Total_5_Star_Reviews"),
    (count(when(col("star_rating") == 5, True))/count(col("vine"))*100).alias("%_5_Star_To_Total")).show()

+----+-----------------+-------------+--------------------+------------------+
|vine|verified_purchase|Total_Reviews|Total_5_Star_Reviews| %_5_Star_To_Total|
+----+-----------------+-------------+--------------------+------------------+
|   Y|                N|           60|                  34|56.666666666666664|
|   N|                Y|         8610|                4940|57.375145180023225|
|   N|                N|         5867|                3272| 55.76955854780978|
+----+-----------------+-------------+--------------------+------------------+
```

Thank you for your time and let me know if you wish to see any additional data.

Jill Hughes
