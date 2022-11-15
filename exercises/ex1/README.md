# Exercise 1 - Replicating data from ABAP CDS Views in SAP Data Intelligence

In this exercise, we will leverage in Data Intelligence Pipelines those ABAP CDS Views that got created during the first Deep Dive demo in the connected S/4HANA system.<br><br>
The use case is
- to obtain the Business Partner master data in S/4HANA's demo application **Enterprise Procurement Model (EPM)** and make the records available for the corporate Data Analysts in HANA Cloud Database table.
- to also persist the transactional data for EPM Sales Orders in S3.
- In both cases, any single change of these data sources in the S/4HANA system has to be instantly and automatically replicated to the related files in S3 in append mode.
- Additionally, the Sales Order data have to be enriched with Customer master data. For the initial load and then on every change committed to the EPM Sales Order data in S/4HANA.

(As a reminder: You can recap the relationship between the relevant EPM table entities that are used in this exercise [here](../ex0))<br>

## Exercise 1.1 - Consume the EPM Business Partner ABAP CDS Views in SAP Data Intelligence

After completing these steps you will have created a Pipeline that reads EPM Customer data from an ABAP CDS View in S/4HANA and displays it in a Terminal UI.

1. Log on to SAP Data Intelligence and enter the Launchpad application. Then start the ***Modeler*** application.
   - Follow the link to your assigned Data Intelligence instance, i.e. https://vsystem.ingress.dh-93awrk8fy.dh-canary.shoot.live.k8s-hana.ondemand.com/app/datahub-app-launchpad/.
   - The tenant name should be ***"dat161"***.
   - In the next pop-up window, enter your assigned user name (e.g. ***"TA99"***) and your individual password received from the user registration.<br><br>
   ![](/exercises/ex1/images/ex1-002c.JPG)<br><br>
   ![](/exercises/ex1/images/ex1-002d.JPG)<br><br>
   - From the Launchpad, start the ***Modeler*** application by clicking on the corresponding tile.<br><br>
   ![](/exercises/ex1/images/ex1-003b.JPG)<br><br>

2.	Make sure you are in the ***Graphs*** tab of the Modeler UI (see left side). Then click the ***+*** symbol and select **Use Generation 1 Operators** in order to create a new Pipeline.<br><br>
![](/exercises/ex1/images/ex1-123b.JPG)<br><br>

3.	Now a new Pipeline canvas is opened on the right side and the Modeler UI automatically switches to the ***Operators*** tab (see left side). In the list of operators, drag the ABAP CDS Reader and drop it into the Pipeline canvas. Click the ABAP CDS Reader node in the canvas one time and then click the ***configuration*** icon.<br><br>
![](/exercises/ex1/images/ex1-006b.JPG)<br><br>

4.	In the configuration panel for the ABAP CDS Reader, specify the S/4HANA Connection: Select `S4_RFC_TechEd`. Then click on the ***Version*** selection icon.<br><br>
![](/exercises/ex1/images/ex1-007b.JPG)<br><br>

5.	In the pop-up window, mark the different versions of the ABAP CDS Reader and read through their documentation. As you can see, the ***ABAP CDS Reader V2*** provides a standard message type output (no conversion from ABAP object required). select this entry and click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-008b.JPG)<br><br>

6.	The ABAP CDS Reader operator shell has received the operator's metadata from S/4HANA and has configured the Pipeline node (for example, the operator now has an output port). Now fill in the configuration parameters in the panel:
      - Label: `Buyer - S/4 ABAP CDS` (Optional)
      - Subscription Type: `New`
      - Subscription Name: `BP001_XXXX`, where XXXX is your user name, for example "BP001_TA99"
      - ABAP CDS Name: `Z_CDS_BUYER_Delta`, which is the "simple" CDS View that got created in the Deep Dive 1 demo
      - Transfer Mode: `Initial Load`
      - ***Records per Roundtrip: 5000***
      - Leave all other parameters as-is.<br><br>
   ![](/exercises/ex1/images/ex1-009b.JPG)<br><br>

7.	From the operator list on the left side, drag the ***Get Header*** operator and drop it into the Pipeline canvas. When searching for the ***Get Header*** operator make sure you include the category **Teched** in your search:<br><br>
	![](/exercises/ex1/images/ex1-128b.JPG)<br><br>	
	The Get Header Operator is a small Python operator that is taking care of adding column names of the CDS View as header line into the output CSV data stream.
	Then connect the output port of the ABAP CDS Reader with the input port of the Get Header operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.  <br><br>
![](/exercises/ex1/images/ex1-137b.JPG)<br><br>
Open the configuration of the ***Get Header*** operator and select ***only once*** for the Add column names configuration parameter. The ***only once*** selection will make sure that the column names of the CDS View will only be added once for all data packages being received from S/4 HANA. <br><br>
![](/exercises/ex1/images/ex1-124b.JPG)<br><br>

8.  From the operator list on the left side, drag the ***Terminal*** operator and drop it into the Pipeline canvas. Then connect the upper output port of the Get Header with the input port of the Terminal operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-010c.JPG)<br><br>

