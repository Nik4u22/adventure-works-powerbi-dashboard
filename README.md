# adventure-works-powerbi-dashboard

<img width="1599" alt="image" src="https://github.com/Nik4u22/adventure-works-powerbi-dashboard/assets/64134540/63324c83-5f19-45e7-a67f-dfec6fc4819f">

<img width="1608" alt="image" src="https://github.com/Nik4u22/adventure-works-powerbi-dashboard/assets/64134540/bbc68e03-9922-4dd3-ab3c-710e3edd181e">

To create a Power BI report using the AdventureWorks database, you will need to follow these general steps:

Connect to the AdventureWorks Database: Open Power BI Desktop and click on "Get Data" from the Home tab. Select your preferred method to connect to the database, such as "SQL Server" or "Azure SQL Database." Provide the necessary connection details, such as server name, database name, and authentication method.

Import Data: Once connected, select the tables or views you want to import into Power BI. You can choose multiple tables that are relevant to your report. Click "Load" to import the data into Power BI.

Design the Report: After importing the data, you will see the "Fields" pane on the right side of the Power BI Desktop. You can drag and drop fields from the imported tables onto the report canvas to create visualizations.

Create Visualizations: Use the visualizations pane on the right side to select and configure different types of visuals, such as charts, tables, or maps. Drag the desired fields into the visualizations to populate them with data. You can also add filters, slicers, and other interactive elements to enhance the report's functionality.

Apply Formatting and Layout: Customize the look and feel of your report by formatting the visuals, adjusting colors, fonts, and sizes. Arrange the visuals on the report canvas to create a visually appealing layout.

Add Calculated Measures: Use DAX (Data Analysis Expressions) to create calculated measures or columns based on the imported data. Calculated measures can perform calculations or aggregations that are not available directly in the source data.

Create Relationships: If your report requires data from multiple tables, establish relationships between them in the "Manage Relationships" window. This ensures that data can be properly combined and analyzed across different tables.

Save and Publish the Report: Save your Power BI report locally and then publish it to the Power BI service if you want to share it with others. Publishing allows you to collaborate, schedule data refreshes, and access the report from various devices.

These steps provide a general overview of creating a Power BI report using the AdventureWorks database. The specific design and layout of your report will depend on your analysis goals and the insights you want to present.

## Data Modeling

<img width="452" alt="image" src="https://github.com/Nik4u22/adventure-works-powerbi-dashboard/assets/64134540/d3729c91-b550-43d8-a41a-d61e8e0a1618">


## DATE TABLE & MEASURES USING DAX

DATE TABLE - NEW COLUMNS
__________________________________________________________________________________________________________________________

Date = CALENDAR(MIN(Fact_sales_tb[order_date]), MAX(Fact_sales_tb[order_date]))

or

Date = CALENDARAUTO(3) // 3 - FiscalYearEndMonth (Calendar will start from 1st April-31st March)

or

Date = CALENDAR(MIN(Fact_sales_tb[order_date]), MAX(Fact_sales_tb[order_date]))
/*
VAR 
    start_Date = MIN(Fact_sales_tb[order_date])
VAR
    end_Date =  MAX(Fact_sales_tb[order_date])
VAR
    date_Table = CALENDAR(start_Date, end_Date)
RETURN
    ADDCOLUMNS(date_Table, 
    "Month", MONTH([Date]),
    "Year", YEAR([Date]),
    "Quarter", QUARTER([Date]),
    "Week_No", WEEKNUM([Date], 1), //1- Sunday, 2-Monday (Week starts from Sunday or Monday)
    "Month_Name", FORMAT([Date], "MMM"),
    "Month_Year", FORMAT([Date], "MMM-yyyy"),
    )
*/ 

Start_of_Year = STARTOFYEAR('Date'[Date], "31/03")

Fiscal Year = YEAR('Date'[Start_of_Year])

Financial_Year(FY) = "FY " & RIGHT('Date'[Fiscal Year], 2) &"_"& RIGHT('Date'[Fiscal Year]+1, 2)

Day = DAY('Date'[Date])

Day_Name = FORMAT(DAY('Date'[Date]), "DDDD")

Today_Date = TODAY()

Today_Day = FORMAT([Today_Date], "DDDD")

Week_Day = WEEKDAY(TODAY(),2) // 2-Week start from Monday

Week_No = WEEKNUM(TODAY())

Month = MONTH('Date'[Date])

Month_Name = FORMAT([Date], "MMM")

Month_Year = FORMAT('Date'[Date], "MMM-yy")

Month_Year_Sort = FORMAT('Date'[Date], "YYYYMM")

Quarter = QUOTIENT(DATEDIFF('Date'[Start_of_Year], 'Date'[Date], MONTH), 3)+1 // Takes APR-MAY-JUNE -> Q1

Quarter_Name = "Q"&'Date'[Quarter]

Quarter_year = 'Date'[Quarter]&"-"&'Date'[Year]

Year = YEAR('Date'[Date])


CALCULATED MEASURES 
__________________________________________________________________________________________________________________________

Total_Sales = SUM(Fact_sales_tb[sales_amount])

Today_Sales = CALCULATE([Total_Sales], FILTER('Date', 'Date'[Date] = TODAY()))
//CALCULATE([Total_Sales], LASTDATE('Date'[Date]))

