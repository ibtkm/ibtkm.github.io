---
layout: post
title: "a little but useful (for me) bot"
comments: true
date: "2014-04-04 00:52:24"
---

many of you are movie geeks and are usually interested in some particular genres. you might like "drama" or somebody else might like "thriller" and etc. the scenario
that I'm gonna talk about has happened to you undoubtedly. you have a bunch of movies in a directory and now you wanna watch one but you don't know them all. thus, you decide to go to imdb and start to search the titles one by one. gazed to the screen, you start to read about genres, score and description and at the end you choose one. seems ok. today, as a result of not having enough of fun, I defined a little challenge for my self: automating this process through an imdb bot.<br>
I didn't find the imdb's REST api so I did the process through web interactions.
I wrote a little script in Ruby language with the help of our friend, "nokogiri" gem to handle the html parsing. you can get it through its <a href="http://github.com/py4/mifb">github page</a>. <br>
here is a screenshot of what it brings for you: <br>
<img src="http://ubuntuone.com/4pRtSgL6PQSxDCIg9qJV7W" width="750px" height="300px">
here is the source: <br>
{% highlight ruby %}
require 'nokogiri'
require 'open-uri'
require 'colorize'
blacklist = ["dvdrip","xvid","maxspeed","720p","hd"]
puts "Enter the directory: "
dir_addr = gets.chomp
names = Dir.entries(dir_addr).reject { |name| name == "." or name == ".." }
names.each do |name|
    name.downcase!
    blacklist.each { |black_name| name.gsub!(black_name,'') if name.include? black_name }
    name = name.gsub(/[^A-Za-z\s]/,' ').strip.gsub(" ","+")
    doc = Nokogiri::HTML(open("http://www.imdb.com/find?q=#{name}&s=all"))
    section = doc.css("div.findSection").select { |section| section.css("h3.findSectionHeader").text == "Titles" }.first
    relative_url = section.css("table.findList").css("tr").first.css("td.result_text").css("a").attr("href").text
    doc = Nokogiri::HTML(open("http://www.imdb.com#{relative_url}"))
    puts ">>> #{doc.css("h1.header").css("span.itemprop").text} <<<".red
    print "genre: ".green
    print doc.css("div.infobar").css("span.itemprop").map { |item| item.text }
    print "\n"
    print "rating: ".green
    puts doc.css("div.star-box-giga-star").text
    print "description: ".green
    puts doc.css("td#overview-top").css("p").select { |p| p.attributes["itemprop"] and p.attributes["itemprop"].value == "description" }.first.text
end
{% endhighlight %}
Your comments and suggestions are welcome.

