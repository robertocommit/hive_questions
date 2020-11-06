- _Please walk us through your thinking of what some first-principle rules you'd follow in designing our data schemas and the whole data pipeline considering our goals. We are planning to index all kinds of social media (i.e. various platforms such as Twitter, Reddit etc., but also platforms that don't exist today, but will in the future), media (podcasts, youtube videos, articles etc. and other media formats that may emerge in the future) in such a way that they can cross-reference each other. We will also need to index a multi-dimensional identities that will tie to these pieces of content (e.g. identifiers connected across multiple social media platforms, but also such entities as Bitcoin wallets, email addresses or phone numbers). The purpose of this question is for us to see your thinking process – we'd like to understand how you would approach such a problem, rather than get any particular answer._
___

This is one of the most fascinating yet complex challenges I have heard.<br>
I try to describe your mission using my own words:

```
The goal is to collect and structure users' social media presence.
```
Below I will try to explain how I would apporach the data schema and the data pipeline for such an ambitous goal.

# Data Schema

The "data framework" I imagine would allow to integrate seamleassy any new social media, independentely from their type.

Following the suggestions you gave in the description, I would divide the problem in four macro sections:
- **User**: A map to allow to know which social media belongs to which user
- **Social**: A map to allow to know for any social media which data are available and their type
- **SocialUser**: A map to store any social media data scraped
- **Scraping**: A map to keep track of each scraping session executed

I image massive JSON storing your information.
For such a purpose, highly performance and distributed NOSQL databases would be necessary.
Not sure which solution would be optimal, if MongdDb, Firebase, or DynamoDB among others.
Recentely I have ready about CockroachDB, claiming they have built the best distrubuted SQL database in the world. 
You are using Arangodb so it is defintely the solution which grants you the biggest benefits.

One of the key question I have is to store the social information itself, or to simply store the link to it.
For example, storing the text of a tweet and its URL, or only the URL?
I am raising this question because of the dynamic nature of the data related to social media.
The number of likes can increase, the text of a facebook post can change, the text of a tweet post cannot be changed, and so on...

Here how I would imagine the four massive JSON.

### User
A user can have multiple social connections,<br>
and each social connection can have multiple addresses (example a user with two twitter accounts).
```json
{
	"user_Ab3231bd5X": {
		"twitter": [
			"social_I9Zdkj909P",
			"social_UI87ZzZ78E"
		],
		"linkedin": [
			"social_UI87ZzZ78E"
		],
		"btc_address": [
			"social_PO90gdT673",
			"social_Zus793GdvB"
		],
	}
}
```

### Social
Any social media has a clear and univoque way to store information.
What is important is to grant flixibility,<br>
allowing versioning in case a social media update itself.
```json
{
	"linkedin": {
		"version_001": {
			"user_page": {
				"url": "string",
				"name": "string",
				"title": "string",
				"description": "string",
				"jobs": [
					{
						"company_name": "string",
						"title": "string",
						"started_date": "date",
						"ended-date": "date"
					}
				]
			},
			"post": {
				"timestamp": "timestamp",
				"text": "string",
				"likes": "numeric",
				"comments": {
					"name": "string",
					"text": "string",
					"timestamp": "timestamp",
				},
			}
		}
	}
}
```

### SocialUser
Here the scraped data are stored.
In this example I will show how I would store LinkedIn data of a certain user.
The same structure can be applied to other social media.
```json
{
	"social_UI87ZzZ78E": { // unique social ID connected to a specific user
		"scraping_73ZHdfF&tz": { // scrpaing session ID
			"timestamp": "2020/11/06 11:52:35",
			"version": "version_002",
			"data": {
				"user_page": {
					"url": "https://www.linkedin.com/in/malcotti/",
					"name": "Roberto Malcotti",
					"title": "Data Engineering, Analytics and Design",
					"description": "My CV: https://www.robimalco.com/Roberto_Malcotti_CV.pdf ",
					"jobs": [
						{
							"company_name": "Bitbond",
							"title": "Data Engineer / Data Analyst",
							"started_date": "2017/10",
							"ended-date": "2019/10"
						},
						{
							"company_name": "Auto1",
							"title": "Marketing Automation Engineer",
							"started_date": "2016/10",
							"ended-date": "2017/10"
						},
					],
				},
				"post": {
					"post_j89JS09OP8": {
						"timestamp": "2020/11/06 11:52:35",
						"text": "this is a sample post",
						"likes": 23,
						"comments": {
							"comment_kj7890gfT6": {
								"name": "Edith",
								"text": "werll done",
								"timestamp": "2020/11/03 11:52:35",
							},
							"comment_hTd8d0gfT6": {
								"name": "Andrea",
								"text": "this is cool",
								"timestamp": "2020/11/03 11:52:35",
							}
						}
					}
				}
			},
		},
	}
}
```

### Scraping
These data can be stored in a pure relational database.
The purpose of this table is to keep track of any scraing sessions and their status.
```sql
create table scrapings (
  id            serial primary key,
  startedat     timestamp not null,
  finishedat    timestamp not null,
  status		boolean,
  userid		varchar(15),
  socialid		varchar(15)
);
```

# Data Pipeline
I would give to the pipeline a cloud based ~ serverless ~ microservices oriented architecture,<br>
keeping completely separated the part of data extration with the part of data analysis.

Each social media would have one specific lambda function dedicated:
- a lambda function to extract the data (wheater using API or pure scraping)

Moreover, common lambda functions are necessary:
- a lambda function to check if the data extracted respect the schema created for a specific social media
- a set of lambda functions to store the data in the dedicated JSON
- a set of lambda functions to check quality of the data triggering other functions in case of failures

In order to run and execute those lambdas, an Apache Airflow running on Kubernetes is necessary.

Millions of daily requests would be necessary, so it might make sense to split the strucure above mentioned by social media.
So for example I would have a K8S Airflow instance only dedicated to Twitter.
This division would produce strong advantages in terms of maintenability of the whole pipeline.