8.	The Terminal displays string type data. Because the output of the ABAP CDS Reader is a message, its body/payload needs to be converted into a string. After you have connected the operator ports you are, hence, prompted for a conversion approach. Select the first of the two options (only the body will be processed) and click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-011b.JPG)<br><br>

9.	The selected converter is now added to the Pipeline. Please click on the ***Auto Layout*** button to arrange the Pipeline nodes.<br><br>
![](/exercises/ex1/images/ex1-012b.JPG)<br><br>
![](/exercises/ex1/images/ex1-012c.JPG)<br><br>

10.	***Save*** your Pipeline.
      - Click on the Disk symbol in the menue bar.<br><br>
      ![](/exercises/ex1/images/ex1-013b.JPG)<br><br>
      - Because you save the Pipeline for the first time, you are prompted for some inputs.<br>
      As Name, enter **`teched.XXXX.EPM_Customer_Replication_to_HANA`**, where **XXXX** is your user name, for example "teched.TA99.EPM_Customer_Replication_to_S3".<br>
      As Description, please enter **`XXXX - Replicate S/4HANA EPM Customer Data to HANA`**,where **XXXX** is your user name, for example "TA99 - Replicate S/4HANA EPM Customer Data to a target HANA Cloud Database". (The description will show up in the Pipeline status information later on.)<br>
      As Catergory, enter **`dat161`**, which is the name under which you can find your Pipeline in the ***Graphs*** tab.<br>
      Click ***OK***.<br><br>
      ![](/exercises/ex1/images/ex1-014b.JPG)<br><br>

11.	After you have saved the Pipeline, it will get validated by SAP Data Intelligence. Check the validation results. If okay, you can now execute the Pipeline by clicking the ***Play*** icon in the menue bar. Then change to the ***Status*** tab in the Modeler UI status section on the lower right side.<br><br>
![](/exercises/ex1/images/ex1-015b.JPG)<br><br>

12.	Once the status of your Pipeline has changed to ***running***, click on the ***Terminal*** operator node one time and then on its ***Open UI*** icon.<br><br>
![](/exercises/ex1/images/ex1-016b.JPG)<br><br>

13.	You should now see that EPM Customer master data is coming in, which proves that the integration with the S/4HANA CDS View is working as expected. By the way: the truncation of payload content can be determined by the ***Max size (bytes)*** parameter in the Terminal operator configuration.<br><br>
![](/exercises/ex1/images/ex1-017b.JPG)<br><br>

14.	***Stop*** the Pipeline execution again by clicking the marked icons in the menue bar or in the status section. The status of the Pipeline will then change from ***running*** to ***stopping*** and finally to ***completed***.<br><br>
![](/exercises/ex1/images/ex1-018b.JPG)<br><br>

**Congratulations!** You have now successfully implemented a Pipeline that consumes an ABAP CDS View from the connected SAP S/4HANA system.
As a next step, you will persist the EPM Customer master data in a SAP HANA Cloud database table.

## Exercise 1.2 - Extend the Pipeline to transfer the Customer data into a HANA Cloud Database with Initial Load mode

After completing the following steps you will have extended the Data Intelligence Pipeline with an additional persistency for the S/4HANA EPM Business Partner data in HANA Cloud. Therefore, a pre-defined S/4HANA CDS View will be used as a data source and loaded into a table located in a SAP HANA Cloud database.

1. If not already done, open the Pipeline from the previous section (**`teched.XXXX.EPM_Customer_Replication_to_HANA`**, where XXXX is your user name). Click on the Terminal operator and then the "waste bin" icon in order to delete the Terminal operator. Do the same for the To String Converter operator. Just keep the the ABAP CDS Reader operator in the Pipeline canvas.<br><br>
![](/exercises/ex1/images/ex1-019b.JPG)<br><br>

2. Make sure that the ***Operators*** tab is in scope in the Modeler UI (see left side). From the list of operators, drag the ***HANA Cient*** operator and drop it into the Pipeline canvas. <br><br>
Now connect the **output port of the Get Header Operator** with the **input port of the HANA Client operator** by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-020b.JPG)<br><br>

3. From the list of operators, drag the ***Wiretap*** operator and drop it in the Pipeline canvas. Similar to a Terminal operator, this operator can wiretap a connection between two operators in a Pipeline and display the traffic to the browser window (or to an external websocket client that connects to this operator). But the Wiretap operator also supports throughput of type message (amongst others), so that no type conversion is needed between the HANA Client and the Wiretap operator.
Now connect the ***output port of the HANA Client*** with the ***input port of the Wiretap operator*** by pulling the mouse pointer from one port to the other while the left mouse button is pressed.
.<br><br>
![](/exercises/ex1/images/ex1-021b.JPG)<br><br>

4. Click on the ***HANA Client*** operator and click its configuration icon. (As a reminder: You can always change from the configuration panel to the online documentation of each standard Pipeline operator, by clicking on the "Show Documentation" icon ![](/exercises/ex1/images/Show_Documentation.JPG) on the upper right side of the Modeler UI above the configuration panel).<br><br>
![](/exercises/ex1/images/ex1-022b.JPG)<br><br>


