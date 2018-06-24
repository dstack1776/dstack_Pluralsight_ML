# Pluralsite ML Coding Challenge
Daniel Stack, dstack1776@gmail.com 

github handle: dstack1776

June 23, 2018

# Instructions

There are two separate Python files. They should be put in the same directory. The first of these is "Pluralsight Class Modules". I have provided it both as a Jupyter Notebook and as a standard Python file. The function main makes a number of calls to parse the csv files, calculate similarity, and create an sqlite database "dstack.db" that stores the results in the table distance_matrix, with three columns - src_usr, dst_usr, and distance. This takes several minutes to complete - there's some optimizations I'd consider (and will discuss below) were this not a model but rather a much larger dataset.

The second file is rest_dstack.py that serves as a RESTful API interface. Like the first program, it is a toy with basic functionality. In this case is is listening to 127.0.0.1:5002. There are two get commands, users and usersn, detailed below.

The folder "data_files_ml_engineer" should be within the folder with these Python files. (i.e. as a sub-folder of the cloned Pluralsight-ML.)

## users
"users" has a single argument, the user handle, which ranges from 1 to 10000, inclusive. It returns a dictionary of the ten nearest user handles (not including itself) and the distance, which ranges from 0 (identical) to 1 (nothing in common). For example, 127.0.0.1:5002/users/9999 will return the ten nearest handles to user handle 9999 as well as the distances from 9999. I ran this program straight out of IDLE. It needs to be run after "Pluralsight Class Modules" as it does a simple sql select from dstack.db.

Example output for 127.0.0.1:5002/users/9999

```python
{"1562": 0.0, "5238": 0.0, "6303": 0.0, "4484": 0.5, "4432": 0.5196152422706632, "906": 0.5204164998665332, "2200": 0.5443310539518175, "3605": 0.5446711546122731, "5071": 0.5608545472127794, "453": 0.5773502691896258}
```

## usersn
"usersn" has two arguments, id and num. id is used for the user handle while num indicates how many handles should be returned, in ascending order, starting with the closest.

Example output for http://127.0.0.1:5002/usersn?id=9999&num=15
```python
{"1562": 0.0, "5238": 0.0, "6303": 0.0, "4484": 0.5, "4432": 0.5196152422706632, "906": 0.5204164998665332, "2200": 0.5443310539518175, "3605": 0.5446711546122731, "5071": 0.5608545472127794, "453": 0.5773502691896258, "637": 0.5773502691896258, "2754": 0.5773502691896258, "5411": 0.5773502691896258, "6188": 0.5773502691896258, "7688": 0.5773502691896258}
```


# Similarity Calculation
## Aborted Effort
When I first looked into this problem, I had initially considered using cosine similarity for the User Assessments. With all of the assessments having numeric values it seemed a reasonable mechanism. However, this entailed creating a column for every assessment type and the code to an **extremely** long time to run, even with this limited dataset. Moreover, it became clear that even were the performance model to be overcome, it was not a good model. There were users who took no assessments - which made them identical to other users who took no assessments, as I needed to give them some numeric value. In the end, this did not appear a good model to use.

## Three-Axis Jaccard Distance
Instead, I made use of three axes of Jaccard distance. I set up an axis for assessments, classes examined, and self-described interests.

Jaccard similarity is used to compare sets. It is the cardinality of the intersection of two sets divided by the union of the two sets. I added some code to make the similarity between empty sets as 0 (so as to avoid divide by zero error). Given the most similar (identical) would have a 1 under this model, I subtracted the result from 1 to get a distance.

I settled on making use of a set compare given the wide variety of interests, courses, and assessments. This avoided hoops I ran into when attempting to make use of cosine similarity.

### Assessments
One thing I did not want to lose on assessments were the scores. I wanted similarity between people who took assessments in a given are but I wanted even greater similarity if their level of experience were similar. To do this I added three possible tags to every assessment - Novice, Proficient, and Expert. Someone who tested as a Novice in Python, for example, would have the entry 'Python-Novice' added to their assessment set. Someone who tested as an Expert in Python have 'Python-Novice', 'Python-Proficient', and 'Python-Expert; added to their assessment set. In this model, two Python Experts would have a strong level of similarity given they would have three matching entries in their sets. Similarly a Python Novice and Python Expert would still have some level of similarity. I used benchmarks of 100 and 200 to show Proficiency and Expertise, aligning with Pluralsight's testing algorithms.

### Interests
Interests were the most straightforward as they were simple binaries - a user was either interested in something or not. I added interests into a user's Interest set.

### Classes
Given the variety of courses, I took advantage of the course tags to simplify areas of interest into broader categories - for each user, instead of using the course name I instead mapped in the tags from the separate csv file, using logic akin to a SQL left join. Like the Assessments, I took advantage of the difficulty of the course. I did not use the time spent on a class's webpage, an area of potential enhancement - perhaps using it to measure an intensity of interest - or to possibly discard entries that a user spent an extremely brief amount of time on.

## Combining the Three Axes
Since I calculated three separate distance axes, I combined them using a simple application of the Pythagorean Theorem, and to keep the result scaled from 0 to 1, I divided this result by the square root of 3. 


# Considerations at Larger Scale

I was able to take advantage of storing a lot of data in memory. Indeed, some of my initial efforts involved building a Jaccard Distance table for everything. It would have been possible to go with this model and not used a database table at all. However, the database table has as an advantage it can be built over time and in pieces. For example, I built the SQL table for one user at a time, comparing it with all potential other users in one 1D array. This was a large data structure - at a larger scale it might be reasonable (or even necessary) to build it in smaller chunks. Conversely, I performed and committed the SQL writes one at a time - by use of transactions - http://bytes.schibsted.com/speeding-up-sqlite-insert-operations/ - it becomes possible to speed up that process. 

To speed up calculations it is possible to take advantage of the distance between a source and destination is commutative - you only need to calculate it once.

# Other Thoughts

One thing I would have liked would have been some information as to what sorts of assessments, class tags, and interests are similar to one another. Domain knowledge could have been used to make such decisions, likely codifying it with a CSV file similar to the classes' tag file.  For example, Pluralsight's webpage breaks the assessments into Development, IT Ops, Data, Security, Creative. These may have been data points appropriate for similarity calculations.

The REST interface is fairly basic and a good opportunity for improvement. This was my first exercise at creating a REST interface - since on my interview with Connor I spoke how I have talent at developing new skills, it seemed an absolutely fair challenge for me to illustrate my capacity to do just that.