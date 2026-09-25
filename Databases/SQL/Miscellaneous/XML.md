# XML in SQL: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
XML in SQL refers to the storage, querying, and manipulation of XML (eXtensible Markup Language) documents within relational database management systems, leveraging native XML data types and specialized query languages like XPath and XQuery.

**Technical Definition**
SQL/XML support encompasses native `xml` data types (SQL Server, Oracle), functions for querying XML content via XPath/XQuery expressions, tools for shredding XML into relational rows (`XMLTABLE`, `nodes()`), XSLT transformation capabilities, XML-specific indexing strategies, and schema validation against DTDs or XSDs.

**Beginner-Friendly Explanation**
Imagine a database column that can hold an entire structured document—like a receipt or an invoice with nested line items—instead of just a simple number or string. XML in SQL lets you store, search, and transform these documents using special queries that understand the document's structure.

### Key Characteristics

- **Native XML Data Types**: SQL Server (`xml`), Oracle (`XMLType`), and PostgreSQL (`xml`) provide dedicated column types that validate well-formedness and support XML-specific operations.
- **XPath and XQuery Integration**: XML content is queried using standard path languages, enabling navigation to specific nodes and values.
- **Shredding Capabilities**: `XMLTABLE` and `nodes()` convert XML fragments into relational rows, enabling joins and set operations.
- **Schema Validation**: Columns can be bound to XML Schema Collections (XSDs) to enforce structural constraints.
- **XML Indexing**: Primary and secondary XML indexes accelerate XPath/XQuery queries.

### Prerequisites

- Basic SQL (SELECT, INSERT, CREATE TABLE)
- Understanding of XML syntax (elements, attributes, namespaces)
- Familiarity with indexing concepts

### Related Programming Areas

- **Data Integration**: Exchanging structured documents between systems
- **Web Services**: SOAP and REST payloads
- **Configuration Storage**: Hierarchical settings
- **Document Management**: Storing and querying semi-structured content

### Core Concepts

| Concept | Description |
|---------|-------------|
| XML Storage | Native `xml`/`XMLType` vs. text/CLOB storage |
| XML Querying | XPath navigation and XQuery expressions |
| XML Extraction | `query()`, `value()`, `exist()` methods; `XMLTABLE` shredding |
| XML Transformation | XSLT processing (limited in-database support) |
| XML Indexing | Primary XML index + secondary PATH/VALUE/PROPERTY indexes |
| Schema Validation | XSD binding via XML Schema Collections |


## 1. XML Storage (Native Data Types vs. Large Text Objects)

### Definitions

**Core Definition**
XML storage determines how XML documents are persisted in a database—either as native XML-typed columns with built-in validation and indexing, or as large text/CLOB columns with XML processing done externally.

**Technical Definition**
SQL Server provides the `xml` data type, Oracle provides `XMLType`, and PostgreSQL offers `xml`. These native types validate well-formedness on insert, support XML-specific methods, and can be indexed. Text/CLOB storage treats XML as opaque character data, requiring application-level parsing.

**Beginner-Friendly Explanation**
Storing XML in a native column is like putting a document in a special filing cabinet that understands its structure. Storing it as text is like putting it in a plain box—you can keep it, but you have to open it yourself to find anything.

### Purposes (All begin with "To")

- **To** store XML documents with automatic well-formedness validation
- **To** enable XPath/XQuery queries directly against stored XML
- **To** support XML-specific indexing for query performance
- **To** enforce XML Schema validation when bound to a schema collection

### Syntax Rules and Structure

**SQL Server**
```sql
-- Untyped XML column
CREATE TABLE T (pk INT PRIMARY KEY, xCol XML);

-- Typed XML column (bound to schema)
CREATE TABLE T2 (pk INT PRIMARY KEY, xCol XML(dbo.MySchemaCollection));
```

**Oracle**
```sql
-- XMLType column
CREATE TABLE po_xml (id NUMBER, po XMLType);

-- With schema validation
CREATE TABLE po_typed (id NUMBER, po XMLType) 
XMLTYPE po STORE AS OBJECT RELATIONAL
XMLSCHEMA "http://example.com/po.xsd" ELEMENT "PurchaseOrder";
```