5. Enter the needed configuration parameters for the ***Write File*** operator. These are:<br>
   - Label: **`Customer Master to HANA`**<br>
   - Connection: Choose type ***Connection Management*** and then the connection ID **`TechEd_HANA`**<br>
   - Output in Batches: **False**<br>
   - Table Name: <br>
	 Click the browse button next to Table parameter: <br><br>
	![](/exercises/ex1/images/ex1-129b.JPG)<br><br>
	
	Select your user schema “XXXX_EPM_TARGET”, where XXXX refers to your user name, e.g. “TA99_EPM_TARGET. <br><br>
	![](/exercises/ex1/images/ex1-023b.JPG)<br><br>

	Double click on your user schema and inside your user schema you will find a pre-defined table with your user id “XXXX_BUSINESS_PARTNER”, e.g. “TA99_BUSINESS_PARTNER” <br><br>
	![](/exercises/ex1/images/ex1-024b.JPG)<br><br>

	Click on ***OK*** to save your target table selection.<br><br>
	![](/exercises/ex1/images/ex1-025b.JPG)<br><br>

	Make sure the following additional configuration settings are applied to define the correct format of the incoming csv data stream:<br>
	- Input Format: ***CSV*** <br>
	- CSV Mode: ***Batch*** <br>
	- CSV Record delimiter: ***\n*** <br>
	- CSV field delimiter: ***,*** <br>
	- CSV quote delimiter: ***“*** <br>
	- CSV header: ***Ignore*** <br>
	- Insert mode: ***INSERT*** <br>
	- Table Initialization: ***None*** <br>
	- Terminate on Error: ***True*** <br>

	![](/exercises/ex1/images/ex1-026b.JPG)<br><br>

	![](/exercises/ex1/images/ex1-027b.JPG)<br><br>

6. Now ***Save*** your Pipeline, verify the validation results and - if okay - run the Pipeline by clicking on the ***Play*** symbol in the menue bar.<br><br>
![](/exercises/ex1/images/ex1-028b.JPG)<br><br>

	In this case the validation shows two warnings that can be ignored. The topic about the missing resources is optional if you want to define specific resources to a graph with a minimum and maximum number of CPU + memory the graph is allowed to use.
	More information can be found here: https://help.sap.com/viewer/1c1341f6911f4da5a35b191b40b426c8/Cloud/en-US/b0a5a31101304ef6be7ce3708aadceea.html?q=resources <br>


7. Once the Pipeline has the status ***running***, click on the ***Wiretap*** operator and then click its ***Open UI*** icon.<br><br>
![](/exercises/ex1/images/ex1-029b.JPG)<br><br>

8. In the ***Wiretap UI*** you should now see the processed messages coming from the HANA Client Operator. In this case it indicates that one successful batch of has been processed to HANA.<br><br>
![](/exercises/ex1/images/ex1-030b.JPG)<br><br>

9. ***Stop*** the Pipeline.<br><br>
![](/exercises/ex1/images/ex1-031b.JPG)<br><br>

10. As a verification of the successful load of the data to the file in S3, we can use the ***Metadata Explorer*** as integrated part of SAP Data Intelligence. Please go back to the Launchpad and click on the corresponding tile.<br><br>
![](/exercises/ex1/images/ex1-032b.JPG)<br><br>

11. In the ***Metadata Explorer*** application main menue, click on ***Browse Connections***.<br><br>
![](/exercises/ex1/images/ex1-033b.JPG)<br><br>

12. In order to easily find our connection to the target HANA Cloud Database, you may leverage the search functionality. Enter `TechEd` into the search field and click on the spyclass icon. Click on the **`TechEd_HANA`** tile.<br><br>
![](/exercises/ex1/images/ex1-034b.JPG)<br><br>

13. On the next screen search for your user schema that you selected in your Pipeline, e.g. XXXX_EPM_TARGET, where XXXX stands for your user ID.<br><br>
![](/exercises/ex1/images/ex1-035b.JPG)<br><br>

14. Select your HANA table that you previously selected in the HANA Client operator and open the factsheet by clicking on the glasses ***View Fact Sheet*** option on the right-hand side of your table:<br><br>
![](/exercises/ex1/images/ex1-036b.JPG)<br><br>

15. If your Pipeline ran successfully, you'll see data in your table once you switch to the tab ***Data Preview***<br><br>
![](/exercises/ex1/images/ex1-037b.JPG)<br><br>

    Now you can see that the EPM Customer data got loaded into a target table in HANA Cloud. Success!

16. In the Overview section you can find additional metadata of the HANA table such as the column names + data types of our Business Partner table:. Success!<br><br>
![](/exercises/ex1/images/ex1-038b.JPG)<br><br>

**Very well done!** You have implemented a Pipeline that extracts Initial Load data from an ABAP CDS View in S/4HANA to a table stored in a target HANA Cloud database.

## Exercise 1.3 - Implement a Pipeline for delta transfer of enhanced EPM Sales Order data from S/4HANA to an S3 Object Store

In the next section, we'll also take care for the Sales Order transaction data from EPM and will right away establish a replication (initial load plus following delta processing) transfer mode.

1. In SAP Data Intelligence, open the ***Modeler*** application. Make sure the scope of the Modeler UI is on tab ***Graphs*** (see left side). Then click the ***+*** button and select **Use Generation 1 Operators** to create a new Pipeline.<br><br>
![](/exercises/ex1/images/ex1-123b.JPG)<br><br>

