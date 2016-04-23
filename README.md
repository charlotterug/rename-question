# rename-question
Responses to Brian's Holbrook's question on other ways to rename a column

At the <a href="http://www.meetup.com/Charlotte-R-Users-Group/events/230305585/">4/18/2016 Charlotte R Users Group meeting</a>, Brian Holbrook shared a snippet of code and asked the group for a better way to do this.

Overview of Solutions<br>
1) The original code from Brian<br>
2) Solutions that involve joins<br>
  2a) Karthik's solution using the merge() function from Base R.<br>
  2b) A dplyr package method<br>
  2c) A sqldf package method<br>

**1) Original Solution**

    data <- data.frame("Activity"=c(1,2,3,4,5,6))
    
    ##Converting Activity code to Activity Name##
    data$Activity <- replace(data$Activity, data$Activity == 1, "Walking")
    data$Activity <- replace(data$Activity, data$Activity == 2, "Walking Upstairs")
    data$Activity <- replace(data$Activity, data$Activity == 3, "Walking Downstairs")
    data$Activity <- replace(data$Activity, data$Activity == 4, "Sitting")
    data$Activity <- replace(data$Activity, data$Activity == 5, "Standing")
    data$Activity <- replace(data$Activity, data$Activity == 6, "Laying")
    
    print(data)
      Activity
    1        1
    2        2
    3        3
    4        4
    5        5
    6        6


**2) Join Related Solutions**

**2a) Karthik's Solution**

There is a function called MERGE in R which appears to solve the SQL xref join issue that was discussed in our last meetup. I tried it and it works exactly the way an SQL outer join would work.

    xref <- data.frame("Activity"=c(1,2,3,4,5,6),
                       "Activity_Name"=c("Walking","Walking Upstairs","Walking Downstairs","Sitting","Standing","Laying"))
                       
    result <- merge(data,xref,by.x = "Activity",by.y = "Activity",all.x = TRUE)
    
    print(result)
    Activity      Activity_Name
    1        1            Walking
    2        2   Walking Upstairs
    3        3 Walking Downstairs
    4        4            Sitting
    5        5           Standing
    6        6             Laying

**2b) dplyr package solution**

Keying off of Karthik's solution, here is how you would do it with dplyr.  Since we want to join on the column "Activity" which is the same name in both columns, dplyr guesses that that's what we want to join on.  The result is piped to a print statement too.

    library(dplyr)
    result <- data %>%
      left_join(xref) %>% 
      print()
    Joining by: "Activity"
    Activity      Activity_Name
    1        1            Walking
    2        2   Walking Upstairs
    3        3 Walking Downstairs
    4        4            Sitting
    5        5           Standing
    6        6             Laying

**2c) sqldf package solution**

A method using the sqldf package, which is significantly slower than dplyr.

    library(sqldf)
    result <- sqldf("select d.Activity, x.Activity_Name
                 from data d left join
                 xref x on 
                    d.Activity=x.Activity")
                    
    print(result)
    Activity      Activity_Name
    1        1            Walking
    2        2   Walking Upstairs
    3        3 Walking Downstairs
    4        4            Sitting
    5        5           Standing
    6        6             Laying
