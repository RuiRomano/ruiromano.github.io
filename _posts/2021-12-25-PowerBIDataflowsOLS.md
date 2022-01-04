---
title: "Power BI Dataflows & Object Level Security (table level)"
date: 2021-12-25
tags: powerbi dataflows security
---

Power BI Dataflow security is managed at the workspace level, this means when you add the user to the workspace he will be able to see all the Dataflows inside the workspace and every table within.

This blog post aims to show you a workaround to achieve table level permission in Dataflows by leveraging:

- [Bring your own storage](https://docs.microsoft.com/en-us/power-bi/transform-model/dataflows/dataflows-azure-data-lake-storage-integration)
- [Attach external CDM Folders](https://ssbipolar.com/2018/12/10/dataflows-in-power-bi-overview-part-7-external-cdm-folders/)
- [Access Control Lists on ADLS Gen 2](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-access-control)

First you need to create a workspace and link it to a storage account, see the link for more details.

![image](https://user-images.githubusercontent.com/10808715/147389047-ab422938-89c3-4875-98e9-03a2d10adaa1.png)

Now every dataflow you create on this workspace will write its data to the linked storage account using the [CDM format](https://docs.microsoft.com/en-us/common-data-model/):

![image](https://user-images.githubusercontent.com/10808715/147389070-1f9147f1-3f40-4a3d-9418-36724cb8b64b.png)

If you want to only allow the end user to see & connect to a subset of tables (ex: Sales and Store), you can do the following:

Copy the url of the “model.json”, you can use [Azure Storage Explorer](https://azure.microsoft.com/en-us/free/storage/search/?&ef_id=Cj0KCQiAnaeNBhCUARIsABEee8XQo0UbTtKASGrK33KIS34a7nPUPS4jXt_Pxt-yF17kyycNRVriY0oaArKpEALw_wcB:G:s&OCID=AID2200245_SEM_Cj0KCQiAnaeNBhCUARIsABEee8XQo0UbTtKASGrK33KIS34a7nPUPS4jXt_Pxt-yF17kyycNRVriY0oaArKpEALw_wcB:G:s&gclid=Cj0KCQiAnaeNBhCUARIsABEee8XQo0UbTtKASGrK33KIS34a7nPUPS4jXt_Pxt-yF17kyycNRVriY0oaArKpEALw_wcB) to easily connect to the storage account and copy the url:

![image](https://user-images.githubusercontent.com/10808715/147389090-3a2659df-721a-4f58-8a46-ebb8d5fe4bd5.png)

The url should be like this:

https://yourstorage.blob.core.windows.net/powerbi/Workspace/Contoso/model.json

Replace “*.blob*” for “*.dfs.*”:

https://yourstorage.dfs.core.windows.net/powerbi/Workspace/Contoso/model.json

Create a new workspace in Power BI and add the end user as viewer:

PS – The end user is inside the security group “RRMSFT | Power Users”

![image](https://user-images.githubusercontent.com/10808715/147389094-52e644f7-273c-4bd3-bf70-250b2c75a172.png)

Create a new Dataflow, select the option “Attach a Common Data Model folder” and copy the url from above:

![image](https://user-images.githubusercontent.com/10808715/147389098-2cb298ca-7c88-48eb-a8a9-ffcca63320d0.png)

You should end with an attached dataflow on the workspace like this:

![image](https://user-images.githubusercontent.com/10808715/147389100-192ab8c2-2ec0-4502-8ed1-41666012c37d.png)

Using the end user credentials in Power BI Desktop, if you try to connect to the Dataflow you will get the following error:

“Power BI can’t access your organization’s Azure Data Lake Storage Gen2 account. Please ask your administrator to restore access, and try again”

![image](https://user-images.githubusercontent.com/10808715/147389106-20051cf4-6f84-4cc9-8e76-6a4b8b8ab568.png)

The error its because the connection to the dataflow storage is made with the end user credentials and he doesn’t have permissions on the data lake storage account.

If we assign the following permissions on the data lake:

- model.json – Read + Execute

![image](https://user-images.githubusercontent.com/10808715/147389112-9e8f25a6-71ad-4258-b218-cffa11f61a2d.png)

- All tables you want to give access to, ex: Sales & Store) – Read + Execute on folder & children

![image](https://user-images.githubusercontent.com/10808715/147389118-acf03f4b-8c59-4e01-9296-56d68f6ecef6.png)

- All parent folders up to the container- Execute

![image](https://user-images.githubusercontent.com/10808715/147389125-01fc680e-c1c5-42ce-adb9-8107b34cfca2.png)

Now if you go back to Power BI Desktop and try to connect to the Dataflow you will notice that the Dataflow tables are listed and the end user can see data for both Store & Sales:

![image](https://user-images.githubusercontent.com/10808715/147389129-296b93a5-10cf-40d8-b4c6-7bd23897e884.png)

But if he tries to connect to any other table (Ex: Customer) it will fail with a Forbidden error:

![image](https://user-images.githubusercontent.com/10808715/147389143-f12a56e8-c15d-434e-a613-5c83f4bd83ac.png)

Its not a perfect solution because the user will still be able to view all the available tables but could be useful versus the alternative of duplicating the dataflow data on multiple workspaces to achieve the same goal.