2. Now a new Pipeline canvas is opened on the right side and the Modeler UI automatically switches the scope to the ***Operators*** tab (see left side). In the list of operators, drag the ABAP CDS Reader and drop it into the Pipeline canvas. Click the ABAP CDS Reader node in the canvas one time and then click the ***configuration*** icon.<br><br>
![](/exercises/ex1/images/ex1-006b.JPG)<br><br>

3.	In the configuration panel for the ABAP CDS Reader, specify the S/4HANA Connection: Select `S4_RFC_TechEd`. Then click on the ***Version*** selection icon.<br><br>
![](/exercises/ex1/images/ex1-007b.JPG)<br><br>

4.	In the pop-up window, select the option ***ABAP CDS Reader V2*** and click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-008b.JPG)<br><br>

6.	The ABAP CDS Reader operator shell has received the operator's metadata from S/4HANA and has configured the Pipeline node (for example, the operator now has an output port). Now fill in the configuration parameters in the panel:
      - Label: `Sales Order - S/4 ABAP CDS` (Optional)
      - Subscription Type: `New`
      - Subscription Name: `SO001_XXXX`, where XXXX is your user name, for example "SO001_TA99"
      - ABAP CDS Name: `Z_CDS_SO_SOI_Delta`, which is the "more complex" CDS View that got created in the Deep Dive 1 demo
      - Transfer Mode: `Replication`
      - ***Records per Roundtrip: 5000***
      - Leave all other parameters as-is.<br><br>
   ![](/exercises/ex1/images/ex1-039b.JPG)<br><br>
   
7. From the operator list on the left side, drag and drop the ***Get Header*** operator into the Pipeline canvas.  When searching for the ***Get Header*** operator make sure you include the category **Teched** in your search:<br><br>
	![](/exercises/ex1/images/ex1-128b.JPG)<br><br>	
	Then connect the output port of the ABAP CDS Reader with the input port of the Get Header operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-138b.JPG)<br><br>
Open the configuration of the ***Get Header*** operator and select ***only once*** for the Add column names configuration parameter. The ***only once*** selection will make sure that the column names of the CDS View will only be added once for all data packages being received from S/4 HANA. This is required as we want to Append the data using the Write File operator. <br><br>
![](/exercises/ex1/images/ex1-130b.JPG)<br><br>

8.	From the operator list on the left side, drag the ***Wiretab*** operator and drop it into the Pipeline canvas. Then connect the output port of the ABAP CDS Reader with the input port of the Wiretab operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-041b.JPG)<br><br>

9. From the list of operators, drag the ***Write File*** operator and drop it in the Pipeline canvas. Then connect the output port of the ABAP CDS Reader with the input port of the Wiretap operator by pulling the mouse pointer from one port to the other while the left mouse button is pressed.<br><br>
![](/exercises/ex1/images/ex1-042b.JPG)<br><br>

10. The message format from the Wiretap operator output must be transformed to a file format. For this reason you are prompted to choose an appropriate converter operator. On the pop-up window, select the first option (transfer the content). Click ***OK***. Afterwards you can again use the ***Auto Layout*** button to re-arrange the operators.<br><br>
![](/exercises/ex1/images/ex1-043b.JPG)<br><br>
  
11. Click on the ***Write File*** operator and click its ***configuration*** icon.<br><br>
![](/exercises/ex1/images/ex1-044b.JPG)<br><br>

12. Enter the needed configuration parameters for the ***Write File*** operator. These are:
   - Label: **`Sales Order to S3`** (Optional)
   - Connection: Choose type ***Connection Management*** and then the connection ID **`DI_DATA_LAKE`**
   - Path mode: **`Static (from configuration)`**
   - Path: **`/shared/DA281/XXXX/Sales_Order_0001.csv`**, where XXXX is your User name, for example "/shared/DA281/TA99/Sales_Order_0001.csv"
   - Mode: **`Append`**
   - Join batches: **`True`**<br><br>
   
   Please note that not all cloud storages do natively support **Append** functionality and you should check the write file approach in your project based on your requirements as well as supported functionality of your favorit cloud storage. In this case we choose **Append** mode for simplification reasons of this exercise.<br><br>
<!--- ![](/exercises/ex1/images/ex1-045b.JPG)<br><br> --->

13.	***Save*** your Pipeline.
      - Click on the Disk symbol in the menue bar.
      - Because you save the Sales Order Pipeline for the first time, you are prompted for some inputs.<br>
      - As Name, enter **`teched.XXXX.EPM_SalesOrder_Replication_to_S3`**, where **XXXX** is your user name, for example "teched.TA99.EPM_SalesOrder_Replication_to_S3".<br>
      - As Description, please enter **`XXXX - Replicate S/4HANA EPM Customer Data to S3`**,where **XXXX** is your user name, for example "TA99 - Replicate S/4HANA EPM Sales  Order Data to S3". (The description will show up in the Pipeline status information later on.)<br>
      - As Catergory, enter **`dat161`**, which is the name under which you can find your Pipeline in the ***Graphs*** tab.<br>
      - Click ***OK***.<br><br>
      ![](/exercises/ex1/images/ex1-046b.JPG)<br><br>

