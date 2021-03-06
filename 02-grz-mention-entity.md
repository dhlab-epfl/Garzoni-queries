### 02. About person mentions and person entities

##### 01. What is the total number of person mentions? (api:02_mentions_01_pm_total)
```sparql
#+ tags:
#+   - mentions

SELECT ( COUNT (distinct ?pm) AS ?NbPersMention )
WHERE { ?pm a common:PersonMention .}
```

##### 02. What is the total number of person entities? (api:02_mentions_02_pe_total)
```sparql
#+ tags:
#+   - number

SELECT ( COUNT (distinct ?pe) AS ?NbPersEntities )
WHERE { ?pe a common:Person .}
```

##### 03. How many person entities have how many mentions? (api:02_mentions_03_pe_pm_proportions)
```sparql
#+ tags:
#+   - mentions

SELECT ( COUNT (distinct ?pe) AS ?NbPersonEntities) ?mentionCount
WHERE
{
	SELECT ?pe (COUNT (distinct ?pm)  AS ?mentionCount)
	WHERE
	{
	    ?pm a common:PersonMention ; core:refersTo ?pe .
	    ?pe a common:Person .
	}
	GROUP BY ?pe
}
GROUP BY ?mentionCount
ORDER BY ASC(?mentionCount)
```

##### 04. What is the average number of mentions per person entity? (api:02_mentions_04_avg_pm_per_pe)
```sparql
#+ tags:
#+   - mentions

SELECT (AVG (?mentionCount) AS ?AvgPersonMention)
WHERE
{
    SELECT ?pe (COUNT (distinct ?pm) AS ?mentionCount)
    WHERE
    {
        ?pm a common:PersonMention ; core:refersTo ?pe .
        ?pe a common:Person .
    }
    GROUP BY ?pe
}
```

##### 05. How many person entities have more than x mentions? (api:02_mentions_05_count_pe_with_more_x_mentions)
```sparql
#+ tags:
#+   - mentions

SELECT (COUNT (distinct ?pe) AS ?NbPe)
WHERE
{
   SELECT ?pe
   WHERE
    {
         ?pm a common:PersonMention ; core:refersTo ?pe .
         ?pe a common:Person .
    }
    GROUP BY ?pe
    HAVING (COUNT (distinct ?pm) > 1)
}
```

##### 06.  Give me all mentions of person entity with id x. (api:02_mentions_06_get_pm_of_pe_with_id)
```sparql
#+ tags:
#+   - mentions

SELECT ?pm
WHERE
{
     ?pe a common:Person ; core:referredBy ?pm ; dhc:uuid '76b3eb35-6b08-4a51-adfe-13495667ca53' .
     ?pm a common:PersonMention
}
```

##### 07. Give me all mentions of person entity with name x. (api:02_mentions_07_get_pm_of_pe_with_name)
```sparql
#+ tags:
#+   - mentions

SELECT ?pm
WHERE
{
     ?pe a common:Person ; core:referredBy ?pm ; rdfs:label 'Marsilio Mesture (spezier)' .
     ?pm a common:PersonMention
}
```

##### 08. Who are the 10 most mentioned person entities? (api:02_mentions_08_get_10_most_mentioned_pe)
```sparql
#+ tags:
#+   - mentions

SELECT ?pe ( COUNT (distinct ?pm) AS ?NbMentions )
WHERE
{
     ?pm a common:PersonMention ; core:refersTo ?pe .
     ?pe a common:Person .
}
GROUP BY ?pe
ORDER BY DESC (COUNT (distinct ?pm))
LIMIT 10
```

##### 09. Get the top-x most mentioned person entities with time window. (api:02_mentions_09_get_10_most_mentioned_pe_withTW)
```sparql
#+ tags:
#+	- mentions
#+ params: ?_topX ?_date_start ?_date_end

SELECT ?pe ( COUNT (distinct ?pm) AS ?NbMentions )
WHERE
{
     ?pm a common:PersonMention ; core:refersTo ?pe .
     ?pe a common:Person .
     ?pm core:isMentionedIn ?c .
     ?c sem:hasTimeStamp ?date .
     BIND(IF(?date = "0"^^<http://www.w3.org/2001/XMLSchema#gYear>,"NO DATE", xsd:dateTime(?date) ) AS ?myDate)
     FILTER (?myDate < '1653-03-15'^^xsd:dateTime)
     FILTER (?myDate > '1630-03-15'^^xsd:dateTime)
}
GROUP BY (?pe)
ORDER BY DESC(?NbMentions)
LIMIT ?_topX
```

##### 10. Who are the person entities mentioned more than X time? (api:02_mentions_10_get_pe_mentioned_more_than)
```sparql
#+ tags:
#+	- mentions
#+ params: ?_times

SELECT ?pe (COUNT (distinct ?pm) AS ?countMention)
WHERE
{
	?pm a common:PersonMention ; core:refersTo ?pe .
	?pe a common:Person .
}
GROUP BY ?pe
HAVING (count(distinct ?pm) > ?_times)
ORDER BY DESC (count(distinct ?pm))
```

##### 11. What is the distribution of roles of person entities mentioned more than X times? (api:02_mentions_11_get_role_distrib_of_pe_mentioned_more_than)
```sparql
#+ tags:
#+	- mentions
#+ params: ?_times

SELECT  ?roleType (COUNT (distinct ?pe) AS ?NbPe)
WHERE
{
	{
	    SELECT ?pe WHERE
	    { ?pm a common:PersonMention ; core:refersTo ?pe . }
	    GROUP BY ?pe
	    HAVING (count(distinct ?pm) > ?_times)
	}
	?pe grz-owl:hasRole/rdf:value ?roleType .
}
GROUP BY ?roleType
```

##### 12. Get number of mentions per person entity (listing all entities) (api:02_mentions_12_get_nb_pm_per_pe)
```sparql
#+ tags:
#+	- mentions

SELECT ?pe (COUNT (distinct ?pm) AS ?NbPm )
WHERE
{
    ?pm core:refersTo ?pe .
    ?pm a common:PersonMention .
    ?pe a common:Person .
}
GROUP BY ?pe
ORDER BY DESC (COUNT (distinct ?pm))
```