Yesterday_Sales = CALCULATE([Total_Sales], FILTER('Date', 'Date'[Date] = TODAY()-1))
//CALCULATE([Total_Sales], LASTDATE('Date'[Date]))

Avg_Daily_Sales = AVERAGEX(VALUES('Date'[Date]), [Total_Sales])

Week_To_Date_Sales = 
VAR 
    _min = today() -WEEKDAY(today() ,2) +1 //Monday week start
VAR
    _max = _min +6
RETURN
    CALCULATE([Total_Sales], FILTER('Date','Date'[Date] >=_min && 'Date'[Date] <= _max))

MTD Sales = TOTALMTD([Total_Sales], 'Date'[Date])

PM MTD Sales = 
VAR 
    PM_Sales = CALCULATE([Total_Sales], PREVIOUSMONTH('Date'[Date]))
RETURN
    IF(ISBLANK(PM_Sales), 0, PM_Sales)

PY MTD Sales = TOTALMTD([Total_Sales], SAMEPERIODLASTYEAR('Date'[Date]))

Avg_Daily_Sales_MTD = CALCULATE(Average(Fact_sales_tb[sales_amount]),DATESMTD('Date'[Date]))

QTD Sales = TOTALQTD([Total_Sales], 'Date'[Date])

PY QTD Sales = TOTALQTD([Total_Sales], SAMEPERIODLASTYEAR('Date'[Date]))

Avg_Daily_Sales_QTD = CALCULATE(Average(Fact_sales_tb[sales_amount]),DATESQTD('Date'[Date]))

YTD Sales = TOTALYTD([Total_Sales], 'Date'[Date], "31/03")

PY YTD Sales = TOTALYTD([Total_Sales], SAMEPERIODLASTYEAR('Date'[Date]), "31/03")

Avg_Daily_Sales_YTD = CALCULATE(Average(Fact_sales_tb[sales_amount]),DATESYTD('Date'[Date], "31/03"))

Total_Cost = CALCULATE(SUM(Fact_sales_tb[total_product_cost]))

Profit = [Total_Sales] - [Total_Cost]

Profit % = [Profit]/[Total_Sales]

MTD Profit = TOTALMTD([Profit], 'Date'[Date])

MTD Profit % = [MTD Profit]/[MTD Sales]

PY MTD Profit = TOTALMTD([Profit], SAMEPERIODLASTYEAR('Date'[Date]))

PY MTD Profit % = [PY MTD Profit]/[PY MTD Sales]

QTD Profit = TOTALQTD([Profit], 'Date'[Date])

QTD Profit % = [QTD Profit]/[QTD Sales]

PY QTD Profit = TOTALQTD([Profit], SAMEPERIODLASTYEAR('Date'[Date]))

PY QTD Profit % = [PY QTD Profit]/[PY QTD Sales]

YTD Profit = TOTALYTD([Profit], 'Date'[Date], "31/03")

YTD Profit % = [YTD Profit]/[YTD Sales]

PY YTD Profit = TOTALYTD([Profit], SAMEPERIODLASTYEAR('Date'[Date]), "31/03")

PY YTD Profit % = [PY YTD Profit]/[PY YTD Sales]

CAGR = (  
    VAR 
        Beginning_Value = CALCULATE(SUM(Fact_sales_tb[sales_amount]),FILTER('Fact_sales_tb', YEAR('Fact_sales_tb'[order_date]) = MIN('Date'[Fiscal_Year])))
    VAR 
        Ending_Value = CALCULATE(SUM(Fact_sales_tb[sales_amount]),FILTER('Fact_sales_tb', YEAR('Fact_sales_tb'[order_date]) = MAX('Date'[Fiscal_Year])))
    VAR 
        No_Of_Years = (MAX('Date'[Fiscal_Year])-MIN('Date'[Fiscal_Year]))

    RETURN 
        CALCULATE((Ending_Value/Beginning_Value)^(1/No_Of_Years)-1)    
    ) 


Avg_Monthly_Sales = AVERAGEX(VALUES('Date'[Month_Year]), [Total_Sales])

Avg_Yearly_Sales = AVERAGEX(VALUES('Date'[Financial_Year(FY)]), [Total_Sales])

Total_Orders = COUNTROWS(Fact_sales_tb)

Today_Orders = CALCULATE([Total_Orders], FILTER('Date', 'Date'[Date]=TODAY()))
//CALCULATE(COUNTROWS(Fact_sales_tb), LASTDATE('Date'[Date])))

Week_To_Date_Orders = 
VAR 
    _min = today() -WEEKDAY(today() ,2) +1 //Monday week start
VAR
    _max = _min +6
RETURN
    CALCULATE([Total_Orders], FILTER('Date','Date'[Date] >=_min && 'Date'[Date] <= _max))

Yesterday_Orders = CALCULATE(COUNTROWS(Fact_sales_tb), FILTER('Date', 'Date'[Date] = TODAY()-1))
//CALCULATE([Total_Sales], LASTDATE('Date'[Date]))

MTD Orders = TOTALMTD(COUNTROWS(Fact_sales_tb), 'Date'[Date])

PM MTD Orders = 
VAR 
    PM_Orders = CALCULATE([Total_Orders], PREVIOUSMONTH('Date'[Date]))
RETURN
    IF(ISBLANK(PM_Orders), 0, PM_Orders)

