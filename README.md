# Striim-S3-Cosmos
Using Striim to migrate data from S3 to cosmos and mask data in the process

## TOOLS REQUIRED 
Striim – unified data integration and Streaming platform

AWS S3 – Object storage service

Azure Cosmos - fully managed NoSQL database 

## OVERVIEW
 
 ![image](https://user-images.githubusercontent.com/54326319/150217370-1f761861-1bcc-4cf6-b5ed-aafe571579e1.png)

 
Cloud to Cloud migration is a common scenario that usually arises due to a wide variety of reasons ranging from costs, security, company wide single cloud adoption or the more popular scenario nowadays on leveraging Hybrid data architectures to maintain no downtime and high tolerance.

“A wise man doesn’t trust all his eggs to one basket”
-Miguel De Cervantes

### End user benefits
This recipe focuses on two main areas
1.	Move data from Object Storage Service (AWS S3) to a NoSql Database (Azure cosmos) - 
Striim allows data to be migrated directly into a NoSQL database like Cosmos in real time, without the need for dumping data into Blob Storage and use custom applications(ADF, Spark etc) to process/move the data .

2.	Mask sensitive data during ETL stage - 
Striim helps in analyzing data from file dumps that have sensitive information by leveraging the inbuilt Event transformers and avoid the need for any staging zones to perform any masking functionality before reaching the target.

## STEP 1: CONFIGURING DOCKER WITH STRIIM
Complete the following steps on Configuring Striim in docker.
Note: Use the below docker run command for the below message during initial setup, the docker run command might generate the following error
 
 ![image](https://user-images.githubusercontent.com/54326319/150217655-9d01946a-a9c3-49ad-a981-c6148e4f9620.png)

Pass the requested arguments with additional -e in the run command.Update the run command with the following syntax
>docker run -p 80:9080 -e "STRIIM_ACCEPT_EULA=Y" -e "FIRST_NAME=Sriram" -e "LAST_NAME=Pasam" -e "COMPANY_NAME=abc" -e "COMPANY_EMAIL_ADDRESS=sp19815n@pace.edu" striim/evalversion
Verify if the docker container is up and running via
>docker ps 
Or access the web UI http://localhost:80. Log in as admin with password admin.


## STEP 2: MANAGING S3 BUCKET 
Log into IAM in AWS console and navigate to Security Credentials to verify if the Access key ID and Secret access key are configured. These will be used by Striim to authenticate and load data from S3.
In a real-world scenario, the CSV data can be multiple files generated in micro batches or hourly feeds. Sample data is as follows and contains no headers nor duplicates.

![image](https://user-images.githubusercontent.com/54326319/150218091-3e06d930-66bc-4646-9822-082a469a38b6.png)
![image](https://user-images.githubusercontent.com/54326319/150218105-5db390b5-52d5-4660-99a8-8ab9ae8bf25f.png)


##  STEP 3: PREPARE A COSMOS DATABASE
Before migrating your data from S3, you would have to create a Cosmos database within Azure to store the migrated data.
•	In the Cosmos homepage, click on Create and go with the default recommended options. 

![image](https://user-images.githubusercontent.com/54326319/150218531-235887c0-ae2e-4bdb-924a-abd1dfd10dc3.png)

•	Update required fields such as Resource group, Account name and location and go ahead to Review and Create.

![image](https://user-images.githubusercontent.com/54326319/150218562-d1c08a60-ee6f-469c-8fb2-2cefd30bd4ee.png)

•	Once deployment has finished, navigate to Data explorer within the database to create a new Database and Container.
![image](https://user-images.githubusercontent.com/54326319/150218665-05eb01c2-bd52-4502-946f-79c9b165adf6.png)

•	Create a new container and set the partition key of your choice, and in case the source data lacks a primary key, we can add a new column name (/id) under Partition key and let Cosmos handle the generation of a unique element.
NOTE: For use cases where data needs to be sent to Synapse analytics as part of downstream flows, click on Enable for Azure Synapse Link during container creation or can be initialized later as well. Please keep in mind Synapse analytics and Cosmos are cost intensive applications.
![image](https://user-images.githubusercontent.com/54326319/150218698-1a8b069c-4861-48ef-970b-446b90ae165c.png)



## STEP 4: STRIIM INTEGRATION
Navigate to a browser and open http://localhost/#landing. The username and password should be ‘admin’. Skip the initial tour of the application and click on Apps to create an application.
![image](https://user-images.githubusercontent.com/54326319/150220054-29b6e842-f6c7-4b38-89d9-fed94e9f4d2f.png)

•	You will be provided with 3 options providing us the flexibility to choose from depending on our use case. If you are a returning user, we can import the template (.tql file) to load previous Striim flows that have been created.

![image](https://user-images.githubusercontent.com/54326319/150220105-ef7e4407-d890-4e40-aa12-f28dcd0e0f2c.png)

•	Striim has the flexibitlity of connecting to different types of sources and target types. For this use case, we can select to Start from scratch

•	Enter the name of the application name and Save. The template will request to configure the S3 and Azure. Check if the source CSV file has a header and initialize if true.

![image](https://user-images.githubusercontent.com/54326319/150220178-0d585724-bbdd-43c4-8b01-6ac7187d7976.png)

![image](https://user-images.githubusercontent.com/54326319/150220258-51c00c97-b775-4fe3-9c6b-e610ba908fc8.png)

•	Click on S3_source_output and select to create a new CQ component.

![image](https://user-images.githubusercontent.com/54326319/150220307-3d1b6ce6-0ea0-4402-b114-7eb3a90f04bd.png)

•	Update the Query with the column names and aliases and point to the output

![image](https://user-images.githubusercontent.com/54326319/150220354-92a1ef52-18e7-40ff-8880-d828ae7c78da.png)

•	Drag and drop field masker from the Event Transformer drop down menu on the left 

![image](https://user-images.githubusercontent.com/54326319/150220380-8eb5e033-0c90-461a-8c3a-cf5b007756e0.png)

•	Point to the new input and and click on ADD FIELD to select the column names and choose the appropriate masking function applicable for the data. Add new Output as masked_output


![image](https://user-images.githubusercontent.com/54326319/150220414-19fbf10d-7717-4663-83eb-0b9fe775d23b.png)


•	Next, we can Drag and drop the CosmosDB from the Target drop down on the left 

![image](https://user-images.githubusercontent.com/54326319/150220440-cdbf954c-8c6a-4067-904a-e68145d993da.png)

•	Navigate to your azure portal and obtain the keys for Striim to authenticate and write the data too. Under the Read-write Keys, obtain the URI and Primary Key and enter them into the Service endpoint and Access key respectively while configuring the cosmosDB Target.

![image](https://user-images.githubusercontent.com/54326319/150220474-fb7204aa-ce46-4037-a930-ad36d4bf83ef.png)

•	Update CosmosDB target as follows 

![image](https://user-images.githubusercontent.com/54326319/150220505-5b17feb5-74fd-41fa-8777-710d8c1e99d4.png)

•	Click on deploy app, followed by Start app.
Note: Cosmos will not accept any duplicates and will lead to run time errors if no check mechanism is implemented to filter out the duplicates.

![image](https://user-images.githubusercontent.com/54326319/150220525-3426c30d-edc7-48ec-9a01-84fa96da83dc.png)

![image](https://user-images.githubusercontent.com/54326319/150220541-e09afdba-8197-4100-a50a-f565566fda3d.png)

![image](https://user-images.githubusercontent.com/54326319/150220564-c249e9c6-bab1-41e0-b6cd-c9b1b3b5c21e.png)


## STEP 5: VALIDATE AND NEXT STEPS
Validate data in cosmos container and check if sensitive data fields are masked. 
![image](https://user-images.githubusercontent.com/54326319/150220603-e23dc90c-6614-4d0f-88ea-c436dd4fdfe3.png)

This recipe shows a quick overview of how to move CSV data from S3 to a NoSql database like Cosmos, and at this point, we can do data analytics by integrating with Synapse analytics to perform any data analytics and build more complex use cases such as combining multiple different tables while data is in transit, apply transformations etc. Happy Striiming!













