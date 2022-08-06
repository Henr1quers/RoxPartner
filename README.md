<h1 align="center"> RoxPartner Teste </h1>

<p>Esse é um teste com o objetivo de conhecer um pouco mais sobre a forma de trabalhar com dados e a resolução de problemas que envolvem engenharia de dados. Analisando os seguintes aspectos: </p>
<ul>
  <li>Fazer a modelagem conceitual dos dados;</li>
  <li>Criação da infraestrutura necessária</li>
  <li>Criação de todos os artefatos necessários para carregar os arquivos para o banco criado;</li>
  <li>Desenvolvimento de SCRIPT para análise de dados;</li>
   <li>Criar um relatório em qualquer ferramenta de visualização de dados. (OPCIONAL) </li>
</ul>

<h2> Modelo Fisio </h2>

![image](https://user-images.githubusercontent.com/92615548/183257472-2a9fff50-8c15-4051-a0ea-c2d7d2acab79.png)

<p>Para e resolução desse projeto foi escolhido a plataforma Microsoft Azure para criar a infraestrutura necessária pelo custo-benefício e por oferecem no primeiro cadastro realizado uma quantia para utilizar dentro da plataforma. Foi utilizado no projeto o Banco de dados SQL utilizando a camada de preço (Uso Geral - Sem servidor: Gen5, 1 vCore) , que atende perfeitamente a necessidade para o projeto. </p>

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
SubTotal FLOAT,
TaxAmt FLOAT,
Freight FLOAT,
TotalDue FLOAT,
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
UnitPrice FLOAT,
UnitPriceDiscount FLOAT,
LineTotal FLOAT,
rowguid VARCHAR(36),
ModifiedDate DATETIME,
PRIMARY KEY (SalesOrderID, SalesOrderDetailID),
CONSTRAINT FK_SalesOrderDetail_SalesOrderHeader_SalesOrderID FOREIGN KEY (SalesOrderID) REFERENCES Sales.SalesOrderHeader (SalesOrderID),
CONSTRAINT FK_SalesOrderDetail_SpecialOfferProduct_SpecialOfferIDProductID FOREIGN KEY (SpecialOfferID,ProductID) REFERENCES Sales.SpecialOfferProduct (SpecialOfferID,ProductID)
)
~~~~

<p>Para o armazenamento dos arquivos CSV foi utilizado Azure Blob Storage, para fazer as impotações dos dados para suas respectivas tabelas. Para fazer a importação dos dados para as tabelas foram criadas tarefas de pipeline, para que copiasse os dados dos arquivos e importasse para tabelas temporárias com os seguintes nomes: </p>

<ul>
  <li>Person.StagePerson</li>
   <li>Production.StageProduct</li>
  <li>Sales.StageCustomer</li>
  <li>Sales.StageSpecialOfferProduct</li>
  <li>Sales.StageSalesOrderHeader</li>
  <li>Sales.StageSalesOrderDetail</li>
</ul>
