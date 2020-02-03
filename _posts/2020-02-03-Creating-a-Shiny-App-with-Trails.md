---
layout: post
title: "Creating a Shiny App via R with Web-Scraped Trails"
date: 2020-02-02
---

Today for all your pleasure, I'm going to go into detail about how to create the R Shiny App that I created [here](https://jordanmyerowitz.shinyapps.io/RShinyAllTrails/). Feel free to play around with it--I also additionally web-scraped information about the Maine hiking trails and clustered them as well to compare with New Hampshire hiking trails. You can read more about how I did that in [this blog post](https://jmyerowitz.github.io/2019/10/31/WebScraping-and-Clustering.html). Buuuut if you want a quick referesher, here's how I did it for the Maine trails:

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

    for(i in trail_links){
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
  
      Sys.sleep(5)
  
    }

#Intializing empty lists for Latitude and Longitude

    Latitude <- c()
    Longitude <- c()

    for(i in trail_links){
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
    
And there you have it! I went on to cluster them as well, but that was done in Python.

I used the following libraries to create the Shiny app and the graphs in it:
library(shiny)
library(shinythemes)
library(plotly)
library(ggplot2)
library(leaflet)

So the first step is to always definte the UI for the application. [This cheat sheet](https://shiny.rstudio.com/images/shiny-cheatsheet.pdf) is super helpful for finding your way for RShiny. 

# Define UI for application
    ui <- fluidPage(

#Selecting theme
    
    theme = shinytheme("cerulean"),
    
    #Title Page
    titlePanel("An In-Depth Look Into Maine and New Hampshire Hiking Trails"),
    
    #Navigation Bar
    navbarPage("Options",
