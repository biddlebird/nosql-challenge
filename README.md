# NoSQL Challenge
This repository contains a project where I analyzed UK Food Standards Agency hygiene ratings data for *Eat Safe, Love* magazine. The goal was to help their journalists and food critics identify establishments for future articles by providing insights into ratings trends and notable locations.

# Table of Contents 
1. [Overview](#Overview)
2. [Dependencies](#Importing-Dependencies)
3. [Part 1: Database and Jupyter Notebook Set Up](#Part-1-Database-and-Jupyter-Notebook-Set-Up)
4. [Part 2: Update the Database](#Update-the-Database)
5. [Part 3: Exploratory](#Exploratory)
6. [Real-Life Applications and Data Analytics](#Real-Life_Applications_and_Data_Analytics)
7. [Data Analytics in Action](#Data_Analytics_in_Action)
8. [Conclusion](#Conclusion)

# Overview
### Assignment Overview

In this assignment, you will work with a NoSQL database containing data about food establishments across the UK. The goal is to perform various tasks, including setting up the database, updating records, and conducting exploratory data analysis to answer specific questions posed by the magazine "Eat Safe, Love." The analysis will help identify which establishments are safe to visit based on their ratings and hygiene scores.

The assignment is divided into three main parts:

1. **Database Setup**:
   - Import the data from a provided `establishments.json` file into a MongoDB database named `uk_food`.
   - Confirm that the data was loaded correctly by listing the databases, collections, and displaying sample documents.

2. **Database Update**:
   - Make specific updates to the database as requested by the magazine editors. This includes adding a new restaurant, updating its details, removing certain records, and ensuring that data types are correctly formatted.

3. **Exploratory Data Analysis**:
   - Perform various queries and aggregations on the `establishments` collection to answer specific questions about food safety and ratings.
   - Tasks include identifying establishments with specific hygiene scores, finding the best-rated establishments in London, and analyzing the data to find the top local authorities with the best or worst hygiene records.

Throughout the assignment, you will use Python with libraries like PyMongo and Pandas to interact with the MongoDB database, perform data manipulations, and visualize the results. The completed analysis will provide valuable insights into food safety across different regions in the UK, helping the magazine make informed recommendations.

This assignment is a practical exercise in working with NoSQL databases, focusing on data manipulation, querying, and analysis using MongoDB.

# Importing Dependencies
```
from pymongo import MongoClient
from pprint import pprint
import pandas as pd
```

# Part 1: Database and Jupyter Notebook Set Up
- Import the provided 'establishments.json' file into MongoDB using the terminal.
- Name the database 'uk_food' and the collection 'establishments'.
- Create an instance of the Mongo Client
- List the database in MongoDB to confirm that 'uk_food' is created with `print(mongo.list_database_names())`
- Verift that the 'establishments' collection exists within 'uk_food'
- Retrieve and display a document from the 'establishments' collection using 'find_one' and 'pprint'
```
# Import libraries
from pymongo import MongoClient
from pprint import pprint

# Create a MongoClient instance
mongo = MongoClient(port=27017)

# Assign the uk_food database to a variable
db = mongo['uk_food']

# Verify the database and collection
print(mongo.list_database_names())  # Should include 'uk_food'
print(db.list_collection_names())   # Should include 'establishments'

# Display a document from the collection
establishments = db['establishments']
document = establishments.find_one()
pprint(document)
```


# Part 2: Update the Database
- Add in the new restaurant: Insert the new restraunt, "Penang Flavours", into the 'establishments' collection with the provided information
- Find and Update BusinessTypeID: Locate the BusinessTypeID for "Restaurant/Cafe/Canteen". Update the "Penang Flavours" entry with the correct BusinessTypeID.
- Remove Dover establishments: count and remove all establishments associated with the Dover Local Authority. Confirm that the removal was successful.
- Data type correction: convert 'longitude' and 'latitude' fields from strings to decimal numbers. Convert the 'RatingValue' field from strings to intergers where applicable.
```
# Import libraries
from pymongo import MongoClient

# Create a MongoClient instance
mongo = MongoClient(port=27017)
db = mongo['uk_food']
establishments = db['establishments']

# Add new restaurant "Penang Flavours"
new_restaurant = {
    "BusinessName": "Penang Flavours",
    "BusinessType": "Restaurant/Cafe/Canteen",
    "BusinessTypeID": "",
    "AddressLine1": "Penang Flavours",
    "AddressLine2": "146A Plumstead Rd",
    "AddressLine3": "London",
    "AddressLine4": "",
    "PostCode": "SE18 7DY",
    "Phone": "",
    "LocalAuthorityCode": "511",
    "LocalAuthorityName": "Greenwich",
    "LocalAuthorityWebSite": "http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress": "health@royalgreenwich.gov.uk",
    "scores": {
        "Hygiene": "",
        "Structural": "",
        "ConfidenceInManagement": ""
    },
    "SchemeType": "FHRS",
    "geocode": {
        "longitude": "0.08384000",
        "latitude": "51.49014200"
    },
    "RightToReply": "",
    "Distance": 4623.9723280747176,
    "NewRatingPending": True
}
establishments.insert_one(new_restaurant)

# Find the BusinessTypeID for "Restaurant/Cafe/Canteen"
result = establishments.find_one({"BusinessType": "Restaurant/Cafe/Canteen"}, {"BusinessTypeID": 1, "BusinessType": 1})
business_type_id = result['BusinessTypeID']

# Update "Penang Flavours" with the found BusinessTypeID
establishments.update_one({"BusinessName": "Penang Flavours"}, {"$set": {"BusinessTypeID": business_type_id}})

# Remove all documents with LocalAuthorityName as "Dover"
dover_count = establishments.count_documents({"LocalAuthorityName": "Dover"})
establishments.delete_many({"LocalAuthorityName": "Dover"})
remaining_count = establishments.count_documents({"LocalAuthorityName": "Dover"})

# Convert longitude and latitude to decimal numbers
establishments.update_many({}, [
    {"$set": {
        "geocode.longitude": {"$toDouble": "$geocode.longitude"},
        "geocode.latitude": {"$toDouble": "$geocode.latitude"}
    }}
])

# Convert RatingValue to integer and handle non-numeric values
non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
establishments.update_many({"RatingValue": {"$in": non_ratings}}, [{"$set": {"RatingValue": None}}])
establishments.update_many({}, [{"$set": {"RatingValue": {"$toInt": "$RatingValue"}}}])

```

# Part 3: Exploratory 
1. Hygiene Score Equal to 20:
  - Find establishments with a hygiene score of exactly 20.
  - Use count_documents to count the results.
  - Display the first document in the results using pprint.
  - Convert the results to a Pandas DataFrame, display the number of rows, and show the first 10 rows.
2. RatingValue Greater Than or Equal to 4 in London:
  - Find establishments in London with a RatingValue of 4 or higher.
  - Use $regex to match establishments under London’s Local Authority, as its name is longer than just "London."
  - Use count_documents to display the number of matching documents.
  - Convert the results to a Pandas DataFrame, and display the first 10 rows.
3. Top 5 Establishments with RatingValue of 5 and Lowest Hygiene Score Near "Penang Flavours":
  - Find the top 5 establishments with a RatingValue of 5, sorted by the lowest hygiene score, near the new restaurant "Penang Flavours."
  - Search within 0.01 degrees on either side of the restaurant's latitude and longitude.
  - Use count_documents to count the number of matching documents.
  - Convert the results to a Pandas DataFrame, and display the first 10 rows.
4. Establishments with a Hygiene Score of 0 by Local Authority:
  - Count how many establishments in each Local Authority area have a hygiene score of 0.
  - Sort the results from highest to lowest.
  - Print out the top ten Local Authority areas.
```
# Import necessary libraries
from pymongo import MongoClient
import pandas as pd
from pprint import pprint

# Create an instance of MongoClient
mongo = MongoClient(port=27017)
db = mongo['uk_food']
establishments = db['establishments']

# 1. Establishments with a hygiene score of 20
query = {"scores.Hygiene": 20}
results = list(establishments.find(query))
df = pd.DataFrame(results)
print(f"Number of establishments with a hygiene score of 20: {len(df)}")
pprint(results[0])
df.head(10)

# 2. Establishments in London with RatingValue >= 4
query = {"LocalAuthorityName": {"$regex": "London"}, "RatingValue": {"$gte": 4}}
results = list(establishments.find(query))
df = pd.DataFrame(results)
print(f"Number of establishments in London with RatingValue >= 4: {len(df)}")
pprint(results[0])
df.head(10)

# 3. Top 5 establishments near "Penang Flavours" with RatingValue of 5 and lowest hygiene score
latitude = 51.49014200
longitude = 0.08384000
degree_search = 0.01
query = {
    "RatingValue": 5,
    "geocode.latitude": {"$gte": latitude - degree_search, "$lte": latitude + degree_search},
    "geocode.longitude": {"$gte": longitude - degree_search, "$lte": longitude + degree_search}
}
sort = [("scores.Hygiene", 1)]
results = list(establishments.find(query).sort(sort).limit(5))
df = pd.DataFrame(results)
print(f"Top 5 establishments near 'Penang Flavours' with RatingValue of 5 and lowest hygiene score:")
pprint(results[0])
df.head(10)

# 4. Establishments with a hygiene score of 0 by Local Authority
pipeline = [
    {"$match": {"scores.Hygiene": 0}},
    {"$group": {"_id": "$LocalAuthorityName", "count": {"$sum": 1}}},
    {"$sort": {"count": -1}},
    {"$limit": 10}
]
results = list(establishments.aggregate(pipeline))
df = pd.DataFrame(results)
print(f"Top 10 Local Authority areas with the highest number of establishments having a hygiene score of 0:")
df.head(10)
```

# Real-Life Applications and Data Analytics

This assignment showcases practical skills in managing and analyzing data within a NoSQL database, which is increasingly important in real-world scenarios. The techniques and concepts learned here can be directly applied to a wide range of real-life problems and data analytics tasks:

1. **Food Safety and Public Health**:
   - The ability to analyze and update food establishment data is crucial for health departments and regulatory agencies. By identifying patterns in hygiene scores and ratings, authorities can target inspections and resources more effectively, ensuring public safety.
   - For example, the insights gained from analyzing establishments with low hygiene scores can help prioritize which locations need immediate attention, potentially preventing foodborne illnesses.

2. **Customer Experience and Recommendation Systems**:
   - Businesses like food magazines, review platforms, and travel guides can use similar analytical methods to curate recommendations for their audiences. By focusing on highly-rated establishments with excellent hygiene scores, they can enhance customer satisfaction and trust.
   - This approach can also be extended to develop personalized recommendation systems that suggest the best dining options based on individual preferences and safety standards.

3. **Business Intelligence and Market Research**:
   - Companies can leverage the techniques from this assignment to gain insights into the competitive landscape. By analyzing the distribution of food establishment ratings and locations, businesses can identify market gaps, opportunities for expansion, and areas where they need to improve their services.
   - For instance, a restaurant chain might analyze the data to understand how their branches compare to competitors in terms of hygiene and customer satisfaction, helping them make data-driven decisions to improve their operations.

4. **Urban Planning and Public Services**:
   - Urban planners and local governments can use this type of analysis to assess the distribution of food establishments in different regions. Understanding where high and low-rated establishments are clustered can inform decisions about where to allocate resources, such as public health initiatives or community support programs.
   - This data-driven approach can also help in planning new commercial zones, ensuring that new developments meet safety and quality standards that align with community needs.

5. **Data-Driven Policy Making**:
   - Policymakers can utilize the insights from this analysis to create or adjust regulations governing food safety. By understanding the trends and patterns in food establishment ratings, they can craft more effective policies that encourage higher standards and better compliance among businesses.

# Data Analytics in Action

The use of NoSQL databases, like MongoDB, in this assignment highlights the flexibility and scalability of modern data storage solutions. In data analytics, this flexibility is key to handling diverse datasets that may not fit neatly into traditional relational databases. The ability to query, update, and analyze such data in real-time is a powerful tool for businesses and organizations looking to make informed decisions quickly.

By working through this assignment, you're building a foundation in data analytics that can be applied to a variety of fields, including public health, customer service, business strategy, and more. The skills gained here—such as data cleaning, querying, aggregation, and data type conversion—are essential for any data analyst or data scientist working in today's data-driven world.

# Conclusion

In this project, you have explored a comprehensive approach to managing and analyzing NoSQL database data. From setting up a MongoDB database to performing complex queries and updates, each step has demonstrated the power of data-driven decision-making. 

You have learned to:
- Import and set up data, ensuring accurate and efficient storage in a NoSQL database.
- Perform critical updates, including adding new entries, correcting data types, and removing irrelevant information.
- Conduct exploratory analysis to gain actionable insights, such as identifying key establishments, filtering based on ratings, and mapping geographical data.

These skills are not only foundational for data analysis but also applicable to a wide range of real-world scenarios, from public health and business intelligence to urban planning and policy-making. The methodologies used in this project reflect best practices in handling and interpreting data, providing valuable insights that can drive better decisions and strategies.

By applying these techniques, you are well-equipped to tackle complex data challenges and contribute meaningfully to any data-driven field. This project serves as a testament to the power of data in solving real-world problems and underscores the importance of effective data management and analysis in making informed decisions.
