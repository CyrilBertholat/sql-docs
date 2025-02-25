---
title: "Lesson 8: Create a data filter"
description: Learn to create a data filter for the data table that you defined for the child report. Create a table-based filter or a query filter for the drillthrough report.
author: maggiesMSFT
ms.author: maggies
ms.date: 09/25/2024
ms.service: reporting-services
ms.subservice: reporting-services
ms.topic: conceptual
ms.custom:
  - updatefrequency5
---
# Lesson 8: Create a data filter
After you add a drillthrough action on the parent report, your next step is to create a data filter for the data table that you defined for the child report.  
  
You can create a table-based filter **or** a query filter for the drillthrough report. This lesson provides instructions for both options.  
  
## Table-based filter  
You need to complete the following tasks to implement a table-based filter.  
  
-   Add a filter expression to the tablix in the child report.  
  
-   Create a function that selects unfiltered data from the **PurchaseOrderDetail** table.  
  
-   Add an event handler that binds the **PurchaseOrderDetail** DataTable to the child report.  
  
### Add a filter expression to the tablix in the child report  
  
1.  Open the child report.  
  
2.  Select a column heading in the tablix, right-click the gray cell that appears above the column heading, and then choose **Tablix Properties**.  
  
3.  Select the **Filters** page, and then choose **Add**.  
  
4.  In the **Expression** filed, select **ProductID** from the drop-down list. This column is where you apply the filter.  
  
5.  Select the equal (**=**) operator in the **Operator** drop-down list.  
  
6.  Select the expression button next to the **Value** field, choose **Parameters** in the **Category** area, and then double-click **productid** in the **Values** area. The **Set expression for: Value** field should now contain expression similar to **=Parameters!productid.Value**.  
  
7.  Select **OK,** and **OK** again in the **Tablix Properties** dialog box.  
  
8.  Save the .rdlc file.  
  
### Create a function that selects unfiltered data from the PurchaseOrderDetail table  
  
1.  In Solution Explorer, expand Default.aspx, and then double-click Default.aspx.cs.  
  
2.  Create a new function that accepts a parameter called **productid** of type Integer. It should return a **datatable** object and do the following:  
  
    1.  Create an instance of the dataset, **DataSet2**, which was created in Step 2 of [Lesson 4: Define a data connection and data table for the child report](../reporting-services/lesson-4-define-a-data-connection-and-data-table-for-child-report.md).  
  
    2.  Create a connection to the SqlServer database to execute the query defined in **Lesson 4: Define a data connection and data table for the child report**.  
  
    3.  The query returns unfiltered data.  
  
    4.  Fill the DataSet instance with the unfiltered data by executing the query.  
  
    5.  Return the **PurchaseOrderDetail** DataTable.  
  
        The function should look similar to the following example, (This example is just for your reference. You can follow any pattern that you want, to fetch the necessary data for child report).  
  
        ```  
        /// <summary>  
            /// Function to query PurchaseOrderDetail table, fetch the  
            /// unfiltered data and bind it with the Child report  
            /// </summary>  
            /// <returns>A dataTable of type PurchaseOrderDetail</returns>  
            private DataTable GetPurchaseOrderDetail()  
            {  
                try  
                {  
                    //Create the instance for the typed dataset, DataSet2 which will   
                    //hold the [PurchaseOrderDetail] table details.  
                    //The dataset was created as part of the tutorial in Step 4.  
                    DataSet2 ds = new DataSet2();  
  
                    //Create a SQL Connection to the AdventureWorks2008 database using Windows Authentication.  
                    using (SqlConnection sqlconn = new SqlConnection("Data Source=.;Initial Catalog=AdventureWorks2022;Integrated Security=SSPI"))  
                    {  
                        //Building the dynamic query with the parameter ProductID.  
                        SqlDataAdapter adap = new SqlDataAdapter("SELECT PurchaseOrderID, PurchaseOrderDetailID, OrderQty, ProductID, ReceivedQty, RejectedQty, StockedQty FROM Purchasing.PurchaseOrderDetail ", sqlconn);  
                        //Executing the QUERY and fill the dataset with the PurchaseOrderDetail table data.  
                        adap.Fill(ds, "PurchaseOrderDetail");  
                    }  
  
                    //Return the PurchaseOrderDetail table for the Report Data Source.  
                    return ds.PurchaseOrderDetail;  
                }  
                catch  
                {  
                    throw;  
                }  
            }  
        ```  
  
