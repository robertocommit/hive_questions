- _Please provide an example of your past work that is the most relevant to this job. We'd like to be able to see a database schema that you've previously designed._
- _Please provide context for the design choices that you've made, so that we can understand why you designed the way you did._

___

I am going to provide you the database schema I developed to sustain data's needs of the Saas [I am currently building](https://www.jeifai.com).

The backend of the Saas is a concurrent web scrapers built using Golang, targeting companies' career pages (currently 470 tech companies are monitored).

When I build tools I want to sit on giants' shoulders: as database I use Postgresql running on Google Cloud Platform.

Usually before writing any line of code, I spend a lot of time thinking how I would my queries look like.<br>
After I create a prototype, I design the schema which would allow me to run such SQL code.<br>
In my opinion, in the same way as nature works, **schemas and queries have to be beautiful**.<br>
As conseguence they tend to be easy to read and easy to maintain.

Design choices have been taken considering the main mission of the Saas:

```
To allow a user to get notified as soon as possibile
when a company publishes a new job offer connected to certain keywords.
```

___

The data architecture is mainly divided in two sections: companies and users.<br>
Here I will explain the high level principles behind my choices.

### Companies
Each company has one entry in table *targets* as well as into *scrapers*.
When a new scraping "wave" starts, an entry in the table *scrapings* is generated.
The results of the scraping wave are stored into a table called *results*.<br>
To get jobs available for a particular company I can simply write:
```sql
SELECT
	ss.name AS company_name,
	r.title AS job_title,
	r.location AS job_location,
	r.url AS job_url
FROM results r
LEFT JOIN scrapings s ON(r.scrapingid = s.id)
LEFT JOIN scrapers ss ON(s.scraperid = ss.id)
WHERE ss.name = 'Google'
AND r.updatedat::DATE = current_date;
```
I also scrape Linkedin's data for each company. Data are stored in a table called *linkedin*.

### Users
The Saas allows each user to select a list of favourite companies, as well as favourite keywords (for example "data" or "analyst").
A user can conseguentely assign a fav keyword to a fav company.<br>

Each user has an entry in the table *users*.
Whenever a user adds a new fav company, a new entry in the table *userstarget*" is created, same for fav keyword with table *userskeywords*.<br>
Matches between fav keywords and fav companies are stored in a table called *userstargetskeywords*.<br>
To get the fav companies and fav keywords of a user, I can write:
```sql
SELECT
	u.name AS user_name,
	t.name AS company_name,
	k.text AS keyword_text
FROM users u
LEFT JOIN userskeywords uk ON(u.id = uk.userid)
LEFT JOIN userstargets ut ON(u.id = ut.userid)
LEFT JOIN userstargetskeywords utk ON(uk.id = utk.userkeywordid AND ut.id = utk.usertargetid)
WHERE u.name = 'Roberto'
AND utk.deletedat IS NULL;
```

## Data Schema
I use Google Drawing to draw the schema, keeping it up to date whenever a change is made.<br>
Moreover thanks to Google Drawing I can play around with the schema trying to give it an optimal layout.<br>
Here you can find how my data schema look like:
![](https://github.com/robimalco/hive_questions/blob/main/images/data_schema.png)