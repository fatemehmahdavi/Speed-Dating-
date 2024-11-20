---
title: "Unlocking the Dynamics of Speed Dating"
author: "Fatemeh Mahdavi Goloujeh"
date: "2024-11-19"
output:
  html_document:
    toc: true
    toc_depth: 3
---

```{r setup, echo=FALSE,include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Imagine sitting across from someone for just four minutes, trying to decide if they could be your perfect match. This intriguing setup formed the basis of Columbia University’s speed dating experiments, where participants rated each other on attributes like Attractiveness, Intelligence, and Fun, and decided whether they’d want to meet again. But what truly drives these decisions? Let’s dive in to uncover the hidden patterns of dating success.

```{r, echo=FALSE, message=FALSE, warning=FALSE}
library(dplyr)
library(readr)
library(tidyverse)
library(ggplot2)
library(plotly)
library(crosstalk)
library(here)
library(visNetwork)
library(htmltools)
library(htmlwidgets)
```

```{r, echo=FALSE, warning=FALSE, message=FALSE}
speed_dating <- read_csv(here("speed Dating Data.csv"))
```
<div class="plot-container">
## What Do We Look for in a Partner?

When it comes to first impressions, not all attributes are equal. Men and women value different things:

Men tend to prioritize Attractiveness and Fun.
Women, on the other hand, give more weight to Intelligence and Sincerity.
This difference reveals how societal norms and expectations shape what we find appealing in a potential partner.
```{r, echo=FALSE, warning=FALSE, message=FALSE}
# Data Cleaning and Aggregation
speed_dating_summary <- speed_dating %>%
  filter(!is.na(attr1_1) & !is.na(sinc1_1) & !is.na(intel1_1) &
         !is.na(fun1_1) & !is.na(amb1_1) & !is.na(shar1_1)) %>%
  group_by(gender) %>%
  summarise(
    AvgAttractive = round(mean(attr1_1), 0),
    AvgSincere = round(mean(sinc1_1), 0),
    AvgIntelligence = round(mean(intel1_1), 0),
    AvgFun = round(mean(fun1_1), 0),
    AvgAmbition = round(mean(amb1_1), 0),
    AvgSharedInterests = round(mean(shar1_1), 0)
  ) %>%
  pivot_longer(
    cols = starts_with("Avg"),
    names_to = "TypeOfAttribute",
    values_to = "Score"
  ) %>%
  mutate(
    TypeOfAttribute = str_replace(TypeOfAttribute, "Avg", ""),
    Gender = ifelse(gender == 1, "Male", "Female")
  )

# Interactive Plot with Dropdown Menu
fig <- plot_ly()

# Add traces for each gender
fig <- fig %>%
  add_trace(
    data = speed_dating_summary %>% filter(Gender == "Male"),
    x = ~Score,
    y = ~reorder(TypeOfAttribute, Score),
    type = 'bar',
    orientation = 'h',
    name = 'Male',
    marker = list(color = '#3777FF')
  ) %>%
  add_trace(
    data = speed_dating_summary %>% filter(Gender == "Female"),
    x = ~Score,
    y = ~reorder(TypeOfAttribute, Score),
    type = 'bar',
    orientation = 'h',
    name = 'Female',
    marker = list(color = '#FFB5C2')
  )
# Adjusted layout for better spacing between y-axis ticks and labels
fig <- fig %>%
  layout(
    title = list(
      text = "Attribute Ratings by Gender in Speed Dating",
      x = 0.5, # Center the title
      y = 0.95 # Push the title slightly upwards
    ),
    xaxis = list(
      title = list(
        text = "Score",
        standoff = 20
      )
    ),
    yaxis = list(
      title = list(
        standoff = 20
      ),
      ticklabelposition = "outside"
    ),
    updatemenus = list(
      list(
        buttons = list(
          list(
            args = list('visible', c(TRUE, TRUE)), # Show both Male and Female traces
            label = "Both",
            method = "restyle"
          ),
          list(
            args = list('visible', c(TRUE, FALSE)), # Show Male, hide Female
            label = "Male",
            method = "restyle"
          ),
          list(
            args = list('visible', c(FALSE, TRUE)), # Show Female, hide Male
            label = "Female",
            method = "restyle"
          )
        ),
        direction = "down",
        title = "Filter by Gender",
        x = 0.1,
        y = 1.2, # Position dropdown
        showactive = TRUE,
        active = 0 # Set "Both" as the default active button
      )
    ),
    width = 800, # Set desired width
    height = 600,
    margin = list(t = 50, b = 100)# Set desired height
  )