**Component Breakdown**

| Database | Native Type | Validation | Indexing Support |
|----------|-------------|------------|------------------|
| SQL Server | `xml` | Well-formedness; XSD if bound | Primary + secondary XML indexes |
| Oracle | `XMLType` | Well-formedness; XSD if registered | XMLIndex, structured storage |
| PostgreSQL | `xml` | Well-formedness | Limited (expression indexes) |

### Annotated Code Examples

**Example 1: Creating XML Columns in SQL Server and Oracle**

```sql
-- SQL Server: Untyped XML
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Description XML
);

INSERT INTO Products VALUES (1, '<Product><Name>Widget</Name><Price>9.99</Price></Product>');

-- SQL Server: Typed XML with schema
CREATE XML SCHEMA COLLECTION ProductSchema AS
N'<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <xsd:element name="Product">
    <xsd:complexType>
      <xsd:sequence>
        <xsd:element name="Name" type="xsd:string"/>
        <xsd:element name="Price" type="xsd:decimal"/>
      </xsd:sequence>
    </xsd:complexType>
  </xsd:element>
</xsd:schema>';

CREATE TABLE TypedProducts (
    ProductID INT PRIMARY KEY,
    Description XML(ProductSchema)
);
```

**Expected Output:**
```
Query OK, 1 row affected
```

**Why This Output Occurs**: The untyped column accepts any well-formed XML. The typed column enforces the structure defined in `ProductSchema`—attempts to insert XML with missing `Price` or invalid types would fail.

### Real-World Cases

- **Order Management**: Storing purchase orders with variable line-item structures
- **API Logging**: Persisting JSON/XML payloads from web services
- **Configuration**: Application settings stored hierarchically

### References

- Microsoft Learn - XML Data Type Methods - https://learn.microsoft.com/es-es/sql/t-sql/xml/xml-data-type-methods?view=sql-server-ver16 
- UTFPR Repository - XML Database Functionality Comparison - http://repositorio.utfpr.edu.br:8080/jspui/bitstream/1/22187/1/PB_EBD_02_2017_04.pdf 
- Microsoft PDF - SQL Server 2008 XML Novelties - https://download.microsoft.com/download/F/1/9/F195FB9E-B7A5-4234-BC14-0C4178D501CA/SQL_Server_2008_novinky.pdf 


## 2. XML Querying (XPath and XQuery Integration)

### Definitions

**Core Definition**
XML querying uses XPath expressions to navigate XML document hierarchies and XQuery (a superset of XPath) to perform complex queries, filtering, and construction.

**Technical Definition**
SQL Server's XML data type provides `query()` (returns XML fragment), `value()` (returns SQL scalar), `exist()` (boolean check), and `nodes()` (rowset generation). Oracle provides `XMLQuery`, `XMLTable`, and `XMLExists`. These methods embed XPath/XQuery expressions that select nodes relative to the XML instance.

**Beginner-Friendly Explanation**
XPath is like a street address for XML nodes: `/PurchaseOrder/LineItems/LineItem[1]/Price` means "go to PurchaseOrder, then LineItems, then the first LineItem, then Price." XQuery adds the ability to filter, loop, and construct new XML.

### Purposes (All begin with "To")

- **To** navigate to specific nodes within XML documents
- **To** filter XML rows based on node values
- **To** extract scalar values for use in SQL comparisons and joins
- **To** construct new XML fragments from query results

### Syntax Rules and Structure

**SQL Server XML Methods**

| Method | Returns | Example |
|--------|---------|---------|
| `query()` | XML fragment | `xCol.query('/a/b')` |
| `value()` | SQL scalar | `xCol.value('(/a/b)[1]', 'int')` |
| `exist()` | 1 or 0 | `xCol.exist('/a[b="c"]')` |
| `nodes()` | Rowset | `xCol.nodes('/a/b') AS T(c)` |

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| XPath expression | Path to target nodes |
| `value()` second arg | SQL type to cast to |
| `nodes()` alias | Virtual table and column names |

### Annotated Code Examples

