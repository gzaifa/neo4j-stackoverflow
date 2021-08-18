# Stackoverflow data and Neo4j
After exploring and analysing the [yelp dataset](https://www.yelp.com/dataset), I was interested to play around with other big datasets. After some googling, I found the [Stack Overflow dataset](https://archive.org/details/stackexchange) from the Internet archive.
I downloaded the following dump files using the torrent link:  
- stackoverflow.com-Posts.7z	- 16.58 GB
- stackoverflow.com-Users.7z	- 857.7 KB
- stackoverflow.com-Tags.7z	- 733.0 MB

1. Unzip the .7z Files
	```shell
	for i in *.7z; do 7za -y -oextracted x $i; done
	```
	
2. 3 xml files were extracted:
	- Posts.xml	- 90.21GB
	- Tags.xml	- 5.5 MB
	- Users.xml	- 5.02 GB
3. Initial graph model visualised using [arrows.app](https://arrows.app/)  
![image](https://user-images.githubusercontent.com/830693/129439317-64409ef9-dd68-4ac9-9a4e-34a231ed7108.png)
4. Convert to csv format for importing using the script provided [python3 to_csv.py extracted]:
	- 53086330 posts records processed in 67mins
	- 14839629 users records processed in 10mins
	- 61061 tag records processed

	
```shell
./bin/neo4j-admin import --multiline-fields=true  --skip-bad-relationships \
--nodes=Post=./import/sof/posts.csv  \
--nodes=User=./import/sof/users.csv  \
--nodes=Tag=./import/sof/tags.csv  \
--nodes=PostType=./import/sof/posttypes.csv \
--relationships=PARENT_OF=./import/sof/posts_rel.csv \
--relationships=HAS_TAG=./import/sof/tags_posts_rel.csv \
--relationships=POSTED=./import/sof/users_posts_rel.csv \
--relationships=IS_TYPE=./import/sof/posts_type_rel.csv \
--database=sof
```
  
5. The import took 5mins on my desktop
<pre>IMPORT DONE in 5m 2s 17ms. 
Imported:
  67987014 nodes
  147194601 relationships
  403463278 properties
Peak memory usage: 1.756GiB</pre>
![image](https://user-images.githubusercontent.com/830693/129184861-ffa84c61-98c6-4aee-a786-a4c871d04acd.png)

## Exploring the data
### Types of post
What is the breakdown of all the posts?
<pre>MATCH(p:Post)-[:IS_TYPE]->(t:PostType)
RETURN t.type, count(*) as numPosts</pre>  
|"t.type"              |"numPosts"|
|:---|---:|
|"Question"            |21286479  |
|"Answer"              |31692495  |
|"Orphaned tag wiki"   |167       |
|"Tag wiki excerpt"    |53423     |
|"Tag wiki"            |53423     |
|"Moderator nomination"|334       |
|"Wiki placeholder"    |5         |
|"Privilege wiki"      |2         |  
  
### Find the Top 10 Users
Both the Cypher commands can find the Top 10 Users, but one is faster than the other. Use EXPLAIN to see why.
|Cypher|Operation time|
|:---|:---|
|MATCH (u:User)-[p:POSTED]->(:Post) RETURN u.userId, u.displayname, count(p) as numposts ORDER BY numposts DESC LIMIT 10|Started streaming 10 records after 16 ms and completed after 72087 ms|
|MATCH (u:User) WITH u, size((u)-[:POSTED]->()) as posts ORDER BY posts desc LIMIT 10 RETURN u.userId, u.displayname, posts|Started streaming 10 records after 1 ms and completed after 6861 ms.|  

But I digress, the results of both commands:  
|"u.userId"|"u.displayname" |"numposts"|
|---:|:---|---:|
|"1144035" |"Gordon Linoff" |81683     |
|"22656"   |"Jon Skeet"     |35308     |
|"3732271" |"akrun"         |29962     |
|"1491895" |"Barmar"        |26590     |
|"6309"    |"VonC"          |25847     |
|"2901002" |"jezrael"       |25089     |
|"548225"  |"anubhava"      |24191     |
|"115145"  |"CommonsWare"   |22354     |
|"19068"   |"Quentin"       |22332     |
|"29407"   |"Darin Dimitrov"|21490     |

### What kind of posts do the top user posts?
Seems like Gordon Linoff is quite the expert at databases and SQL. He has only answered questions and have not started a question thread himself!
<pre>MATCH (u:User {userId: '1144035'})-[:POSTED]-(answers:Post)<-[:PARENT_OF]-(q:Post)-[:HAS_TAG]->(t:Tag)
RETURN t.tagId, count(distinct answers) as numAnswers
ORDER BY numAnswers DESC LIMIT 10
</pre>  
|"t.tagId"        |"numAnswers"|
|:---|---:|
|"sql"            |72822       |
|"mysql"          |24294       |
|"sql-server"     |20309       |
|"oracle"         |8436        |
|"postgresql"     |7322        |
|"tsql"           |5205        |
|"database"       |4604        |
|"join"           |3421        |
|"php"            |3370        |
|"sql-server-2008"|2517        |
  
### Top answerers in Neo4j
<pre>MATCH (t:Tag {tagId:'neo4j'})<-[:HAS_TAG]-()
       -[:PARENT_OF]->()<-[:POSTED]-(u:User) 
WITH u, count(*) AS freq ORDER BY freq DESC LIMIT 10
RETURN u.displayname,freq</pre>
|"u.displayname"       |"freq"|
|:---|---:|
|"cybersam"            |2253  |
|"Michael Hunger"      |1826  |
|"InverseFalcon"       |1316  |
|"Stefan Armbruster"   |981   |
|"Luanne"              |554   |
|"Christophe Willemsen"|548   |
|"stdob--"             |386   |
|"Brian Underwood"     |374   |
|"Bruno Peres"         |362   |
|"logisima"            |348   |

And other categories that they are active in
<pre>
MATCH (neo:Tag {tagId:"neo4j"})<-[:HAS_TAG]-()
      -[:PARENT_OF]->()<-[:POSTED]-(u:User) 
WITH neo,u, count(*) as freq order by freq desc limit 10
MATCH (u)-[:POSTED]->()<-[:PARENT_OF]-(p)-[:HAS_TAG]->(other:Tag)
WHERE NOT (p)-[:HAS_TAG]->(neo)
WITH u,other,count(*) AS freq2 ORDER BY freq2 DESC 
RETURN u.displayname, collect(distinct other.tagId)[1..5] AS tags
</pre>  
|"u.displayname"       |"tags"                                                |
|:---|:---|
|"stdob--"             |["node.js","express","three.js","mapbox-gl-js"]       |
|"cybersam"            |["java","javascript","node.js","arrays"]              |
|"Stefan Armbruster"   |["groovy","intellij-idea","grails-plugin","tomcat"]   |
|"Brian Underwood"     |["neo4j.rb","ruby","ruby-on-rails-3","activerecord"]  |
|"Bruno Peres"         |["cordova","angularjs","android","ionic-framework"]   |
|"Luanne"              |["spring-data-neo4j","java","spring","neo4j-ogm"]     |
|"Christophe Willemsen"|["php","doctrine-orm","spring-boot","java"]           |
|"logisima"            |["javascript","graph","java","playframework"]         |
|"Michael Hunger"      |["spring-data-neo4j","java","nosql","graph-databases"]|
|"InverseFalcon"       |["java","enterprise","web-applications","graph"]      |  