fig

```

</div> <div style="margin-bottom: 90px;"> This is the text that comes below the plot. </div>

<div class="plot-container">
## Do Career Paths Shape Dating Preferences?

Your career can say a lot about what you value in a partner:

STEM professionals lean towards Intelligence and Shared Interests, perhaps seeking someone they can connect with intellectually.
Those in Creative Arts often value Fun and Ambition, suggesting a preference for dynamic and exciting partnerships.
This shows how our work can influence what we want in our personal lives.





```{r, echo=FALSE, message=FALSE, warning=FALSE}
# Prepare data for visualization
career_data <- speed_dating %>%
  filter(!is.na(career_c)) %>%  # Ensure career field data is present
  group_by(career_c) %>%        # Group by career field
  summarise(
    Attractiveness = mean(attr1_1, na.rm = TRUE),
    Sincerity = mean(sinc1_1, na.rm = TRUE),
    Intelligence = mean(intel1_1, na.rm = TRUE),
    Fun = mean(fun1_1, na.rm = TRUE),
    Ambition = mean(amb1_1, na.rm = TRUE)
  ) %>%
  pivot_longer(
    cols = -career_c,
    names_to = "Attribute",
    values_to = "Average_Score"
  )

# Map career codes to descriptive labels
career_labels <- c(
  "1" = "Lawyer", "2" = "Academic/Research", "3" = "Psychologist",
  "4" = "Doctor/Medicine", "5" = "Engineer", "6" = "Creative Arts",
  "7" = "Banking/Business", "8" = "Real Estate", "9" = "International Affairs",
  "10" = "Undecided", "11" = "Social Work", "12" = "Speech Pathology",
  "13" = "Politics", "14" = "Sports", "15" = "Other", "16" = "Journalism",
  "17" = "Architecture"
)

# Replace career codes with labels
career_data <- career_data %>%
  mutate(Career_Field = career_labels[as.character(career_c)])

# Create an interactive horizontal stacked bar chart
fig <- plot_ly(
  data = career_data,
  x = ~Average_Score,
  y = ~Career_Field,
  color = ~Attribute,
  type = 'bar',
  orientation = 'h',  # Horizontal bars
  text = ~paste("Attribute:", Attribute, "<br>Average Score:", round(Average_Score, 2)),
  textposition = 'auto',  # Display numbers directly on bars
  texttemplate = '%{x:.2f}',  # Format numbers to 2 decimal places
  hoverinfo = 'text',
  colors = "viridis"
)

# Customize the layout
fig <- fig %>%
  layout(
    title = list(
      text = "<b>How Attributes Are Valued Across Career Fields</b>",
      font = list(size = 20, family = "Arial, sans-serif", color = "black"),
      y = 2
    ),
    xaxis = list(
      title = list(
        text = "Average Attribute Score",
        font = list(size = 16, family = "Arial, sans-serif", color = "black")
      ),
      tickfont = list(size = 14, family = "Arial, sans-serif")
    ),
    yaxis = list(
      title = list(
        font = list(size = 16, family = "Arial, sans-serif", color = "black")
      ),
      tickfont = list(size = 12, family = "Arial, sans-serif"),
      categoryorder = "total descending",  # Order by total score
      ticklen = 10,  # Increase the distance between ticks and labels
      ticks = "outside"  # Position ticks outward
    ),
    legend = list(
      title = list(text = "<b>Attributes</b>", font = list(size = 14)),
      font = list(size = 12)
    ),
    barmode = "stack",  # Stack bars for each career field
 margin = list(l = 120, r = 40, t = 60, b = 40),  # Adjust margins
    height = 600,  # Reduced height
    width = 800   # Adjust width for better proportions
  )