**Example 1: Querying XML with query(), value(), and exist() (SQL Server)**

```sql
-- Table with XML column
CREATE TABLE Sales.CustomerOrderSummary (
    OrderID INT PRIMARY KEY,
    OrderSummary XML
);

-- Insert sample XML
INSERT INTO Sales.CustomerOrderSummary VALUES (1,
'<SalesOrders>
  <Order>
    <OrderHeader>
      <CustomerName>Tailspin Toys</CustomerName>
      <OrderID>1001</OrderID>
    </OrderHeader>
    <OrderDetails>
      <Product ProductID="119" ProductName="Chocolate Sharks"/>
    </OrderDetails>
  </Order>
</SalesOrders>');

-- value(): extract scalar
SELECT OrderSummary.value('(/SalesOrders/Order/OrderHeader/CustomerName)[1]', 'nvarchar(100)') AS CustomerName
FROM Sales.CustomerOrderSummary;

-- query(): extract XML fragment
SELECT OrderSummary.query('/SalesOrders/Order/OrderDetails/Product') AS Products
FROM Sales.CustomerOrderSummary;

-- exist(): filter rows
SELECT * FROM Sales.CustomerOrderSummary
WHERE OrderSummary.exist('/SalesOrders/Order/OrderHeader/CustomerName[text()="Tailspin Toys"]') = 1;
```

**Expected Output:**
```
CustomerName
Tailspin Toys

Products
<Product ProductID="119" ProductName="Chocolate Sharks" />

(1 row returned from exist filter)
```

**Why This Output Occurs**: `value()` extracts the text of the `CustomerName` node as a SQL `nvarchar`. `query()` returns the `Product` element as an XML fragment. `exist()` returns 1 when the XPath expression finds a match, filtering the row.

### Real-World Cases

- **Order Processing**: Extracting customer names and order IDs from XML orders
- **Log Analysis**: Filtering log entries by XML content
- **Data Integration**: Transforming XML payloads into relational columns

### References

- Microsoft Learn - XML Data Type Methods - https://learn.microsoft.com/es-es/sql/t-sql/xml/xml-data-type-methods?view=sql-server-ver16 
- Pro T-SQL Programmer's Guide - Chapter 12: XML Querying - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2095000/61756933fdf2e50244a46addd53f56ee 
- SQL Server Advanced Data Types - Chapter 4: Querying and Shredding XML - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2259000/865fd50c20d837b78dd4ca2575d7d40f 


## 3. XML Extraction (EXTRACTVALUE, XMLTABLE for Shredding)

### Definitions

**Core Definition**
XML extraction (shredding) converts XML documents into relational rows and columns, enabling SQL joins, aggregations, and set operations on XML data.

**Technical Definition**
`XMLTABLE` (SQL standard, Oracle, PostgreSQL) and `nodes()` (SQL Server) generate virtual tables from XML fragments. Each selected node becomes a row, and XPath expressions map node values to columns. Oracle's deprecated `EXTRACTVALUE` was replaced by `XMLCAST(XMLQuery(...))`.

**Beginner-Friendly Explanation**
Shredding is like unpacking a suitcase: the XML document is the suitcase, and shredding takes out each item (node) and lays them out in rows and columns like a spreadsheet.

### Purposes (All begin with "To")

- **To** convert nested XML collections into relational rows
- **To** join XML data with existing relational tables
- **To** aggregate values stored within XML documents
- **To** create relational views over XML content

### Syntax Rules and Structure

**SQL Server (nodes() + value())**
```sql
SELECT T.c.value('@ProductID', 'int') AS ProductID,
       T.c.value('@ProductName', 'nvarchar(100)') AS ProductName
FROM OrderSummary.nodes('/SalesOrders/Order/OrderDetails/Product') AS T(c);
```

