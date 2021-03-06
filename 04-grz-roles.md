### 04. About roles

##### 01. What is the role distribution of person mentions? (api:04_roles_01_role_distrib_per_pm)
```sparql
#+ tags:
#+   - roles

SELECT (STRAFTER(STR(?role), "#") AS ?role) ?numberOfMentions (?numberOfMentions*100/?total AS ?percent)
WHERE
{
  { SELECT (COUNT (distinct ?pm) AS ?total) WHERE {?pm a common:PersonMention.} }

  { SELECT ?role (COUNT (distinct ?pm) AS ?numberOfMentions)
    WHERE {?pm a common:PersonMention; grz-owl:hasRole ?role .}
  }
}
GROUP BY ?role ?numberOfMentions ?total
ORDER BY DESC(?percent)
```

##### 02. What is the total number of person entities having role x ? (api:04_roles_02_nb_pe_with_role_x)
```sparql
#+ tags:
#+   - roles
# param: ?_role

SELECT (COUNT (distinct ?pe) AS ?NbEntity)
WHERE
{ ?pe a common:Person ; grz-owl:hasRole ?role . ?role rdf:value grz-owl:Apprentice .}
```

##### 03. What is the total number of person entities having role x, with time window? (api:04_roles_03_nb_pe_with_role_x_withTW)
```sparql
#+ tags:
#+   - roles
# param: ?_date_start ?_date_end ?_role

SELECT ?year (COUNT (distinct ?pe) AS ?NbApprenticeEntity)
WHERE
{
  ?pe a common:Person ; grz-owl:hasRole ?roleStmt.
  ?roleStmt sem:hasTimeStamp ?date ; rdf:value grz-owl:Apprentice .
  BIND(IF(?date = "0"^^<http://www.w3.org/2001/XMLSchema#gYear>,"NO DATE", xsd:dateTime(?date) ) AS ?myDate)
  BIND(IF(?myDate != "NO DATE", year(?myDate), "NODATE") AS ?year)
   FILTER (?year > ?_date_start)
   FILTER (?year < ?_date_end)
}
GROUP BY ?year
ORDER BY ASC (?year)
```

##### 04. What is the role distribution per gender (on entities)? (api:04_roles_04_role_distrib_per_gender)
```sparql
#+ tags:
#+   - roles

SELECT ?role ?countWomen ?countMen
WHERE
{
  {SELECT ?role (COUNT (distinct ?women) AS ?countWomen)
    WHERE {?women a common:Person; foaf:gender "female"; grz-owl:hasRole/rdf:value ?role .}}

  {SELECT ?role (COUNT (distinct ?men) AS ?countMen)
    WHERE {?men a common:Person; foaf:gender "male"; grz-owl:hasRole/rdf:value ?role .}}
}
GROUP BY ?role
```

##### 05. Role distribution: Give me an overview of all and per gender (api:04_roles_05_role_distrib_overview)
```sparql
#+ tags:
#+   - roles

SELECT
?role
?countMentions (?countMentions*100/?total AS ?percentAll)
?countWomen (?countWomen*100/?totalWomen AS ?percentWomenReRole)
?countMen (?countMen*100/?totalMen As ?percentMenReRole)
WHERE
{
  { SELECT (COUNT (distinct ?pm) AS ?total) WHERE {?pm a common:PersonMention ; foaf:gender ?g. } }
  { SELECT (COUNT (distinct ?pm) AS ?totalMen) WHERE {?pm a common:PersonMention; foaf:gender "male".} }
  { SELECT (COUNT (distinct ?pm) AS ?totalWomen) WHERE {?pm a common:PersonMention; foaf:gender "female".} }

  { SELECT ?role (COUNT (distinct ?pm) AS ?countMentions)
    WHERE {?pm a common:PersonMention; grz-owl:hasRole ?role .}}

  {SELECT ?role (COUNT (distinct ?women) AS ?countWomen)
    WHERE {?women a common:PersonMention; foaf:gender "female"; grz-owl:hasRole ?role .}}

  {SELECT ?role (COUNT (distinct ?men) AS ?countMen)
    WHERE {?men a common:PersonMention; foaf:gender "male"; grz-owl:hasRole ?role .}}
}
GROUP BY ?role ?countMentions ?total
```

