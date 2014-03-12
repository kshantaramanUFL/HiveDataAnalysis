Data Analysis of Enron and Netflix Dataset and Visualization Using Hive and Tableau


WHAT ARE THE DATA PROCESSING PIPELINES USED IN THE PROJECT
The data processing pipeline used in the project in the AWS Data Pipeline that helps us to process and 
move data between different the AWS Compute Component, Amazon EMR and AWS Storage Component,Amazon S3.

KINDS OF ANALYTICS AND QUERIES 

a.	NETFLIX
1) Exploratory analytics: This means analyzing the data set to find previously unknown relationships that 
discover new connections which are useful for defining future studies/questions. Example:  Most popular movie of all time, Months where User activity is high
2) Inferential analytics: This indicates analyzing the dataset to test theories about the 
context of the dataset in general based on a sample of subjects taken from the dataset. 
That is, using a relatively small sample of data to say something about a bigger population. Example:  Top 10 highest rated movies of all time
Queries:
Base Table Creation
CREATE EXTERNAL TABLE IF NOT EXISTS titles(mid INT, yearOfRelease INT, title STRING) ROW FORMAT 
DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3://spring-2014-ds/movie_dataset/movie_titles/';

CREATE EXTERNAL TABLE IF NOT EXISTS ratings(mid INT, customer_id INT, rating INT, date STRING) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3://spring-2014-ds/movie_dataset/movie_ratings/';

i.	AVERAGE RATING OF ALL MOVIES RELEASED IN A YEAR.
The average rating of all movies released in the year is calculated by calculating the average rating of each movie.

CREATE EXTERNAL TABLE avg_per_year(yearOfRelease INT, average DOUBLE) ROW FORMAT 
DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3://sayee-proj2/movies/';

INSERT INTO TABLE avg_per_year SELECT titles.yearofrelease, AVG(m_avg .ravg)  AS avg_py
FROM(SELECT ratings.mid, AVG(ratings.rating) AS ravg FROM ratings GROUP BY ratings.mid) m_avg 
JOIN titles on m_avg .mid = titles.mid GROUP BY titles.yearofrelease ORDER BY avg_py DESC;

ii.	TOP 20 POPULAR MOVIES OF ALL TIME
The most popular movies of all times are those movie that have higher rating and those that have been reviewed the most number of times.

CREATE EXTERNAL TABLE most_popular(title STRING, rating DOUBLE, customer INT) ROW FORMAT 
DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3://sayee-proj2/movies/';

INSERT INTO TABLE most_popular select titles.title,movie_sum.mov_avg as average,movie_sum.cust 
from titles JOIN (select ratings.mid,SUM(ratings.rating) as tot,AVG(Ratings.rating) as mov_avg,count(*) as cust 
from ratings group by ratings.mid order by tot desc limit 20) Movie_sum ON titles.mid=movie_sum.mid Order by average desc;

iii.	USER ACTIVITY PER MONTH AND YEAR
This queries returns those months and their corresponding years in which the Netflix forum usage has been high. That is, most number of movies have been rated.

CREATE EXTERNAL TABLE user_activity(month INT, year INT, number INT) ROW FORMAT 
DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3://sayee-proj2/movies/';

INSERT INTO TABLE user_activity select date_format.mm,date_format.yy,count(*) as cou from ratings 
JOIN (select distinct month(ratings.date) as mm,year(ratings.date) as yy from ratings) date_format 
ON month(ratings.date)=date_format.mm and year(ratings.date)=date_format.yy group by date_format.mm,date_format.yy order by cou desc;

iv.	 TOP TEN HIGHEST RATED TITLES OF ALL TIME

This queries the data set to find the top 10 highest rated titles that exist in the dataset

CREATE EXTERNAL TABLE top_ten(title STRING, year INT, rating DOUBLE) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION 's3://sayee-proj2/movies/'; 
INSERT INTO TABLE top_ten SELECT titles.title, titles.yearofrelease, movies.average as aver FROM titles 
JOIN ( SELECT ratings.mid as movid, AVG(ratings.rating) as average FROM ratings GROUP BY ratings.mid ORDER by average DESC LIMIT 10 )  
movies ON movies.movid = titles.mid ORDER BY aver DESC;


v.	TOP 10 MOVIES THAT HAVE BEEN RATED BY THE TOP 10 REVIEWERS

Top Reviewers are those who have written the most number of reviews. This query finds those reviewers and return the average rating of 
the top 10 movie titles reviewed by them.

CREATE EXTERNAL TABLE movies_maxrated(title STRING, average Double,numReviews INT) ROW FORMAT 
DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3://sayee-proj2/movies/'; 
INSERT INTO TABLE movies_maxrated select titles.title,mov_count.rat_avg as rate,Mov_Count.nr from 
titles JOIN (select SUM(ratings.rating) as rev_sum,mid,Avg(Ratings.rating) as rat_avg,count(*) as nr 
from ratings JOIN (select customer_id,count(*) as numReviews from ratings group by customer_id order by numReviews desc limit 10) 
temp ON temp.customer_id=ratings.customer_id group by mid order by rev_sum desc limit 10) Mov_Count ON Mov_count.mid=titles.mid order by rate desc;

b.	ENRON

Descriptive Analytics: This mode of analytics target analyzing the data to describe the main features of the data. 
Example: Most forwarded e-mail, Person who received most number of emails.

