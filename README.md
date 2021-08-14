# Stackoverflow data and Neo4j
After exploring and analysing the [yelp dataset](https://www.yelp.com/dataset), I was interested to play around with other big datasets. After some googling, I found the [Stack Overflow dataset](https://archive.org/details/stackexchange) from the Internet archive.
I downloaded the following dump files using the torrent link:  
- stackoverflow.com-Posts.7z	- 16.58 GB
- stackoverflow.com-Users.7z	- 857.7 KB
- stackoverflow.com-Tags.7z	- 733.0 MB

1. Unzip the .7z Files
	<pre>for i in *.7z; do 7za -y -oextracted x $i; done</pre>
2. 3 xml files were extracted:
	- Posts.xml	- 90.21GB
	- Tags.xml	- 5.5 MB
	- Users.xml	- 5.02 GB
3. Initial graph model visualised using [arrows.app](https://arrows.app/)  
![image](https://user-images.githubusercontent.com/830693/129433164-10db4ced-cb96-4466-b7e0-07d79be111dc.png)
4. Convert to csv format for importing using the script provided [python3 to_csv.py extracted]:
	- 53086330 posts records processed in 67mins
	- 14839629 users records processed in 10mins
	- 61061 tag records processed

	
<pre>./bin/neo4j-admin import --multiline-fields=true  --skip-bad-relationships \
--nodes=Post=./import/sof/posts.csv  \
--nodes=User=./import/sof/users.csv  \
--nodes=Tag=./import/sof/tags.csv  \
--relationships=PARENT_OF=./import/sof/posts_rel.csv \
--relationships=HAS_TAG=./import/sof/tags_posts_rel.csv \
--relationships=POSTED=./import/sof/users_posts_rel.csv \
--database=sof
</pre>
  
5. The import took 5mins on my desktop
<pre>IMPORT DONE in 5m 2s 17ms. 
Imported:
  67987014 nodes
  147194601 relationships
  403463278 properties
Peak memory usage: 1.756GiB</pre>
![image](https://user-images.githubusercontent.com/830693/129184861-ffa84c61-98c6-4aee-a786-a4c871d04acd.png)

## Exploring the data
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
