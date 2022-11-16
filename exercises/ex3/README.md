# Exercise 3 - Integrate ABAP CDS Views in SAP Data Intelligence Replication Management Flow
For the integration of ABAP CDS Views in S/4HANA, SAP Data Intelligence provides standard ABAP operator shells that can easily be configured according to your use cases. In principal, you can leverage the following approach:

> **Important**:
> 
> Replace the subfix XX within this exercise with your user ID of this exercise session.

1. Logon to SAP Data Intelligence to access the Launchpad application and click on the Modeler tile.

![DI Overview Launchpad](img/DI_Launchap_Overview.png)

2. In the DI Modeler click on the right side the Replication tab in the middle to access the Replication Management Functionalities in Data Intelligence

![Replication Flow](img/DI_RMS_Button.png)

3. Click on the Plus icon at the top of your window to create a new Replication Flow

![Plus Button RMS](img/DI_RMS_Create_RF.png)

4. Enter a name for the Replication Flow in the pop-up window

![RF Name](img/DI_RF_Name.png)

5. With the newly visible window we need to specify two different main parameter setting. One for the source system and the second for the target system. 
   
   Select the following details from the dropdown to set up the replication flow:
   - Source:
     - Source Connection: **S4_HANA (ABAP)**
     - Container: **CDS**
   - Target
      - Target Connection: **HANA_CLOUD (HANA_DB)**
      - Container: **TAXX_EPM**

![RF Configuration](img/RF_Configuration.png)

1. After filling the necessary properties for the Replication Flow, we click on the Tasks tab to replicate the CDS View **Z_CDS_EPM_SO**. 
2. Click on the Create button to create a task for the CDS View replication

![RF Create Task](img/RF_Create_Task.png)

1. Select the CDS View by navigating to the TMP folder and click on the CDS View **Z_CDS_EPM_SO**. To add the CDS View to the Replication Flow click the Ok button on the Pop-Up window
2. Repeat step 8 for the CDS views **Z_CDS_EPM_PO** and **Z_CDS_EPM_PD**.

3.  After adding all three CDS views to the created Replication Flow, change the Load Type of each CDS View from Initial Only to Initial and Delta over the drop-down menu in the column Load Type.

![RF Task Overview](img/RF_Replication_Details.png)

11. For the three CDS Views we define different preparations techniques which can be used for the replication. We differentiate between Filter and Mapping preparations
12. For the CDS View Z_CDS_EPM_PD we create a new column for the target table. Click therefore in the Mapping Column on the ![Mapping Icon](img/Mapping_Icon.png) icon
    - In the opened window click on the Create Button to create a new column
    - Change the Column Name in the last field to the name **TimestampReplication**
    - Set the Data Type to timestamp from the drop down
    - Add for the Column Expression the following function: CURRENT_UTCTIMESTAMP
    ![RF Mapping Timestamp](img/RF_Mapping_Timestamp.png)
    - Click on the **OK** Button to save the mapping for the CDS View Z_CDS_EPM_PD

13. For the CDS View Z_CDS_EPM_PO we want to filter the column GrossAmount for all rows which are >= 800 €. Click to filter the CDS View on the ![RF Filter Icon](img/Filter_Icon.png) icon
    - Scroll on the opened window until the column GrossAmount
    - Click on the left side on the left side on the field GrossAmount
    - Change the = operator to an >= from the drop-down
    -  Enter in the Value field on the ríght side the value **800**
    -  Click on the **OK** Button to save the mapping for the CDS View Z_CDS_EPM_PO
    ![RF Filter GrossAmount](img/RF_Filter_GrossAmount.png)

14. For the CDS View Z_CDS_EPM_SO we combine the mapping and filtering together for our replication. For the sales orders we are interested filtering for all printers within our orders. We want to also add within our replicated dataset a new column storing the exact timestamp of our replication
    - For the filtering of all printers click on the filter icon and scroll down to the column ProductCategory
    - In total three different printers are present within the ProductCategory column: **Laser Printers**, **Ink Jet Printers** and **Multifunction Printers**
    - To filter for all items we click two times on the Add button to have three rows present for the three product categories
    - Enter for each row in the filtering option one of the Product Categories as displayed in below picture
    ![RF Filter ProductCategory](img/RF_Filter_ProductCategory.png)


15. Click on the Validate button ![RF Validate Button](img/RF_Validate_Button.png)  to check if every check runs through smoothly. An ideal validation is displayed in belows image

![RF Validation Check](img/RF_Validation_Check.png)

17. After everything went through in the validation check we can deploy our Replication Flow by clicking on the deploy button ![RF Deploy Button](img/RF_Deploy_Button.png)

18. After the deployment of the Replication Flow is completed, we can press the run button ![RF Run Button](img/RF_Run_Button.png) to start the replication with our specified settings

19. After the replication is finished, click on the Data Intelligence Modeler banner at the top of the screen

20. In the Drop-Down menu click on the Metadata Explorer row to browse the replicated data

![DropDown Metadata Explorer](img/DI_Pipeline_DropDown.png)

20. In the newly opened tab click on the Browse Connections option to navigate the HANA database we replicated our data to

![Metadata Explorer](img/Metadata_Explorer_BrowseConnections.png)

21. To navigate to the replicated datasets click through with the following steps
    - Click on HANA_CLOUD tile
    - Click on TECHED_DA281 tile
    - Click on the replicated datasets **Z_CDS_EPM_SO**, **Z_CDS_EPM_PO**, **Z_CDS_EPM_PD**
    - Click for each dataset on the Data Preview tab and explore for each dataset the data replicated

22.  After the replication flow has successfully run and you explored the data, execute the Custom ABAP operator created in [Exercise 2](../ex2/README.md#exercise-15---using-a-custom-abap-operator-to-verify-your-delta-replication-of-epm-sales-orders) to create new sales orders.