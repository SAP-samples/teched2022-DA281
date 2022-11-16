## Exercise 1.5 - Using a custom ABAP Operator to verify your Delta Replication of EPM Sales Orders
You can now test the delta processing capabilities of the ABAP CDS View based data extraction. A nice task would be to check if Pipeline for Sales Orders replications and enrichment that you have built in [Exercise 1.4 - Extend the Pipeline for joining Sales Order with Customer data for each change in Sales Orders and persist results in S3](../ex1#exercise-13---implement-a-pipeline-for-delta-transfer-of-enhanced-epm-sales-order-data-from-s4hana-to-an-s3-object-store) is really processing the delta records from EPM in S/4HANA.<br><br>

1. In case your pipeline ***EPM_SalesOrder_Replication_Enrich_to_S3*** from exercise 1.4 is still running, you can directly go to step 5. In case you have stopped the pipeline, click on the ***Graphs*** tab of the Modeler UI (see left side). Then enter your user name in the search field (if you made your user name a part of the Pipeline names or descriptions) and start the search. You will now get a list of the pipelines that you have implemented. Click on your 'EPM_SalesOrder_Replication_Enrich_to_S3' Pipeline icon. If the displayed name is too short to recognize a unique name, just hover with your mouse over the Pipeline icons (see below).<br><br>
When searching for your pipeline, make sure you have selected the category ***da281*** in the available categories.<br><br>
![](/exercises/ex2/images/ex2-033c.jpg)<br><br>
![](/exercises/ex2/images/ex2-012c.jpg)<br><br>

2. The Pipeline is opened in the canvas area of the Modeler UI.<br><br>

3. Make sure a Wiretap operator exists between the ***Get Header*** and ***To File*** Operator and the Sales Order to S3 Write File Operator:<br><br>
   ***Note:*** In case you have kept the Wiretap Operator between ***Get Header*** and ***Write File*** before as described in Exercise 1.4 Step 4. you can ignore this step. In case you have removed it, please add the Wiretap operator inside your pipeline and add it between the ***Get Header*** and ***Write File*** Operator.<br><br>
![](/exercises/ex2/images/ex2-013b.JPG)<br><br>

4. ***Save*** the graph and now ***Run*** it <br><br>
![](/exercises/ex2/images/ex2-028c.jpg)<br><br>

5. If you see that the Pipeline changes to a ***running*** status, open the Wiretap UI.<br><br>
![](/exercises/ex2/images/ex2-014c.jpg)<br><br>

6. You can see now that the initial load from the ABAP CDS View in S/4HANA was successfully conducted. Please **leave the Wiretap UI open**.<br><br>
![](/exercises/ex2/images/ex2-015b.JPG)<br><br>

7. Now create your own Pipeline for generating EPM Sales Order records like already shown in [Deep Dive 2.2](../../exercises/dd2/README.md#deep-dive-22---integrate-the-custom-abap-operator-in-a-sap-data-intelligence-pipeline).<br>
In order to not have you navigating back and forth, all steps of that Deep Dive session for creating the Pipeline for EPM Sales Order creation are also included in this exercise document.<br><br>

8.	In the DI Modeler, make sure you are in the ***Graphs*** tab (see left side) and click the ***+*** symbol and select **Use Generation 1 Operators** in order to create a new Pipeline.<br><br>
![](/exercises/ex2/images/dd2-029b.jpg)<br><br>

9.	A new Pipeline canvas opens and the design focus automatically changes to the ***Operators*** tab (see left side). Drag the ***Custom ABAP Operator*** icon from the Operator list and drop it onto the canvas. Then do one click on the ***Custom ABAP Operator*** node in the canvas and open the configuration panel by clicking on the related symbol.<br><br>
![](/exercises/ex2/images/dd2-018b.jpg)<br><br>

10.	In the configuration panel on the right side, select the ***ABAP Connection*** (RFC or Websocket RFC connection) to the SAP S/4HANA system that provides the ABAP Operator. If done, click on the selection button of the field for the ***ABAP Operator***.<br><br>
![](/exercises/ex2/images/dd2-019b.jpg)<br><br>

11.	From the pop-up window, select the custom ABAP Operator that you want to call from the Pipeline. In our case, it's the ABAP Operator that triggers the execution of a specific ABAP Report variant on the S/4HANA system, and sends a confirmation back to the DI pipeline. Hence, choose ***Operator Class: Creation of EPM Sales Order*** and click ***OK***<br><br>
![](/exercises/dd2/images/dd2-020b.jpg)<br><br>

12.	As you can see, the ABAP Operator node in the Pipeline canvas gets automatically updated with the operator's name in S/4HANA and the ports that we have defined in the previous section of this Deep Dive demo. (The `GET_INFO( )`method in our operator's ABAP class provides the corresponding meta information.)<br><br>
![](/exercises/dd2/images/dd2-021b.jpg)<br><br>

13.	For verifying the functionality of the ABAP Operator call, we'll be using a ***Terminal*** Operator in the Pipeline. This operator allows the sending of user inputs and the reception of the results. Drag the ***Terminal*** icon from the Operator list and drop it onto the Pipeline canvas. Then
    - connect the output port of the ABAP Operator with the input port of the Terminal Operator and
    - connectthe output port of the Terminal Operator with the input port of the ABAP Operator.
    - ***Save*** the Pipeline.
<br><br>
![](/exercises/ex2/images/dd2-022b.jpg)<br><br>

14.	When saving the Pipeline, you are prompted for the name of the pipeline (including namespace information), a description, and the category under which the Pipeline can be found in the ***Graphs*** tab of the Modeler. Please enter the following parameters prompted in the pop-up windows:<br>
    - Name: `teched.XXXX.EPM_FM_Call_SO_Generator`, where XXXX is your user name, for example "teched.TA99.EPM_FM_Call_SO_Generator"
    - Description: `XXXX - Generate EPM SO data via ABAP FM call`, where XXXX is your user name, for example "TA99 - Generate EPM SO data via ABAP FM call"
    - Category: `da281`.
    - Finally click ***OK***.<br><br>
![](/exercises/ex2/images/ex2-009c.JPG)<br><br>

15.	You can now start the Pipeline by clicking the ***Play*** symbol in the menue bar.<br><br>
![](/exercises/ex2/images/dd2-025b.jpg)<br><br>

16.	Change back to the ***Status*** tab of the status section in the Modeler UI. Once the status has turned to ***running***, click one time on the ***Terminal*** node and open the Terminal UI with a click on the corresponding icon.<br><br>
![](/exercises/ex2/images/dd2-026b.jpg)<br><br>

17.	In the lower section of the Terminal UI (the Input Prompt), you can now enter any input or just press ***Return*** in order to send a trigger signal to the ABAP Operator in S/4HANA. As a response for each input from our custom ABAP Operator, you will receive a confirmation message in the upper section of the Terminal UI, what proves our ABAP operator code functionality and the integration success.<br>
Now produce several Sales Order records.<br>
![](/exercises/ex2/images/dd2-027b.jpg)<br><br>

18. In the Browser tabs, switch back to the ***Wiretap UI*** of your Replication and Enrichment Pipeline. You should now see that a message with new records came in, which all have the "U" indicator for Updates. This verifies the delta-enablement of the ABAP CDS View and the immediate integration with Data Intelligence.<br><br>
![](/exercises/ex2/images/ex2-018b.JPG)<br><br>

19. Now please throw one last inspecting glance at the files on the DI Data Lake. Here, you can again use the data browser feature in the Data Intelligence ***Metadata Explorer***. Open the application via the Launchpad.<br><br>
![](/exercises/ex2/images/ex2-019b.JPG)<br><br>

20. In the Metadata Explorer main screen, click on ***Browse Connections***.<br><br>
![](/exercises/ex2/images/ex2-020b.JPG)<br><br>

21. Click on the Connection **"DI_DATA_LAKE"** and drill further down to the folders **"shared"** --> **"DA281"** --> that of **your user** (e.g. "TA99").<br><br>
![](/exercises/ex2/images/ex2-021c.jpg)<br><br>
![](/exercises/ex2/images/ex2-022c.jpg)<br><br>

22. Browse into the specified folder inside your user folder **Enriched Sales Order**. <br><br>
    ![](/exercises/ex2/images/ex1-122b.JPG)<br><br>
    
23. Inside the folder, you will now see that a new CSV file has been generated for the changes that were triggered via the custom operator. Click on the ***More Actions*** menu (the three dots) of the newly created file - that contains the delta records - and click on ***View Fact Sheet***. <br><br>
    ![](/exercises/ex2/images/ex2-040c.jpg)<br><br>
    ***Please Note***: In case you have stopped the pipeline after exercise 1.4 and restarted it in this exercise as defined in exercise 1.5, you will see an additional file in the S3 storage that represent a second initial load of our enriched Sales Order Data. The reason for this is that the second start of the pipeline using the ***Replication*** mode has triggered an additional initial load after starting the pipeline for the second time.<br><br>
    ![](/exercises/ex2/images/ex2-034b.JPG)<br><br> 
24. You can now see that also the processing of the delta data within the Pipeline to the DI Data Lake worked seamlessly. The delta records contain the enrichments and the "U" indicator that with which updates are flagged by the ABAP CDS View CDC framework.<br><br>
![](/exercises/ex2/images/ex2-041c.jpg)<br><br>

25. After checking the data that has been replicated to The DI Data Lake in the Metadata Explorer, please stop your running pipelines.<br><br>
![](/exercises/ex2/images/ex2-029b.JPG)<br><br>

**Very well done!** You successfully followed the integration approach for data processing and functional execution between S/4HANA and SAP Data Intelligence and know how to realize such implementations.<br><br>

Please proceed now to [Exercise 2 - Integrate ABAP CDS Views in SAP Data Intelligence Replication Management Flow](../../exercises/ex3/README.md#exercise-3---integrate-abap-cds-views-in-sap-data-intelligence-replication-management-flow) 


*****************************************************
<br> **Table of Contents / Navigation**
<br>
- [Overview and Getting Started](../../exercises/ex0/README.md#overview-and-getting-started)
  - [Deep Dive demos vs. Exercises](../../exercises/ex0/README.md#deep-dive-vs-exercise-sections-in-this-document)
  - [Short introduction to the Enterprise Procurement Model (EPM) in ABAP systems](../../exercises/ex0/README.md#short-introduction-to-the-enterprise-procurement-model-epm-in-sap-s4hana)
  - Access to the exercises' Data Intelligence environment (**will be provided after the session**)
- [Deep Dive 1 - ABAP CDS View based data extraction in SAP Data Intelligence](../../exercises/dd1/README.md#deep-dive-1---abap-cds-view-based-data-extraction-in-sap-data-intelligence)
- [Deep Dive 2 - Creating a Custom ABAP Operator and making use of it in an SAP Data Intelligence Pipeline](../../exercises/dd2/README.md)
- [Deep Dive 3- Technical Background for Replication Flows in SAP Data Intelligence](../../exercises/dd3/README.md)

- [Overview and Getting Started](exercises/ex0/README.md#overview-and-getting-started)

- [Exercise 1 - Replicating data from S/4HANA ABAP CDS Views in SAP Data Intelligence](../../exercises/ex1/)
    - [Exercise 1.1 - Consume the EPM Business Partner ABAP CDS Views in SAP Data Intelligence](../../exercises/ex1#exercise-11-sub-exercise-1-description)
    - [Exercise 1.2 - Extend the Pipeline to transfer the Customer data into a HANA Cloud Database with Initial Load mode](../../exercises/ex1#exercise-12-sub-exercise-2-description)
    - [Exercise 1.3 - Implement a Pipeline for delta transfer of enhanced EPM Sales Order data from S/4HANA to an S3 Object Store](../../exercises/ex1#exercise-13-sub-exercise-1-description)
    - [Exercise 1.4 - Extend the Pipeline for joining Sales Order with Customer data for each change in Sales Orders and persist results in S3](../../exercises/ex1#exercise-14-sub-exercise-1-description)
    - [Exercise 1.5 - Using a custom ABAP Operator to verify your Delta Replication of EPM Sales Orders](README.md)

- [Exercise 2 - Integrate ABAP CDS Views in SAP Data Intelligence Replication Management Flow](../../exercises/ex3/README.md#exercise-3---integrate-abap-cds-views-in-sap-data-intelligence-replication-management-flow)
<br><br>
