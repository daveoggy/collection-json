## OData Query Options Extension ##

Introduction
============

The [OData Query Options][1] define a syntax for querying resources. Queries are encoded into a [Query][4] string by combining [Query Options][1] with [Operations][2], [Functions][2], and the name property of the [data][3]. Queries are executed with a HTTP.GET to a URI that understands the OData Query Options syntax.

OData functionality beyond the OData Query Options is out of scope of this extension. Specifically a client that recognizes this extension should only assume server compatibility with [OData Uri Conventions - 4.1. System Query Options][1] through [4.9. Inlinecount System Query Option ($inlinecount)][5]. 

| OData Uri Conventions (out of scope) |
|--------------------------------------|
| Service Root URI                     |
| Resource Path                        |
| Custom Query Options                 |
| Service Operation Parameters         |

odata Extension
===============

odata object
-------------

**Parameters**


| Name | Type | Description                                     | Required |
|------|------|-------------------------------------------------|----------|
| href | URI  | The url cable of decoding OData encoded queries | TRUE     |
| data | data | A collection of data objects                    | FALSE    |

Extension to the [data][3] object
---------------------------------

**Additional Parameter**

| Name          | Type          | Description                                     | Required |
|---------------|---------------|-------------------------------------------------|----------|
| odata-options | odata-option  | Collection of odata-options                     | FALSE    |

odata-option Object
-------------------

**Parameters**

| Name       | Type     | Description                                                                                | Required |
|------------|----------|--------------------------------------------------------------------------------------------|----------|
| option     | String   | The name of an OData Query Option                                                          | TRUE     |
| enabled    | Boolean  | Is the query option allowed by the server for the parent data object                       | TRUE     |
| operations | String[] | An array of OData operations and functions supported by the server                         | FALSE    |
| values     | String[] | A list of available available values expected by the server for this data object           | FALSE    |

The odata-option object should contain the name of an [odata query option][1]. It should also contain a boolean value for the enabled property indicating if the operation is accepted by the href. In the case of the $filter option a list of acceptable operations and functions can be described. In all cases if the odata-options property is not null, all options must be explicitly enabled. All $filter operations must be explicitly enabled or be considered disabled by the client.

If the odata-option is not included the client is free to assume any possible OData Query is acceptable, but will have to execute the query to know for sure.

Example
-------

```javascript
{   
    "odata": {
        "href"="http://example.orf/Examples"
    }
}
```

The URI http://example.orf/Examples is compatible with OData Query Options. No limitations on the OData Query Options are defined. Clients are free to execute any legal OData query against the resource. Clients should however be aware that some queries may result in an error. Clients should expect to handle errors from the server in the form of HTTP error status code or via [Cj Error Extension][6] Extension.

Example Including odata-options
-------------------------------

```javascript
{
  "odata": {
    "href": "http://example.org/Examples",
    "data": [
      {
        "name": "Name",
        "prompt": "Example Name",
        "odata-options": [
            { "option": "$orderby", "enabled": true },
            { "option": "$top","enabled": true },
            { "option": "$skip","enabled": true },
            { "option": "$filter","enabled": true, "operations": [ "Eq", "substringof" ] },
            { "option": "$expand", "enabled": false },
            { "option": "$format", "enabled": false },
            { "option": "$select", "enabled": true },
            { "option": "$inlinecount", "enabled": true }
        ]
      },
      {
        "name": "CreatedDt",
        "prompt": "Date Created",
        "odata-options": [ 
            { "option": "$filter", "allowed": true, "operations": ["Gt", "Lt" ] }
        ]
      }
    ]
  }
}
```

The URI http://example.orf/Examples is compatible with OData Query Options. The server supports: $orderby, $top, $skip, $select, and $inlinecount with the 'Name' data property. \$filter is also enabled when using the operations Eq (Equals) and substringof (Contains). \$expand and $format are not enabled. Client should build OData Query Option queries using only the enabled Query Options, Operations, and Functions. Clients should expect to handle errors from the server in the form of HTTP error status code or via [Cj Error Extension][6] Extension.

Simple System Query Options
---------------------------
 - $orderby
 - $top
 - $skip
 - $expand
 - $format
 - $select
 - $inlinecount

These query options require only enabling or disabling using the odata-option object.

 Complex System Query Options
----------------------------
 - $filter

The server should define valid operations within the operation array. Valid values for the operations are any operation or function as follows:

The available operations to $filter are:

