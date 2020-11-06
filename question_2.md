- _- Please walk us through your thinking of what some first-principle rules you'd follow in designing our data schemas and the whole data pipeline considering our goals. We are planning to index all kinds of social media (i.e. various platforms such as Twitter, Reddit etc., but also platforms that don't exist today, but will in the future), media (podcasts, youtube videos, articles etc. and other media formats that may emerge in the future) in such a way that they can cross-reference each other. We will also need to index a multi-dimensional identities that will tie to these pieces of content (e.g. identifiers connected across multiple social media platforms, but also such entities as Bitcoin wallets, email addresses or phone numbers). The purpose of this question is for us to see your thinking process – we'd like to understand how you would approach such a problem, rather than get any particular answer._
___

This is one of the most interesting challenges I have heard.<br>
I try to describe your mission usoing my own words:

```
The goal is to collect and structure users' digital presence.
```

The "data framework" I imagine would allow to integrate seamleassy any new sources, independentely from their type.

Following the suggestions you gave in the description, I would divide the problem in three macro sections:
- storing digital data scraped from any possibile sources
- creating a map allowing to know which data belongs to a which user
- for any data source available, create a map of how the data should look like 

I image massive JSON storing these information.
For such a purpose, highly performance and distributed NOSQL databases would be necessary.
Not sure which solution would be optimal, if MongdDb, Firebase, or DynamoDB among others.
Recentely I have ready about CockroachDB, claiming they have built the best distrubuted SQL database in the world. 
You are using Arangodb so it is defintely the solution which grants you the biggest benefits, but honestely I have not a good knowledge about it.

One of the key question I would ask you, is if you aim to store the social information itself, or if you would simply store the link to it.
For example, do you store the text of a tweet and its URL, or do you store only the URL?
I am raising this question because of the dynamic nature of the data related to social media.
The number of likes can increase, the text of a facebook post can change, the text of a twwt post cannot be changed, and so on.

Despite many uncertainties, here how I would imagine the three massive JSON.

## User JSON
A user can have multiple social connections, and each social connection can have multiple addresses (example a user with two twitter accounts).
```json
{
	"user_Ab3231bd5X": {
		"twitter": {
			"social_I9Zdkj909P",
			"social_UI87ZzZ78E"

		"linkedin": "social_UI87ZzZ78E",
		"linkedin": "social_UI87ZzZ78E",
	}
}
```
