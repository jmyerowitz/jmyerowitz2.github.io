---
layout: post
title: "Clustering Similar New Hampshire Trails via WebScraping"
date: 2019-10-31
---

Recently, my Master's program (UNH Analytics and Data Science, represent!) had a small module on webscraping using R via Rvest. I looked at a website called [alltrails.com](https://www.alltrails.com). It's a really handy site for information on trails all around the world. I've only used it locally for New England trails, but I've seen trails in Spain pop up as well.

I'm an avid hiker. I'm originally from Maine, so I wanted to get a feel for the different trails in New Hampshire. New Hampshire has trails that range from a nice stroll to a strenuous hike up Mt. Washington. I had the thought that it would be cool to pick new trails based off the characteristics of previous trails that I had hiked--sounds like a good opportunity to cluster! 

As is good web-scraping practice, I checked out the robots.txt file for the website. Some things were off limits for me, such as the website APIs and user info. But that's alright, I only want information about the trails! Specifically, New Hampshire trails. I accessed the information needed from this specific [webpage](https://www.alltrails.com/us/new-hampshire). On the right of the page is a loading bar for the best trails in New Hampshire, ranked from best to worst by the members of the alltrails community. Using RSelenium, it would be possible to load all the links via clicking the "Load" button repeatedly. Admittingly, I did not do this.

One day I'll feel comfortable with RSelenium. For now though, I manually loaded the webpage side scroll and saved it to my local machine. From this local file, I scraped the links that would lead me to each individual trails site and the information that I needed.  
So this link: 
![Image](https://github.com/jmyerowitz/jmyerowitz.github.io/blob/master/images/Alltrails1.PNG)

Gets me all this information:
![Image](https://github.com/jmyerowitz/jmyerowitz.github.io/blob/master/images/Alltrails2.PNG)

You probably noticed a trail map of all the trails. I originally went to that map link to try and find latitude and longitude coordinates. Needless to say, I was unsuccessful in that endeavor. But I hit a lucky break: hidden away in the html source code of each link page are the latitude and longitude coordinates for the start of each trail. Here's come example scode for how I pulled the links from the sidebar and pulled the relevant information from each link. 

#Setting object to pipe through for trail urls
currentpage <- alltrails

#Piping through for the url of each trail that I want
trail_links <- currentpage %>%
  html_nodes("li.sortable") %>% #Going through each "sortable" node that's a trail
  html_nodes("a.item-link") %>% #going to that trail link
  html_attr("href") #pull the individual trail link
trail_links

#Initializing empty lists to grab each trail Distance, Elevation, Route Type, Description, Name, and Difficulty
Distance <- c()
Elevation <- c()
Route_type <- c()
Description <- c()
Trail_name <- c()
Difficulty <- c()


#Looping through each link for the above attributes (but using html_text :)) in the first chunk
for(i in trail_link_1){
  trail_info <- read_html(i)
  distance <- trail_info %>% 
    html_nodes("section#trail-stats") %>% #This section is pulling the distance from the web page
    html_node("span.distance-icon") %>% 
    html_node("span.detail-data.xlate-none") %>% 
    html_text() 
  Distance <- append(Distance, distance)
  elevation <- trail_info %>% 
    html_nodes("section#trail-stats") %>% #This one is pulling elevation
    html_node("span.elevation-icon") %>% 
    html_node("span.detail-data.xlate-none") %>% 
    html_text() 
  Elevation <- append(Elevation, elevation)
  route_type <- trail_info %>% 
    html_nodes("section#trail-stats") %>% #And this one is pulling route type
    html_nodes("span.detail-data") %>% 
    nth(3) %>% # I couldn't get it to work without using "nth", something about detail data beginning the other trail stats
    html_text()
  Route_type <- append(Route_type, route_type)
  description <- trail_info %>% 
    html_node("p#auto-overview.xlate-google.line-clamp-4") %>% #pulling the description
    html_text()
  Description <- c(Description, description)
  trail_name <- trail_info %>% 
    html_node("div#title-and-difficulty") %>% #Grabbing the trail name
    html_node("h1.xlate-none") %>% 
    html_text()
  Trail_name <- append(Trail_name, trail_name)
  trail_info <- read_html(i)
  difficulty <- trail_info %>% 
    html_nodes("div#difficulty-and-rating") %>% #Getting the difficulty
    html_node("span.diff") %>% 
    nth(1) %>%  #Using nth again
    html_text() 
  Difficulty <- append(Difficulty, difficulty)

}

for(i in trail_1){
  trail_info <- read_html(i)
  latitude <- trail_info %>% 
    html_nodes("a#sidebar-map") %>% 
    html_nodes("div") %>%
    html_node("meta") %>% #Navigating to this node had two "meta' lines: The first had the latitude
    nth(1) %>% 
    html_attr("content") 
  Latitude <- append(Latitude, latitude)
  longitude <- trail_info %>% 
    html_nodes("a#sidebar-map") %>% 
    html_nodes("div") %>% 
    html_nodes("meta") %>% #And the second had longitude
    nth(2) %>% 
    html_attr("content")
  Longitude <- append(Longitude, longitude)
 
}

I elected to use the html source code rather than xpaths, as xpaths can change rather easily on a website. The nth() function is from Rtidyverse, and is really handy in this case. I constructed a dataframe from the info, and I was ready to cluster!

I elected to use python to cluster my trails dataframe. I ignored my Latitude and Longitide columns for clustering, as I didn't want the trails to cluster on geographic information, only characteristics. I clustered on the continuous variables Distance and Elevation, and the categorical variables Route Type and Difficulty.

Route Type and Difficulty had three values each (Loop; Out & Back; and Point-to-Point, and Easy; Moderate; and Hard, respectively). By one-hot encoding this variables, I had 8 variables total. After standardizing them and running outlier analysis for 2% of the data, I identified by outliers.

#Outliers


These trails are extradordinarily long--the type of trails that requires a multi-day hike. 

After outlier analysis, I ran PCA because why not? And I created 4 principle components that explained 90% of the variation in the data. Using K-Means Clustering (which most certainly is not the best clustering technique, but this isn't an intense problem), I then ran a for loop to generate 5-12 clusters. I ended up selecting 6 clusters, as they seemed the most stable. From here, I loaded Rleaflet to play around with map vsiaulizations.