### Add an event handler that binds the PurchaseOrderDetail DataTable to the child report  
  
1.  Open Default.aspx in designer view.  
  
2.  Right-click the ReportViewer control, and then select **Properties**.  
  
3.  On the **Properties** page, select the **Events** icon.  
  
4.  Double-click the **Drillthrough** event.  
  
    This action adds an event handler section in the code, which should look similar to the following block.  
  
    ```  
    protected void ReportViewer1_Drillthrough(object sender, Microsoft.Reporting.WebForms.DrillthroughEventArgs e)  
    {  
    }  
    ```  
  
5.  Complete the event handler. It should include the following functionality.  
  
    1.  Fetch the child report object reference from the *DrillthroughEventArgs* parameter.  
  
    2.  Call the function, **GetPurchaseOrderDetail**  
  
    3.  Bind the **PurchaseOrderDetail** DataTable with the report's corresponding data source.  
  
        The completed event handler code should look similar to the following example.  
  
        ```  
        protected void ReportViewer1_Drillthrough(object sender, Microsoft.Reporting.WebForms.DrillthroughEventArgs e)  
            {  
                try  
                {  
                     //Get the instance of the Target report.  
                    LocalReport report = (LocalReport)e.Report;  
  
                     //Binding the DataTable to the Child report dataset.  
                    //The name DataSet1 which can be located from,   
                    //Go to Design view of Child.rdlc, Click View menu -> Report Data  
                    //You'll see this name under DataSet2.  
                    report.DataSources.Add(new ReportDataSource("DataSet1", GetPurchaseOrderDetail()));  
                }  
                catch (Exception ex)  
                {  
                    Response.Write(ex.Message);  
                }  
            }  
        ```  
  
6.  Save the file.  
  
## Query filter  
You need to complete the following tasks to implement a query filter.  
  
-   Create a function that selected filtered data from the **PurchaseOrderDetail** table.  
  
-   Add an event handler that retrieves parameter values and binds the **PurchaseOrderDetail** DataTable to the child report.  
  
### Create a function that selects filtered data from the PurchaseOrderDetail table  
  
1.  In Solution Explorer, expand Default.aspx, and then double-click Default.aspx.cs.  
  
2.  Create a new function that accepts a parameter, **productid**, of type Integer and returns a **datatable** object and does the following.  
  
    1.  Creates an instance of the dataset, **DataSet2**, which was created in Step 2 of [Lesson 4: Define a data connection and data table for child report](../reporting-services/lesson-4-define-a-data-connection-and-data-table-for-child-report.md).  
  
    2.  Create a connection to the SqlServer database to execute the query defined in **Lesson 4: Define a data connection and data table for the child report**.  
  
    3.  The query includes a parameter, **productid**, to make sure the data returned is filtered based on the **ProductID** selected in the parent report.  
  
    4.  Fill the DataSet instance with the filtered data by executing the query.  
  
    5.  Return the **PurchaseOrderDetail** DataTable.  
  
        The function should look similar to the following example, (This example is just for your reference. You can follow any pattern that you want, to fetch the necessary data for child report).  
  
        ```  
        /// <summary>  
            /// Function to query PurchaseOrderDetail table and filter the  
            /// data for a specific ProductID selected in the Parent report.  
            /// </summary>  
            /// <param name="productid">Parameter passed from the Parent report to filter data.</param>  
            /// <returns>A dataTable of type PurchaseOrderDetail</returns>  
            private DataTable GetPurchaseOrderDetail(int productid)  
            {  
                try  
                {  
                    //Create the instance for the typed dataset, DataSet2 which will   
                    //hold the [PurchaseOrderDetail] table details.  
                    //The dataset was created as part of the tutorial in Step 4.  
                    DataSet2 ds = new DataSet2();  
  
                    //Create a SQL Connection to the AdventureWorks2008 database using Windows Authentication.  
                    using (SqlConnection sqlconn = new SqlConnection("Data Source=.;Initial Catalog=AdventureWorks2022;Integrated Security=SSPI"))  
                    {  
                        //Building the dynamic query with the parameter ProductID.  
                        SqlCommand cmd = new SqlCommand("SELECT PurchaseOrderID, PurchaseOrderDetailID, OrderQty, ProductID, ReceivedQty, RejectedQty, StockedQty FROM Purchasing.PurchaseOrderDetail where ProductID = @ProductID", sqlconn);  
                  
                        // Sets the productid parameter.  
                        cmd.Parameters.Add((new SqlParameter("@ProductID", SqlDbType.Int)).Value = productid);  
  
                        SqlDataAdapter adap = new SqlDataAdapter(cmd);  
                        //Executing the QUERY and fill the dataset with the PurchaseOrderDetail table data.  
                        adap.Fill(ds, "PurchaseOrderDetail");  
                    }  
  
                    //Return the PurchaseOrderDetail table for the Report Data Source.  
                    return ds.PurchaseOrderDetail;  
                }  
                catch  
                {  
                    throw;  
                }  
            }  
        ```  
  
