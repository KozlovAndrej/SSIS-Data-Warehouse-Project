# SSIS-Data-Warehouse-Project
Design and implement a data warehouse using SSIS to extract, transform, and load order data from Excel files.
Here's the updated README file with the SQL schema included:

---

# Data Warehousing Project with SSIS

## Project Overview

The objective of this exercise is to assess the ability to design and implement a simple data warehousing structure and populate it with data using SQL Server Integration Services (SSIS). This project involves extracting data from an Excel file, transforming it as necessary, and loading it into a new data warehouse (DWH) schema. The final data will be used to create a report with visualizations and key performance indicators (KPIs).

## Data Warehouse Schema

The following schema was designed to organize and store the data extracted from the provided Excel file:

### Dimension Tables

```sql
CREATE TABLE DimProducts (
    ProductID INT PRIMARY KEY,
    ProductName NVARCHAR(255),
    SupplierID INT,
    CategoryID INT,
    QuantityPerUnit NVARCHAR(50),
    UnitCost DECIMAL(18, 2),
    UnitPrice DECIMAL(18, 2),
    UnitsInStock INT,
    UnitsOnOrder INT
);

CREATE TABLE DimCustomers (
    CustomerID INT PRIMARY KEY,
    CompanyName NVARCHAR(255),
    ContactName NVARCHAR(255),
    City NVARCHAR(255),
    Country NVARCHAR(255),
    DivisionID INT,
    "Address" NVARCHAR(255),
    Fax NVARCHAR(50),
    Phone NVARCHAR(50),
    PostalCode NVARCHAR(50),
    StateProvince NVARCHAR(50)
);

CREATE TABLE DimCategories (
    CategoryID INT PRIMARY KEY,
    CategoryName NVARCHAR(255),
    "Description" NVARCHAR(255)
);

CREATE TABLE DimShippers (
    ShipperID INT PRIMARY KEY,
    CompanyName NVARCHAR(255)
);

CREATE TABLE DimDivisions (
    DivisionID INT PRIMARY KEY,
    DivisionName NVARCHAR(255)
);
```

### Fact Tables

```sql
CREATE TABLE StagingOrders (
    OrderID INT,
    OrderDate DATE,
    CustomerID INT,
    EmployeeID INT,
    ShipperID INT,
    Freight DECIMAL(18, 2)
);

CREATE TABLE FactOrders (
    OrderID INT PRIMARY KEY,
    OrderDate DATE,
    CustomerID INT,
    EmployeeID INT,
    ShipperID INT,
    Freight DECIMAL(18, 2),
    FOREIGN KEY (CustomerID) REFERENCES DimCustomers(CustomerID),
    FOREIGN KEY (ShipperID) REFERENCES DimShippers(ShipperID)
);

CREATE TABLE FactOrderDetails (
    OrderID INT,
    "LineNo" INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(18, 2),
    Discount DECIMAL(18, 2),
    IsStockAvailable BIT,
    PRIMARY KEY (OrderID, "LineNo"),
    FOREIGN KEY (OrderID) REFERENCES FactOrders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES DimProducts(ProductID)
);

CREATE TABLE FactShipments (
    OrderID INT,
    "LineNo" INT,
    ShipperID INT,
    CustomerID INT,
    ProductID INT,
    EmployeeID INT,
    ShipmentDate DATE,
    PRIMARY KEY (OrderID, "LineNo"),
    FOREIGN KEY (ShipperID) REFERENCES DimShippers(ShipperID),
    FOREIGN KEY (CustomerID) REFERENCES DimCustomers(CustomerID),
    FOREIGN KEY (ProductID) REFERENCES DimProducts(ProductID)
);
```

### Aggregation Table

```sql
CREATE TABLE AggShipments (
    ShipmentDate DATE,
    OrderCount INT,
    TotalPrice DECIMAL(18, 2),
    PRIMARY KEY (ShipmentDate)
);
```

### Staging Tables

```sql
CREATE TABLE ETLControlTable (
    TableName NVARCHAR(255) PRIMARY KEY,
    LastLoadDate DATETIME
);

CREATE TABLE Staging_DimProducts (
    ProductID INT,
    ProductName NVARCHAR(255),
    SupplierID INT,
    CategoryID INT,
    QuantityPerUnit NVARCHAR(50),
    UnitCost DECIMAL(18, 2),
    UnitPrice DECIMAL(18, 2),
    UnitsInStock INT,
    UnitsOnOrder INT,
    LoadDate DATETIME
);

CREATE TABLE Staging_DimCustomers (
    CustomerID INT,
    CompanyName NVARCHAR(255),
    ContactName NVARCHAR(255),
    City NVARCHAR(255),
    Country NVARCHAR(255),
    DivisionID INT,
    Address NVARCHAR(255),
    Fax NVARCHAR(50),
    Phone NVARCHAR(50),
    PostalCode NVARCHAR(50),
    StateProvince NVARCHAR(50),
    LoadDate DATETIME
);

CREATE TABLE Staging_DimCategories (
    CategoryID INT,
    CategoryName NVARCHAR(255),
    Description NVARCHAR(255),
    LoadDate DATETIME
);

CREATE TABLE Staging_DimShippers (
    ShipperID INT,
    CompanyName NVARCHAR(255),
    LoadDate DATETIME
);

CREATE TABLE Staging_DimDivisions (
    DivisionID INT,
    DivisionName NVARCHAR(255),
    LoadDate DATETIME
);

CREATE TABLE Staging_FactOrders (
    OrderID INT,
    OrderDate DATE,
    CustomerID INT,
    EmployeeID INT,
    ShipperID INT,
    Freight DECIMAL(18, 2),
    LoadDate DATETIME
);

CREATE TABLE Staging_FactOrderDetails (
    OrderID INT,
    "LineNo" INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(18, 2),
    Discount DECIMAL(18, 2),
    IsStockAvailable BIT,
    LoadDate DATETIME
);

CREATE TABLE Staging_FactShipments (
    OrderID INT,
    "LineNo" INT,
    ShipperID INT,
    CustomerID INT,
    ProductID INT,
    EmployeeID INT,
    ShipmentDate DATE,
    LoadDate DATETIME
);
```

## Project Steps

1. **Design a DWH schema**: Create a schema for the data warehouse based on the data structure and reporting requirements.
2. **Use SSIS to ETL the data**:
    - **Extract** data from the Excel file.
    - **Transform** the data to fit the DWH schema, ensuring data integrity and consistency.
    - **Load** the data into the new DWH schema.
3. **Handle potential challenges**: Ensure the ETL process handles potential inconsistencies and challenges in the data.
4. **Date formatting**: Load dates in the format YYYY-MM-DD.
5. **Stock check column**: Add a new column to indicate if there are enough units in stock to fulfill an order.
6. **Incremental data loads**: Ensure the DWH can handle incremental data loads, identifying new records, updating existing records, and maintaining data consistency.
7. **Aggregation table**: Create a table that aggregates the number of orders and total price for each shipment date.

## Reporting and Analysis

1. **Create Report using Excel as a source**: The report will be based on the transformed data stored in the DWH.
2. **Data Transformations**: All data transformations should be performed using a BI tool, not by modifying the original Excel file.
3. **Visualize Data and KPIs**:
    - Analyze sales figures and compare with the previous year (current year is the maximum year from the data).
    - Analyze sales margin.
    - Visualize data across various business dimensions: Division, Country, Region, State/Province, Product Line, Product Group, Product, Customer.
    - Analyze average sales per transaction and per customer.
    - Show the number of customers in a given period YTD vs LY YTD.
    - Provide suggestions for additional analysis that could be beneficial to users of the report.