**Oracle (XMLTABLE)**
```sql
SELECT xt.*
FROM po_xml p,
     XMLTABLE('/PurchaseOrder/LineItems/LineItem'
              PASSING p.po
              COLUMNS
                item_num   NUMBER        PATH '@ItemNumber',
                description VARCHAR2(100) PATH 'Description') xt;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `nodes()` / `XMLTABLE` | Generates rowset from node collection |
| `value()` / `PATH` | Maps node/attribute to column |
| Aliases | Virtual table and column names |

### Annotated Code Examples

**Example 1: Shredding XML into Relational Rows (SQL Server)**

```sql
-- XML with multiple Product elements
DECLARE @xml XML = '
<SalesOrders>
  <Order>
    <OrderDetails>
      <Product ProductID="119" ProductName="Chocolate Sharks" Quantity="2"/>
      <Product ProductID="120" ProductName="Gummy Bears" Quantity="5"/>
    </OrderDetails>
  </Order>
</SalesOrders>';

-- Shred Product elements into rows
SELECT
    T.c.value('@ProductID', 'int') AS ProductID,
    T.c.value('@ProductName', 'nvarchar(100)') AS ProductName,
    T.c.value('@Quantity', 'int') AS Quantity
FROM @xml.nodes('/SalesOrders/Order/OrderDetails/Product') AS T(c);
```

**Expected Output:**
```
ProductID | ProductName       | Quantity
119       | Chocolate Sharks  | 2
120       | Gummy Bears       | 5
```

**Why This Output Occurs**: The `nodes()` method creates one row per `Product` element. Each `value()` call extracts a specific attribute from the current node context (`T.c`).

### Real-World Cases

- **Reporting**: Generating line-item reports from XML orders
- **Data Migration**: Moving XML data into normalized tables
- **Analytics**: Joining XML order details with product master data

### References

- Oracle XML DB Developer's Guide - XMLTable Examples - https://docs.oracle.com/en/database/oracle/oracle-database/26/adxdb/xml-db-developers-guide.pdf 
- Pro T-SQL Programmer's Guide - nodes() Method - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2095000/61756933fdf2e50244a46addd53f56ee 
- Oracle XML DB Developer's Guide - Creating Relational Views with XMLTable - https://docs.oracle.com/en/database/oracle/oracle-database/26/adxdb/xml-db-developers-guide.pdf 


## 4. XML Transformation (XSLT Processing Inside the Database)

### Definitions

**Core Definition**
XML transformation applies XSLT stylesheets to XML documents, converting them into other XML formats, HTML, or plain text directly within the database engine.

**Technical Definition**
XSLT processing in databases is limited. SQL Server supports XSLT via the `xml` data type's `transform()` method (deprecated in some versions). Oracle provides XSLT processing through XMLType methods. Most databases recommend external transformation for complex XSLT.

**Beginner-Friendly Explanation**
XSLT is a language for saying "take this XML and rearrange it into that format." For example, turning an XML invoice into an HTML receipt. Some databases can do this internally, but it's often better done outside.

### Purposes (All begin with "To")

- **To** convert XML data into presentable HTML or other formats
- **To** restructure XML without application-level code
- **To** apply consistent transformations across stored documents

### Syntax Rules and Structure

**XSLT Template Structure**
```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:template match="/">
    <html>
      <body>
        <xsl:apply-templates select="//Product"/>
      </body>
    </html>
  </xsl:template>
  <xsl:template match="Product">
    <p><xsl:value-of select="@ProductName"/></p>
  </xsl:template>
</xsl:stylesheet>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `xsl:template` | Rule matching specific nodes |
| `xsl:apply-templates` | Recursively apply rules |
| `xsl:value-of` | Insert node value into output |

### Annotated Code Examples

**Example 1: XSLT Processing Model (Conceptual)**

```xml
<!-- Source XML -->
<Products>
  <Product name="Widget" price="9.99"/>
  <Product name="Gadget" price="19.99"/>
</Products>

<!-- XSLT Stylesheet -->
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:template match="/">
    <table>
      <xsl:for-each select="Products/Product">
        <tr>
          <td><xsl:value-of select="@name"/></td>
          <td><xsl:value-of select="@price"/></td>
        </tr>
      </xsl:for-each>
    </table>
  </xsl:template>
</xsl:stylesheet>
```

