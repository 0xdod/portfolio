+++ 
draft = false
date = 2020-12-25T20:38:40+01:00
title = "Web scraping with go and go-colly"
description = "Building a web scrapper in golang"
slug = "web scraping with go" 
tags = ["web scraping", "go", "colly"]
categories = ["go", "golang"]
externalLink = ""
series = []
featured_image ="/images/posts/golang.png"
+++

In this post, I will show you how to use Go and [colly](https://go-colly.org) for web scraping by trying to scrape the frontpage of a popular Nigerian forum called [Nairaland](https://nairaland.com/).

##### Web Scraping
[Web scraping](https://en.wikipedia.org/wiki/Web_scraping) is a form of data extraction that basically extracts data from websites. A web scraper is usually a bot that uses the HTTP protocol to access websites, extract HTML elements from them and use the data for various purposes.
I'll be sharing with you how you can scrape a website with minimal effort in go, let's go :rocket:

First, you will need to have go installed on your system and know the basics of the language before you can proceed.
We'll start by creating a folder to house our project, open your terminal and create a folder 
```bash
mkdir scraping-with-go
cd scraping-with-go 
```
Then initialize a go module, using the go toolchain
```bash
go mod init github.com/<Username>/<projectname>/
```
Replace the username and project name with appropriate values, by now we should have two files in our folder called go.mod and go.sum, these will track our dependencies.
Next we go get colly with the following command
```bash
go get -v github.com/gocolly/colly
```
Then we can get our hands dirty. Create a new main.go file and fire up your favorite text editor or IDE.
```go
package main

// Post is a struct representing a single post in nairaland
type Post struct {
	Author string
	URL    string
	Body   string
	Title  string
}
```
The above is the data structure we will be storing a single post in, it will contain necessary information about a single post. This was all I needed to populate my database, I was not interested in getting the comments since we all know how toxic the comments section of  forums can be :)
```go
func main() {
	URL := "https://nairaland.com/"
	log.Println("Visiting", URL)

	c := colly.NewCollector(colly.CacheDir("./nl_cache"))
	postCollector := c.Clone()
	posts := make([]Post, 0, 100)
}
```
We need to call the NewCollector function to create our web scrapper, then using CSS selectors, we can identify specific elements to extract data from. The main idea is that we target specific nodes, extract data, build our data structure and dump it in a json file. After inspecting the nairaland HTML structure (which I think is quite messy), I was able to target the specific nodes I wanted. 

```go
...
c.OnHTML("td.featured.w a[href]", func(e *colly.HTMLElement) {
		postCollector.Visit(e.Attr("href"))
	})
```
The OnHTML method registers a callback function to be called every time the scrapper comes across an html node with the selector we passed in. The above code visits every link of frontpage news 
```go
...
postCollector.OnHTML(`table[summary="posts"] tbody`, func(e *colly.HTMLElement) {
		log.Println("Post found", e.Request.URL)
		var p Post
		p.Title = e.ChildText("tr:first-child td.bold.l.pu a[href]:nth-child(4)")
		p.Author = e.ChildText(`tr:first-child td.bold.l.pu:first-child a[class="user"]`)
		p.Body = e.ChildText("tr:nth-child(2) td.l.w.pd:first-child div.narrow")
		p.URL = e.Request.URL.String()
		posts = append(posts, p)
	})

	c.OnRequest(func(r *colly.Request) {
		log.Println("Visiting: ", r.URL)
	})
	c.OnResponse(func(r *colly.Response) {
		log.Println("Visited: ", r.Request.URL)
	})
	c.Visit(URL)
```
What is happening here is that when we visit each link to a frontpage news, we extract the title, url, body and author name using CSS selectors to identify where they are located, we then build up our post struct with this data and append it to our slice. The OnRequest and OnResponse functions registers a callback each to be called when our scrapper makes a request and receives a response respectively. With this data at our disposal, we can then serialize it into json to be dumped on disk. There are other storage backends you can use if you want to do something advanced, checkout the [docs](https://go-colly.org/docs). We then make a call to c.Visit to visit our target website.
```go
...
var buf bytes.Buffer
	enc := json.NewEncoder(&buf)
	enc.SetIndent(" ", "\t")
	err := enc.Encode(posts)
	if err != nil {
		log.Println("failed to serialize response: ", err)
		return
	}
	err = ioutil.WriteFile("posts.json", buf.Bytes(), 0644)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(buf.String())
```
We use the standard library's json package to serialize json then write it to a file on disk, and voila we have written our first scrapping tool in golang, easy right?. Armed with this tool, you can conquer all the web, but remember to check the *robots.txt* file which tells you what data you can scrape and how to handle the data. You can read more about the robots file [here](http://www.robotstxt.org), and remeber to visit the [docs](http://go-colly.org) to learn more there's a ton of great examples you can follow along there. Cheers  :v:

Thank you for reading