### Add an event handler that retrieves parameter values and binds the PurchaseOrderDetail DataTable to the child report  
  
1.  Open Default.aspx in designer view.  
  
2.  Right-click the ReportViewer control, and then select **Properties**.  
  
3.  On the **Properties** pane, select the **Events** icon.  
  
4.  Double-click the **Drillthrough** event.  
  
    This action adds an event handler section in the code that should look similar to the following example.  
  
    ```  
    protected void ReportViewer1_Drillthrough(object sender, Microsoft.Reporting.WebForms.DrillthroughEventArgs e)  
    {  
    }  
    ```  
  
5.  Complete the event handler. It should include the following functionality.  
  
    1.  Fetch the Child report object reference from the *DrillthroughEventArgs* parameter.  
  
    2.  Get the child report parameter list from the child report object fetched.  
  
    3.  Iterate through the parameter's collection and retrieve the value for the parameter, **ProductID**, passed from the parent report.  
  
    4.  Call the function, **GetPurchaseOrderDetail**, and pass the value for parameter **ProductID**.  
  
    5.  Bind the **PurchaseOrderDetail** DataTable with the Report's corresponding data source.  
  
        The completed event handler code should look similar to the following example.  
  
        ```  
        protected void ReportViewer1_Drillthrough(object sender, Microsoft.Reporting.WebForms.DrillthroughEventArgs e)  
            {  
                try  
                {  
                    //Variable to store the parameter value passed from the MainReport.  
                    int productid = 0;  
  
                    //Get the instance of the Target report.  
                    LocalReport report = (LocalReport)e.Report;  
  
                    //Get all the parameters passed from the main report to the target report.  
                    //OriginalParametersToDrillthrough actually returns a Generic list of   
                    //type ReportParameter.  
                    IList<ReportParameter> list = report.OriginalParametersToDrillthrough;  
  
                    //Parse through each parameters to fetch the values passed along with them.  
                    foreach (ReportParameter param in list)  
                    {  
                        //Since we know the report has only one parameter and it is not a multivalued,   
                        //we can directly fetch the first value from the Values array.  
                        productid = Convert.ToInt32(param.Values[0].ToString());  
                    }  
  
                    //Binding the DataTable to the Child report dataset.  
                    //The name DataSet1 which can be located from,   
                    //Go to Design view of Child.rdlc, Click View menu -> Report Data  
                    //You'll see this name under DataSet2.  
                    report.DataSources.Add(new ReportDataSource("DataSet1", GetPurchaseOrderDetail(productid)));  
                }  
                catch (Exception ex)  
                {  
                    Response.Write(ex.Message);  
                }  
            }  
        ```  
  
6.  Save the file.  
  
## Next step

> [!div class="nextstepaction"]
> [Lesson 9: Build and run the application](../reporting-services/lesson-9-build-and-run-the-application.md)