**Expected Output:**
```html
<table>
  <tr><td>Widget</td><td>9.99</td></tr>
  <tr><td>Gadget</td><td>19.99</td></tr>
</table>
```

**Why This Output Occurs**: The XSLT processor matches the root, iterates over each `Product` element, and outputs a table row for each. XSLT's processing model selects templates based on pattern matching and recursively applies them.

### Real-World Cases

- **Reporting**: Generating HTML reports from XML data
- **Data Exchange**: Converting between XML dialects
- **Publishing**: Transforming content for different platforms

### References

- VLDB 2002 - Incorporating XSL Processing Into Database Engines - http://www.vldb.org/conf/2002/S04P03.pdf 
- Stack Overflow - Generating SQL using XML and XSLT - https://stackoverflow.com/questions/319159/generating-sql-using-xml-and-xslt 
- Microsoft PDF - SQL Server XML Features (XSLT) - https://download.microsoft.com/download/F/1/9/F195FB9E-B7A5-4234-BC14-0C4178D501CA/SQL_Server_2008_novinky.pdf 


## 5. XML Indexing (Namespaces, Primary/Secondary Indexes)

### Definitions

**Core Definition**
XML indexing creates auxiliary index structures on XML columns to accelerate XPath/XQuery queries, which would otherwise require expensive full-document scans.

**Technical Definition**
SQL Server requires a **primary XML index** on the XML column first (which shreds the XML into an internal table), then optional **secondary XML indexes**: PATH (for path-based queries), VALUE (for value-based queries), and PROPERTY (for property retrieval). Indexes are namespace-aware and must be created on the base table with a clustered primary key.

**Beginner-Friendly Explanation**
An XML index is like a book's index: instead of reading every page (scanning every XML document), the database can jump directly to the relevant nodes. Namespaces matter because XML elements with the same name in different namespaces are different.

### Purposes (All begin with "To")

- **To** dramatically reduce XPath/XQuery query execution time
- **To** support different query patterns via specialized secondary indexes
- **To** enable efficient filtering and joining on XML node values

### Syntax Rules and Structure

**SQL Server XML Index Creation**
```sql
-- Primary XML index (required first)
CREATE PRIMARY XML INDEX PXML_Col ON T(xCol);

-- Secondary PATH index (for path-based queries)
CREATE XML INDEX IXML_Path ON T(xCol)
USING XML INDEX PXML_Col FOR PATH;

-- Secondary VALUE index (for value-based queries)
CREATE XML INDEX IXML_Value ON T(xCol)
USING XML INDEX PXML_Col FOR VALUE;

-- Secondary PROPERTY index (for property retrieval)
CREATE XML INDEX IXML_Property ON T(xCol)
USING XML INDEX PXML_Col FOR PROPERTY;
```

**Component Breakdown**

| Index Type | Optimized For | Example Query |
|------------|---------------|---------------|
| Primary | Base structure for secondary indexes | N/A |
| PATH | Path-based queries | `/a/b/c` |
| VALUE | Value-based queries | `//*[text()="X"]` |
| PROPERTY | Property retrieval | `value()` calls |

**Syntax Rules**

- Base table must have a clustered primary key before creating XML indexes 
- Only one primary XML index per XML column allowed
- Namespaces must be declared in queries using the same URIs as the indexed data 

### Annotated Code Examples

**Example 1: Creating XML Indexes and Measuring Improvement (SQL Server)**

```sql
-- Base table with XML column
CREATE TABLE JobCandidate (
    JobCandidateID INT PRIMARY KEY,
    Resume XML
);

-- Primary XML index
CREATE PRIMARY XML INDEX PXML_Resume ON JobCandidate(Resume);

-- Secondary PATH index for education queries
CREATE XML INDEX IXML_Education ON JobCandidate(Resume)
USING XML INDEX PXML_Resume FOR PATH;

-- Query with namespace (must match indexed data)
SELECT Resume.query('
declare namespace ns = "http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/Resume";
for $m in /ns:Resume
where $m/ns:Education/ns:Edu.Level[.="Bachelor"]
return $m/ns:Name
') AS BachelorNames
FROM JobCandidate;
```

