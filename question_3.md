- _Please provide an example where you have gathered data in a scalable, cost-efficient and creative way. (This can be from using an API, scraping or any other method)
If you considered other potential solutions to the problem of gathering this data, please explain your decision making process. We'd like to understand what solutions you took into considerations and how you made your decision for the winning approach._
___

Also for this question, I am going to provide you an answer based on the Saas [I am currently building](https://www.jeifai.com).

There are also other characteristics I want for my scraper:
- Fast
- Running  independently from each other
- Running in parallel from each other
- Simple to read
- Simple to maintain
- Simple to debug
- Give me a good way to run headless Chrome

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
The most important aspect is to produce code as identical as possibile to each other.<br>
The two main tools used to scrape are:
* *[Colly](http://go-colly.org/)*: Golang scraping best library
* *[Chromedp](https://github.com/chromedp/chromedp)*: Run an headless Google Chrome instance

Not all the career pages are identical:
During the last months, I have been able to divide job career' pages in different tiers.
* *HTML*: data are stored into the HTML response, Colly is used.
* *API*: data are returned after an API call, Colly is used.
* *Javascript*: data are stored into the HTML, but after Javascript renders the page, Colly and Chromedp are used.
* *API_POST*: data are returned after an initial API call to get the cookies, Colly and Chromedp are used.
* *Pagination*: if any of the category above presents pagination, it is necessary to implement a logic for it.

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
I wanted to run my scraper from CLI, so I started using Cobra.

Now it is possibile to run a scraper from CLI using:
```bash
./JeifaiBack scrape -s=[scraper_name] -r=[true/false]
```
* -s select any scraper name
* -r true if results need to be saved, false otherwise (might be useful for testing purposes)