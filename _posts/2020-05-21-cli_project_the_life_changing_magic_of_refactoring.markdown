---
layout: post
title:      "CLI project: the life changing magic of refactoring"
date:       2020-05-21 23:43:56 -0400
permalink:  cli_project_the_life_changing_magic_of_refactoring
---


<br>
When I started the software engineering bootcamp at Flatiron in April, the last time I learned any new coding skills was the early 2000s, when my mom gave me one of her textbooks and I coded my own LiveJournal and MySpace pages. 

Not exactly up-to-date for 2020. 

<br>

![](https://media.giphy.com/media/PkRdyGsaKp7R0QAKAG/giphy.gif)

<br><br>
I was excited to get started, but had no idea what to expect or how much I would learn in a short time. Now at the end of the first project it is a great time to reflect. Although my CLI is very simple, to create it I have learned the basics of: Github, object-oriented-programming, defining methods and variables in my first language (Ruby), and probably most importantly how to source information from my peers and Google.  Pretty cool.


Working on this project I began to fall in love with code, but I really lit up while refactoring at the end because it came together for me how important well written code is. I got my app working, and then I spent a few hours refactoring. The end result was code that is more readable, simpler, and more flexible to grow over time. 

The importance of this was solidified when I had a conversation with the developer at my start-up last week. Over the last year I have taken on product management, and in that time have become used to bugs popping up all over the site whose source couldn't easily be traced. And then when a bug would get fixed, another unrelated one would pop up as a result. I thought this was normal, and to extent it is (tech is going to break sometimes), but in our conversation I found out how over engineered our code base is. **These are the principles we violated at the beginning:**
<br>

### DRY (Don't repeat yourself): 
Because we have code that is duplicated, even small changes cause a big headache. I was quite amused when I found out in researching more about this that the opposite could be called WET: "write every time" or, perhaps more accurately, "waste everyone's time". I think it was one of those "it's funny because it's true" moments.

### Abstraction:
Where we have 100s of lines of code that could have been simplified to 10, what the code is actually trying to do is not easily understood. This makes our code base complicated to debug, and frustrating to build on. Ultimately costing more than just building it better to start with.

<br>

Turns out, getting a project done quickly is all fun and games until everything starts breaking. 

<br><br>


![](https://media.giphy.com/media/wz8OwIImMOaOc/giphy.gif)



<br><br>



I found this definition I like on a [Medium post](https://medium.com/madhash/are-you-writing-too-much-code-fb4a9605375):


> Refactoring is the act and process of restructuring code to meet the growth of the codebase and maintain the long term stability of cohesive executions. 
> 
<br>

<br><br>

# I would like to dedicate the blog post about my first project to my new favorite topic: refactoring.
<br><br>
![](https://media.giphy.com/media/Y2GFtz19vL1Zu/giphy.gif)

<br>


I look forward to getting feedback on my first project and learning how I can make my code even more elegant. In the future, whether I am looking at my own code, working on a team, or managing a developer, my focus is going to be on **code that is readable, abstract, and closed for modification but open for extension.**


<br>


## What I considered to refactor my code


* Is each method only doing one job? (the single-responsibility-principle)
* Have I repeated anything? (keep it DRY!)
* How flexible is this code; can I use it in different ways without having to modify it? (open-closed principle)

<br>

## In conclusion

I am very excited about the fundamentals that I learned so far, and I look forward to always **striving for elegance**.


<br><br><br><br>




#### Some specific refactoring examples from my CLI project
[Github link](https://github.com/agiletkiewicz/ethical-activewear)

<br><br>

**Made the Brand class more flexible**
<br>

I have a `Brand` class responsible for creating and storing instances of ethical activewear brands. At first, I had a single method (`#create_from_page`) creating a new instance of the `Brand` class and using css selectors to extract information from a Nokogiri XML element to assign corresponding attributes. 

This was **not flexible**, because what if I wanted to create an instance of a Brand using a different data source? That wouldn't work. 

Also, this method **did not follow the single-responsibility principle** because it was doing more than one job. 


I refactored the code so that I can instantiate the Brand class with an attribute hash. That way, I can get the data from anywhere and if the attributes vary, the method won't break. I also wrote a `#create method` that wraps `.new` to not only instantiate with a hash of attributes, but also stores the new instance. 


<br>

Original code:



Scraper class

```
   def get_page
      Nokogiri::HTML(open("https://goodonyou.eco/ultimate-guide-ethical-activewear/"))
   end
  
  
   def scrape_page
    get_page.css("div.article__block .layout-block").each do |brand|
       EthicalActivewear::Brand.create_from_page(brand)
    end
  end
```


Brand class

```
 def self.create_from_page(brand_object)
    new_brand = self.new
    new_brand.name = brand_object.css("h2.layout-block__heading").text
    new_brand.url = brand_object.css("h2.layout-block__heading a").attribute("href").value
    new_brand.description = brand_object.css("div.layout-block__main div.layout-block__content.rich-text").text
    new_brand.rating = brand_object.css("div.layout-block__aside div.meta-labels").text
    new_brand.shipping = brand_object.css("div.layout-block__main div.layout-block__content figure.layout-block__image-container").text
    new_brand.save
  end
  
  def save
    @@all << self
  end
```

<br>

Refactored code:


Scraper class

```
   def get_page
      Nokogiri::HTML(open("https://goodonyou.eco/ultimate-guide-ethical-activewear/"))
   end
  

   def scrape_page
     get_page.css("div.article__block .layout-block").each do |brand|
       attribute_hash = {}
       attribute_hash[:name] = brand.css("h2.layout-block__heading").text
       attribute_hash[:url] = brand.css("h2.layout-block__heading a").attribute("href").value
       attribute_hash[:description] = brand.css("div.layout-block__main div.layout-block__content.rich-text").text
       attribute_hash[:shipping] = brand.css("div.layout-block__main div.layout-block__content figure.layout-block__image-container").text
       attribute_hash[:rating] = brand.css("div.layout-block__aside div.meta-labels").text
       EthicalActivewear::Brand.create(attribute_hash)
    end
  end
```


Brand class

```
  def initialize(attributes)
    attributes.each {|key, value| self.send(("#{key}="), value)}
  end
  
  
  def self.create(attributes)
    new_brand = self.new(attributes) 
    new_brand.save
  end
  
  
  def save
    @@all << self
  end
```


<br>


**Made sure each method in the CLI class has only one responsibility**


When scraping for the attributes of brands, the text needed some formatting. Originally, I wrote that code in my `CLI` method `#print_brand`. 

I thought that this was not very readable, and also shouldn't be the responsibility of the `CLI` class. So I wrote custom getter methods in the `Brand` class instead. 

<br>

Original code:


Brand class

```
attr_accessor :name, :url, :description, :rating, :shipping
```

CLI class

```
 def print_brand(index)
   brand = EthicalActivewear::Brand.all[index]
   formatted_description = brand.description.split(/(See the rating)/)
   formatted_shipping = brand.shipping.split("–")
   puts ""
   puts "----------------------Brand:---------------------".light_blue
   puts "Name: #{brand.name}"
   puts brand.rating
   puts "" 
   puts "------------------Description:------------------".light_blue
   puts formatted_description[0] 
   puts ""
   puts "----------------------Shop:----------------------".light_blue
   puts formatted_shipping[1].strip
   puts "Website: #{brand.url}"
   puts ""
 end
```

<br>

Refactored code



Brand class

```
  attr_accessor :name, :url, :rating
  attr_reader :description, :shipping
	
	
	def description=(string)
    formatting_array = string.split(/(See the rating)/)
    #description comes out with 'see the rating' and 'shop' at the end
    @description = formatting_array[0].strip
  end
  
  
  def shipping=(string)
    formatting_array = string.split("–")
    #shipping information comes after description of product pictured
    @shipping = formatting_array[1].strip
  end
```



CLI class

```
 def print_brand(index)
   brand = EthicalActivewear::Brand.all[index]
   puts ""
   puts "----------------------Brand---------------------".light_blue
   puts ''
   puts "Name: #{brand.name}"
   puts brand.rating
   puts "" 
   puts "------------------Description------------------".light_blue
   puts ''
   puts brand.description
   puts ""
   puts "----------------------Shop----------------------".light_blue
   puts ''
   puts brand.shipping
   puts "Website: #{brand.url}"
   puts ""
 end
```
