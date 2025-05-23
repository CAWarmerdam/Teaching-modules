---
title: "ggplot"
output: 
  html_document:
    toc: true # table of content true
    depth: 3  # upto three depths of headings (specified by #, ## and ###)
    number_sections: true  ## if you want number sections at each table header
    theme: united  # many options for theme, this one is my favorite.
    highlight: tango  # specifies the syntax highlighting style
---

# Introduction
 
You have already explored the Iris dataset by determining differences in petal and sepal size of different types of these plants. In this tutorial you will learn how to visualize these difference using different kinds of plots with an external package called GGplot2.

GGplot2 is an R package created by Hadley Wickham in 2005. It can highly improve the quality and aesthetic of your graphs. R also has standard build-in plotting functions, but for now we will be focusing on GGplot2.

Scroll through [http://www.r-graph-gallery.com/portfolio/ggplot2-package/](http://www.r-graph-gallery.com/portfolio/ggplot2-package/) to see some examples of the plots you can make. Clicking on a plot shows the code that was used to make it. 


```{r install, eval=F}
# if you haven't installed it yet, install ggplot2
install.packages('ggplot2')
```
First, we have to load the libraries needed for plotting the data, using the ```library()``` function. 

```{r}
library(ggplot2)
```

# Plotting

## Easy example
Now we have loaded the ggplot library lots of new methods are available. The first function you always need when making a ggplot is ```ggplot()```. Let's try the basic function ```ggplot()``` and ```geom_point()``` to generate a scatter plot.  

```{r plot, eval=FALSE}
ggplot(iris, aes(x=Sepal.Length, y=Sepal.Width))
```

The first argument in ```ggplot()``` is the dataset you want to use to generate the plot, in this case we're using the iris dataset again. The second argument is the ```aes()``` or aesthetic function, where you can set which variables of the dataset are used for the available visual properties. In the case of this scatterplot we're only interested in the x and y position of each data point.

If everything works properly the above command will generated an empty plot. This is because ggplot doesn't know what kind of plot you wish to make. The function ```geom_point()``` will tell ggplot to make a scatterplot. Adding these two functions  ```ggplot(...) + geom_point()``` will result in a scatterplot.
```{r}
ggplot(iris, aes(x=Sepal.Length, y=Sepal.Width)) + geom_point()
```

__A1. Make a scatterplot between the Petal Length and Petal Width__

## Histrograms
One of the first things to check when you are visualy exploring a dataset is the distribution of it's variables. A common way to do this is by plotting a histogram. With ggplot you can do this easily by using ```geom_histogram()```. For instance we can look at the distribution of petal lengths with the following command:
```{r, message=FALSE, warning=FALSE}
ggplot(iris, aes(x=Petal.Length)) + geom_histogram()
```

As you can see the ```aes()``` function only takes one argument (x) since we are only interested in the distribution of one variable

The length of every bin is the amount of observation within the range (width) of each bin. You can customize the width of each bin with the ```binwidth``` argument. This might be useful if the dataset contains many more observations:
```{r}
ggplot(iris, aes(x=Petal.Length)) + geom_histogram(binwidth = 0.05)
```

__A2. Make a histogram of the Sepal length__

## Boxplots

A box plot is another quick way of examining one or more sets of data graphically. The box shows where the majority of the data points are located, the band in the box is the median and the whiskers shows the complete range of values.  Box plots may seem more primitive than a histogram but they do have some advantages. They take up less space and are therefore particularly useful for comparing distributions between several groups or sets of data.
Since the iris dataset includes different species we can use boxplots to compare the distribution of values between these groups. The aesthetic for the plot for x-axis are the groups defined in Species, and the y axis will be the sepal length. Finally we add the ```geom_boxplot()``` function to tell ggplot we want a boxplot.

```{r}
ggplot(iris, aes(x=Species, y=Sepal.Length)) + geom_boxplot()
```

__A3. Make a boxplot of the Sepal width__


## Saving plots

In ggplot it is possible to add shapes to a plot by simply adding to the ```ggplot()``` function. Therefor it can be convenient to store the result of ```ggplot()``` inside a variable.
```{r}
sepal.length.boxplot <- ggplot(iris, aes(x=Species, y=Sepal.Length)) + geom_boxplot()
```
Notice that this won't generate any graphical output. By calling the variable ```sepal.length.boxplot``` again the plot will be generated.

__A4. Call the variable petal.length.boxplot to draw the plot__

# Improving plots

## Adding shapes

Now we have stored the plot, we can add shapes to the plot by adding to the ```sepal.length.boxplot``` variable. The functions ```geom_hline()``` and ```geom_vline()``` are used to add horizontal and vertical lines to a plot. For instance we can draw a horizontal line over the plot to indicate what the mean is of sepal lengths for the complete dataset. The only argument these functions need is the interception of the x- or y-axis, but it's also possible the define a color.
Let's make the mean stand out and color it red.


```{r}
sepal.width.boxplot <- ggplot(iris, aes(x=Species, y=Sepal.Width)) + geom_boxplot()
# First calculate the mean
sepal.width.mean <- mean(iris$Sepal.Width)
# Add a horizontal line to the plot
sepal.width.boxplot + geom_hline(yintercept = sepal.width.mean, color = "red")
```

__A5. Plot boxplot + mean for Sepal width__


## Adding text

Previously we have determined if there was a significant differences in petal lengths using a t-test. The p-value yielded by this test is very informative and can be added to plot easily. 

__A6. Run a t-test to determine the difference in means of sepal length between *Iris virginica* and *Iris versicolor*, and store the p-value in a variable called t.test.p.value __
```{r, include=FALSE}
t.test.p.value <- t.test(iris[iris$Species == 'virginica',]$Sepal.Length, iris[iris$Species == 'versicolor',]$Sepal.Length)$p.value
```

To annotate the boxplot with the p-value of the t-test, we can use the ```annotate()``` function. We need to provide what type of annotation we want, the x and y position and the actual text of the annotation. But first we have to format the label properly.
```{r}
# Round the p-value to 3 digits
t.test.p.value <- signif(t.test.p.value, 3)
# Create a string by pasting "p-value =" and the p-value
p.value.label <- paste("p-value =", t.test.p.value)
# Plot the boxplot + the annotation
sepal.length.boxplot + annotate("text", x = 1.5, y = 8, label =  p.value.label)
```

__A7. Put the text in the lower right corner__


## Adding layers

While ggplot makes good plots right out off the box, they are also highly adjustable. Basicly every line or text of the plot can be adjusted. Let's enhance our boxplot a bit by adding the actual data points as dots using the ```geom_point``` function.
```{r, eval=FALSE}
sepal.length.boxplot + geom_point()
```
Because we are plotting in one dimension, the above statement will draw all dots on a single line. Since some measurements might overlap it is a good idea to add some random noise to the 'x' position of each point. We can adjust the x position by using the ```position``` argument of the ```geom_point()``` function. The random noise can be added using ```"jitter"``` as value. To move the points a bit to the background we can also adjust the size, color and transparency using the ```size, color, alpha``` arguments.
Let's store the result of the plot in ```sepal.length.boxplot``` again 
```{r}
# Different parts of ggplot are added together using '+'. After the '+' the next part can be 
# one line below. This makes it easier to read
sepal.length.boxplot <- sepal.length.boxplot + 
                        geom_point(position = "jitter", size=1, color=c("blue"), alpha=0.5)
sepal.length.boxplot
```

## Changing axis labels

Sometimes it is usefull to change the sizes of the labels. For instance when you want to use your plots for a presentation or for a poster you want to increase the font size of the labels, so that the people in the back of the room also have an idea what you are talking about. We can do this by using the ```theme()``` function. Type ```?theme``` in your console to see what arguments can be used when calling this function.
As you can see there are a lot of arguments available, we can use the ```axis.title``` argument to adjust both axes at once. To change them separately we could use ```axis.title.x``` and ```axis.title.y```.

```{r, eval=F}
sepal.length.boxplot + theme(axis.title = element_text(size = 15))
```
Also the the text of the label on the y axis "Sepal.Length" is not despriptive enough. All the measurements of the iris dataset are in centimeters. To change the text of the y label we can use the ```ylab()``` function and add it to plot like we are used to. Also store the result inside ```sepal.length.boxplot``` so we don't need to type it again.
```{r}
sepal.length.boxplot <- sepal.length.boxplot + 
                        theme(axis.title = element_text(size = 15)) + 
                        ylab("Sepal length (cm)")
sepal.length.boxplot
```

The size of the text of the species beneath the boxplots can also be increased by adding more arguments to the ```theme()``` function. The argument we need to change the text of axis is ```axis.text.x```

```{r}
sepal.length.boxplot <- sepal.length.boxplot + 
                        theme(axis.text.x = element_text(size = 13))
sepal.length.boxplot
```

To make a beter distinction between the two boxplot we could also color the boxes
```{r}
sepal.length.boxplot + geom_boxplot(aes(fill=Species))
```

As you can see the boxplot is drawn on top of ```geom_point```, this is because we have added the boxplot as another layer on top of the previous plot.

__A8. Recreate the colored boxplot above so that the points lay on top of the boxplot. __

Not everyone likes the grey background. An easy way to remove these (and to make the plot look less like a default ggplot) is to add ```theme_bw()```

```{r}
sepal.length.boxplot + theme_bw()
```

# Grouped plots

## Melting data

There are many situations where data is presented in a format that is not ready to dive straight to exploratory data analysis or to use a desired statistical method. The reshape2 package for R provides useful functionality to avoid having to hack data around in a spreadsheet prior to import into R.

The melt function takes data formatted as a matrix with a set of columns, like our data, and formats it into a single column. For some application of GGplot we need this format. To make use of the function we need to specify a data frame, the id variables (which will be left at their settings) and the measured variables (columns of data) to be stacked. The default assumption on measured variables is that it is all columns which are not specified as id variables.

First load reshape2 library, so we can use the melt function.
```{r}
library(reshape2)
```

Let's have a look at our dataframe again.
```{r}
head(iris)
```
Our id variable is Species, so we use this as value of the ```id.vars``` argument of the melt function. All the other columns are variables we'de like to use and therefor we don't have to specify ```measure.vars``` argument. 
```{r}
iris.melt <- melt(iris, id.vars = "Species")
```

This will format the data in the following way:
```{r}
head(iris.melt)
```


## Facets

Now have have melted our data we can use the facet_wrap function. The facet approach partitions a plot into a matrix of panels. Each panel shows a different subset of the data.

We can plot the molten data frame by plotting the distribution of every variable as boxplots.
```{r, eval=F}
ggplot(iris.melt, aes(x=variable, y=value)) + geom_boxplot()
```
Every variable now contains the value of every Iris species in the dataframe. If we want to plot the distribution of every Species separately we can use the ```face_grid()``` funtion. In this function we only have te specify a formula with the rows (of the tabular display) on the left hand side of ```~``` and the columns (of the tabular display) on the right hand side (RHS) of ```~```; the dot in the formula is used to indicate there should be no faceting on this dimension (either row or column). We only want split up the data into one dimension (columns), and only need the specify the variable on the RHS.

```{r}
ggplot(iris.melt, aes(x=variable, y=value)) + geom_boxplot() + facet_grid(.~Species)
```

It's probably more informative to group the variables instead of the species.

__A9. Use the facet_grid() function so that that the boxplots are grouped by the measured variables like the plot below __

```{r, echo=F}
ggplot(iris.melt, aes(x=Species, y=value)) + geom_boxplot() + facet_grid(~variable)
```

## Violin plots

A violin plot is another method of plotting numeric data. It is similar to box plot with a rotated kernel density plot on each side. The violin plot is similar to box plots, except that they also show the probability density of the data at different values. A violin plot is more informative than a plain box plot. In fact while a box plot only shows summary statistics: mean/median, interquartile ranges and outliers, the violin plot shows the full distribution of the data.
We can make violin plots using the ```geom_violin()``` function.

__A10. Use the geom_violin() function to create a similar plot like the last boxplot, grouped by the measured variables. Also, color the boxes by species and draw actual data points on top of the violin plot (should look like below)  __

Add ```+ theme(axis.text.x=element_text(angle=45, hjust=1))``` to rotate the x-axis labels so they don't overlap.

```{r, echo=F}
ggplot(iris.melt, aes(x=Species, y=value, fill=Species)) + 
  geom_violin() + 
  facet_grid(~variable) + 
  geom_point(position = "jitter", alpha=0.5, size=0.8)+
  theme(axis.text.x=element_text(angle=45, hjust=1))
```

# Using plots to find anomalies

## Coloring points above a threshold

Sometimes you want to quickly see which points are above or below a certain threshold. Coloring can be done by first creating a vector with which points you want to color.

```{r}
# ifelse first parameter is what you want to evaluate, e.g. for every value in 
# iris$Sepal.Length test if it is higher than 7.6 (True) or lower than 7.6 (False).
# The second parameter is the value that it should return if True, the third
# parameter the value it should return if False
outlier <- ifelse(iris$Sepal.Length > 7.6, "Outlier", "Non-outlier")
iris$Sepal.Length
outlier
```

The outlier vector can then be given as a vector to the color parameter. Using ```geom_text``` we also add a label to those points.
```{r}
ggplot(iris, aes(x=Sepal.Length, y=Sepal.Width, color=outlier)) +
  geom_point() +
  geom_text(aes(label = Sepal.Width), data = iris[iris$Sepal.Length>7.6,], color="black")
```

# Simpsons' paradox

One great way of showing the power of good visualization is simpsons paradox. Look again at the Iris correlations from last time:

```{r,echo=F}
# correlation between attributes for all species together
cor(iris[1:4])
```

```{r,message=FALSE}
# correlation between attributes for species separatly
library(dplyr)
cormat_result <- iris %>% group_by(Species) %>%  do(cormat = cor(.[1:4]))
cormat_result[[2]]
```

You might have noticed that the Sepal Width is negatively correlated with Sepal Length when correlating for all species together, but when correlating using the separate species they are all positive. Plotting the species together shows the slight negative correlation:

```{r}
ggplot(data = iris, aes(Sepal.Length, Sepal.Width)) + 
              geom_point() + 
              geom_smooth(method = "lm")
```


__A11. Redo the above plot but segment using the Species. Explain the opposite correlation__

Add ```scales = "free_x"``` as parameter to the ```facet_grid()``` function. Compare the difference before and after adding this. 

# Saving plots to file
Finally, when you are finished with your plots you will want to save them to a file so that you can add them to your presentation or poster. When using ```ggplot``` the easiest way to save is ```ggsave```

```{r, eval=F }
plot1 <- ggplot(data = iris, aes(Sepal.Length, Sepal.Width))+
  geom_point()
# Running ggsave() will save the last plot you made
ggsave('filename.png')
# The file type is based on the extension you use.
ggsave('filename.pdf')
# When making figures for posters or presentations 
# only use either eps or pdf as these do not lose quality when resizing
# (read https://thepoliticalmethodologist.com/2013/11/25/making-high-resolution-graphics-for-academic-publishing/ if you want to know more).

plot2 <- ggplot(data = iris, aes(Sepal.Length, Sepal.Width))+
            geom_boxplot()

# You can save as plot stored in a variable by adding the parameter plot
# This saves plot1 instead of plot2, which would be the case if you didn't add plot=plot1
ggsave('filename.pdf', plot=plot1)

# Usually you will want to change the width, size and resolution. For presentations when using
# one figure per slide you want it a bit wider than high, so you can add width and height.
# For posters you will probably want to use 300 dpi (dots per inch) for resolution
ggsave('filename.pdf', plot=plot1, dpi=300, width=12, height=8)
```

