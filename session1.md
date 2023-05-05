---
title: "Session 1: TMS FlexCel"
layout: default
nav_order: 2
has_children: false
---

# TMS FlexCel: Reporting Everywhere

The session will explain how to create a database-driven report with preview. In order to provide reports to web clients, the report will be loosely coupled to the user interface and can thus be used inside of an XData server as well.

![](images/01/preview.png)

## Creating the template

The template will be completely created in Microsoft Excel. 

{: .note}
Obviously, you could generate the template document using FlexCel API as well. However, designing in Microsoft Excel is much easier.

![](images/01/template.png)

- Information for the yard sale record is not linked to the dataset. Instead, two placeholders are defined which are filled with values at run-time on a field-by-field basis. The report iteration does not affect this:
  1. `#YardSaleTitle`: Title of the yard sale
  1. `#YardSaleEventDates`: Date and time of the event

- The logo of the yard sale - its thumbnail to be precise - will be loaded into the placeholder `#YardSaleThumb`.\
![](images/01/templateimg.png)

There is one remaining ingredient in the template. This is also the one that most Delphi developers struggle with. Microsoft Excel allows you to define *ranges* that are identified by names. These names are used by FlexCel (not Excel!) to determine the scope of placeholders you use in that range.

In the first screenshot of the template, notice the identifier `__P__`. This is the name of the selected range shown in the template. It encompasses all the placeholders that start with `#P.`. Further, the two underscores `__` mean that for each record inside of the dataset that is linked as `P`, this range will be inserted. Imagine you link this `P` to a `TDataSet` or `TList` that contains all the participants for your record. The placeholders inside of this range can then access all individual properties or fields of `P`. In this example, we insert `Name`, `Address`, and `Categories` into the report for each record. If you have a look at the database model, you might hesitate a bit. These fields do not exist. Well, the magic of Delphi allows us to define calculated fields that are calculated when needed. Thus, there is no restriction when using FlexCel what kind of field you want to use. As long as its value can be converted to `TValue`, you are good to go.

## Report Manager data module

The report manager hosts the operations to 
- retrieve the data from the database,
- load the report template, and
- create the report.

### Setting up the database connection

All database queries are hosted inside the data module without a `FDConnection` instance. At run-time a connection will be linked to each query passed by the main form view controller.

```pascal
constructor TReportManager.Create(AOwner: TComponent; AConnection: TFDConnection);
begin
  inherited Create( AOwner );

  FConnection := AConnection;
  Sale.Connection := FConnection;
  Participants.Connection := FConnection;
  ParticipantCategories.Connection := FConnection;
end;
```
The easiest way to do this is to provide a custom constructor. Hiding the default constructor, a connection `AConnection` now needs to be passed to the data module when created. That connection will then be stored in the field `FConnection`. Each query will also be linked to that connection.

### Retrieving data with queries

All queries are found on the designer canvas and make use of parameters. 

{: .warning}
**Always** use parameters. Never build your queries at run-time using strings.

1. `Sale`: Select the yard sale that to create the report for:
```sql
SELECT * FROM YardSales WHERE Id = :Id
```
2. `Participants`: Select all participants of that sale using `SalesId` as filter. 
```sql
SELECT * FROM SalesParticipant P  
    WHERE SalesId = :SalesId
```
3. `ParticipantCategories`: Define another query used as detail to a participant listing all their categories.
```sql
SELECT Id, Name FROM ParticipantItemCategories
    LEFT JOIN ItemCategories i ON i.Id = IdCategory
    WHERE IdParticipant = :Id 
```
This query will be managed by FireDAC. It will always list the categories of the participant that is selected in the query. The parameter `Id` will be filled through the `MasterSource` link.  The field is set as the only field in `MasterFields`.\
![](images/01/participantcategories.png)\
The data source `sourceParticipants` simply points to the dataset `Participants`.\
![](images/01/sourceparticipants.png)

### Loading the template

The template will be stored inside of the executable as a resource. This removes all clutter from the application of files that have to be deployed with it. Everything is nicely included inside of the executable. Sadly, because we use MySQL we need to deploy that particular client library, but the library most likely can be found on most user's systems already. 

{: .note}
Some database client libraries can be statically linked into the executable with FireDAC. Other database frameworks do also might not require client libraries.





