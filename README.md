# Views-and-Triggers--SQL
Views and Triggers- SQL

CREATE the following tables –
a. Emp_triggers– Add Identity Column (1000,1)
b. Emphistory
*/

CREATE TABLE Emp_Triggers
(EmpID INT PRIMARY KEY NOT NULL IDENTITY (1000,1),
EmpName VARCHAR (50) NULL,
DeptId INT)

CREATE TABLE EmpHistory
(EmpID INT,
DeptID INT NULL,
IsActive INT )

/*

TRIGGERS
a. Trigger 1 - Build a trigger on the emp table after insert that adds a record into the emp_History table and marks IsActive column to 1
*/

CREATE TRIGGER TrigAfterInsert ON [dbo].EmpHistory
FOR INSERT
AS
	declare @EmpID INT;
	declare @DeptID INT;
	declare @IsActive INT;


	select @EmpID=i.EmpID from inserted i;	
	select @DeptID=i.DeptID from inserted i;	
	select @IsActive=1


	insert into EmpHistory
		   (EmpID,DeptID,IsActive) 
	values(@EmpID,@DeptID,@IsActive);

	PRINT 'AFTER INSERT trigger fired.'
/*
b. Trigger 2 – Build a trigger on the emp table after an update of the empname or deptid column - It updates the subsequent empname and/or deptid in the emp_history table.

*/
CREATE TRIGGER trgAfterUpdates ON [dbo].[Emp_Triggers]
FOR UPDATE
AS
declare @Empid INT;
declare @EmpName VARCHAR(50);
declare @deptID INT;

	select @EmpID=i.EmpID from inserted i;	
	select @EmpName=i.EmpName from inserted i;	
	select @DeptId=i.DeptId from inserted i;	

	if update(EmpID)
		set @EmpID='Updated Record -- After Update Trigger.';
	if update(DeptId)
		set @DeptId='Updated Record -- After Update Trigger.';

	insert into Emp_Triggers(EmpID,EmpName,DeptId) 
	values(@EmpID,@EmpName,@DeptId);

	PRINT 'AFTER UPDATE Trigger fired.'
/*

c. Build a trigger on the emp table after delete that marks the isactive status = 0 in the emp_History table.
*/

CREATE TRIGGER TriggersAfterDelete ON [dbo].[EmpHistory] 
AFTER DELETE
AS

	declare @EmpID INT;
	declare @DeptID INT;
	declare @IsActive INT;

	select @EmpID=d.EmpID from deleted d;	
	select @DeptID=d.DeptID from deleted d;	
	select @IsActive=d.IsActive from deleted d;	
	set    @IsActive =0;

	insert into EmpHistory
				( EmpID,DeptID,IsActive) 
	values(@EmpID,@DeptID,@IsActive);

	PRINT 'AFTER DELETE TRIGGER fired.'
/*

Run this script – Results should show 10 records in the emp history table all with an active status of 0
--SUCCESSFUL---
*/
INSERT INTO dbo.Emp_Triggers
SELECT 'Ali',1000
INSERT INTO dbo.Emp_Triggers
SELECT 'Buba',1000
INSERT INTO dbo.Emp_Triggers
SELECT 'Cat',1001
INSERT INTO dbo.Emp_Triggers
SELECT 'Doggy',1001
INSERT INTO dbo.Emp_Triggers
SELECT 'Elephant',1002
INSERT INTO dbo.Emp_Triggers
SELECT 'Fish',1002
INSERT INTO dbo.Emp_Triggers
SELECT 'George',1003
INSERT INTO dbo.Emp_Triggers
SELECT 'Mike',1003
INSERT INTO dbo.Emp_Triggers
SELECT 'Anand',1004
INSERT INTO dbo.Emp_Triggers
SELECT 'Kishan',1004
DELETE FROM dbo.Emp_Triggers

----SUCCESSFUL------

/*
--4. Create 5 views – Each view will use 3 tables and have 9 columns with 3 coming from each table.

--a. Create a view using 3 Human Resources Tables (Utilize the WHERE clause)
*/

CREATE VIEW V_EmployeeDetails

