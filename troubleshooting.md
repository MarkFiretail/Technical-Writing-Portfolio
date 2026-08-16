# Troubleshooting Exercises

## Troubleshooting Exercise Set Up

-   All exercises will use the Clarity database TrainCube for the OLAP connection.
-   All exercises contain an error that must be corrected.
-   The exercises are located in **C:\\Clarity Systems\\ClarityServer\\Web\\Templates\\Training\\Troubleshooting.**
-   You are encouraged to work in pairs or in groups.

## Troubleshooting \#1: Unable to Create Page Option

### Description:

Template name is **\\Web\\Templates\\Troubleshooting\\Case 1**

The user is attempting to create a page option, but the drop-down list for the member list is empty.

![](media/Unable%20to%20create%20page%20option.png)

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#2: Column Titles Not Displaying

### Description:

Template name is **\\Web\\Templates\\Troubleshooting\\Case 2**

The user runs the template, but none of the title captions for the columns appears. The titles should read January..December.

![](media/Column%20titles%20not%20displaying.png)

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#3: Data not displaying correctly

### Description:

Template name is **\\Web\\Templates\\Troubleshooting\\Case 3**

![](media/Data%20not%20displaying%20correctly.png)

The user runs the template, but only one of the three rows expected appears. The rows should be **Income before Taxes, Income Taxes** and **Net Operating Revenue**.

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#4: Errant Data Appearing

### Description:

Template name is \\Web\\Templates\\Troubleshooting\\Case 4

![](media/Filter%20data%20mistakenly%20displays.png)

The user runs the template and instead of data in the January and February columns, there is filter information appearing.

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#5: Column Totals in the Incorrect Location

### Description:

Template name is \\Web\\Templates\\Troubleshooting\\Case 5

![](media/Column%20totals%20in%20incorrect%20location.png)

The user runs the template and a number of blank lines are inserted between the last row of the data and the total row. These lines should not be there.

1.  What caused the problem?
2.  How would you correct the error?

## Troubleshooting \#6: Template Not Saving

### Description:

Template name is \\Web\\Templates\\Troubleshooting\\Case 6

The user runs the template, inputs a value, saves, then retrieves the data. The value input is not being saved.

![](media/Template%20not%20saving.png)

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#7: Column Headings Not Displaying Correctly

### Description:

Template name is **\\Web\\Templates\\Troubleshooting\\Case 7**

The user runs the report and the headings for the time columns are not appearing in the correct place.

![](media/Column%20headings%20not%20displaying%20correctly.png)

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#8: Error – Named Range Does Not Exist

### Description:

Template name is **\\Web\\Templates\\Troubleshooting\\Case 8**

The user runs the template and receives the following error message:

![](media/Named%20range%20does%20not%20exist.png)

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting \#9: Captions Not Displaying

### Description:

Template name is **\\Web\\Templates\\Troubleshooting\\Case 9**

The user runs the report and the headings for the account rows are displaying names and not the captions.

![](media/Captions%20not%20displaying.png)

1.  What caused the problem?
2.  How would you correct the error?
3.  Correct the error.

## Troubleshooting Answers

| Case #     | Problem and Resolution                                    |
| ---------- | --------------------------------------------------------- |
| Case 1     | The report created forgot to create the **OLAP Member Lists** before creating the **Page Options**.  Once a **Member List** is added, the drop-down will no longer be empty. |
| Case 2     | **ColumnRangeMeta** is set to a single row – the hidden row.  To correct this, you must alter the **Column Range** from (**$11:$11)** to include the row above the title and the title row (**$11:$12**) |
| Case 3     | **RowRangeR1** is set to not insert and is set to a single row, hence only the first row will display.  To correct, set **RowRangeR1** to allow insertion. |
| Case 4     | On the **Data** – **Olap Data Map** – **Page Filters** window, the placement for the **Page Filters** is set to **RowRangeR1** and **ColumnRangeC1** instead of **RowRangeFilter** and **ColumnRangeFilter**, resulting in an errant placement of the filter data.  Correct by changing the location to **RowRangeFilter** and **ColumnRangeFilter**. |
| Case 5     | When defining the **Page Filters** for the row, the location of the **RowRangePageFilter**, which has **Insert Additional Rows** enabled is above the location of the total line.  Move the **Named Range** for **RowRangePageFilter** to be below the row containing the column totals. |
| Case 6     | The **ColumnRangeAction** Column range was created, but in the **Data** settings, the **Action column** was never set.  Set **Action** to **ColumnRangeAction**, save and the template will now save. |
| Case 7     | The **Meta** range was not set in the **OLAP Map** for the columns.  In the **Data** pane click on **OlapDataMapMain**.  Click the grid icon next to **Column Members** and set the dropdown to **RowRangeMeta**. |
| Case 8     | The named range for **RowRangeR1** was not generated in Excel.  Using Excel select **Insert > Name > Define** and add the named range **RowRanges.RowRangeR1.** In the **Refers to:** field select row 13 on the grid. The field will display **=Sheet1!$13:$13** |
| Case 9     | On the **Data Pane** the **Row Members** have been set to **Names**.  Uncheck **Include Names** and check **Include Captions.** |