14.	After you have saved the Pipeline, it will get validated by SAP Data Intelligence. Check the validation results. If okay, you can now execute the Pipeline by clicking the ***Play*** icon in the menue bar. Then change to the ***Status*** tab in the Modeler UI status section on the lower right side.<br><br>
![](/exercises/ex1/images/ex1-047b.JPG)<br><br>

	In this case the validation shows one warning that can be ignored. The topic about the missing resources is optional if you want to define specific resources to a graph with a minimum and maximum number of CPU + memory the graph is allowed to use.<br>
	More information can be found here: https://help.sap.com/viewer/1c1341f6911f4da5a35b191b40b426c8/Cloud/en-US/b0a5a31101304ef6be7ce3708aadceea.html?q=resources 


15.	Once the status of your Pipeline has changed to ***running***, click on the ***Wiretap*** operator node one time and then on its ***Open UI*** icon.<br><br>
![](/exercises/ex1/images/ex1-048b.JPG)<br><br>

16. In the ***Wiretap UI*** you should now see the processed Sales Order messages coming from the ABAP CDS Reader. This proves that the integration with the S/4HANA CDS View is working as expected.<br><br>
![](/exercises/ex1/images/ex1-049b.JPG)<br><br>

17. For validating the correct creation of the file in S3, please leverage the Data Intelligence Metadata Explorer again. Go back to the Launchpad and start the Metadata Explorer application, if not already launched from the previous exercise.<br><br>

18. In the ***Metadata Explorer*** application main menue, click on ***Browse Connections***.<br><br>
![](/exercises/ex1/images/ex1-050b.JPG)<br><br>

19. In order to easily find our connection to the target S3 Object Store, you may leverage the search functionality. Enter `TechEd_s` into the search field and click on the spyclass icon. Click on the **`DI_DATA_LAKE`** tile.<br><br>
![](/exercises/ex1/images/ex1-051b.JPG)<br><br>

20. On the next drill-down view, click on the **`DAT161`** directory that you had specified in the ***Write File*** operator, and then drill down to your specific User directory, for example **`TA99`**.<br><br>
![](/exercises/ex1/images/ex1-052b.JPG)<br><br>

21. If your Pipeline ran successfully, you'll find a file with your specified name (`Sales_Order.csv`) Click on the glasses icon ***View Fact Sheet*** to open up the Fact Sheet for the CSV file. <br><br>
![](/exercises/ex1/images/ex1-053b.JPG)<br><br>

22. In the ***Fact Sheet***, which provides some more overview and statistical information about the new file, go to tab ***Data Preview***.<br><br>
![](/exercises/ex1/images/ex1-054b.JPG)<br><br>

23. Success! Now you can see that the EPM Customer data got loaded into the target file in S3. While the Pipeline is running, this file would get automatically updated with each change in the S/4HANA data sources (the tables `SNWD_SO`, `SNWD_SO_I`, `SNWD_PD`, `SNWD_TEXTS` joined through the ABAP CDS View `Z_CDS_SO_SOI_Delta`).<br><br>
![](/exercises/ex1/images/ex1-055b.JPG)<br><br>

24. However, in order to go on with the exercise, please stop the Pipeline for now. We will validate the delta processing in Exercise 2.<br><br>
![](/exercises/ex1/images/ex1-056b.JPG)<br><br>

**Congratulations!** You have created the Sales Order extraction from a delta-enabled, more complex ABAP CDS View into the S3 Object Store. Because you have chosen the transfer mode ***"Replication"*** in the CDS Reader operator configuration, the Pipeline has conducted the Initial Load and would now wait for any changes (inserts, updates, deletions) in the S/HANA EPM Sales Order object as long as it is running.

As a next step, you will enrich the Sales Order data with some Customer Details (e.g. Name and Legal Form) during the Replication process.


## Exercise 1.4 - Extend the Pipeline for joining Sales Order with Customer data for each change in Sales Orders and persist results in S3

In this last part of the S/4HANA ABAP CDS View intergration exercise, you will establish a Pipeline that replicated the Sales Order data from the delta-enabled ABA CDS View in S/4HANA and joins it with some of the details that you have replicated from the Customer master data, i.e. company name and legal form.

1. In SAP Data Intelligence, open the ***Modeler*** application. Make sure the scope of the Modeler UI is on tab ***Graphs*** (see left side). If not still open from the previous exercise part, search for your Sales Order Pipeline by entering your user name into the search field on top of the list of Pipelines. Click on Pipeline with the label `XXXX - Replicate S/4HANA EPM Sales  Order Data to S3`, where XXXX is your user name, for example "TA99 - Replicate S/4HANA EPM Sales  Order Data to S3". The Pipeline will get opened then or will get the focus on the UI if it was already opened. If you cannot see the complete label of the Pipeline, just hover with your mouse over the pipeline item in the list.<br><br>
![](/exercises/ex1/images/ex1-057b.JPG)<br><br>

2. Click on the pull-down menue of the ***Save*** icon (disk symbol) and choose ***Save As***.<br><br>
![](/exercises/ex1/images/ex1-058b.JPG)<br><br>