AS
	SELECT        b.BusinessEntityID,a.JobTitle,c.DepartmentID,c.Name AS DeptName,c.GroupName,b.StartDate,b.EndDate,a.VacationHours,a.SickLeaveHours
	FROM	      [AdventureWorks2019].[HumanResources].[Employee] a
	LEFT JOIN     [AdventureWorks2019].[HumanResources].[EmployeeDepartmentHistory]b
	ON            a.BusinessEntityID= b.BusinessEntityID
	LEFT JOIN	  [AdventureWorks2019].[HumanResources].[Department]c
	ON            b.DepartmentID= c.DepartmentID
	WHERE         a.JobTitle='Chief Executive Officer'
GO
--b. Create a view using 3 Person Tables (Utilize 3 system functions)

CREATE VIEW V_AddressDetails

AS

	 SELECT	    a.AddressLine1,a.City,b.StateProvinceCode,b.Name AS StateProvince,b.StateProvinceID,a.PostalCode,REPLACE (c.CountryRegionCode,'US','USA')StateCode,UPPER (c.Name) AS CountryName,c.ModifiedDate as OldUpdate , GETDATE () AS NewUpdate
	 FROM		[AdventureWorks2019].[Person].[Address]a
	 LEFT JOIN	[AdventureWorks2019].[Person].[StateProvince]b
	 ON			a.StateProvinceID =b.StateProvinceID
	 LEFT JOIN	[AdventureWorks2019].[Person].[CountryRegion]c
	 ON			b.CountryRegionCode =c.CountryRegionCode

GO
--c. Create a view using 3 Production Tables (Utilize the Group By Statement)

CREATE VIEW V_InventoryInStock

AS


	SELECT	  MAX ( b.Quantity)MaximumQuantity,b.ProductID ,a.Name ProductName,a.ProductNumber,  a.Color,c.LocationID,c.Name LocationName,b.Shelf,c.CostRate
	FROM		[AdventureWorks2019].[Production].[Product] a
	LEFT JOIN	[AdventureWorks2019].[Production].[ProductInventory]b
	ON			a.ProductID= b.ProductID
	LEFT JOIN	 [AdventureWorks2019].[Production].[Location]c
	ON		   b.LocationID =c.LocationID
	GROUP BY   b.ProductID ,a.Name ,a.ProductNumber,a.Color, b.Quantity,c.LocationID,c.Name ,b.Shelf,c.CostRate


GO
--d. Create a view using 3 Purchasing Tables (Utilize the HAVING clause)

CREATE VIEW V_HighOrderQtyFromVendorsWithGoodCredit
AS

	SELECT		c.AccountNumber,c.Name,c.CreditRating,a.ProductID,SUM (a.UnitPrice)SumOfPrice, b.MinOrderQty,b.MaxOrderQty,b.UnitMeasureCode,a.OrderQty,a.DueDate,b.AverageLeadTime
	FROM		[AdventureWorks2019].[Purchasing].[PurchaseOrderDetail]a
	LEFT JOIN	[AdventureWorks2019].[Purchasing].[ProductVendor]b
	ON			a. ProductID = b.ProductID
	LEFT JOIN	[AdventureWorks2019].[Purchasing].[Vendor]c
	ON			b.BusinessEntityID = c.BusinessEntityID
	WHERE	    a.OrderQty >300
	GROUP BY	c.AccountNumber,c.Name,c.CreditRating,a.ProductID, a.UnitPrice, b.MinOrderQty,b.MaxOrderQty,b.UnitMeasureCode,a.OrderQty,a.DueDate,b.AverageLeadTime
	HAVING	    c.CreditRating <2

GO
--e. Create a view using 3 Sales Tables (Utilize the CASE Statement)

CREATE VIEW V_CCStatusforSalesOrders
AS

	 SELECT a.LineTotal,a.SalesOrderID,a.CarrierTrackingNumber,b.OrderDate,b.TotalDue,b.PurchaseOrderNumber,c.CardType,c.ExpMonth,c.Expyear,
   
	   CASE
		 WHEN c.ExpYear = 2005 THEN  'EXPIRED'
		 WHEN c.ExpYear = 2006 THEN ' Expiring Soon'
		 ELSE 'Current'
		 END AS ExpStatus

	 FROM	    [AdventureWorks2019].[Sales].[SalesOrderDetail] a
	 LEFT JOIN	[AdventureWorks2019].[Sales].[SalesOrderHeader]b
	 ON			a.SalesOrderID =b.SalesOrderID
	 LEFT JOIN  [AdventureWorks2019].[Sales].[CreditCard] c
	 ON         b.CreditCardID =c.CreditCardID


GO
