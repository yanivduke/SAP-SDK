#
# SAP Development

#
## SAP business one (SB1) SDK integration

# introduction

This article includes a basic knowledge about **S** AP **D** evelopment **K** it (SDK).

[SAP](https://en.wikipedia.org/wiki/SAP_Business_One) software is an ERP that provides several modules which handles different business processes.

The SDK allows software customizations and integration in a safe, complete and organize environment, as opposed to writing direct SQL statements to SQL Server&#39;s SAP database.

SAP business one ERP software works with many databases - the key ones are DB2, MS SQL, Oracle and HANA.

It is possible to read data directly from database, but it is **forbidden** to add new data or change data (write operation) directly to database because of the following reasons:

- SAP warranty – SAP wont supports damages caused by direct change of SAP data.
- Data integrity – depended data won&#39;t be update.
- Performance – update deadlocks and other performance problem caused by writing data without que or buffer.


# principles

In order to use the SB1 SDK properly it is important to understand some basic principles:

Data integrity – update via SB1 SDK force the same validation as the SAP client application, so you cannot change prices on closed status document or add document without fill-in all mandatory fields, and more other validation rules that happend on application screen like values range and type.

Document identity is not DocNum but DocEntry!

The SDK includes two main part Di API – for data manipulations, and Ui API – for user interface modifications.

This article focused on the Di API.


# installation

The SDK can be found on the SB1 client installation pack, under **Packages\SDK**.
After installation the Di DLL can be added to the .NET project.
![Adding Reference](reference1.png "Adding Reference")

![Di API DLL Added to solution](explorer2.png "Di API DLL Added to solution")

In some cases, the project must be configured as 64bit in order to be compile successfully.
![Platform target x64](build3.png "Platform target x64")


#  Objects update model

A code example for company&#39;s database connection.

**C#**

'''csharp
int res = 0;string strret = &quot;&quot;;SAPbobsCOM.Company oCompany = new SAPbobsCOM.Company();oCompany.DbServerType = SAPbobsCOM.BoDataServerTypes.dst\_MSSQL2008;oCompany.Server = &quot;127.0.0.1&quot;; //ip or server name for the SQL Server oCompany.UseTrusted = false; oCompany.CompanyDB = &quot;db\_name&quot;;oCompany.UserName = &quot;SomeUser&quot;;oCompany.Password = &quot;\*\*\*\*\*\*\*\*&quot;; oCompany.Connect();oCompany.GetLastError(out res, out strret);strret = oCompany.GetLastErrorDescription();MessageBox.Show(strret);  
'''



After connecting the company&#39;s DB other actions can be done, for example add a new order document.

**C#**

```csharp
SAPbobsCOM.Documents oDoc = oCompany.GetBusinessObject(SAPbobsCOM.BoObjectTypes.oOrders);SAPbobsCOM.Document\_Lines oLines = oDoc.Lines; oDoc.CardCode = &quot;12341234&quot;;oDoc.DocDate = DateTime.Now;oDoc.DocDueDate = DateTime.Now.AddDays(10); oLines.ItemCode = &quot;ABC12345&quot;;oLines.Price = 100; oLines.Add();oLines.ItemCode = &quot;ABCD4321&quot;;oLines.Price = 120; int res = oDoc.Add(); if (res != 0){    MessageBox.Show(oCompany.GetLastErrorDescription());}else{    MessageBox.Show(oCompany.GetNewObjectKey()); //@scope\_identity}  
```

For updating an exist order document,  DocEntry must be fetched first by SQL query or other method.

**C#**

|  SAPbobsCOM.Documents oDocToUpdate = oCompany.GetBusinessObject(SAPbobsCOM.BoObjectTypes.oOrders);SAPbobsCOM.Document\_Lines oLinesToUpdate = oDocToUpdate.Lines; int DocEntry = 217022334; if(oDocToUpdate.GetByKey(DocEntry)){    oLinesToUpdate.SetCurrentLine(1);    oLinesToUpdate.Price = 50;     oLinesToUpdate.Add();    oLinesToUpdate.SetCurrentLine(2);     oLinesToUpdate.ItemCode = &quot;-&quot;;    oLinesToUpdate.Price = 100;     res = oDocToUpdate.Update();} |
| --- |





# xml update model

קיימת דרך נוספת לבצע הוספה ועדכון למסמכים – ע&quot;י שליחת XML, בצורה זו אפשר לפתח מנגנון כללי יותר לצורך תמיכה ביותר מסוג אובייקט אחד, ובנוסף המנגנון פועל קצת יותר מהר.

מומלץ מאוד להוספה, אך עלול לעשות בעיות בעדכון – שדות שלא יוזנו ל XML המעדכן ידרסו בערך ריק.

ע&quot;י קבלת הסכימה ניתן לבצע מניפולציות על קוד ה XML של הרבה סוגי אובייקטים.

קבלת סכמה

**C#**

|  oCompany.XmlExportType = SAPbobsCOM.BoXmlExportTypes.xet\_ExportImportMode;oCompany.XMLAsString = false; string xmlSchema = oCompany.GetBusinessObjectXmlSchema(SAPbobsCOM.BoObjectTypes.oBusinessPartners);System.IO.File.WriteAllText(@&quot;C:\TEMP\bp.xml&quot;, xmlSchema);  |
| --- |

המרה ל XML

**C#**

|  SAPbobsCOM.BusinessPartners oBP = oCompany.GetBusinessObject(SAPbobsCOM.BoObjectTypes.oBusinessPartners);oBP.GetByKey(&quot;12186938&quot;);System.IO.File.WriteAllText(&quot;C:\\TEMP\\bp.xml&quot;, oBP.GetAsXML());  |
| --- |

הוספה ל XML

**C#**

|  int count = oCompany.GetXMLelementCount(&quot;C:\\TEMP\\bp.xml&quot;);for(int i=0; i &lt; count; i++){     SAPbobsCOM.BoObjectTypes type = oCompany.GetXMLobjectType(&quot;C:\\TEMP\\bp.xml&quot;, i);     oBP = oCompany.GetBusinessObjectFromXML(&quot;C:\\TEMP\\bp.xml&quot;, i);     res = oBP.Add();} |
| --- |
