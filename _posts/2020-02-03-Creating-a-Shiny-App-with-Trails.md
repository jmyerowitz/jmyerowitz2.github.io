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

So the first step is to always definte the UI for the application. [This cheat sheet](https://shiny.rstudio.com/images/shiny-cheatsheet.pdf) is super helpful for finding your way for RShiny. [Shinythemes](https://rstudio.github.io/shinythemes/) allows you to easily code a theme to your pleasing--they have several themes to choose from.

I used a fluidPage for a UI (see below). As a forewarning, the following code will  be incomplete as it's very long and I don't want to show everything. I wouldn't want to give up my trade secrets just yet!
#Define UI for application
    ui <- fluidPage(

I ended up choosing the theme "cerulean," and the below code selects it for me!
#Selecting theme
    
    theme = shinytheme("cerulean"),
    
Below is an example of my sidebar and how the widgets in the Shiny App will be used to create a reactive dataframe that is graphed by Plotly. 
#Sidebar

                        sidebarPanel(
                            h3("How to Use the Widgets"),
                            helpText("Select your inputs below in order to",
                                     "find and visualize trails across",
                                     "New Hampshire and Maine"),
                            radioButtons(inputId = "state", h3("Select State"), 
                                         choices = list("New Hampshire", "Maine"), selected = "New Hampshire"),
                            checkboxGroupInput(inputId = "cluster", h3("Cluster Selection"), choices = c(0,1,2,3,4,5,6), 
                                               selected = c(0,1,2,3,4,5,6)),
                            selectInput(inputId = "difficulty", h3("Trail Difficulty"), 
                                        choices = list("Easy", "Moderate", "Hard"), selected = "Easy"),
                            checkboxGroupInput("checkGroup2", h3("Route Type"),
                                               choices = list("Loop", "Out & Back", "Point to Point"), selected = "Loop"),
                            sliderInput("slider1", h3("Elevation (feet)"), min = 0, max = 46000, value = 46000),
                            sliderInput("slider2", "Distance (miles)", min = 0, max = 170, value = 170),
                            actionButton("submit", "Apply Changes", icon("refresh"))
                        ),
                        
In the server function, I have a dataframe that reacts to the above widgets and is used to display those filtered trails as graphs.

    #Creating reactive dataframe that works with widgets input 
    df <- eventReactive(input$submit, {
        filter(traildf, State == input$state, Difficulty == input$difficulty, Route_Type == input$checkGroup2, 
               Cluster_number == input$cluster, Elevation_Feet <= input$slider1, Distance_Miles <= input$slider2)
         
    })
    
Pretty neat, huh? You can think of the UI (or user interface) as the aesthetic design for the app, while the server function is the meat and potatoes code that gets the pretty graphs working. 

Below is an example of a leaflet map with a borrowed image from [flaticon](https://www.flaticon.com/authors/freepik). Show them some love--they have some really useful icons for any leaflet maps that you want to make!

    #Defining icon use
    hikingIcon <- makeIcon(
        iconUrl = "https://image.flaticon.com/icons/svg/71/71423.svg",
        iconWidth = 20, iconHeight = 30,
    )
    
    #Defining rank as a character so that is can be shown via pop-up
    traildf$Rank <-as.character(traildf$Rank)
    
    #Leaflet as same trails as above, but with rank
    output$filteredleaflethiking <-renderLeaflet({
        leaflet(df()) %>%
        addTiles() %>%
        addMarkers(lng= ~Longitude, lat= ~Latitude, icon = hikingIcon, popup = ~Rank)
    })
    
The output name is also in the UI (wherever you want to put it). Below is where mine is (as defined by leafletOutput("filteredleaflethiking). 

                                tabPanel("Mapping the Filtered Trails",
                                         h3("Trails with Their Description"),
                                         h4("This tab can be fitlered, so filter away!"),
                                         h5("Icons made by", a("https://www.flaticon.com/authors/freepik")),
                                         leafletOutput("filteredleaflet"),
                                         h5("Trails with Their Rank"),
                                         leafletOutput("filteredleaflethiking")),
                                         
You'll notice that I have two leaflet outputs in the above code, and that's true! I do have two leaflet maps in my shiny app that show up together.  

If you're interested in seeing more code from the app, let me know and I can write more about it! Until next time!
