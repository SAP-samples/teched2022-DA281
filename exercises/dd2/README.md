# Deep Dive 2 - Creating a Custom ABAP Operator and making use of it in an SAP Data Intelligence Pipeline

In this section we will demonstrate how to create a custom ABAP operator in SAP S/4HANA and trigger its execution from a Pipeline in SAP Data Intelligence.<br><br>
The ABAP operator that we will be implementing provides a feature which is needed by some of the other Deep Dive sessions and Exercises of this workshop when it comes to extracting and replicating delta information from an ABAP source system. We will realize functionality for triggering the execution of an ABAP program in the source system that generates Sales Order data for the Enterprise Procurement Model (EPM).<br>
The operator can then be used in this workshop to 'remote control' the creation of EPM Sales Order records from SAP Data Intelligence pipelines without having the need to logon to the SAP GUI of S/4HANA. We can leverage the Custom ABAP operator to demonstrate and verify the delta provisioning capabilities of the CDS Reader and the Replication Flows.<br><br>

## Implementation of Custom ABAP Operators
The ABAP operators are developed in the S/4HANA system as regular development objects, hence, they can be transported within the system landscape.<br>

The basis for the integration with SAP Data Intelligence are the ABAP Pipeline Engine in SAP S/4HANA and the ABAP Subengine in SAP Data Intelligence, both linked with either an RFC or a Websocket RFC connection.

After a new ABAP Operator has been created, it can immediately be used in the SAP Data Intelligence Modeler by leveraging its dynamic ABAP Integration operator. These operators in SAP Data Intelligence are actually shells that point to the original implementation in the connected S/4HANA system.

There are two variants of this operator type available in SAP Data Intelligence:
1. SAP ABAP Operator: This can be used with any ABAP operator delivered by SAP (in namespace `com.sap`). Examples for out-of-the-box operators are (the already known) ABAP CDS Reader, ODP Reader, SLT Connector, Cluster Table Splitter (for Business Suite systems), and the ABAP Converter. 
2. Custom ABAP Operator: This can be used with any ABAP operator created by customers (in namespace `customer.<xyz>`).<br><br>
![](/exercises/dd2/images/dd2-001a.JPG)