# Display the plot
fig
```
</div> <div class="text-container"> </div>
</div> <div style="margin-bottom: 90px;"> </div>

<div class="plot-container">
```{r, echo=FALSE}
# Filter and prepare edge data
network_data <- speed_dating %>%
  filter(match == 1) %>%
  select(iid, pid)

# Extract gender information for nodes
node_data <- speed_dating %>%
  select(iid, gender) %>%
  distinct()

# Ensure all nodes in the edges exist in the nodes dataframe
unique_ids <- unique(c(network_data$iid, network_data$pid))  # All unique IDs in edges
nodes <- data.frame(
  id = unique_ids,  # Use unique IDs as node IDs
  label = as.character(unique_ids),  # Add labels for visualization
  gender = ifelse(unique_ids %in% node_data$iid, node_data$gender[match(unique_ids, node_data$iid)], NA)
) %>%
  filter(!is.na(gender)) %>%  # Remove unknown genders
  mutate(
    group = ifelse(gender == 1, "Men", "Women"),
    shape = "image",
    image = ifelse(
      group == "Men",
      "https://cdn-icons-png.flaticon.com/512/2202/2202112.png",  # Men icon
      "https://cdn-icons-png.flaticon.com/512/11107/11107554.png"   # Women icon
    ),
    color = ifelse(
      group == "Men", 
      list(background = "lightblue"),  # Light blue background for men
      list(background = "pink")       # Pink background for women
    )
  )

# Rename columns in edges for visNetwork
edges <- network_data %>%
  rename(from = iid, to = pid) %>%
  filter(from %in% nodes$id & to %in% nodes$id)  # Keep edges with known nodes only

# Create the visNetwork visualization
visNetwork(nodes, edges) %>%
  visNodes(
    shapeProperties = list(useBorderWithImage = TRUE),
    fixed = TRUE  # Hold the icons fixed
  ) %>%
  visEdges(color = list(color = "gray")) %>%  # Default edge color to gray
  visLayout(randomSeed = 123) %>%
  visInteraction(
    multiselect = TRUE,
            # Enables keyboard navigation
    dragNodes = FALSE,         # Prevent dragging nodes
    zoomView = FALSE        # Disable zoom in and out
  ) %>%
  visEvents(
    selectNode = "function(properties) {
      var selectedNode = properties.nodes[0];
      this.body.data.edges.update(this.body.data.edges.map(function(edge) {
        return {
          id: edge.id,
          color: edge.from === selectedNode || edge.to === selectedNode ? 'green' : 'gray'
        };
      }));
    }",
    deselectNode = "function(properties) {
      this.body.data.edges.update(this.body.data.edges.map(function(edge) {
        return {
          id: edge.id,
          color: 'gray'  // Reset to default color
        };
      }));
    }"
  ) %>%
  visOptions(
    highlightNearest = TRUE
  ) %>%
  visLegend(
    useGroups = FALSE,
    addNodes = data.frame(
      label = c("Men", "Women"),
      shape = c("image", "image"),
      image = c(
        "https://cdn-icons-png.flaticon.com/512/2202/2202112.png",
        "https://cdn-icons-png.flaticon.com/512/11107/11107554.png"
      ),
      color = c("lightblue", "pink")
    ),
    position = "right"
  )
