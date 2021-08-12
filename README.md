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
![image](https://user-images.githubusercontent.com/830693/129159808-fda852b0-8fc0-4b7d-a122-054770498866.png)
4. Convert to csv format for importing using the script provided:
	- 53086330 posts records processed in 67mins
	- 14839629 users records processed in 10mins
	- 61061 tag records processed

	
<pre>./bin/neo4j-admin import --multiline-fields=true  \
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