In order to use the ABAP Subengine, the following prerequisites have to be met:
1. A supported ABAP system is available (see table below)
2. The ABAP system can be reached via RFC or WebSocket RFC
3. A user with the necessary authorizations (see SAP Note [2855052](https://launchpad.support.sap.com/#/notes/2855052))
4. An RFC or Websocket RFC connection has been created in the Connection Manager
5. Whitelisting of operators and data objects (tables, views, etc.) in the S/4HANA system (see SAP Note [2831756](https://launchpad.support.sap.com/#/notes/2831756)).<br>

The ABAP Pipeline Engine is supported starting from the following releases (you can also run this scenario with a SAP Business Suite system, but then it is required to install the (non-modifying) DMIS add-on on that system.)<br>
ABAP Edition | Minimum version | Recommended Version
------------ | --------------- | -------------------
S/4HANA on Premise | OP1909 + Note 2873666 |
S/4HANA Cloud | CE2002 | 
Netweaver >= 7.52 (ECC, BW, SRM, ...) | Add-On DMIS 2018 SP02 + Note 2845347 | Latest DMIS 2018 SP 
Netweaver >= 7.00 & < 7.52 (ECC, BW, SRM, ...) | Add-On DMIS 2011 SP17 + Note 2857333 or 2857334 | Latest DMIS 2011 SP
Netweaver >= 4.6C & < 7.00 via dedicated SLT Server | Add-On DMIS 2011/2018 considering DMIS [version dependency](https://help.sap.com/doc/7824ad784dd242928fade9b62cdb171c/3.0.04/en-US/Installation_Guide_2020.pdf) | Use latest DMIS SP
> **Note:**
> Please always also consult the most up-to-date Product Availability Matrix (PAM).

![](/exercises/dd2/images/dd2-001b.JPG)

<br>ABAP Operators are created in the ABAP System by implementing the BAdI: `BADI_DHAPE_ENGINE_OPERATOR`.<br>
The BAdI implementation consists of a class with **two methods** that must be redefined. It is recommended that the BAdI implementation extends the abstract class
`cl_dhape_graph_oper_abstract`.
- GET_INFO: Returns metadata about the operator
- NEW_PROCESS (with its local class `lcl_process`): Creates a new instance of the operator.

`lcl_process` uses a simple event-based model and can be implemented by redefining one or more of the following methods:
- ON_START: Called once, before the graph is started
- ON_RESUME: Called at least once, before the graph is started or resumed
- STEP: Called frequently (loop)
- ON_SUSPEND: Called at least once, after the graph is stopped or suspended
- ON_STOP: Called once, after the graph is stopped

## Deep Dive 2.1 - Create a custom ABAP Operator in SAP S/4HANA

Like the CDS Views, custom ABAP Operators could also be manually implemented in S/4HANA (Class Builder) or in the ABAP Development Tools (ADT) on Eclipse.<br>
However, in order to reduce manual activities to a minimum, there is a framework available that supports you in the creation of all artifacts in the S/4HANA ABAP backend that are required for your own ABAP operator. That framework consists of two reports that must be executed in sequence:
- `DHAPE_CREATE_OPERATOR_CLASS`: Generate an implementation class
- `DHAPE_CREATE_OPER_BADI_IMPL`: Create and configure a BAdI implementation

After running these two reports, the generated implementation class can be adapted as needed.<br>
Here is a step-by-step guideline for creating a custom ABAP Operator. In the specific use case below, the ABAP Operator in S/4HANA should receive a input table or CDS View name and send back the record count of the given object to the Pipeline ABAP Operator in Data Intelligence.

1. Logon to the SAP GUI of your conneted S/4HANA system and run transaction `DHAPE` (SAP Data Intelligence - Operator Workbench). Then click on the button ***Generate Class***.<br><br>
   ![](/exercises/dd2/images/dd2-002a.jpg)<br>

   **Alternatively**, you can also directly open the underlying ABAP report by running transaction SE38 (ABAP Editor) and entering DHAPE_CREATE_OPERATOR_CLASS. Then just ***Execute*** (![](/exercises/dd2/images/Execute.jpg) or ***F8***) this report.<br><br>
   ![](/exercises/dd2/images/dd2-002b.jpg)<br>

2. Enter the required parameters and ***Execute***.<br><br>
   ![](/exercises/dd2/images/dd2-003a.jpg)<br>

3. Now assign a package or choose 'Local Object', then ***Save*** (![](/exercises/dd2/images/Save.JPG)).<br><br>
   ![](/exercises/dd2/images/dd2-004a.jpg)<br>

4. You should now see the following screen. Close that windows by clicking ***Exit*** (or ***Shift+F3***).<br><br>
   ![](/exercises/dd2/images/dd2-005a.jpg)<br>

5. You are now back on the SAP Data Intelligence Operator Workbench (transaction `DHAPE`). Click on “Generate BAdI Implementation”<br><br>
   ![](/exercises/dd2/images/dd2-006a.jpg)<br>

   **Alternatively**, you can also directly open the underlying ABAP report by running transaction SE38 (ABAP Editor), entering `DHAPE_CREATE_OPER_BADI_IMPL`, and ***Execute*** (![](/exercises/dd2/images/Execute.jpg) or ***F8***) this report.<br><br>

6. Enter the required parameters and ***Execute*** (![](/exercises/dd2/images/Execute.jpeg)).<br><br>
   ![](/exercises/dd2/images/dd2-007a.jpg)<br>

7. Now assign a package and ***Save*** (![](/exercises/dd2/images/Save.jpg)), or click 'Local Object'.<br>
   The Enhancement Implementation tool then starts, as you can see in the status line. This may take a couple of seconds.<br><br>
   ![](/exercises/dd2/images/dd2-008a.jpg)<br>

8. On the next screen (Enhancement Implementation), double-click on ***Implementing Class*** on the left side, then double-click on the name of your Implementing Class, in this case `ZCL_DHAPE_OPER_GEN_EPM_SO`.<br><br>
   ![](/exercises/dd2/images/dd2-009b.jpg)<br>

9. This opens the Class Builder (`SE24`). Double click on the `GET_INFO` method. This method determines the 'look and feel' of the operator in a Data Intelligence pipeline, hence, the input and output ports, or which fields are displayed in the configuration panel of the operator. We will just need to assign the input and output ports of the ABAP Operator. Configuration parameters are not needed in our use case. To keep it simple, we will also not assign any documentation to the operator.<br><br>
   ![](/exercises/dd2/images/dd2-010b.jpg)<br>

10. In the method `GET_INFO`, open to the `Change`view ***(icon or Ctrl+F1)***<br>
    Since we don't need configuration parameters (properties), the related line in the NEW_PROCESS method can be commented out (using a double-quote character). The rest can be left as is.<br>
    The code should then look as follows:<br>
   
    ```abap
      METHOD IF_DHAPE_GRAPH_OPERATOR~GET_INFO.
        rs_info = VALUE #(
          name         = 'customer.teched.socreate'
          description  = 'Creation of EPM Sales Order'(DSC)
          extensible   = abap_false
          component    = 'com.sap.abap.base'
          documentid   = 'ZCL_DHAPE_OPER_GEN_EPM_SO'
          iconsrc      = '../base/SAP_ABAP_Logo.png'
          internal     = abap_false
          subengine    = 'v6'
          inports      = VALUE #( ( name = 'in'  type = 'string' ) )
          outports     = VALUE #( ( name = 'out' type = 'string' ) )
          "properties   = VALUE #( ( name = 'myparameter' type = 'string' ) )
        ).
      ENDMETHOD.
    ```
    Then click the ***Save*** button.<br><br>
    ![](/exercises/dd2/images/dd2-011b.jpg)<br><br>

11. Go back to the Class Builder and double-click on the `NEW_PROCESS` method in order to implement the wanted functionality for our ABAP Operator.<br><br>
    ![](/exercises/dd2/images/dd2-012b.jpg)<br>

12. On the next screen, double click on the local class `lcl_process`.<br><br>
    ![](/exercises/dd2/images/dd2-013b.jpg)<br>

13. As we are going to implement a new method `on_data`, we have to declare the method in the class definition.<br><br>
    Open the `Change`view ***(icon or Ctrl+F1)*** and add the following code snippet:<br>
    ```abap
      PRIVATE SECTION.
      METHODS: on_data.
    ```
    ![](/exercises/dd2/images/dd2-014b.jpg)<br><br>

14.  Overwrite the existing `step( )` method with the following code:<br>
     ```abap
     METHOD if_dhape_graph_process~step.
       rv_progress = abap_false.
       CHECK mv_alive = abap_true.

       IF mo_out->is_connected( ) = abap_false.
         IF mo_in->is_connected( ).
           mo_in->disconnect( ).
         ENDIF.
         rv_progress = abap_true.
         mv_alive = abap_false.
       ELSE.
         IF mo_in->has_data( ).
           CHECK mo_out->is_blocked( ) <> abap_true.
           rv_progress = abap_true.
           on_data( ).
         ELSEIF mo_in->is_closed( ).
           mo_out->close( ).
           rv_progress = abap_true.
           mv_alive = abap_false.
         ELSEIF mo_in->is_connected( ) = abap_false.
           mo_out->disconnect( ).
           rv_progress = abap_true.
           mv_alive = abap_false.
         ENDIF.
       ENDIF.
     ENDMETHOD.
     ```

If `has_data( )` returns true, i.e. if the ABAP Operator receives a signal from the corresponding Data Intelligence Pipeline operator, we call the `on_data( )` method, which contains the wanted functionality (receive the record count based on a given table or CDS View).<br>
Include the following lines after the `step( )` method:

```abap
  METHOD on_data.
    DATA lv_data TYPE string.
    mo_in->read_copy( IMPORTING ea_data = lv_data ).

    "Execute the report variant of the EPM Data Generation report
    "that creates one Sales Order Header and related Sales Order items
    SUBMIT SEPM_DG_EPM_STD_CHANNEL USING SELECTION-SET 'Z_EPM_GEN_SO_1' AND RETURN.
    lv_data = '--> 1 additional EPM Sales Order with up to 10 related Sales Order Items created.'.

    mo_out->write_copy( lv_data ).
  ENDMETHOD.
```

We can outcomment the parameter value retrieval (see line 30 in screenshot below)<br><br>
![](/exercises/dd2/images/dd2-014c.jpg)<br><br>

Now click the ***Save*** button.<br><br>
The complete code of the local class `lcl_process` should now look as follows:
<br>

```abap
CLASS lcl_process DEFINITION INHERITING FROM cl_dhape_graph_proc_abstract.

  PUBLIC SECTION.
    METHODS: if_dhape_graph_process~on_start  REDEFINITION.
    METHODS: if_dhape_graph_process~on_resume REDEFINITION.
    METHODS: if_dhape_graph_process~step      REDEFINITION.
  
  PRIVATE SECTION.
    METHODS: on_data.

  
    DATA:
      mo_util         TYPE REF TO cl_dhape_util_factory,
      mo_in           TYPE REF TO if_dhape_graph_channel_reader,
      mo_out          TYPE REF TO if_dhape_graph_channel_writer,
      mv_myparameter  TYPE string.

ENDCLASS.

CLASS lcl_process IMPLEMENTATION.

  METHOD if_dhape_graph_process~on_start.
    "This method is called when the graph is submitted.
    "Note that you can only check things here but you cannot initialize variables.
  ENDMETHOD.

  METHOD if_dhape_graph_process~on_resume.
    "This method is called before the graph is started.

    "Read parameters from the config here.
    "mv_myparameter = to_upper( if_dhape_graph_process~get_conf_value( '/Config/myparameter' ) ).

    "Do initialization here.
    mo_util     = cl_dhape_util_factory=>new( ).
    mo_in       = get_port( 'in' )->get_reader( ).
    mo_out      = get_port( 'out' )->get_writer( ).
  ENDMETHOD.

  METHOD if_dhape_graph_process~step.
    rv_progress = abap_false.
    CHECK mv_alive = abap_true.

    IF mo_out->is_connected( ) = abap_false.
      IF mo_in->is_connected( ).
        mo_in->disconnect( ).
      ENDIF.
      rv_progress = abap_true.
      mv_alive = abap_false.
    ELSE.
      IF mo_in->has_data( ).
        CHECK mo_out->is_blocked( ) <> abap_true.
        rv_progress = abap_true.
        on_data( ).
      ELSEIF mo_in->is_closed( ).
        mo_out->close( ).
        rv_progress = abap_true.
        mv_alive = abap_false.
      ELSEIF mo_in->is_connected( ) = abap_false.
        mo_out->disconnect( ).
        rv_progress = abap_true.
        mv_alive = abap_false.
      ENDIF.
    ENDIF.
  ENDMETHOD.

  METHOD on_data.
    DATA lv_data TYPE string.
    mo_in->read_copy( IMPORTING ea_data = lv_data ).

    "Execute the report variant of the EPM Data Generation report
    "that creates one Sales Order Header and related Sales Order items
    SUBMIT SEPM_DG_EPM_STD_CHANNEL USING SELECTION-SET 'Z_EPM_GEN_SO_1' AND RETURN.
    lv_data = '--> 1 additional EPM Sales Order with up to 10 related Sales Order Items created.'.

    mo_out->write_copy( lv_data ).
  ENDMETHOD.

ENDCLASS.
```

<br>

***Save*** the local class and activate (![](/exercises/dd2/images/Activate.jpg)) your ABAP Operator implementations.<br>
![](/exercises/dd2/images/dd2-014d.jpg)<br><br>

15. After you have clicked the Activation button (and if you haven't activated your new artifacts before), you are prompted for a selection of objects. Check both and confirm (![](images/Confirm_black.JPG)).<br><br>
    ![](/exercises/dd2/images/dd2-015b.jpg)<br><br>

The ABAP Operator implementation is now finished. The operator can immediately be used in SAP Data Intelligence Pipeline. The next section of this Deep Dive demo describes how this is done.<br><br>

## Deep Dive 2.2 - Integrate the custom ABAP Operator in a SAP Data Intelligence Pipeline

SAP Data Intelligence provides multiple Operator shells for the integration with ABAP Operators in SAP S/4HANA. On the one hand, there are the Operator shells that point to pre-defined ABAP Operators in ABAP systems, such as ABAP CDS Reader, ODP Reader, SLT Connector or the ABAP Converter. On the other hand, you can also trigger custom ABAP Operators by using the "Custom ABAP Operator" shell.<br>
Technically, the approaches for calling function modules in S/4HANA are the same in both cases. The only difference is the namespace under which these ABAP Operators are maintained and selected. While the pre-built Operators belong to the namespace `com.sap.`, custom ABAP Operators are assigned to `customer`.

The integration of ABAP Operators is done via Pipelines in the SAP Data Inteligence Modeler.

1.	Logon to SAP Data Intelligence to access the Launchpad application and click on the ***Modeler*** tile.<br><br>
![](/exercises/dd2/images/dd2-016b.jpg)<br><br>

2.	In the DI Modeler, make sure you are in the ***Graphs*** tab (see left side) and click the ***+*** symbol and select **Use Generation 1 Operators** in order to create a new Pipeline.<br><br>
![](/exercises/dd2/images/dd2-029b.jpg)<br><br>

3.	A new Pipeline canvas opens and the design focus automatically changes to the ***Operators*** tab (see left side). Drag the ***Custom ABAP Operator*** icon from the Operator list and drop it onto the canvas. Then do one click on the ***Custom ABAP Operator*** node in the canvas and open the configuration panel by clicking on the related symbol.<br><br>
![](/exercises/dd2/images/dd2-018b.jpg)<br><br>

4.	In the configuration panel on the right side, select the ***ABAP Connection*** (RFC or Websocket RFC connection) to the SAP S/4HANA system that provides the ABAP Operator. If done, click on the selection button of the field for the ***ABAP Operator***.<br><br>
![](/exercises/dd2/images/dd2-019b.jpg)<br><br>

5.	From the pop-up window, select the custom ABAP Operator that you want to call from the Pipeline. In our case, it's the ABAP Operator that triggers the execution of a specific ABAP Report variant on the S/4HANA system, and sends a confirmation back to the DI pipeline. Hence, choose ***Operator Class: Creation of EPM Sales Order*** and click ***OK***<br><br>
![](/exercises/dd2/images/dd2-020b.jpg)<br><br>

6.	As you can see, the ABAP Operator node in the Pipeline canvas gets automatically updated with the operator's name in S/4HANA and the ports that we have defined in the previous section of this Deep Dive demo. (The `GET_INFO( )`method in our operator's ABAP class provides the corresponding meta information.)<br><br>
![](/exercises/dd2/images/dd2-021b.jpg)<br><br>

7.	For verifying the functionality of the ABAP Operator call, we'll be using a ***Terminal*** Operator in the Pipeline. This operator allows the sending of user inputs and the reception of the results. Drag the ***Terminal*** icon from the Operator list and drop it onto the Pipeline canvas. Then connect<br><br>
- the output port of the ABAP Operator with the input port of the Terminal Operator and
- the output port of the Terminal Operator with the input port of the ABAP Operator.
Then ***Save*** the Pipeline.<br><br>
![](/exercises/dd2/images/dd2-022b.jpg)<br><br>

8.	For saving the Pipeline, you are prompted for the name of the pipeline (including namespace information), a description, and the category under which the Pipeline can be found in the ***Graphs*** tab of the Modeler. Fill in the needed and click ***OK***.<br><br>
![](/exercises/dd2/images/dd2-023b.jpg)<br><br>

9.	The Pipeline now gets validated by SAP Data Intelligence. You can see the results in the ***Validation*** tab of the status section in the Modeler UI. You can ignore the warning in the validation message for now. It just says that the default settings for a pipeline's resource allocations (memory, CPU) is used if you don't specify dedicated settings. If otherwise okay, you can now start the Pipeline by clicking on the ***Play*** symbol in the menue bar.<br><br>
![](/exercises/dd2/images/dd2-025b.jpg)<br><br>

10.	Change back to the ***Status*** tab of the status section in the Modeler UI. Once the status has turned to ***running***, click one time on the ***Terminal*** node and open the Terminal UI with a click on the corresponding icon.<br><br>
![](/exercises/dd2/images/dd2-026b.jpg)<br><br>

11.	In the lower section of the Terminal UI (the Input Prompt), you can now enter any input or just press ***Return*** in order to send a trigger signal to the ABAP Operator in S/4HANA. As a response for each input from our custom ABAP Operator, you will receive a confirmation message in the upper section of the Terminal UI, what proves our ABAP operator code functionality and the integration success.<br><br>
![](/exercises/dd2/images/dd2-027b.jpg)<br><br>
	
12.	Don't forget to stop the Pipeline again, using either the ***stop*** icon in the menue bar or the one in the status area.<br><br>
![](/exercises/dd2/images/dd2-028b.jpg)<br><br>

## Summary

We've now successfully implemented a custom ABAP Operator in S/4HANA and consumed it from a Data Intelligence Pipeline.<br><br>

The next part of the DA281 TechEd workshop is [Deep Dive 3- Technical Background for Replication Flows in SAP Data Intelligence](../../exercises/dd3/README.md)

<br><br>


*****************************************************
<br> **Table of Contents / Navigation**
<br>
- [Overview and Getting Started](../../exercises/ex0/README.md#overview-and-getting-started)
  - [Deep Dive demos vs. Exercises](../../exercises/ex0/README.md#deep-dive-vs-exercise-sections-in-this-document)
  - [Short introduction to the Enterprise Procurement Model (EPM) in ABAP systems](../../exercises/ex0/README.md#short-introduction-to-the-enterprise-procurement-model-epm-in-sap-s4hana)
  - Access to the exercises' Data Intelligence environment (**will be provided after the session**)
- [Deep Dive 1 - ABAP CDS View based data extraction in SAP Data Intelligence](../../exercises/dd1/README.md#deep-dive-1---abap-cds-view-based-data-extraction-in-sap-data-intelligence)
- [Deep Dive 2 - Creating a Custom ABAP Operator and making use of it in an SAP Data Intelligence Pipeline](README.md)
- [Deep Dive 3- Technical Background for Replication Flows in SAP Data Intelligence]../../(exercises/dd3/README.md)

- [Overview and Getting Started](exercises/ex0/README.md#overview-and-getting-started)

- [Exercise 1 - Replicating data from S/4HANA ABAP CDS Views in SAP Data Intelligence](../../exercises/ex1/)
    - [Exercise 1.1 - Consume the EPM Business Partner ABAP CDS Views in SAP Data Intelligence](../../exercises/ex1#exercise-11-sub-exercise-1-description)
    - [Exercise 1.2 - Extend the Pipeline to transfer the Customer data into a HANA Cloud Database with Initial Load mode](../../exercises/ex1#exercise-12-sub-exercise-2-description)
    - [Exercise 1.3 - Implement a Pipeline for delta transfer of enhanced EPM Sales Order data from S/4HANA to an S3 Object Store](../../exercises/ex1#exercise-13-sub-exercise-1-description)
    - [Exercise 1.4 - Extend the Pipeline for joining Sales Order with Customer data for each change in Sales Orders and persist results in S3](../../exercises/ex1#exercise-14-sub-exercise-1-description)
    - [Exercise 1.5 - Using a custom ABAP Operator to verify your Delta Replication of EPM Sales Orders](../../exercises/ex2/)

- [Exercise 2 - Integrate ABAP CDS Views in SAP Data Intelligence Replication Management Flow](../../exercises/ex3/README.md#exercise-3---integrate-abap-cds-views-in-sap-data-intelligence-replication-management-flow)
- <br><br>