| Operator             | Description           | Example                                             |
|----------------------|-----------------------|-----------------------------------------------------|
| Logical Operators    |                       |                                                     |
| Eq                   | Equal                 | /Suppliers?$filter=Address/City eq ‘Redmond’       |
| Ne                   | Not equal             | /Suppliers?$filter=Address/City ne ‘London’        |
| Gt                   | Greater than          | /Products?$filter=Price gt 20                      |
| Ge                   | Greater than or equal | /Products?$filter=Price ge 10                      |
| Lt                   | Less than             | /Products?$filter=Price lt 20                      |
| Le                   | Less than or equal    | /Products?$filter=Price le 100                     |
| And                  | Logical and           | /Products?$filter=Price le 200 and Price gt 3.5    |
| Or                   | Logical or            | /Products?$filter=Price le 3.5 or Price gt 200     |
| Not                  | Logical negation      | /Products?$filter=not endswith(Description,’milk’) |
| Arithmetic Operators |                       |                                                     |
| Add                  | Addition              | /Products?$filter=Price add 5 gt 10                |
| Sub                  | Subtraction           | /Products?$filter=Price sub 5 gt 10                |
| Mul                  | Multiplication        | /Products?$filter=Price mul 2 gt 2000              |
| Div                  | Division              | /Products?$filter=Price div 2 gt 4                 |
| Mod                  | Modulo                | /Products?$filter=Price mod 2 eq 0                 |
| Grouping Operators   |                       |                                                     |
| ( )                  | Precedence grouping   | /Products?$filter=(Price sub 5) gt 10              |

The functions available to $filter are:

| Function                                               | Example                                                             |
|--------------------------------------------------------|---------------------------------------------------------------------|
| String Functions                                       |                                                                     |
| bool substringof(string po, string p1)                 | ~$filter=substringof(‘Alfreds’, CompanyName) eq true               |
| bool endswith(string p0, string p1)                    | ~$filter=endswith(CompanyName, ‘Futterkiste’) eq true              |
| bool startswith(string p0, string p1)                  | ~$filter=startswith(CompanyName, ‘Alfr’) eq true                   |
| int length(string p0)                                  | ~$filter=length(CompanyName) eq 19                                 |
| int indexof(string p0, string p1)                      | ~$filter=indexof(CompanyName, ‘lfreds’) eq 1                       |
| string replace(string p0, string find, string replace) | ~$filter=replace(CompanyName, ‘ ‘, ”) eq ‘AlfredsFutterkiste’      |
| string substring(string p0, int pos)                   | ~$filter=substring(CompanyName, 1) eq ‘lfreds Futterkiste’         |
| string substring(string p0, int pos, int length)       | ~$filter=substring(CompanyName, 1, 2) eq ‘lf’                      |
| string tolower(string p0)                              | ~$filter=tolower(CompanyName) eq ‘alfreds futterkiste’             |
| string toupper(string p0)                              | ~$filter=toupper(CompanyName) eq ‘ALFREDS FUTTERKISTE’             |
| string trim(string p0)                                 | ~$filter=trim(CompanyName) eq ‘Alfreds Futterkiste’                |
| string concat(string p0, string p1)                    | ~$filter=concat(concat(City, ‘, ‘), Country) eq ‘Berlin, Germany’  |
| Date Functions                                         |                                                                     |
| int day(DateTime p0)                                   | ~$filter=day(BirthDate) eq 8                                       |
| int hour(DateTime p0)                                  | ~$filter=hour(BirthDate) eq 0                                      |
| int minute(DateTime p0)                                | ~$filter=minute(BirthDate) eq 0                                    |
| int month(DateTime p0)                                 | ~$filter=month(BirthDate) eq 12                                    |
| int second(DateTime p0)                                | ~$filter=second(BirthDate) eq 0                                    |
| int year(DateTime p0)                                  | ~$filter=year(BirthDate) eq 1948                                   |
| Math Functions                                         |                                                                     |
| double round(double p0)                                | ~$filter=round(Freight) eq 32                                      |
| decimal round(decimal p0)                              | ~$filter=round(Freight) eq 32                                      |
| double floor(double p0)                                | ~$filter=filter=round(Freight) eq 32                               |
| decimal floor(decimal p0)                              | ~$filter=floor(Freight) eq 32                                      |
| double ceiling(double p0)                              | ~$filter=ceiling(Freight) eq 33                                    |
| decimal ceiling(decimal p0)                            | ~$filter=floor(Freight) eq 33                                      |
| Type Functions                                         |                                                                     |
| bool IsOf(type p0)                                     | ~$filter=isof(‘NorthwindModel.Order’)                              |
| bool IsOf(expression p0, type p1)                      | ~$filter=isof(ShipCountry, ‘Edm.String’)                           |

  [1]: http://www.odata.org/documentation/odata-v2-documentation/uri-conventions/#4_Query_String_Options
  [2]: http://www.odata.org/documentation/odata-v2-documentation/uri-conventions/#45_Filter_System_Query_Option_filter
  [3]: http://amundsen.com/media-types/collection/format/#arrays-data
  [4]: http://tools.ietf.org/html/rfc3986#section-3.4
  [5]: http://www.odata.org/documentation/odata-v2-documentation/uri-conventions/#49_Inlinecount_System_Query_Option_inlinecount
  [6]: https://github.com/mamund/collection-json/blob/master/extensions/errors.md
