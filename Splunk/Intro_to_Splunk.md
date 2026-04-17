# INtro top SPlunk Course 


What is splunk 
Running searches


Splunk Unified data platform


at the heart of Splunk is the index, which contain the data form all sorts of input. THe inspectors go to work processing the dat and labelinig it with a Sourc etype. Time stamps are normallised for consistence.



# Useintth splunk web interface


3 roles - admin power and the user 
- `Admin` is the most powerful. Can install apps , injest dat and create knowledge objects for all users
=- `power` role can craate and sjare knowldege objeject of all users of an app, and do real time object . ONly see own and shared Knowldge object. 



Get apps fro msplunkbase to get new apps or build your own


Search and reporting apps interface


App bar allows you to navigate hte aplication


# Searching 

Click the `Data Summary` you get  break donw by Hosts, Source and SOurce type

Source types ( classification of Dta ) 
- File Path
- Host, ip , fqdn 
- Port

## Search syntax (SPL)

( iS 7 days is a good default time frame ?? ) 

Save 
Search result tabs 
Timeline 


- `Save As` Allos us to save the results as a `knowldge object`


- `Transforming Commands` are commands that create statistics and visualizations. transform dat into data tables.


Search action allow:
- edidt , inspect and deleted


Search jobs last for ten mins ( 7 days if shared) 

Export in Raw, CSV, json


### Three Search Modes
- Fast  ( feild discovery is disabled , only useing default fields)
- Smart ) toggle behavious based on search type
- Verbose



## Exploreing Events 

GQ: Can a search read files for look for the word ".pem" in a file 

Time is normallised but the timestamp is based on the time zone set in ones user account


Highlight text in the searhc result and then we can right cloickk on it for more features.


## USeing Searhc Terms

- Search terms are not case sensitive
- `*` wildcard
- Boolean Order of Evaliuation : `NOT` -> `OR` -> `AND` 
- Parenthesis can be used to set presidents/Nesting `failed NOT (success OR accepted)` 
- quotes can be used for terms `"failed something or other"`
- backslash can be used to escape quotes `\"failed something or other\"`



## Whata rw commands ?

Built from 5 componants :
- searrhc terms
= commands
- functions 
- argumants ) variables
- clauses ( for grouping or deifined)

for example the following search: 

```spl
index=network sourcetype-cisco_wsa_squid usage=Violoation | stats count(usage) as Visits
```

- Search term : `index=network sourcetype-cisco_wsa_squid usage=Violoation` 
- Command : `stats` (-i)
- Function : `count()` (-i)
- Argument : `usage`
- Clause : `as` (-i)
- Field : `Visits` 

separated by a pipie `|` 

We could then extend this search based on declared result to find users who visited "Violation" pages, more than once. (`1>`)


```spl
index=network sourcetype-cisco_wsa_squid usage=Violoation | stats count(usage) as Visits by cs_username | search Visits >1
```

Most Powerful Default Fields ( extracted at index time )
- `Time`
- `index`
- `source`
- `host`
- `sourcetype`

Tips 
- Term search "inclusion" is best 
- Use `OR` when possible ( instead of wildcards)
- Filter early in the pipline


## Knowldge objects 
5 categories 
- Data Interpretation
- Data Classification
- Data Enrichment 
- Data Models
- Data Normalization

KO are useful becasue they can be shared, saved, used by multiple people. 

Knowledge managers Responsibilities
- Oversee KO Creation
- Normalize Event Data
- Implement naming conventions
- Create Data Models





## Knowldge objects 
Tools to to discover and analyze our data. Can be created bu one user and shared with others, Saved, reused bby multiple people or in multiple apps. They can be used in a search.

## 5 categories of knowledge objects  

#### Data Interpretation
Some fields are automatically extracted from data based on the source type but additional fields can be manually extracted (`action` ,`catagoryID` ,`clientip` ,`product_name`) but additnal fields can be extracted `Feild Extractions` for additianl insight. `Calcualted fields` are added to events at search times based on the values of existing fields.


#### Data Classification
`Event Types` allow you catagorise events based opn search terms . `Transactions` are conceptually related events accross time
#### Data Enrichment 

`Lookups` allow you to ad other fields and values to events , not include in the eindexed data. 
`Workflow actions` aloows to create links within event that interact with external resource or narrow our search.
#### Data Models
`Hierarchical structed datasets` which may consist of `events`, `searches` and or `transactions`
#### Data Normalization
`Tags` allow us to designate descriptive names for _key:value_ pairs. They enable us to search for events whihc contain particular feild values . THink of them as labels for the fata. 
`Field Aliases` give us a wayty to normalize data over multiple resopuirces.  We can assign 1 or more allias to any extracted feild . And aply fields from a look up tab;e. 