Base Table Creation
CREATE EXTERNAL TABLE emails(eid STRING, timestampD STRING, fromPerson STRING, toPerson STRING, cc STRING, subject STRING, context STRING) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION 's3n://spring-2014-ds/enron_dataset/';

i.	10 MOST POPULAR PEOPLE IN THE COMPANY
This queries retrieves from the dataset the person who has received the most number of emails

CREATE TABLE most_emails (toPerson STRING, numCount STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION 's3n://sayee-proj2/emails/q1';

INSERT INTO TABLE most_emails SELECT receiver, COUNT(emails.eid) AS no FROM emails 
LATERAL VIEW EXPLODE(SPLIT(emails.toPerson,',')) exp AS receiver GROUP BY receiver SORT BY no DESC LIMIT 10;

ii.	TOP 10 PEOPLE LISTENING TO MOST NUMBER OF EMAILS

This query returns the employees whose e-Mail IDs are present most number of times in the cc field.

CREATE TABLE many_cc (sender STRING, no STRING) ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3n://sayee-proj2/emails/q2';

INSERT INTO TABLE many_cc SELECT cc_person, COUNT(DISTINCT sender) AS no 
FROM (SELECT emails.fromPerson AS sender, TRIM(Incc) as cc_person 
FROM emails LATERAL VIEW EXPLODE(SPLIT(emails.cc,',')) emailsExploded AS Incc WHERE NOT (incc = '')) cc_tab GROUP BY cc_person SORT BY no DESC LIMIT 10;

iii.	10 PEOPLE WHO HAVE FORWARDED MOST NUMBER OF EMAILS

The query retrieves those people who have forwarded most number of emails.

CREATE TABLE max_fwd (who STRING, no STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3n://sayee-proj2/emails/q3';

INSERT INTO TABLE max_fwd SELECT fromPerson, COUNT(*) AS no FROM emails WHERE subject LIKE 'FW:%' GROUP BY fromPerson SORT BY no DESC LIMIT 10;

iv.	10 PEOPLE WHO FLOOD THE EMAIL SYSTEM

Flooding the system is defined as sending and receiving most number of emails. This query returns those top 10 people 
who have received and sent the most number of emails.

CREATE TABLE max_emails (who STRING, no STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE LOCATION 's3n://sayee-proj2/emails/q4';
INSERT INTO TABLE max_emails SELECT to_tab.fromPerson as who, SUM(to_tab.numTo+cc_tab.numCC) as tot 
FROM (SELECT fromPerson, COUNT(emails.eid) AS numTo FROM emails LATERAL VIEW EXPLODE(SPLIT(emails.toPerson,',')) exp 
AS receiver GROUP BY fromPerson) to_tab JOIN (SELECT fromPerson, COUNT(emails.eid) 
AS numCC FROM emails LATERAL VIEW EXPLODE(SPLIT(emails.cc,',')) exp AS receiver GROUP BY fromPerson) 
cc_tab ON to_tab.fromPerson = cc_tab.fromPerson GROUP BY to_tab.fromPerson SORT BY tot DESC LIMIT 10;

VISUALIZATION USED IN TABLEAU

a.	NETFLIX
The visualizations used for this dataset listed according to query are
Query 1: A scatter plot is plotted with Year on the X Axis and Average Rating on the Y Axis 
and a trend line is shown depicting the quality of movies through the years.
 
Query 2: The results of the query are shown in the form of a tree-map 
with each node denoting a movie name with a value pair of Number of Ratings on the X Axis and Average Ratings on the Y Axis.
 
Query 3: The results are depicted in the form of a bar chart with months on the X Axis grouped according to years and the number of users on the Y Axis.
 
Query 4: The data results are shown in the form a text table categorized by the year in which the 
movie was released thus facilitating the knowledge in which year the top most rated movie was released.
 
Query 5: The data is visualized in the form of Packed Bubbles with the size of each 
bubble directly proportional to the average rating of the movie given by those 10 reviewers.
 
b.	ENRON

The visualizations used for this dataset listed according to query are.

Query 1: Data visualization is done in the form of a pie chart with the 
largest sector in the pie depicting the most popular person. That is, the person involved in most emails.


Query 2: Query output is shown in the form of Horizontal bars and the longest horizontal bar denotes the person most cc’ed to.
 
Query 3: A line graph visualization of the output data is depicted and the highest point in the graph is the person who forwarded the most number of e-mails.
 
Query 4: The visualization for this query is shown in the form of packed bubbles with bubble size 
reflecting a person’s usage of the system. Bigger the bubble, greater the usage.
 

RUNTIME EXPERIENCE FOR QUERIES AND VISUALIZATIONS
Queries ranged from 2-9 Map Reduce Jobs. Queries with more number of aggregations, sorting and joining 
took more time to execute because of the ore number of Map Reduce Jobs. 
Tableau provided many suitable data visualization options the data using graphs, lines, histograms etc.

DIFFICULTIES FACED AND LEARNING
We faced initial difficulties in analyzing the ENRON dataset for the queries as discussed above because of the presence of the ‘,’ operator 
in between each e-mail. However that was overcome by the use LATERAL VIEW ((EXPLODE mentioned in one of the reference links specified in the previous sections.
Other difficulties faced in the project were loading the results of the query data in the S3 bucket. 
After some amount of googling, the use of create table and INSERT INTO TABLE solved the problem.
