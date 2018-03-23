SAP DEVELOPMENT
SAP BUSINESS ONE (SB1) SDK INTEGRATION
INTRODUCTION
This article includes a basic knowledge about SAP Development Kit (SDK).
SAP software is an ERP that provides several modules which handles different business processes.
The SDK allows software customizations and integration in a safe, complete and organize environment, as opposed to writing direct SQL statements to SQL Server’s SAP database.
SAP works with many databases - the key ones are DB2, MS SQL, Oracle and HANA.
It is possible to read data directly from database, but it is forbidden to add new data or change data (write operation) directly to database because of the following reasons:
•	SAP warranty – SAP wont supports damages caused by direct change of SAP data.
•	Data integrity – depended data won’t be update.
•	Performance – update deadlocks and other performance problem caused by writing data without que or buffer.
PRINCIPLES
In order to use the SB1 SDK properly it is important to understand some basic principles:
Data integrity – update via SB1 SDK force the same validation as the SAP client admin, so you cannot change prices on closed status document or add document without all mandatory fields.
Document identity is not DocNum but DocEntry.
The SDK includes two main part Di API – for data manipulations, and Ui API – for user interface modifications.
This article focused on the Di API.

 

INSTALLATION
The SDK can be found on the SB1 client installation pack, under Packages\SDK.
לאחר ההתקנה אפשר להוסיף לפרויקטים מבוססי .NET את ה DLLים של ה SDK.
 Adding Reference 
 
Di API DLL Added to solution
 

In some cases, the project must be configured as 64bit in order to be compile successfully.
 
Platform target x64
 

 OBJECTS UPDATE MODEL
A code example for company’s database connection.
C#

int res = 0;
string strret = "";

SAPbobsCOM.Company oCompany = new SAPbobsCOM.Company();
oCompany.DbServerType = SAPbobsCOM.BoDataServerTypes.dst_MSSQL2008;
oCompany.Server = "127.0.0.1"; //ip or server name for the SQL Server

oCompany.UseTrusted = false;

oCompany.CompanyDB = "db_name";
oCompany.UserName = "SomeUser";
oCompany.Password = "********";

oCompany.Connect();
oCompany.GetLastError(out res, out strret);
strret = oCompany.GetLastErrorDescription();
MessageBox.Show(strret);


 

After connecting the company’s DB other actions can be done, for example add a new order document.
C#

SAPbobsCOM.Documents oDoc = oCompany.GetBusinessObject(SAPbobsCOM.BoObjectTypes.oOrders);
SAPbobsCOM.Document_Lines oLines = oDoc.Lines;

oDoc.CardCode = "12341234";
oDoc.DocDate = DateTime.Now;
oDoc.DocDueDate = DateTime.Now.AddDays(10);

oLines.ItemCode = "ABC12345";
oLines.Price = 100;

oLines.Add();
oLines.ItemCode = "ABCD4321";
oLines.Price = 120;

int res = oDoc.Add();

if (res != 0)
{
    MessageBox.Show(oCompany.GetLastErrorDescription());
}
else
{
    MessageBox.Show(oCompany.GetNewObjectKey()); //@scope_identity
}

For updating an exist order document,  DocEntry must be fetched first by SQL query or other method.
C#

SAPbobsCOM.Documents oDocToUpdate = oCompany.GetBusinessObject(SAPbobsCOM.BoObjectTypes.oOrders);
SAPbobsCOM.Document_Lines oLinesToUpdate = oDocToUpdate.Lines;

int DocEntry = 217022334;

if(oDocToUpdate.GetByKey(DocEntry))
{
    oLinesToUpdate.SetCurrentLine(1);
    oLinesToUpdate.Price = 50;

    oLinesToUpdate.Add();
    oLinesToUpdate.SetCurrentLine(2);

    oLinesToUpdate.ItemCode = "-";
    oLinesToUpdate.Price = 100;

    res = oDocToUpdate.Update();
}




