title: "Web Usability – Avoid an Extra Page Load with OpenSearch"
date: 2011-04-05 14:40:55
tags:
- WebDev
- Misc
---

Now a days most of the websites, blogs or any applications that are on internet will have a search functionality which is great!!!. Before the Omnibox concept was introduced by google chrome, if we want to search something in google, as a user I need to go to www.google.com and type the search query. After the introduction of Omnibox, user don’t need to open the google website instead you could do the search from the address bar itself. From the usability point of view, it is a great functionality IMHO.

From the point of google search application, they have done it smartly. As a web developer how could you provide the same usability feature to your own website. Some of the website I frequently visits has done like this.

![StackOverflow Opensearch](http://cdn.rajeeshcv.com/images/2011/04/clip_image001_thumb.png)

If you are in google chrome, after you type the stackoverflow.com a message will be displayed on the address bar saying that *Press Tab to search Stack Overflow*. If you press tab, you can directly type the search query in the address bar itself and pressing enter will show the results. This basically allows you to do a quick search, instead of going to the website and finding the search box and pressing search button(a long process is it? Smile)

Recently I was working on a hobby project called [ifscfinder] , which is a website were you could search for a *Indian Finacial System Code* of any banks in India. It simply provide a search box where user could enter the Bank/Branch name and it will list the banks that matches the  query.  So I wanted to have the same feature like stackoverflow in my [ifscfinder] too.

After googling for sometime, I found about the opensearch which helps to achieve this functionality. You could read more about opensearch from here http://www.opensearch.org

What I had do was, create an xml file(OpenSearch description document), it will have URL format which needs to be called when user enter a search query. ifscfinder xml file looks like this

```xml
<?xml version="1.0" encoding="UTF-8"?>
<OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/">
  <ShortName>IFSC Finder</ShortName>
  <Description>Search IFSC: Find  Indian Financial System Code of a Bank</Description>
  <InputEncoding>UTF-8</InputEncoding>
  <Url type="text/html"
       template="http://ifscfinder.apphb.com/Search?q={searchTerms}"/>
</OpenSearchDescription>
```

After this is created, I upload it the website. In order for the browser to sense whether my website opensearch, I had to add link tag to the head element

`<link rel="search" type="application/opensearchdescription+xml" title="IFSCFinder.com" href="/opensearchdescription.xml" />`

With this [ifscfinder] has got a nice quick search option in google chrome

![ifscfinder opensearch](http://cdn.rajeeshcv.com/images/2011/04/clip_image002_thumb.png)

Firefox also detect the opensearch , which mean you could add this site to the search engine list in firefox

![Firefox Opensearch](http://cdn.rajeeshcv.com/images/2011/04/image_thumb.png)

IE9 doesn’t care about this at all(don’t know why!!!)

[IfscFinder]: http://ifscfinder.apphb.com/