3. You are prompted for some inputs. Please enter the following values.
    - As Name, enter **`teched.XXXX.EPM_SalesOrder_Replication_Enrich_to_S3`**, where **XXXX** is your user name, for example "teched.TA99.EPM_SalesOrder_Replication_Enrich_to_S3".<br>
    - As Description, please enter **`XXXX - Replicate and Enrich S/4HANA EPM Sales Order Data to S3`**,where **XXXX** is your user name, for example "TA99 - Replicate and Enrich S/4HANA EPM Sales Order Data to S3". (The description will show up in the Pipeline status information later on.)<br>
    - As Catergory, enter **`dat161`**, which is the name under which you can find your Pipeline in the ***Graphs*** tab.<br>
    - Then click ***OK***.<br><br>
![](/exercises/ex1/images/ex1-059b.JPG)<br><br>

4. A new Pipeline has been created as a copy of your original Sales Order replication Pipeline. You will be using the Write File Operator outputs (i.e. the file path and name) as a trigger for the join processes with the Customer master data.<br><br>
   
   Optinally you can either keep the Wiretap operator in the pipeline or remove it and directly connect the Get Header Operator with the Write File Operator as it is being done in the following steps. 

5. Open the configuration of the File Writer operator and enter the new target path/file name **`/shared/DA281/TA99/Sales_Order_Staging.csv`**. Change Mode to **`Overwrite`**. Then save the Pipeline.<br><br>
![](/exercises/ex1/images/ex1-060b.JPG)<br><br>

6. Open the configuration of the ***Get Header*** operator and change the Add column names configuration parameter to ***for each batch***. The ***for each batch*** selection will make sure that the column names of the CDS View will only be added once for all data packages being received from S/4 HANA. This is required as we want to overwrite the staging file, which will then be joined with additional metadata from the customer master data for each processed file.<br><br>
![](/exercises/ex1/images/ex1-125b.JPG)<br><br>

7. Open the configuration of the ***CDS Reader Operator*** operator and change the name of the subscription to a new name, e.g. adding a ***_Enriched*** at the end.<br><br>
![](/exercises/ex1/images/ex1-139b.JPG)<br><br>

8. Click on the ***Operators*** tab on the left side in order to get the list of operators. Find the ***Graph Terminator*** operator and drag it from the operator list into the canvas. Then connect the upper output port ("file") of the File Writer with the input port of the Graph Terminator operator.<br><br>
The Graph Terminanor allows us to run the Pipeline once, and when the new file got generated, the File Writer will send a termination signal to the Graph Terminator, which then stops the Pipeline. We do this just for the purpose of generating the new output file as a reference / import sample for the next steps.<br><br>
![](/exercises/ex1/images/ex1-061b.JPG)<br><br>

9. ***Save*** and ***Run*** the Pipeline. Wait until the Pipeline status turns from ***pending*** to ***processing*** and finally to ***completed***.<br><br>
![](/exercises/ex1/images/ex1-062b.JPG)<br><br>

10. After the completion of the Pipeline execution, **remove the Graph Terminator** operator from the Pipeline again.<br><br>
![](/exercises/ex1/images/ex1-063b.JPG)<br><br>

11. From the list of operators on the left side, drag the ***Structured File Consumer*** operator and drop it into the Pipeline canvas. Then open the configuration panel and do the following entries:
   - Increase the **Fetch size** to `10000`
   
   Then click the pencil of the **Source Object** parameter to configure the source file located in the S3 connection. Drill down to your individual folder under **DAT161** and select the file **`Sales_Order_Staging`**. Then click ***OK***.<br><br>
   ![](/exercises/ex1/images/ex1-131b.JPG)<br><br>
     
12. Select Service to SDL and select the previously used Connection ID DI_DATA_LAKE: <br><br> 
![](/exercises/ex1/images/ex1-065b.JPG)<br><br>

13. Then click the ***S3 Source File*** folder icon to browse through the S3 bucket. Drill down to your individual folder under ***DAT161*** and select the file ***Sales_Order_Staging.csv***. Then click ***OK***. <br><br>
![](/exercises/ex1/images/ex1-066b.JPG)<br><br>


	Once you select the target file, the dialog will automatically capture metadata of the csv file, e.g. delimiters and whether the file includes header information or not.<br><br>
	![](/exercises/ex1/images/ex1-067b.JPG)<br><br>

	In the Source option you will now see that file path has been updated: <br><br>
	![](/exercises/ex1/images/ex1-068b.JPG)<br><br>

14. Set the parameter Fall ***on String Truncation*** to false by switching it off: <br><br>
![](/exercises/ex1/images/ex1-069b.JPG)<br><br>

	You are now able to open the ***Data Preview*** of the connected file in S3. Give it a try, if you want!<br><br>
	For the Preview you can choose between the preview of data how is stored in the source or the adapted data in case you have performed any changes:<br><br>
	![](/exercises/ex1/images/ex1-070b.JPG)<br><br>
	![](/exercises/ex1/images/ex1-071b.JPG)<br><br>
	
    Navigate back to your pipeline using the ***Back*** button:<br><br>
    ![](/exercises/ex1/images/ex1-133b.JPG)<br><br>

