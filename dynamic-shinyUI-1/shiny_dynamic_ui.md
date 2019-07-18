# Dynamic UI Elements in Shiny

At STATWORX we regularly deploy our project results with the help of Shiny. It is not only an easy way of letting potential users interact with your R-code, it's also fun to design a good-looking app. 

One of the biggest strengths of Shiny is it's inherent reactivity, after all being reactive to user input is a web-applications prime purpose. It is unfortunate that many apps seem to only make use of Shiny's reactivity on the server side while keeping the UI completely static. This does not have to be necessarily bad - some apps would not profit from having dynamic UI elements and adding them regardless could quickly result in the app feeling gimmicky. But in many cases adding reactivity to the UI can not only result in less clutter on the screen, but also cleaner code. And we all like that, don't we? 

### The Tools

Shiny natively provides convenient tools to turn the UI of any app reactive to input. We are namely going to look at the `renderUI` (in conjunction with `lapply`) and `conditionalPanel` functions. 

`renderUI` is helpful because it frees us from the chains of having to define what kind of object we'd like to render in our `render` function. `renderUI` can render any UI element, so we could, for example, let the type of the content of our `uiOutput` be reactive to an input instead of being set in stone.

`conditionalPanel` simply lets us conditionally show or hide UI elements and thus helps us reduce visual clutter. 

### `renderUI`

Imagine a situation where you are tasked with building a dashboard showing the user three different KPIs for three different countries. The most obvious approach would be to specify the position of each KPI-box on the UI side of the app and creating each element on the server side with the help of `shinydashboard::renderValueBox` as seen in the example below. 

```R
library(shiny)
library(shinydashboard)

ui <- dashboardPage(
  
  dashboardHeader(),
  dashboardSidebar(),
  
  dashboardBody(column(width = 4, 
                       fluidRow(valueBoxOutput("ch_1", width = 12)),
                       fluidRow(valueBoxOutput("jp_1", width = 12)),
                       fluidRow(valueBoxOutput("ger_1", width = 12))),
                column(width = 4,
                       fluidRow(valueBoxOutput("ch_2", width = 12)),
                       fluidRow(valueBoxOutput("jp_2", width = 12)),
                       fluidRow(valueBoxOutput("ger_2", width = 12))),
                column(width = 4, 
                       fluidRow(valueBoxOutput("ch_3", width = 12)),
                       fluidRow(valueBoxOutput("jp_3", width = 12)),
                       fluidRow(valueBoxOutput("ger_3", width = 12)))
  )
)

server <- function(input, output) {
  
  output$ch_1 <- renderValueBox({
    valueBox(value = "CH",
             subtitle = "Box 1")
  })
  
  output$ch_2 <- renderValueBox({
    valueBox(value = "CH",
             subtitle = "Box 2")
  })
  
  output$ch_3 <- renderValueBox({
    valueBox(value = "CH",
             subtitle = "Box 3",
             width = 12)
  })
  
  output$jp_1 <- renderValueBox({
    valueBox(value = "JP",
             subtitle = "Box 1",
             width = 12)
  })
  
  output$jp_2 <- renderValueBox({
    valueBox(value = "JP",
             subtitle = "Box 2",
             width = 12)
  })
  
  output$jp_3 <- renderValueBox({
    valueBox(value = "JP",
             subtitle = "Box 3",
             width = 12)
  })
  
  output$ger_1 <- renderValueBox({
    valueBox(value = "GER",
             subtitle = "Box 1",
             width = 12)
  })
  
  output$ger_2 <- renderValueBox({
    valueBox(value = "GER",
             subtitle = "Box 2",
             width = 12)
  })
  
  output$ger_3 <- renderValueBox({
    valueBox(value = "GER",
             subtitle = "Box 3",
             width = 12)
  })
}

shinyApp(ui = ui, server = server)
```

This might be a working solution to the task at hand, but it is hardly an elegant one. The valueboxes take up a large amount of space in our app and even though they can be resized or moved around, we always have to look at all the boxes, regardless of which ones are currently of interest. The code is also highly repetitive and largely consists of copy-pasted code chunks. This is of course suboptimal. A much more elegant solution would be to only show the boxes for each unit of interest (in our case countries) as chosen by the user. Here's where `renderUI` comes in.

`renderUI` not only allows us to render UI objects of any type, but also integrates well with the `lapply` function. This means that we don't have to render every valuebox separately, but let `lapply` do this repeptitive job for us. 

Assuming we have any kind of input named "select" in our app, the following code chunk will generate a valuebox for each element selected with that input. The generated boxes will show the name of each respective element as value and have their subtitle set to "Box 1". 

``````R
lapply(seq_along(input$select), function(i) {
      fluidRow(
        valueBox(value = input$select[i],
               subtitle = "Box 1",
               width = 12)
      )
    })
``````

How does this work exactly? The `lapply` function iterates over each element of our input called "select" and executes whatever code we feed it once per element. In our case that means `lapply` takes the elements of our input and creates a valuebox embedded in a fluidrow for each (technically it just spits out the corresponding HTML code that would create that). 

This has multiple advantages: 

- Only boxes for chosen elements are shown, reducing visual clutter and showing what really matters.
- We have effectively condensed 3 `renderValueBox` calls into a single `renderUI` call, reducing copy-pasted sections in our code. 

If we apply this to our app our code will look something like this:

```R
library(shiny)
library(shinydashboard)

ui <- dashboardPage(
  dashboardHeader(),
  
  dashboardSidebar(
    selectizeInput(
      inputId = "select",
      label = "Select countries:",
      choices = c("CH", "JP", "GER"),
      multiple = TRUE)
  ),
  
  dashboardBody(column(4, uiOutput("ui1")),
                column(4, uiOutput("ui2")),
                column(4, uiOutput("ui3")))
  )

server <- function(input, output) {
  
  output$ui1 <- renderUI({
    req(input$select)
    
    lapply(seq_along(input$select), function(i) {
      fluidRow(
        valueBox(value = input$select[i],
               subtitle = "Box 1",
               width = 12)
        )
    })
  })
  
  output$ui2 <- renderUI({
    req(input$select)
    
    lapply(seq_along(input$select), function(i) {
      fluidRow(
        valueBox(value = input$select[i],
               subtitle = "Box 2",
               width = 12)
      )
    })
  })
  
  output$ui3 <- renderUI({
    req(input$select)
    
    lapply(seq_along(input$select), function(i) {
      fluidRow(
        valueBox(value = input$select[i],
               subtitle = "Box 3",
               width = 12)
      )
    })
  })
}

shinyApp(ui = ui, server = server)
```

The UI now dynamically responds to our inputs in the `selectizeInput`. This means that users can still show all KPI boxes if needed - but they won't have to. In my opinion this flexibility is what shiny was designed for - letting users interact with R-code dynamically. We have also effectively cut down on copy-pasted code by 66% already! There is still some repetition in the multiple `renderUI` function calls, but the server side of our app is already much more pleasing to read and make sense of that in the static example of our app. 