<h1 align="center"> RoxPartner Teste </h1>

<p>Esse é um teste com o objetivo de conhecer um pouco mais sobre a forma de trabalhar com dados e a resolução de problemas que envolvem engenharia de dados. Analisando os seguintes aspectos: </p>
<ul>
  <li>Fazer a modelagem conceitual dos dados;</li>
  <li>Criação da infraestrutura necessária</li>
  <li>Criação de todos os artefatos necessários para carregar os arquivos para o banco criado;</li>
  <li>Desenvolvimento de SCRIPT para análise de dados;</li>
   <li>Criar um relatório em qualquer ferramenta de visualização de dados. (OPCIONAL) </li>
</ul>

<h2> Modelo Lógico </h2>

![image](https://user-images.githubusercontent.com/92615548/183257472-2a9fff50-8c15-4051-a0ea-c2d7d2acab79.png)

<h2> Resolução </h2>

<p>Para e resolução desse projeto foi escolhido a plataforma Microsoft Azure para criar a infraestrutura necessária pelo custo-benefício e por oferecem no primeiro cadastro realizado uma quantia para utilizar dentro da plataforma. Foi criada uma conta no Azure, onde subi o Banco de Dados SQL utilizando a camada de preço (Uso Geral - Sem servidor: Gen5, 1 vCore), Blob Storege para armazenamento dos arquivos para importação para o Banco de Dados e Data Factory para criação de Pipelines que atende perfeitamente a necessidade para o projeto. </p>

![image](https://user-images.githubusercontent.com/92615548/183257381-1b5cc99c-b3f6-4f2b-8d5d-81fb2625d0ae.png)

<p>Depois de analisar todos os arquivos CSV para serem importado para o banco de dados, ficou decidido que seria criado um banco de dados com o nome RoxPartner e dentro do mesmo seria criado Schemas e sua tabelas relacionadas, como mostrado a baixo:</p>

<ul>
  <li>
    Person (Schema)
    <ul>
      <li>Person (Table)</li>
    </ul>
  </li>
  <li>
    Production (Schema)
    <ul>
      <li>Product (Table)</li>
    </ul>
  </li>
  <li>
    Production (Schema)
    <ul>
      <li>Customer (Table)</li>
      <li>SalesOrderDetail (Table)</li>
      <li> SalesOrderHeader (Table)</li>
      <li>SpecialOfferProduct (Table)</li>
    </ul>
  </li>
</ul>

<p>Para a criação dos Schemas e Tabelas foi utilizado os seguintes scripts:</p>

~~~~
CREATE SCHEMA Person;
~~~~

~~~~
CREATE SCHEMA Production;
~~~~

~~~~
CREATE SCHEMA Sales;
~~~~

~~~~
CREATE TABLE Production.Product
(
ProductID INT NOT NULL,
Name VARCHAR(50),
ProductNumber VARCHAR(10),
MakeFlag INT,
FinishedGoodsFlag INT,
Color VARCHAR(20),
SafetyStockLevel INT,
ReorderPoint INT,
StandardCost FLOAT(10),
ListPrice FLOAT(10),
Size VARCHAR(5),
SizeUnitMeasureCode VARCHAR(5),
WeightUnitMeasureCode VARCHAR(10),
Weight FLOAT(10),
DaysToManufacture INT,
ProductLine VARCHAR(5),
Class VARCHAR(5),
Style VARCHAR(5),
ProductSubcategoryID INT,
ProductModelID INT,
SellStartDate DATE,
SellEndDate DATE,
DiscontinuedDate DATE,
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY (ProductID)
)
~~~~

~~~~
CREATE TABLE Person.Person
(
BusinessEntityID INT NOT NULL,
PersonType VARCHAR(4),
NameStyle INT,
Title VARCHAR(50),
FirstName VARCHAR(50),
MiddleName VARCHAR(50),
LastName VARCHAR(50),
Suffix VARCHAR(4),
EmailPromotion INT,
AdditionalContactInfo VARCHAR(1700),
Demographics VARCHAR(700),
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY(BusinessEntityID)
)
~~~~

~~~~
CREATE TABLE Sales.Customer
(
CustomerID INT NOT NULL,
PersonID INT,
StoreID INT,
TerritoryID INT,
AccountNumber VARCHAR(20),
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY (CustomerID),
CONSTRAINT FK_Customer_Person_PersonID FOREIGN KEY (PersonID) REFERENCES person.person (BusinessEntityID)
)
~~~~

~~~~
CREATE TABLE Sales.SalesOrderHeader
(
SalesOrderID INT NOT NULL,
RevisionNumber INT,
OrderDate DATE,
DueDate DATE,
ShipDate DATE,
Status INT,
OnlineOrderFlag INT,
SalesOrderNumber VARCHAR(10),
PurchaseOrderNumber VARCHAR(20),
AccountNumber VARCHAR(20),
CustomerID INT,
SalesPersonID INT,
TerritoryID INT,
BillToAddressID INT,
ShipToAddressID INT,
ShipMethodID INT,
CreditCardID INT,
CreditCardApprovalCode VARCHAR(20),
CurrencyRateID INT,
SubTotal decimal(10,4),
TaxAmt decimal(10,4),
Freight decimal(10,4),
TotalDue decimal(10,4),
Comment VARCHAR(50),
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY (SalesOrderID),
CONSTRAINT FK_SalesOrderHeader_Customer_CustomerID FOREIGN KEY (CustomerID) REFERENCES Sales.Customer (CustomerID)
)
~~~~

~~~~
CREATE TABLE Sales.SpecialOfferProduct
(
SpecialOfferID INT NOT NULL,
ProductID INT NOT NULL,
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY (SpecialOfferID, ProductID),
CONSTRAINT FK_SpecialOfferProduct_Product_ProductID FOREIGN KEY (ProductID) REFERENCES Production.Product (ProductID)
)
~~~~

~~~~
CREATE TABLE Sales.SalesOrderDetail
(
SalesOrderID INT NOT NULL,
SalesOrderDetailID INT NOT NULL,
CarrierTrackingNumber VARCHAR(20),
OrderQty INT,
ProductID INT,
SpecialOfferID INT,
UnitPrice decimal(10,4),
UnitPriceDiscount decimal(10,4),
LineTotal FLOAT,
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY (SalesOrderID, SalesOrderDetailID),
CONSTRAINT FK_SalesOrderDetail_SalesOrderHeader_SalesOrderID FOREIGN KEY (SalesOrderID) REFERENCES Sales.SalesOrderHeader (SalesOrderID),
CONSTRAINT FK_SalesOrderDetail_SpecialOfferProduct_SpecialOfferIDProductID FOREIGN KEY (SpecialOfferID,ProductID) REFERENCES Sales.SpecialOfferProduct (SpecialOfferID,ProductID)
)
~~~~

<h3>Diagrama de dados após criação das tabelas </h3>

![image](https://user-images.githubusercontent.com/92615548/183262140-da0a1c5d-412a-42c3-9ca4-77f507a4abdd.png)

<p>Para o armazenamento dos arquivos CSV foi utilizado Azure Blob Storage. Para fazer a importação dos dados para as tabelas foram criadas tarefas de pipeline, para que copiasse os dados dos arquivos e importasse para tabelas temporárias com os seguintes nomes: </p>

<ul>
  <li>Person.StagePerson</li>
   <li>Production.StageProduct</li>
  <li>Sales.StageCustomer</li>
  <li>Sales.StageSpecialOfferProduct</li>
  <li>Sales.StageSalesOrderHeader</li>
  <li>Sales.StageSalesOrderDetail</li>
</ul>

![image](https://user-images.githubusercontent.com/92615548/183261445-525d8f42-c295-42a8-832a-4e85d3293015.png)

<p>Após a importação dos dados dos arquivos para as tabelas temporárias, foram criadas procedures para que fosse tratados os dados das mesmas e fosse copiado para a tabela real com os dados devidamente tratados. Abaixo segue script das procedures utilizadas para tratar os dados, copiar para as tebelas reais e excluir as tabelas temporárias.</p>

~~~~
CREATE OR ALTER     PROCEDURE [dbo].[Proc_Insert_Person] AS
BEGIN
	INSERT INTO Person.person
	SELECT * FROM Person.StagePerson;

	DROP TABLE Person.StagePerson;
END
~~~~

~~~~
create or ALTER   PROCEDURE [dbo].[Proc_Insert_Customer] AS
BEGIN
	INSERT INTO Sales.customer
	SELECT  CustomerID,
		CASE 
			WHEN PersonID = 'NULL' THEN null
			ELSE PersonID END,
			CASE 
			WHEN StoreID = 'NULL' THEN null
			ELSE StoreID END,
			CASE 
			WHEN TerritoryID = 'NULL' THEN null
			ELSE TerritoryID END,
		AccountNumber,
		rowguid,
		ModifiedDate 
	FROM Sales.StageCustomer

	DROP TABLE Sales.StageCustomer;
END
~~~~

~~~~
create or ALTER     PROCEDURE [dbo].[Proc_Insert_Product] AS
BEGIN
	INSERT INTO Production.product
	SELECT	  ProductID,
			  Name,
			  ProductNumber,
			  CASE WHEN MakeFlag = 'NULL' THEN NULL ELSE MakeFlag END,
			  CASE WHEN FinishedGoodsFlag = 'NULL' THEN NULL ELSE FinishedGoodsFlag END,
			  Color,
			  CASE WHEN SafetyStockLevel = 'NULL' THEN NULL ELSE SafetyStockLevel END,
			  CASE WHEN ReorderPoint = 'NULL' THEN NULL ELSE ReorderPoint END,
			  CAST(REPLACE(TRIM(StandardCost),',','.') AS float),
			  CAST(REPLACE(TRIM(ListPrice),',','.') AS float),
			  Size,
			  SizeUnitMeasureCode,
			  WeightUnitMeasureCode,
			  case When Weight = 'NULL' THEN null ELSE CAST(REPLACE(TRIM(Weight),',','.') AS float) END,
			  CASE WHEN DaysToManufacture = 'NULL' THEN NULL ELSE DaysToManufacture END,
			  ProductLine,
			  Class,
			  Style,
			  CASE WHEN ProductSubcategoryID = 'NULL' THEN NULL ELSE ProductSubcategoryID END,
			  CASE WHEN ProductModelID = 'NULL' THEN NULL ELSE ProductModelID END,
			  convert(date,case When SellStartDate = 'NULL' THEN null ELSE SellStartDate  END),
			  convert(date,case When SellEndDate = 'NULL' THEN null ELSE SellEndDate  END),
			  convert(date,case When DiscontinuedDate = 'NULL' THEN null ELSE DiscontinuedDate END),
			  rowguid,
			  convert(datetime,ModifiedDate)
  FROM Production.StageProduct

	DROP TABLE Production.StageProduct
END
~~~~

~~~~
create or ALTER PROCEDURE [dbo].[Proc_Insert_SalesOrderDetail] AS
BEGIN 
	INSERT INTO Sales.Salesorderdetail
	SELECT	[SalesOrderID],
			[SalesOrderDetailID],
			[CarrierTrackingNumber],
			[OrderQty],
			[ProductID],
			[SpecialOfferID],
			convert(FLOAT,replace([UnitPrice], ',', '.')),
			convert(FLOAT,replace([UnitPriceDiscount], ',', '.')),
			convert(FLOAT,replace([LineTotal], ',', '.')),
			[rowguid],
			[ModifiedDate]
	FROM Sales.StageSalesorderdetail

	DROP TABLE Sales.StageSalesorderdetail
END
~~~~

~~~~
create or ALTER   PROCEDURE [dbo].[Proc_Insert_SalesOrderHeader] AS
BEGIN 
	INSERT INTO Sales.salesorderheader
	SELECT CASE WHEN SalesOrderID = 'NULL' THEN NULL ELSE SalesOrderID END,
       RevisionNumber,
       OrderDate,
       DueDate,
       ShipDate,
       Status,
       OnlineOrderFlag,
       SalesOrderNumber,
       PurchaseOrderNumber,
       AccountNumber,
       CASE WHEN CustomerID = 'NULL' THEN NULL ELSE CustomerID END,
       CASE WHEN SalesPersonID = 'NULL' THEN NULL ELSE SalesPersonID END,
       CASE WHEN TerritoryID = 'NULL' THEN NULL ELSE TerritoryID END,
       CASE WHEN BillToAddressID = 'NULL' THEN NULL ELSE BillToAddressID END,
       CASE WHEN ShipToAddressID = 'NULL' THEN NULL ELSE ShipToAddressID END,
       CASE WHEN ShipMethodID = 'NULL' THEN NULL ELSE ShipMethodID END,
       CASE WHEN CreditCardID = 'NULL' THEN NULL ELSE CreditCardID END,
       CreditCardApprovalCode,
       CASE WHEN CurrencyRateID = 'NULL' THEN NULL ELSE CurrencyRateID END,
       convert(float,replace(SubTotal, ',', '.')),
       convert(float,replace(TaxAmt, ',', '.')),
       convert(float,replace(Freight, ',', '.')),
       convert(float,replace(TotalDue, ',', '.')),  
	   Comment,
	   rowguid,
       ModifiedDate
  FROM Sales.StageSalesorderheader

	DROP TABLE Sales.StageSalesorderheader
END
~~~~

~~~~
create or ALTER     PROCEDURE [dbo].[Proc_Insert_SpecialOfferProduct] AS
BEGIN 
	INSERT INTO [Sales].[specialofferproduct]
	SELECT * FROM [Sales].[StageSpecialofferproduct]

	DROP TABLE [Sales].[StageSpecialofferproduct]
END
~~~~


<p>Depois da criação das procedures foi criado mais uma tarefa de pipeline, para que executasse as procedures para a importação dos dados para as tabelas reais, de forma que a ordem de importação respeitasse as ligações das Primary Key e Foreign Key, como podemos ver na imagem abaixo:</p>

![image](https://user-images.githubusercontent.com/92615548/183261765-4f436f78-bd57-4aa5-a7c4-76644710647e.png)

<p>E por fim foi criado outra tarefa de pipeline para que executasse as outras tarefas automaticamente e na ordem correta. Como podemos abaixo: </p>

![image](https://user-images.githubusercontent.com/92615548/183261849-3ed7d622-70aa-4587-b23a-e0329558ea44.png)

<h2>Análise  de dados </h2>

<p>Com base na solução implantada responda aos seguintes questionamentos:</p>
<p>1 - Escreva uma query que retorna a quantidade de linhas na tabela Sales.SalesOrderDetail pelo campo SalesOrderID, desde que tenham pelo menos três linhas de detalhes.</p>

~~~~
SELECT SalesOrderID, COUNT (*) AS qtd
    FROM Sales.salesorderdetail
GROUP BY SalesOrderID
  HAVING COUNT (*) >= 3
~~~~

<p>2 - Escreva uma query que ligue as tabelas Sales.SalesOrderDetail, Sales.SpecialOfferProduct e Production.Product e retorne os 3 produtos (Name) mais vendidos (pela soma de OrderQty), agrupados pelo número de dias para manufatura (DaysToManufacture).</p>

~~~~
SELECT DaysManufacture, name, qtd
  FROM (SELECT p.DaysToManufacture AS DaysManufacture,
               ROW_NUMBER ()
               OVER (PARTITION BY p.DaysToManufacture
                     ORDER BY SUM (d.OrderQty) DESC)
                  AS b,
               p.Name              AS name,
               SUM (d.OrderQty)    AS qtd
          FROM Sales.salesorderdetail AS   d
  INNER JOIN Sales.specialofferproduct as o ON o.ProductID = d.ProductID 
  INNER JOIN Production.product as P ON p.ProductID = o.ProductID
GROUP BY name,p.DaysToManufacture) as c
WHERE b <= 3;
~~~~

<p>3 - Escreva uma query ligando as tabelas Person.Person, Sales.Customer e Sales.SalesOrderHeader de forma a obter uma lista de nomes de clientes e uma contagem de pedidos efetuados.</p>

~~~~
SELECT c.CustomerID                          AS CustomerID,
       CONCAT (p.FirstName, ' ', p.LastName) AS Name,
       COUNT (*)                             AS qtdPedidos
  FROM Sales.SalesOrderHeader AS   h
  INNER JOIN Sales.Customer as c ON h.CustomerID = c.CustomerID
  INNER JOIN Person.Person as p ON c.PersonID = p.BusinessEntityID
GROUP BY c.PersonID,c.CustomerID,p.FirstName,p.LastName
~~~~

<p>4 - Escreva uma query usando as tabelas Sales.SalesOrderHeader, Sales.SalesOrderDetail e Production.Product, de forma a obter a soma total de produtos (OrderQty) por ProductID e OrderDate.</p>

~~~~
SELECT p.ProductID, 
	   OrderDate,
	   sum(OrderQty) qtd
  FROM Sales.SalesOrderHeader AS   h 
  INNER JOIN Sales.SalesOrderDetail AS d ON d.SalesOrderID  = h.SalesOrderID
  INNER JOIN Production.Product AS p ON d.SalesOrderID  = h.SalesOrderID
WHERE d.ProductID = p.ProductID
GROUP BY p.ProductID, OrderDate
ORDER BY p.ProductID
~~~~

<p>5 - Escreva uma query mostrando os campos SalesOrderID, OrderDate e TotalDue da tabela Sales.SalesOrderHeader. Obtenha apenas as linhas onde a ordem tenha sido feita durante o mês de setembro/2011 e o total devido esteja acima de 1.000. Ordene pelo total devido decrescente.</p>

~~~~
  SELECT SalesOrderID, OrderDate, TotalDue
    FROM Sales.SalesOrderHeader
   WHERE OrderDate BETWEEN '2011-09-01' AND '2011-09-30' AND TotalDue > 1000
ORDER BY OrderDate
~~~~


