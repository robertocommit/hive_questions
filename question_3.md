- _Please provide an example where you have gathered data in a scalable, cost-efficient and creative way. (This can be from using an API, scraping or any other method)
If you considered other potential solutions to the problem of gathering this data, please explain your decision making process. We'd like to understand what solutions you took into considerations and how you made your decision for the winning approach._
___

Also for this question, I am going to provide you an answer based on the Saas [I am currently building](https://www.jeifai.com).

Below I will try to explain how I scrape the data, how I clean/segment them and also my final goal.

# Data Scraping

Here the characteristics I want for my scraper:
- Fast
- Running  independently from each other
- Running in parallel from each other
- Simple to read
- Simple to maintain
- Simple to debug
- Easy to integrate with headless Chrome

So I decided to start learning and using Golang for this project.

### Scraper's flow	

* Start main function
* Connect to the database
* Get the scraper name (for example "Google")
* Start scraping session
* Scrape
* Save results to the database

### Scrapers' structure
All the scrapers are included in a single file called *scrapers.go*.<br>

Not all the career pages are identical,<br>
but it is really important to produce code as identical as possibile to each other.<br>

During the last months, I have been able to divide job career' pages in different tiers:
* *HTML*: data are stored into the HTML response, Colly is used.
* *API*: data are returned after an API call, Colly is used.
* *Javascript*: data are stored into the HTML, but after Javascript renders the page, Colly and Chromedp are used.
* *API_POST*: data are returned after an initial API POST call to get the cookies, Colly and Chromedp are used.
* *Pagination*: if any of the category above presents pagination, it is necessary to implement a logic for it.

The two main tools used to scrape are:
* *[Colly](http://go-colly.org/)*: Golang scraping best library
* *[Chromedp](https://github.com/chromedp/chromedp)*: Run an headless Google Chrome instance

### How to create a new scraper?
The creation of a new scraper is divided in two different part:
* Add in the database the new target and the new scraper
    * Name, career's url and host url are necessary
```golang
func main() {
    scraper_name := "Google"
    jobs_url := "https://www.google.com/careers"
    host_url := "https://www.google.com"
    scraper := Scraper{scraper_name, jobs_url, host_url}
    scraper.CreateScraper()
}
```

* Create the algorithm to scrape
    * Often it is good practice to build and test the scraper in a separate folder.
    * Here an example fo scraper which extract the information directly from the HTML.
```golang
func (runtime Runtime) Google() (results Results) {
    start_url := "https://career.google.com"
    type Job struct {
        Url      string
        Title    string
        Location string
        Type     string
    }
    c := colly.NewCollector()
    c.OnHTML("a", func(e *colly.HTMLElement) {
        if strings.Contains(e.Attr("class"), "job-box-link") {
            result_title := e.ChildText(".jb-title")
            result_url := e.Attr("href")
            result_description := e.ChildTexts("span")[0]
            result_location := e.ChildTexts("span")[2]
            results.Add(
                runtime.Name,
                result_title,
                result_url,
                result_location,
                Job{
                    result_url,
                    result_title,
                    result_location,
                    result_description,
                },
            )
        }
    })
    c.OnRequest(func(r *colly.Request) {
        fmt.Println(Gray(8-1, "Visiting"), Gray(8-1, r.URL.String()))
    })
    c.OnError(func(r *colly.Response, err error) {
        fmt.Println(Red("Request URL:"), Red(r.Request.URL))
    })
    c.Visit(start_url)
    return
}
```

### **How to run a scraper?**
I wanted to run my scraper from CLI, [so I started using Cobra](https://github.com/spf13/cobra).

Now it is possibile to run a scraper from CLI using:
```bash
./JeifaiBack scrape -s=[scraper_name] -r=[true/false]
```
* -s select any scraper name
* -r true if results need to be saved, false otherwise (might be useful for testing purposes)

# Data Cleaning/Segmentation

In am currently working on this part of the project.
I want my data to be clean because I aim to learn how to apply ML techniques to my data.

At the moment I extract these information:
 - company_name 
 - job_date
 - job_title
 - job_location
 - job_url

I want to enrich my data cleaning them and extracting useful information. 

### Country
The first important step is to extract Country from the location.<br>
In order to do so I am using Python, Pandas and a [fantastic dataset](http://www.geonames.org/) containings geo details of around 200K cities in the world.<br>
The best part of the dataset is that each city has also the translation of their name in multiple langiages.

### Seniority
The second step is to segment the data according to the seniority level of the position.<br>
I want to classify if a job is dedicated to a Senior or Junior position.
```sql
SELECT
    r.id,
	r.title,
	CASE
		WHEN LOWER(r.title) SIMILAR TO '%(chief|officer|president|director|executive|partner|advisor|founder)%' THEN 4
		WHEN LOWER(r.title) SIMILAR TO '%(manager|principal|lead|specialist|architect|scientist|expert|head|leader|staff|senior|security|consultant|management|deputy|researcher|administrator|master|phd|counsel)%' THEN 3
		WHEN LOWER(r.title) SIMILAR TO '%(engineer|engineering|ingenieur|developer|analyst|sales|product|business|development|operations|customer|marketing|project|technician|designer|supervisor|recruiter|coordinator|associate|account|support|assistant|service|technical|operator|controller|planner|trainer)%' THEN 2
		WHEN LOWER(r.title) SIMILAR TO '%(intern|ausbildung|praktikum|praktikant|internship|junior|student|werkstudent|trainee)%' THEN 1
	END AS tier
FROM results r;
```

### Information Technology<br>
I failed in trying to segment a job based on the department (for example Sales VS marketing).<br>
So I decided to use another apporach, identify which jobs are related to information technology (software, develoeper, analyst...).<br>
Using Stackoverflow survey I have been able to identify the most important Tags of their questions,
so now I have a list of ~300 keywords dedicated to IT jobs, which I can use to segment the job titles.

### What's next
I am currently in the process of learning Pytorch to start apply learning techniques to my data.