**Expected Output:**
```
BachelorNames
<ns:Name xmlns:ns="..."><ns:Name.First>John</ns:Name.First>...</ns:Name>
```

**Why This Output Occurs**: The PATH secondary index enables efficient navigation to `Education/Edu.Level` nodes. Without the index, SQL Server would shred every XML document on-the-fly. The namespace declaration must match the namespace used when the XML was inserted .

### Real-World Cases

- **HR Systems**: Searching resumes by education level or skill
- **Product Catalogs**: Filtering products by XML specification attributes
- **Financial Documents**: Finding transactions by XML tag values

### References

- Microsoft Learn - Creating XML Indexes (German) - https://learn.microsoft.com/de-de/sql/relational-databases/xml/create-xml-indexes?view=sql-server-linux-ver16 
- Microsoft Learn - Creating XML Indexes (Portuguese) - https://learn.microsoft.com/pt-pt/sql/relational-databases/xml/create-xml-indexes?view=sql-server-ver15 
- Pro T-SQL Programmer's Guide - XML Indexes and Namespaces - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2095000/61756933fdf2e50244a46addd53f56ee 


## 6. DTD and XML Schema Definition (XSD) Validation

### Definitions

**Core Definition**
XML schema validation ensures that stored XML documents conform to a predefined structure (DTD or XSD), enforcing element order, data types, and cardinality constraints at insert/update time.

**Technical Definition**
SQL Server uses **XML Schema Collections** (`CREATE XML SCHEMA COLLECTION`) bound to XML columns via `XML(CollectionName)`. Oracle uses `DBMS_XMLSCHEMA.registerSchema` and associates schemas with `XMLType` columns. Validation occurs automatically on insert/update.

**Beginner-Friendly Explanation**
An XSD is like a contract for XML documents: "You must have exactly one Name element, then one Price element, and Price must be a decimal." Binding a schema to a column means the database won't accept documents that break the contract.

### Purposes (All begin with "To")

- **To** enforce structural and data-type constraints on XML content
- **To** prevent malformed or incomplete documents from entering the database
- **To** enable query optimization based on known schema structure
- **To** support typed XML with better performance characteristics

### Syntax Rules and Structure

**SQL Server**
```sql
-- Create schema collection
CREATE XML SCHEMA COLLECTION MySchema AS
N'<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <xsd:element name="Product" type="ProductType"/>
  <xsd:complexType name="ProductType">
    <xsd:sequence>
      <xsd:element name="Name" type="xsd:string"/>
      <xsd:element name="Price" type="xsd:decimal"/>
    </xsd:sequence>
  </xsd:complexType>
</xsd:schema>';

-- Bind to column
CREATE TABLE Products (
    ID INT PRIMARY KEY,
    Data XML(MySchema)
);
```

**Oracle**
```sql
-- Register schema
BEGIN
  DBMS_XMLSCHEMA.registerSchema(
    'ProductSchema',
    '<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">...</xsd:schema>'
  );
END;
/

-- Create table with schema-bound XMLType
CREATE TABLE Products (
    ID NUMBER PRIMARY KEY,
    Data XMLType
) XMLTYPE Data STORE AS OBJECT RELATIONAL
XMLSCHEMA "ProductSchema" ELEMENT "Product";
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Schema Collection | Named collection of XSDs |
| Column binding | Associates schema with XML column |
| Validation | Automatic on insert/update |

### Annotated Code Examples

**Example 1: XSD Validation with Wildcards (SQL Server)**

```sql
-- Schema with wildcard allows extension elements
CREATE XML SCHEMA COLLECTION OrderSchema AS
N'<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <xsd:element name="Order">
    <xsd:complexType>
      <xsd:sequence>
        <xsd:element name="CustomerName" type="xsd:string"/>
        <xsd:element name="OrderTotal" type="xsd:decimal"/>
        <xsd:any namespace="##other" processContents="skip" minOccurs="0" maxOccurs="unbounded"/>
      </xsd:sequence>
    </xsd:complexType>
  </xsd:element>
</xsd:schema>';

CREATE TABLE Orders (ID INT PRIMARY KEY, Data XML(OrderSchema));

