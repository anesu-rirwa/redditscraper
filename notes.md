# Web Crawler

in order to get info from reddit (about Game of Thrones) you will have to first run a `crawler` on it.

> A crawler is a program that browses websites and downloads content. Sometimes crawlers are also referred to as spiders.

A crawler needs a starting point to start crawling. We will be using the [GOT subreddit](https://www.reddit.com/r/gameofthrones/), this will be the crawlers start URL.

## Scrapy Shell

To start the scrapy shell in your command line, type:

```bash
scrapy shell
```

To run the crawler in the shell type:

```bash
fetch("https://www.reddit.com/r/gameofthrones/")
```

When you crawl something with scrapy, it returns a “response” object that contains the downloaded information. Let’s see what the crawler has downloaded:

```bash
view(response)
```

## Extracting the title of posts

Scrapy provides ways to extract information from HTML based on css selectors like class, id, etc. Let’s find the css selector for the title, right-click on any post’s title, and select “Inspect” or “Inspect Element”:

As can be seen,  the css class “title” is applied to all <p> tags that have titles. This will help in filtering out titles from the rest of the content in the response object:

```bash
response.css(".title::text").extract()
```

Here response.css(..) is a function that helps extract content based on css selector passed to it. The ‘.’ is used with the title because it’s a css Also, you need to use `“::text”` to tell your scraper to extract only the text content of the matching elements.

## Extracting vote counts for each post

Now this one is tricky. On inspecting, you get three scores:

The “score” class is applied to all three, so it can’t be used as a unique selector is required. On further inspection, it can be seen that the selector that uniquely matches the vote count that we need is the one that contains both “score” and “unvoted.”

When more than two selectors are required to identify an element, we use them both. Also, since both are CSS classes, we have to use “.” with their names. Let’s try it out first by extracting the first element that matches:

```bash
response.css(".score.unvoted").extract_first()
```

To extact all the votes:

```bash
response.css(".score.unvoted::text").extract()
```

## Recap

- response – An object that the scrapy crawler returns. This object contains all the information about the downloaded content.
- response.css(..) – Matches the element with the given CSS selectors.
- extract_first(..) – Extracts the “first” element that matches the given criteria.
- extract(..) – Extracts “all” the elements that match the given criteria.

## Creating a Scrapy Project

To create a scrapy project, type the following command in your terminal:

```bash
scrapy startproject reddit
```

This will create a folder named “reddit” with the following structure:

```bash
reddit/
    scrapy.cfg
    reddit/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
```

- scrapy.cfg – A configuration file that tells Scrapy how to run the project.
- items.py – A file that defines the data model.
- pipelines.py – A file that contains code to process the extracted data.
- settings.py – A file that contains project settings.
- spiders/ – A directory where you’ll later put your spiders.
- spiders/__init__.py – A file that tells Python that this directory contains Python packages.
- __init__.py – A file that tells Python that this directory contains Python packages.

## Creating a Spider

A spider is a class that defines how a certain site (or a group of sites) will be scraped, including how to perform the crawl (i.e. follow links) and how to extract structured data from their pages (i.e. scraping items). In other words, spiders are the place where you define the custom behaviour for crawling and parsing pages for a particular site (or, in some cases, a group of sites).

To create a spider, type the following command in your terminal:

```bash
scrapy genspider reddit_spider reddit.com
```

This will create a file named “reddit_spider.py” in the spiders directory. Open it and you’ll see the following code:

```python
import scrapy

class RedditbotSpider(scrapy.Spider):
    name = "redditbot"
    allowed_domains = ["www.reddit.com/r/gameofthrones/"]
    start_urls = ["https://www.reddit.com/r/gameofthrones/"]

    def parse(self, response):
        pass

```

- name: Name of the spider, in this case, it is “redditbot”. Naming spiders properly becomes a huge relief when you have to maintain hundreds of spiders.
- allowed_domains: An optional list of strings containing domains that this spider is allowed to crawl. Requests for URLs not belonging to the domain names specified in this list won’t be followed.
- parse(self, response): This parse function is called whenever the crawler successfully crawls a URL. Remember the response object from earlier? This is the same response object that is passed to the parse(..).

After every successful crawl, the parse(..) method is called, and so that’s where you write your extraction logic. Let’s add the logic written earlier to extract titles, time, votes, etc., in the parse method:

```python
def parse(self, response):
        #Extracting the content using css selectors
        titles = response.css('.title.may-blank::text').extract()
        votes = response.css('.score.unvoted::text').extract()
        times = response.css('time::attr(title)').extract()
        comments = response.css('.comments::text').extract()
       
        #Give the extracted content row wise
        for item in zip(titles,votes,times,comments):
            #create a dictionary to store the scraped info
            scraped_info = {
                'title' : item[0],
                'vote' : item[1],
                'created_at' : item[2],
                'comments' : item[3],
            }

            #yield or give the scraped info to scrapy
            yield scraped_info
```

> Note: Here, yield scraped_info does all the magic. This line returns the scraped info(the dictionary of votes, titles, etc.) to scrapy, which in turn processes it and stores it.

Run the spider using the following command:

```bash
scrapy crawl redditbot
```

