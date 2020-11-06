_Please provide an example of your past work that is the most relevant to this job. We'd like to be able to see a database schema that you've previously designed. 	Please provide context for the design choices that you've made, so that we can understand why you designed the way you did._

I am going to provide you the database schema I developed to sustain data's needs of the Saas I am currently building (https://www.jeifai.com).
The core of the tool is a web scrapers build using Golang, targeting companies' career pages (currentely 470 tech companies are monitored).

Usually when I design a data schema, I spend a lot of time thinking how I would the queries look like.
After I write a prototype query, I design the schema which would allow me to run such SQL code.
Queries have to be beautiful, easy to read and easy to maintain.

Data architecture is divided in three sections: companies, users and matches.

# Companies
Every company has one entry both in table "targets" as well as into "scrapers".
When a new scraping "wave" starts, an entry in the table "scrapings" is generated.
The results of the scrapers are stored into a table called "results".
To get the jobs available for a particular company I can write:
```sql
SELECT
	ss.name
	r.title,
	r.location,
	r.url
FROM results r
LEFT JOIN scrapings s ON(r.scrapingid = s.id)
LEFT JOIN scrapers ss ON(s.scraperid = ss.id)
WHERE ss.name = 'Google'
AND r.updatedat::DATE = current_date;
```
I also scrape Linkedin's data for each company. This data are stored in a table called "linkedin".

# Users
The Saas allows each user to select a list of favourite companies, as well as favourite keywords (for example "data" or "analyst").
A user can conseguentely assign a fav keyword to a fav company.
The goal is to allow a user to get notified as soon as possibile whenever a company is following publishes a new job offer connected to his keywords.

Each user has an entry in the table "users".
Whenever a user adds a new fav company, a new entry in the table "userstargets" is created, same for fav keyword with table "userskeywords".
The match between fav keywords and fav companies are stored in a table called "userstargetskeywords".

# Match
When the daily scraping wave is done, I run a few scripts to make the matches and to notify the users about the matches.
Each match session is stored in a table "matchings", and each notification wave is stored in a table "notifiers".