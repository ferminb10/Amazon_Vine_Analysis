# Amazon_Vine_Analysis
## Resources:
<details>
<summary>Dataset:</summary>

  - [US Apparel Dataset](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt)
</details>

<details>
<summary>Software and IDE:</summary>
  
  - Google Colab 
  - PostgreSQL
    - pgAdmin

</details>

<details>
<summary>Cloud:</summary>

  - Amazon Web Services `AWS`
    - Simple Storage Service `S3`
    - Relational Database Service `RDS`
</details>

## Overview of the analysis of the Vine Program:
The overview of this project was analyzing Amazon reviews written by members of the paid Amazon Vine program. The Amazon Vine program is a service that allows manufacturers and publishers to receive reviews for their products. Companies like SellBy pay a small fee to Amazon and provide products to Amazon Vine members, who are then required to publish a review.

In this project, we had access to approximately 50 datasets. Each one contains reviews of a specific product, from clothing apparel to wireless products. For the analysis we picked the [US Apparel Dataset](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt) and used PySpark to perform the ETL process to extract the dataset, transform the data, connect to an AWS RDS instance, and load the transformed data into pgAdmin. We then used PySpark on Google Colab to determine if there is any bias toward favorable reviews from Vine members in the selected dataset.

### _Perform ETL on Amazon Product Reviews_
Using our knowledge of the cloud ETL process, we created an AWS RDS database with tables in pgAdmin, and picked the [US Apparel Dataset](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt). All datasets have the same schemata, as shown in this image:

![image](https://user-images.githubusercontent.com/107281474/194203528-72813683-b3f1-4835-9fc6-0b3b94d20360.png)

We then created a new database with `Amazon RDS` and also from `pgAdmin` connected to our `Amazon RDS` server.
In `pgAdmin`, we ran a new query to create the tables for our new database using the code from the provided `challenge_schema.sql` file.
After running the query, we created the following four tables in our database: 
- `customers_table`
- `products_table`
- `review_id_table`
- `vine_table`

![Screen Shot 2022-10-03 at 11 58 15 AM](https://user-images.githubusercontent.com/107281474/194401810-b396aef8-0bb5-469d-8e41-27fa99066c7d.png)

![Screen Shot 2022-10-03 at 12 00 09 PM](https://user-images.githubusercontent.com/107281474/194402310-8a25a16a-4d20-41cd-8f86-11fdc292ccd2.png)

Originally we downladed the `Amazon_Reviews_ETL_starter_code.ipynb`, then uploaded the file as a Google Colab Notebook to begin the ETL process. The reason why Google Colab was chosen for this project was primarily in order for `PySpark` to run. The whole DataFrame was then transformed and divided into four smaller DataFrames for better understanding of the dataset and overall analysis of the project. 

_customers_table DataFrame_
- Cell Code:
```
customers_df = df.groupby("customer_id").agg({"customer_id": "count"}).withColumnRenamed("count(customer_id)", "customer_count")
customers_df.show()
```

<img width="1212" alt="Screen Shot 2022-10-08 at 2 09 41 PM" src="https://user-images.githubusercontent.com/107281474/194727964-486e8448-3623-49b5-9dcc-e553606da2ab.png">

_products_table DataFrame_
- Cell Code:
```
products_df = df.select(["product_id", "product_title"]).dropDuplicates(["product_id"])
products_df.show()
```

<img width="917" alt="Screen Shot 2022-10-08 at 3 06 13 PM" src="https://user-images.githubusercontent.com/107281474/194729385-68498cf8-604c-4365-805a-1a6b71c2aa4f.png">


_review_id_table DataFrame_
- Cell Code:
```
review_id_df = df.select(["review_id", "customer_id", "product_id", "product_parent", to_date("review_date", 'yyyy-MM-dd').alias("review_date")])
review_id_df.show()
```

<img width="1185" alt="Screen Shot 2022-10-08 at 3 00 26 PM" src="https://user-images.githubusercontent.com/107281474/194729343-b1e9e4e5-d991-4f66-8039-182686b645aa.png">

_vine_table DataFrame_
- Cell Code: 
```
vine_df = df.select(["review_id", "star_rating", "helpful_votes", "total_votes", "vine", "verified_purchase"])
vine_df.show()
```

<img width="1188" alt="Screen Shot 2022-10-08 at 3 02 45 PM" src="https://user-images.githubusercontent.com/107281474/194729350-643f1434-9d6f-45ce-b5c8-4db57e3fd7d6.png">

The next iteration of the analysis was loading the DataFrames into `pgAdmin` and make the connection to oue `AWS RDS` instance. Then we loaded the DataFrames that correspond to tables in `pgAdmin` and ran a query to check that the tables have been populated.

### _Determine Bias of Vine Reviews_
Using our knowledge of PySpark, we then determined if there is any bias towards reviews that were written as part of the Vine program. For this analysis, we determined if having a paid Vine review makes a difference in the percentage of 5-star reviews. We first, filtered the data and created a new DataFrame to retrieve all the rows where the `total_votes` count is equal to or greater than 20 to pick reviews that are more likely to be helpful and to avoid having division by zero errors later on. Next, we filtered the new DataFrame created previously and created a new DataFrame to retrieve all the rows where the number of `helpful_votes` divided by `total_votes` is equal to or greater than _50%_. Then, we filtered the DataFrame or table created previously and made a new DataFrame  that retrieves all the rows where a review was written as part of the Vine program (paid), _vine == 'Y'_. Then we duplicated the last table and modified it to where this time we retrieved all the rows where the review was not part of the Vine program (unpaid), _vine == 'N'_. Lastly, we determined the _total number of reviews_, _the number of 5-star reviews_, and _the percentage of 5-star reviews_ for _the two types of review (paid vs unpaid)_.

## Results: 
- How many Vine Reviews and non-Vine reviews were there?
  - Total Vine Reviews: _33_
  - Total non-Vine Reviews: _45,388_

- How many Vine reviews were 5 stars? How many non-Vine reviews were 5 stars?
  - 5-star Vine reviews: _15_
  - 5-star non-Vine reviews: _23,733_

- What percentage of Vine reviews were 5 stars? What percentage of non-Vine reviews were 5 stars?
  - Percentage of 5-star Vine Reviews: _45.45%
  - Percentage of 5-star non-Vine Reviews: _52.29%

_Technical Analysis:_

<img width="1001" alt="Screen Shot 2022-10-09 at 3 19 49 PM" src="https://user-images.githubusercontent.com/107281474/194782135-19e44be9-e239-4537-9e1b-e781145a6586.png">

## Summary: 
From the results, the Vine Program is not sufficient for the apparel category that was chosen for the analysis. Out of the total helpful reviews, less than _50%_ of them were 5-star rated which is very significant and important for categorical review/placement. The same pattern is true and apparent for the unpaid reviews. We can also infer that the Vine program is not popular in this category. We can safely say for certain that there is in fact no bias for the Vine member reviews. One recommendation that could be enacted and incorporated for better bias conclusion is a significantly larger dataset either through oversampling or undersampling.

_Recommended Supporting Analysis:_

One recommendation that can be implemented to support this statement is calculating the mean of each star ratings on each program's review and see if there are any trends or significant incentives. Additionally, we could incorporate a NLP sentiment analysis to analyze key words or phrases used in the majority of the reviews. 
