- _Please walk us through your thinking of what some first-principle rules you'd follow in designing our data schemas and the whole data pipeline considering our goals. We are planning to index all kinds of social media (i.e. various platforms such as Twitter, Reddit etc., but also platforms that don't exist today, but will in the future), media (podcasts, youtube videos, articles etc. and other media formats that may emerge in the future) in such a way that they can cross-reference each other. We will also need to index a multi-dimensional identities that will tie to these pieces of content (e.g. identifiers connected across multiple social media platforms, but also such entities as Bitcoin wallets, email addresses or phone numbers). The purpose of this question is for us to see your thinking process – we'd like to understand how you would approach such a problem, rather than get any particular answer._
___

This is one of the most fashinating yet complex challenges I have heard.<br>
I try to describe your mission usoing my own words:

```
The goal is to collect and structure users' digital presence.
```
The "data framework" I imagine would allow to integrate seamleassy any new sources, independentely from their type.

Following the suggestions you gave in the description, I would divide the problem in four macro sections:
- **UserJSON**: A map to allow to know which data belongs to which user
- **SocialJSON**: A map to allow to know for any social media which data are available and their type
- **SocialUserJSON**: A map to store any digital data scraped from any possibile sources
- **ScrapingJSON**: A map to keep track of each scraping session executed

I image massive JSON storing your information.
For such a purpose, highly performance and distributed NOSQL databases would be necessary.
Not sure which solution would be optimal, if MongdDb, Firebase, or DynamoDB among others.
Recentely I have ready about CockroachDB, claiming they have built the best distrubuted SQL database in the world. 
You are using Arangodb so it is defintely the solution which grants you the biggest benefits, but honestly I have not a good knowledge about it.

One of the key question I would ask you is if you aim to store the social information itself, or if you would simply store the link to it.
For example, do you store the text of a tweet and its URL, or do you store only the URL?
I am raising this question because of the dynamic nature of the data related to social media.
The number of likes can increase, the text of a facebook post can change, the text of a twwt post cannot be changed, and so on.


Here how I would imagine the three massive JSON.

## UserJSON
A user can have multiple social connections, and each social connection can have multiple addresses (example a user with two twitter accounts).
```json
{
	"user_Ab3231bd5X": {
		"twitter": [
			"social_I9Zdkj909P",
			"social_UI87ZzZ78E"
		],
		"linkedin": ["social_UI87ZzZ78E"],
		"btc_address": [
			"social_PO90gdT673",
			"social_Zus793GdvB"
		],
	}
}
```

## SocialJSON
Any social media has a clear way to store information.
What is important is to grant flixibility to manage multiple versions in case a social netowrk update its page.
```json
{
	"linkedin": {
		"version_kj76GGhs78": {
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


```
## SocialUserJSON
Ij this example I will show how I would store LinkedIn data of a certain user.
The same structure can be applied to other social network.
```json
{
	"social_UI87ZzZ78E": { // unique social ID connected to a specific user
		"scraping_73ZHdfF&tz": { // scrpaing session ID
			"timestamp": "2020/11/06 11:52:35",
			"version": "version_kj76GGhs78",
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