15. Now connect the **upper output port ("file") of the File Writer** operator with the **input port of the Structured File Consumer** operator. The ***Structured File Consumer*** operator takes the signal on the input port just as a trigger for commencing its logic. It's an optional input but our approach ensures that the operator is only executed after the file from the previous node has been successfully written on S3. (If the input port of a Structured File Consumer is not connected, the operator will start with the Pipeline execution.)<br><br>
	When you create the link between the operators, a conversion of the data type is required from type `message.file` to type `message`. The ***Converter*** is automatically proposed when the link between the ports is established. Choose the option for ***Path extraction*** (which reflects the minimum transfer payload, as we use the input just as a trigger signal).<br><br>
	![](/exercises/ex1/images/ex1-072b.JPG)<br><br>
	![](/exercises/ex1/images/ex1-073b.JPG)<br><br>

16. Now we need to a ***Table Consumer*** operator for the Customer master data extraction from HANA Cloud. Go to the operator tab on the left-hand panel to drag and drop the Table Consumer operator in your graph.<br><br>
![](/exercises/ex1/images/ex1-074b.JPG)<br><br>

	In case the ***To File Operator*** has been added in a similar way as above, you can easily move it to another position within your pipeline depending on your needs. <br><br>

17. On this new operator, open the configuration panel and click the **S3 Source File** folder icon to browse through the S3 bucket. Drill down to your individual folder under **DAT161** and select the file **`Customer_Master.csv`**. Then click ***OK***..<br><br>
![](/exercises/ex1/images/ex1-075b.JPG)<br><br>

	Select the service to “HANA_DB” and choose “TechEd_HANA” as Connection ID, which has been used as target for storing the Customer Master Data in our previous exercise. <br><br>
	![](/exercises/ex1/images/ex1-076b.JPG)<br><br>

	Click on the Browse button of the Source parameter: <br><br>
	![](/exercises/ex1/images/ex1-077b.JPG)<br><br>

	The following selectin screen appears. Now drilldown into your user HANA schema in HANA Cloud where you have stored your Customer Data and confirm the selection with a click on ***OK***, e.g.: <br><br>
	xxxx_EPM_TARGET / xxxx_BUSINESS_PARTNER, where xxxx stands for your user ID, e.g. TA99_EPM_TARGET / TA99_BUSINESS_PARTNER  <br><br> 

	![](/exercises/ex1/images/ex1-078b.JPG)<br><br>

18. Once you selected the Customer data in HANA, the dialog will automatically fetch the metadata of the table and show on the bottom of the screen: <br><br> 
![](/exercises/ex1/images/ex1-079b.JPG)<br><br>

19. From the list of operators on the left side, drag the ***Data Transform*** operator and drop it into the Pipeline canvas. Then connect the ***output port of the Structured File Consumer*** and the output port of the ***Table Consumer *** with the ***Data Transform box***. This will create the needed input ports of the Data Transform operator.<br><br>
![](/exercises/ex1/images/ex1-080b.JPG)<br><br>

20. Double-click on the ***Data Transform*** operator.<br><br>
![](/exercises/ex1/images/ex1-081b.JPG)<br><br>

21. In the details view, from the list of Data Operations on the left side, drag the ***Join*** operation into the canvas with the existing two ***input nodes*** and connect these with the ***Join*** operation.<br><br>
![](/exercises/ex1/images/ex1-082b.JPG)<br><br>

22. Double-click on the ***Join*** operation.<br><br>
![](/exercises/ex1/images/ex1-083b.JPG)<br><br>

23. Re-arrange the position of input tables on the canvas as per your convenience.<br><br>
![](/exercises/ex1/images/ex1-084b.JPG)<br><br>

24. Connect ***"BUYERGUID" field of Join_Input1*** (Sales Order) with the ***"NODEKEY" field of Join_Input2*** (Customer). This will map the Customer GUID fields of the two tables. We're using an INNER JOIN..<br><br>
![](/exercises/ex1/images/ex1-085b.JPG)<br><br>

	You can leave the default settings displayed in the Join Definition screen: <br><br>
	![](/exercises/ex1/images/ex1-086b.JPG)<br><br>

25. Proceed to tab ***Columns*** in order to select the output fields. Click on the ***Auto map by name*** icon (see red box), which enter all fields of the two input nodes to the list of output fields.<br><br> 
![](/exercises/ex1/images/ex1-087b.JPG)<br><br>

26. Now you can choose if you want to remove some columns in the target data set, e.g. in this case we are selecting & removing the following two columns as an example:<br><br>
![](/exercises/ex1/images/ex1-088b.JPG)<br><br>

27. Navigate back.<br><br>
![](/exercises/ex1/images/ex1-089b.JPG)<br><br>

28. Click with the right mouse button on the output port of the ***Join*** operation. Then click on ***Create Data target***.<br><br>
![](/exercises/ex1/images/ex1-090b.JPG)<br><br>

29. A new output node has been created, which will now be available on the Data Transfer operator. Navigate further back.<br><br>
![](/exercises/ex1/images/ex1-091b.JPG)<br><br>

30. Back on the Pipeline canvas, drag the ***Structured File Producer*** operator from the list of operators into the Pipeline canvas. Then connect the ***output port of the Data Transform*** with the ***input port of the Structured File Producer***.<br><br>
![](/exercises/ex1/images/ex1-092b.JPG)<br><br>