##### 06. Give me all person entities who had 2 different roles (e.g. master & apprentice). (api:04_roles_06_pe_with_doubleRole)
```sparql
#+ tags:
#+   - roles
#+ params: ?_role1 ?_role2

# N.B: roles can be changed/added and more information can be asked about the person entity.

SELECT ?pe (COUNT (distinct ?pm) AS ?nbMentions)
WHERE
{
  ?pe  a common:Person ; grz-owl:hasRole ?role. ?role rdf:value grz-owl:Apprentice ; rdf:value grz-owl:Master .
  ?pe core:referredBy ?pm .
}
GROUP BY ?pe
HAVING(?nbMentions > 1)
ORDER BY DESC (?nbMentions)
```

##### 07. How many entities have different roles? (api:04_roles_07_nb_pe_with_doubleRole)
```sparql
#+ tags:
#+   - roles
#+ params: ?_role1 ?_role2

# N.B: roles can be changed/added

SELECT (COUNT (distinct ?pe) AS ?NbPe)
WHERE
{
  ?pe  a common:Person ; grz-owl:hasRole ?role . ?role rdf:value grz-owl:Apprentice ; rdf:value grz-owl:Master .
}
```

##### 08. Give me the details (person/role/date) for person entities having both apprentice and master roles. (api:04_roles_08_details_pe_with_doubleRole)
```sparql
#+ tags:
#+   - roles

# N.B.: more role can be added (e.g. guarantor)
# For counting, replace the first line by SELECT COUNT distinct ?pe

SELECT ?pe ?peName ?roleType ?year
WHERE
  {
    {
       ?pe  grz-owl:hasRole ?roleStatement; rdfs:label ?peName .
       ?roleStatement rdf:value ?roleType ; sem:hasTimeStamp ?date .
       BIND(IF(?date = "0"^^<http://www.w3.org/2001/XMLSchema#gYear>,"NO DATE", xsd:dateTime(?date) ) AS ?myDate)
       BIND(IF(?myDate != "NO DATE", year(?myDate), "NODATE") AS ?year)
    }
    {

     SELECT ?pe
     WHERE {?pe a common:Person ; grz-owl:hasRole ?role . ?roel rdf:value grz-owl:Master ; rdf:value grz-owl:Apprentice .}
      GROUP BY ?pe
     }
  }
GROUP BY ?roleType
ORDER BY ASC (UCASE(str(?pe))) ?year
```

##### 09. Give me the apprentices who have the same guarantor in 2 different contracts. (api:04_roles_09_app_with_sameGuar_across_contracts)
```sparql
#+ tags:
#+   - roles

SELECT ?app ?appName ?guar ?guarName (?contract1 AS ?contract) (?date1 AS ?date)
WHERE
{
  ?guar a common:Person ; grz-owl:hasRole ?role ; rdfs:label ?guarName . ?role rdf:value grz-owl:Guarantor  .
  ?app a common:Person ; grz-owl:hasRole ?role ; rdfs:label ?appName. ?role rdf:value grz-owl:Apprentice   .
  ?app common:hasGuarantor ?rel. ?rel rdf:value ?guar .
  ?app core:referredBy ?ment1, ?ment2 .
  ?ment1 core:isMentionedIn ?contract1  .
  ?ment2 core:isMentionedIn ?contract2 .
  ?contract1 sem:hasTimeStamp ?date1 .
  ?contract2 sem:hasTimeStamp ?date2 .
  FILTER (?contract1 != ?contract2)
  FILTER (?ment1 != ?ment2)
}
GROUP BY ?app ?guar ?guarName ?appName
ORDER BY ?app
```

##### 10. TO BE REVISED WITH PROFESSION THESAURUS - Number of guarantor per contract given a profession. (api:04_roles_10_nb_guar_per_contract_with_prof_x)
```sparql
#+ tags:
#+   - roles

SELECT ?numberOfGuar (COUNT (distinct ?app) AS ?NbApp)
WHERE
{
  SELECT ?app (COUNT (distinct ?guar) AS ?numberOfGuar)
  WHERE
  {
    ?guar a grz-owl:Person .
    ?app a grz-owl:Person .
    ?master a grz-owl:Person .
    ?guar  grz-owl:role ?role . ?role rdfs:value grz-owl:Guarantor .
    ?master  grz-owl:role ?role . ?role rdfs:value  grz-owl:Master .
    ?app  grz-owl:role ?role . ?role rdfs:value  grz-owl:Apprentice .
    ?app grz-owl:has_master/grz-owl:value ?master .
    ?app grz-owl:has_guarantor/grz-owl:value ?guar .
    ?master grz-owl:profession/grz-owl:value/grz-owl:professionCategory "stampa" .
  }
  GROUP BY ?app
}
GROUP BY ?numberOfGuar
ORDER BY ASC (?numberOfGuar)
```