```
</div> <div style="margin-bottom: 150px;"></div>

<div style="text-align: center;">

## The Art of Being Selective

Is being "picky" worth it? We found that those who give higher ratings are often rated highly in return, suggesting a reciprocal dynamic in dating. However, overly selective participants don’t always see better results—sometimes, keeping an open mind can lead to unexpected matches.

<div style="display: flex; justify-content: center; margin-top: 20px;">

```{r, echo=FALSE,message=FALSE, warning=FALSE}
selectivity_data <- speed_dating %>%
  filter(!is.na(attr1_1), !is.na(attr_o)) %>%
  group_by(iid) %>%  # Group by participant ID
  summarise(
    Avg_Rating_Given = mean(attr1_1, na.rm = TRUE),  # Average rating given
    Avg_Rating_Received = mean(attr_o, na.rm = TRUE) # Average rating received
  )

# Create an interactive scatter plot
fig <- plot_ly(
  data = selectivity_data,
  x = ~Avg_Rating_Given,
  y = ~Avg_Rating_Received,
  type = 'scatter',
  mode = 'markers',
  marker = list(size = 10, color = '#377eb8', opacity = 0.7), # Color-blind-friendly blue
  text = ~paste(
    "Rating Given: ", round(Avg_Rating_Given, 2), "<br>",
    "Rating Received: ", round(Avg_Rating_Received, 2)
  ),
  hoverinfo = 'text'
)

# Add a trend line
fig <- fig %>%
  add_trace(
    type = 'scatter',
    mode = 'lines',
    x = selectivity_data$Avg_Rating_Given,
    y = predict(lm(Avg_Rating_Received ~ Avg_Rating_Given, data = selectivity_data)),
    line = list(color = '#e41a1c', dash = 'solid'), # Color-blind-friendly red
    name = 'Trend Line'
  )

# Customize the layout
fig <- fig %>%
  layout(
    title = list(
      text = "<b>Do More Selective People Receive Higher Ratings?</b>",
      font = list(size = 18, family = "Arial, sans-serif", color = "black"),
      y = 0.95  # Adjusted to move the title slightly down
    ),
    xaxis = list(
      title = list(
        text = "Average Attractiveness Ratings Given",
        font = list(size = 16, family = "Arial, sans-serif", color = "black")
      ),
      tickfont = list(size = 14, family = "Arial, sans-serif")
    ),
    yaxis = list(
      title = list(
        text = "Average Attractiveness Ratings Received",
        font = list(size = 16, family = "Arial, sans-serif", color = "black")
      ),
      tickfont = list(size = 14, family = "Arial, sans-serif")
    ),
    showlegend = FALSE
  )

# Display the interactive plot
fig
```
</div> 
</div> <div class="text-container"> </div>
</div> <div style="margin-bottom: 90px;"> </div>

<div style="text-align: center;">
## Are Shared Interests the Key to a Match?

Hobbies play a bigger role than you might think. Activities like hiking and reading are strong indicators of match likelihood. Why? Shared interests provide a common ground—a foundation for a relationship. On the flip side, some hobbies, like gaming, have a weaker connection to match success, possibly due to personal or cultural biases.

<div style="display: flex; justify-content: center; margin-top: 20px;">

```{r, echo=FALSE}
activity_match <- speed_dating %>%
  select(sports, hiking, gaming, clubbing, reading, match) %>%
  pivot_longer(cols = -match, names_to = "Activity", values_to = "Score") %>%
  group_by(Activity) %>%
  summarize(avg_score = round(mean(Score, na.rm = TRUE),0), match_rate = mean(match, na.rm = TRUE))

plot_ly(activity_match, x = ~avg_score, y = ~Activity, type = "bar", orientation = "h",
        marker = list(color = "#88CCEE")) %>%
  layout(
    title = list(
      text = "How Your Hobbies Affect Match Likelihood",
      font = list(size = 16, family = "Arial, sans-serif", color = "black"),
      x = 0.5, # Center horizontally
      y = 0.95 # Adjust vertical position (reduce value if necessary)
       # Adds padding at the top
    ),
    xaxis = list(
      title = "Average Preferred Score", 
      font = list(size = 12, family = "Arial, sans-serif", color = "black")
    ),
    yaxis = list(title = ""),
    margin = list(t = 50) # Increase top margin
  )


```
</div> 