31. Open the configuration panel of the ***Structured File Producer*** operator aby clicking on the Edit / Pencil icon of the Target Configuration to start the configuration of your target S3 scenario:<br>
    ![](/exercises/ex1/images/ex1-107b.JPG)<br><br>
	
	Select SDL as ***Service*** and use Connection ID ***DI_DATA_LAKE***. <br><br>
	![](/exercises/ex1/images/ex1-111b.JPG)<br><br>
	
	Click on the ***Browse*** Button of ***Target*** parameter: <br><br>
	![](/exercises/ex1/images/ex1-109b.JPG)<br><br>
	
	Browse to the path in S3 with /shared/DA281/XXXX, where XXXX stands for your user ID e.g. /shared/DA281/TA99 and click on + icon: <br><br>
	![](/exercises/ex1/images/ex1-110b.JPG)<br><br>
	
	Enter a name of the target folder, which we will sue to store our files, e.g. ***Enriched_Sales_Order*** and click on ***Add*** <br><br>
	![](/exercises/ex1/images/ex1-120b.JPG)<br><br>
	
	You will now see the folder is being added in the S3 Object store. Select it and click on ***OK***.<br><br>
	
	The Target is now being updated to the folder you specified, which we will use to store our enriched sales orders in form of part-files.<br><br>
	![](/exercises/ex1/images/ex1-121b.JPG)<br><br>
	
	In addition please check the following configurations: <br><br>

    - **Format:** `CSV`
    - Leave the **CSV Properties** as-is
    - **Header:** ON
    - **Write Part Files** should be `True`
    - **Mode** should be `Append` <br><br>
	
	![](/exercises/ex1/images/ex1-112b.JPG)<br><br>
	
    ***Save*** the Pipeline <br><br>
	
	In case the following warning message appears, check the Save layout changes box and click on OK.<br><br>
	![](/exercises/ex1/images/ex1-093b.JPG)<br><br>
	
	When selecting the Structured File Producer Operator, you will now see that the Target status has been updated to Configured.<br><br>
	![](/exercises/ex1/images/ex1-105b.JPG)<br><br>

32. Open the configuration of the ***CDS Reader Operator*** operator and change the name of the subscription to a new name, e.g. adding a ***_Enriched_Delta*** at the end. This is required as a new initialization inlcuding the initial load is requiring a new subscription on the SAP S/4HANA side:<br><br>
![](/exercises/ex1/images/ex1-140b.JPG)<br><br>

33. ***Run*** the Pipeline: <br><br>
![](/exercises/ex1/images/ex1-095b.JPG)<br><br>

34. When the Pipeline is in status ***running***, you can take a look into your folder in the S3 bucket, again using the ***Data Intelligence Metadata Explorer*** application.     After drilling to your specifiec folder DAT161/XXXX, where XXXX stand for your user ID e.g. TA99, you should see the folder that was specified in the *Structured File Producer*  operator (= "Enriched_Sales_Order").<br><br>
    ![](/exercises/ex1/images/ex1-122b.JPG)<br><br>
    
    Inside the directory you will see that one part file has been generated where our enriched sales orders are stored. Click the ***Glasses*** icon on that tile in order to see the ***Data Preview*** of the ***Fact Sheet***.<br><br>
    ![](/exercises/ex1/images/ex1-106b.JPG)<br><br>
    Optionally, you can switch the view in the Metadata Explorer using the ***List View*** button:
    ![](/exercises/ex1/images/ex1-136b.JPG)<br><br>
    ![](/exercises/ex1/images/ex1-114b.JPG)<br><br>

35. As you can see in the Data Preview tab, the two columns with the company name and the legal form of the Customer Master Data table have been added to the Sales Order data.<br><br>
![](/exercises/ex1/images/ex1-116b.JPG)<br><br>

36. As long as the Pipeline is running, you would now receive any updates in S/4HANA on the EPM Sales Order data, enriched with the lookups of the EPM Customer master data. In this particular example every update in the source S/4 HANA system would trigger a new file being generated in the specified S3 folder. The approach is a simple example for our hands-on session and can be further extended in a real project, e.g. by merging smaller delta files into bigger ones based on a defined file size threshold. The file in S3 would look as displayed below. In case an update happened int he source S/4 HANA system you get the ***update*** indicator "U" in the column "/1DH/OPERATION".<br><br>
![](/exercises/ex1/images/ex1-118b.JPG)<br><br>

## Summary

**Congrats!** You have now implemented a Pipeline that receives the Initial Loads and the Change Data Capture (CDC) information from different S/4HANA ABAP CDS Views.<br>

As a matter of fact, in this virtual workshop at TechEd, we can't provide access to the SAP GUI of the connected SAP S/4HANA system in order to run the EPM data generation transaction **`SEPM_DG`**.

In the next exercise we will mitigate this problem and allow you to execute a variant of this ABAP Report, leveraging the ABAP Function Module calls from SAP Data Intelligence.
Please continue to [Exercise 2 - Triggering the execution of a function module in a remote S/4HANA system](../ex2/README.md).

<br><br>

*****************************************************
<br> **Table of Contents / Navigation**