-- Valid: includes wildcard element from other namespace
INSERT INTO Orders VALUES (1,
'<Order xmlns="http://adventure-works.com/order" xmlns:shp="http://adventure-works.com/shipping">
  <CustomerName>Graeme Malcolm</CustomerName>
  <OrderTotal>299.99</OrderTotal>
  <shp:Delivery>Express</shp:Delivery>
</Order>');

-- Invalid: missing required OrderTotal
INSERT INTO Orders VALUES (2,
'<Order xmlns="http://adventure-works.com/order">
  <CustomerName>Jane Doe</CustomerName>
</Order>');
-- Error: validation failed
```

**Expected Output:**
```
Query OK, 1 row affected
Error: XML Validation: Invalid content. Expected element(s): 'OrderTotal'
```

**Why This Output Occurs**: The schema requires `CustomerName` and `OrderTotal` in sequence. The wildcard `xsd:any` with `processContents="skip"` allows the `shp:Delivery` element from another namespace. The second insert fails because `OrderTotal` is missing.

### Real-World Cases

- **B2B Integration**: Enforcing EDI/XML document standards
- **Regulatory Compliance**: Ensuring financial reports match XSD specifications
- **Data Quality**: Preventing incomplete documents from entering systems

### References

- Microsoft PDF - SQL Server 2008 XML Schema Validation - https://download.microsoft.com/download/F/1/9/F195FB9E-B7A5-4234-BC14-0C4178D501CA/SQL_Server_2008_novinky.pdf 
- UTFPR Repository - DTD/XSD Validation Comparison - http://repositorio.utfpr.edu.br:8080/jspui/bitstream/1/22187/1/PB_EBD_02_2017_04.pdf 
- Charles University - XML Schema Collections and Validity Checking - https://www.ksi.mff.cuni.cz/~svoboda/courses/171-NPRG039/lectures/Lecture-05-Databases-IBM-MS.pdf 


## Consolidated References

- Microsoft Learn - XML Data Type Methods - https://learn.microsoft.com/es-es/sql/t-sql/xml/xml-data-type-methods?view=sql-server-ver16 
- Oracle XML DB Developer's Guide - https://docs.oracle.com/en/database/oracle/oracle-database/26/adxdb/xml-db-developers-guide.pdf 
- VLDB 2002 - Incorporating XSL Processing Into Database Engines - http://www.vldb.org/conf/2002/S04P03.pdf 
- Microsoft Learn - Creating XML Indexes (German) - https://learn.microsoft.com/de-de/sql/relational-databases/xml/create-xml-indexes?view=sql-server-linux-ver16 
- UTFPR Repository - XML Database Functionality Comparison - http://repositorio.utfpr.edu.br:8080/jspui/bitstream/1/22187/1/PB_EBD_02_2017_04.pdf 
- Pro T-SQL Programmer's Guide (PDF) - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2095000/61756933fdf2e50244a46addd53f56ee 
- Stack Overflow - Generating SQL using XML and XSLT - https://stackoverflow.com/questions/319159/generating-sql-using-xml-and-xslt 
- Microsoft Learn - Creating XML Indexes (Portuguese) - https://learn.microsoft.com/pt-pt/sql/relational-databases/xml/create-xml-indexes?view=sql-server-ver15 
- Charles University - XML Schema Collections - https://www.ksi.mff.cuni.cz/~svoboda/courses/171-NPRG039/lectures/Lecture-05-Databases-IBM-MS.pdf 
- SQL Server Advanced Data Types (PDF) - /hdd/m0103/deepseek/datasets/pilimi-zlib/libgen_rs_non_fic/2259000/865fd50c20d837b78dd4ca2575d7d40f 
- Microsoft PDF - SQL Server 2008 XML Novelties - https://download.microsoft.com/download/F/1/9/F195FB9E-B7A5-4234-BC14-0C4178D501CA/SQL_Server_2008_novinky.pdf 
- Oracle XML DB Developer's Guide - Creating Relational Views - https://docs.oracle.com/en/database/oracle/oracle-database/26/adxdb/xml-db-developers-guide.pdf 