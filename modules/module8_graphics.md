% R Bootcamp, Module 8: Graphics
% August 2019, UC Berkeley
% Dana Seidel (built off of material by Kellie Ottoboni and Chris Krogslund)



# By way of introduction...

* 3 main facilities for producing graphics in R: **base**, **`ggplot2`**, and **`lattice`**
* In practice, these facilities are grouped into two camps: "basic" and "advanced"
* A better formulation: quick/dirty v. involved/fancy

And here's some motivation - we can produce a plot like [this](gapminder.pdf) with a few lines of code.

(Compare to the [famous gapminder plot](https://s3-eu-west-1.amazonaws.com/static.gapminder.org/GapminderMedia/wp-uploads/20161019161829/screenshot2016.jpg).)


# Base graphics

The general call for base plot looks something like this:


```r
plot(x = , y = , ...)
```
Additional parameters can be passed in to customize the plot:

* type: scatterplot? lines? etc
* main: a title
* xlab, ylab: x-axis and y-axis labels
* col: color, either a string with the color name or a vector of color names for each point

More layers can be added to the plot with additional calls to `lines`, `points`, `text`, etc.




```r
gapChina <- gap %>% filter(country == "China")
plot(gapChina$year, gapChina$gdpPercap)
```

![](figure/unnamed-chunk-2-1.png)

```r
plot(gapChina$year, gapChina$gdpPercap, type = "l",
     main = "China GDP over time",
     xlab = "Year", ylab = "GDP per capita") # with updated parameters
points(gapChina$year, gapChina$gdpPercap, pch = 16)
points(x = 1977, y = gapChina$gdpPercap[gapChina$year == 1977],
       col = "red", pch = 16)
```

![](figure/unnamed-chunk-2-2.png)

# Other plot types in base graphics

These are a variety of other types of plots you can make in base graphics.


```r
boxplot(lifeExp ~ year, data = gap)
```

![](figure/unnamed-chunk-3-1.png)

```r
hist(gap$lifeExp[gap$year == 2007])
```

![](figure/unnamed-chunk-3-2.png)

```r
plot(density(gap$lifeExp[gap$year == 2007]))
```

![](figure/unnamed-chunk-3-3.png)

```r
barplot(gapChina$pop, width = 4, names.arg = gapChina$year, 
                               main = "China population")
```

![](figure/unnamed-chunk-3-4.png)

# Object-oriented plots
* Base graphics often recognizes the object type and will implement specific plot methods
* lattice and ggplot2 generally **don't** exhibit this sort of behavior


```r
gap_lm <- lm(lifeExp ~ log(gdpPercap) + year, data = gap)

# Calls plotting method for class of the dataset ("data.frame")
plot(gap[,c('pop','lifeExp','gdpPercap')])
```

![ ](figure/unnamed-chunk-4-1.png)

```r
# Calls plotting method for class of gap_lm object ("lm"), print first two plots only
plot(gap_lm, which=1:2)
```

![ ](figure/unnamed-chunk-4-2.png)![ ](figure/unnamed-chunk-4-3.png)

# Pros/cons of base graphics, ggplot2, and lattice

Base graphics is

a) good for exploratory data analysis and sanity checks

b) inconsistent in syntax across functions: some take x,y while others take formulas

c) defaults plotting parameters are ugly, and it can be difficult to customize

d) that said, one can do essentially anything in base graphics with some work

`ggplot2` is

a) generally more elegant

b) more syntactically logical (and therefore simpler, once you learn it)

c) better at grouping

d) able to interface with maps

`lattice` is

a) faster than ggplot2 (though only noticeable over many and large plots)

b) simpler than ggplot2 (at first)

c) better at trellis graphs than ggplot2

d) able to do 3d graphs

We'll focus on ggplot2 as it is very powerful, very widely-used and allows one to produce very nice-looking graphics without a lot of coding.


# Basic usage: `ggplot2`

The general call for `ggplot2` graphics looks something like this:


```r
# NOT run
ggplot(data = , aes(x = ,y = , [options])) + geom_xxxx() + ... + ... + ...
```

Note that `ggplot2` graphs in layers in a *continuing call* (hence the endless +...+...+...), which really makes the extra layer part of the call.


```r
... + geom_xxxx(data = , aes(x = , y = ,[options]), [options]) + ... + ... + ...
```
You can see the layering effect by comparing the same graph with different colors for each layer


```r
p <- ggplot(data = gapChina, aes(x = year, y = lifeExp)) +
                 geom_point(color = "red")
p
```

![ ](figure/unnamed-chunk-7-1.png)

```r
p + geom_point(aes(x = year, y = lifeExp), color = "gray") + ylab("life expectancy") +
    theme_minimal()
```

![ ](figure/unnamed-chunk-7-2.png)

# Grammar of Graphics

`ggplot2` syntax is very different from base graphics and lattice. It's built on the **grammar of graphics**.
The basic idea is that the visualization of all data requires four items:

1) One or more **statistics** conveying information about the data (identities, means, medians, etc.)

2) A **coordinate system** that differentiates between the intersections of statistics (at most two for `ggplot`, three for `lattice`)

3) **Geometries** that differentiate between off-coordinate variation in *kind*

4) **Scales** that differentiate between off-coordinate variation in *degree*

`ggplot2` allows the user to manipulate all four of these items.


```r
# Scatterplot
ggplot(gapChina, aes(x = year, y = lifeExp)) + geom_point() +
                          ggtitle("China's life expectancy")
```

![](figure/unnamed-chunk-8-1.png)

```r
# Line (time series) plot
ggplot(gapChina, aes(x = year, y = lifeExp)) + geom_line() +
                          ggtitle("China's life expectancy")
```

![](figure/unnamed-chunk-8-2.png)

```r
# Boxplot
ggplot(gap, aes(x = factor(year), y = lifeExp)) + geom_boxplot() +
                          ggtitle("World's life expectancy")
```

![](figure/unnamed-chunk-8-3.png)

```r
# Histogram
gap2007 <- gap %>% filter(year == 2007)
ggplot(gap2007, aes(x = lifeExp)) + geom_histogram(binwidth = 5) +
                          ggtitle("World's life expectancy")
```

![](figure/unnamed-chunk-8-4.png)


# `ggplot2` and tidy data

* `ggplot2` plays nice with `dplyr` and pipes. If you want to manipulate your data specifically for one plot but not save the new dataset, you can call your `dplyr` chain and pipe it directly into a `ggplot` call.


```r
# This combines the subsetting and plotting into one step
gap %>% filter(year == 2007) %>% 
        ggplot(aes(x = lifeExp)) + geom_histogram(binwidth = 5) +
                          ggtitle("World's life expectancy")
```

![](figure/unnamed-chunk-9-1.png)

* Base graphics/lattice and `ggplot2` have one big difference: `ggplot2` **requires** your data to be in tidy format. For base graphics, it can actually be helpful *not* to have your data in tidy format.

For example, here `ggplot` treats `country` as an aesthetic parameter that differentiates groups of values, whereas base graphics treats each (year, medal) pair as a set of inputs to the plot.

Here's ggplot with the data in a tidy format.



```r
# ggplot2 call
head(gap)
```

```
##       country year      pop continent lifeExp gdpPercap
## 1 Afghanistan 1952  8425333      Asia  28.801  779.4453
## 2 Afghanistan 1957  9240934      Asia  30.332  820.8530
## 3 Afghanistan 1962 10267083      Asia  31.997  853.1007
## 4 Afghanistan 1967 11537966      Asia  34.020  836.1971
## 5 Afghanistan 1972 13079460      Asia  36.088  739.9811
## 6 Afghanistan 1977 14880372      Asia  38.438  786.1134
```

```r
ggplot(data = gap, aes(x = year, y = lifeExp)) +
            geom_line(aes(color = country), show.legend = FALSE)
```

![](figure/unnamed-chunk-10-1.png)

Is that a useful plot? 

And here's use of base graphics, taking advantage of non-tidy, wide-formatted data.


```r
# Base graphics call
gap_wide <- gap %>% select(country, year, lifeExp) %>% spread(country, lifeExp)
gap_wide[1:5, 1:5]
```

```
##   year Afghanistan Albania Algeria Angola
## 1 1952      28.801   55.23  43.077 30.015
## 2 1957      30.332   59.28  45.685 31.999
## 3 1962      31.997   64.82  48.303 34.000
## 4 1967      34.020   66.22  51.407 35.985
## 5 1972      36.088   67.69  54.518 37.928
```

```r
plot(gap_wide$year, gap_wide$China, col = 'red', type = 'l', ylim = c(40, 85))
lines(gap_wide$year, gap_wide$Turkey, col = 'green')
lines(gap_wide$year, gap_wide$Italy, col = 'blue')
legend("right", legend = c("China", "Turkey", "Italy"),
                fill = c("red", "green", "blue"))
```

![](figure/unnamed-chunk-11-1.png)


# Pros/cons of `ggplot2`

* Allows you to add features in "layers"
* Automatically adjusts spacing and sizing as you add more layers
* Requires data to be in tidy format
* Syntax is different from base R -- there is a learning curve
* Plots are actually objects. You can assign them to a variable and do things with it (more on this later)

# An overview of syntax for various `ggplot2` plots

We've already seen these initial ones.

X-Y scatter plots:


```r
ggplot(gapChina, aes(x = year, y = lifeExp)) + geom_point() +
                          ggtitle("China's life expectancy")
```

![ ](figure/unnamed-chunk-12-1.png)

X-Y line plots: 


```r
ggplot(gapChina, aes(x = year, y = lifeExp)) + geom_line() +
                          ggtitle("China's life expectancy")
```

![ ](figure/unnamed-chunk-13-1.png)

Histograms:


```r
gap2007 <- gap %>% filter(year == 2007)
ggplot(gap2007, aes(x = lifeExp)) + geom_histogram(binwidth = 5) +
                          ggtitle("World's life expectancy")
```

![](figure/unnamed-chunk-14-1.png)

Densities:


```r
ggplot(gap2007, aes(x = lifeExp)) + geom_density() + 
                          ggtitle("World's life expectancy")
```

![ ](figure/unnamed-chunk-15-1.png)

Boxplots:


```r
# Notice that here, you must explicitly convert numeric years to factors
ggplot(data = gap, aes(x = factor(year), y = lifeExp)) +
            geom_boxplot() 
```

![ ](figure/unnamed-chunk-16-1.png)


"Trellis" plots:


```r
ggplot(data = gap, aes(x = lifeExp)) + geom_histogram(binwidth = 5) +
            facet_wrap(~year)
```

![ ](figure/unnamed-chunk-17-1.png)

Contour plots:


```r
data(volcano) # Load volcano contour data
volcano[1:10, 1:10] # Examine volcano dataset (first 10 rows and columns)
```

```
##       [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
##  [1,]  100  100  101  101  101  101  101  100  100   100
##  [2,]  101  101  102  102  102  102  102  101  101   101
##  [3,]  102  102  103  103  103  103  103  102  102   102
##  [4,]  103  103  104  104  104  104  104  103  103   103
##  [5,]  104  104  105  105  105  105  105  104  104   103
##  [6,]  105  105  105  106  106  106  106  105  105   104
##  [7,]  105  106  106  107  107  107  107  106  106   105
##  [8,]  106  107  107  108  108  108  108  107  107   106
##  [9,]  107  108  108  109  109  109  109  108  108   107
## [10,]  108  109  109  110  110  110  110  109  109   108
```

```r
volcano3d <- melt(volcano) # Use reshape2 package to melt the data into tidy form
head(volcano3d) # Examine volcano3d dataset (head)
```

```
##   Var1 Var2 value
## 1    1    1   100
## 2    2    1   101
## 3    3    1   102
## 4    4    1   103
## 5    5    1   104
## 6    6    1   105
```

```r
names(volcano3d) <- c("xvar", "yvar", "zvar") # Rename volcano3d columns

ggplot(data = volcano3d, aes(x = xvar, y = yvar, z = zvar)) +
            geom_contour() 
```

![ ](figure/unnamed-chunk-18-1.png)

tile/image/level plots:


```r
ggplot(data = volcano3d, aes(x = xvar, y = yvar, z = zvar)) +
            geom_tile(aes(fill = zvar)) 
```

![ ](figure/unnamed-chunk-19-1.png)

# Anatomy of `aes()`


```r
# NOT run
ggplot(data = , aes(x = , y = , color = , linetype = , shape = , size = ))
```

These four aesthetic parameters (`color`, `linetype`, `shape`, `size`) can be used to show variation in *kind* (categories) and variation in *degree* (numeric).

Parameters passed into `aes` should be *variables* in your dataset.

Parameters passed to `geom_xxx` outside of `aes` should *not* be related to your dataset -- they apply to the whole figure.


```r
ggplot(data = gap, aes(x = year, y = lifeExp)) +
            geom_line(aes(color = country), show.legend = FALSE)
```

![ ](figure/unnamed-chunk-21-1.png)

Note what happens when we specify the color parameter outside of the aesthetic operator. `ggplot2` views these specifications as invalid graphical parameters.


```r
ggplot(data = gap, aes(x = year, y = lifeExp)) +
            geom_line(color = country)
```

```
## Error in layer(data = data, mapping = mapping, stat = stat, geom = GeomLine, : object 'country' not found
```

```r
ggplot(data = gap, aes(x = year, y = lifeExp)) +
            geom_line(color = "country")
```

```
## Error in grDevices::col2rgb(colour, TRUE): invalid color name 'country'
```

```r
## this works but only makes sense if we restrict to one country
ggplot(data = gapChina, aes(x = year, y = lifeExp)) +
            geom_line(color = "red")
```

![ ](figure/unnamed-chunk-22-1.png)

# Using aesthetics to highlight features

Differences in kind


```r
## color as the aesthetic to differentiate by continent
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(aes(color = continent)) + scale_x_log10()

## point shape as the aesthetic to differentiate by continent
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(aes(shape = continent)) + scale_x_log10()

## line type as the aesthetic to differentiate by country
gapOceania <- gap %>% filter(continent %in% 'Oceania')
ggplot(data = gapOceania, aes(x = year, y = lifeExp)) +
            geom_line(aes(linetype = country)) + scale_x_log10()
```

![ ](figure/unnamed-chunk-23-1.png)![ ](figure/unnamed-chunk-23-2.png)![ ](figure/unnamed-chunk-23-3.png)

Differences in degree


```r
## point size as the aesthetic to differentiate by population
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(aes(size = pop)) + scale_x_log10()

## color as the aesthetic to differentiate by population
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(aes(color = pop)) + scale_x_log10() +
            scale_color_gradient(low = 'lightgray', high = 'black')
```

![ ](figure/unnamed-chunk-24-1.png)![ ](figure/unnamed-chunk-24-2.png)

Multiple non-coordinate aesthetics (differences in kind using color, degree using point size)


```r
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(aes(size = pop, color = continent)) + scale_x_log10()
```

![ ](figure/unnamed-chunk-25-1.png)

# Changing options in ggplot2

`ggplot` handles options in additional layers.

### Labels


```r
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() +
  xlab(label = "GDP per capita") +
  ylab(label = "Life expectancy") +
  ggtitle(label = "Gapminder") 
```

![ ](figure/unnamed-chunk-26-1.png)

### Axis and point scales


```r
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point() 
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(size=3) 
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(size=1) 
```

![ ](figure/unnamed-chunk-27-1.png)![ ](figure/unnamed-chunk-27-2.png)![ ](figure/unnamed-chunk-27-3.png)

### Colors

```r
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(color = colors()[11]) 
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(color = "red") 
```

![ ](figure/unnamed-chunk-28-1.png)![ ](figure/unnamed-chunk-28-2.png)

### Point Styles and Widths


```r
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(shape = 3) 
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(shape = "w") 
ggplot(data = gap, aes(x = gdpPercap, y = lifeExp)) +
            geom_point(shape = "$", size=5) 
```

![ ](figure/unnamed-chunk-29-1.png)![ ](figure/unnamed-chunk-29-2.png)![ ](figure/unnamed-chunk-29-3.png)

### Line Styles and Widths


```r
ggplot(data = gapChina, aes(x = year, y = lifeExp)) +
            geom_line(linetype = 1) 
ggplot(data = gapChina, aes(x = year, y = lifeExp)) +
            geom_line(linetype = 2) 
ggplot(data = gapChina, aes(x = year, y = lifeExp)) +
            geom_line(linetype = 5, size = 2) 
```

![ ](figure/unnamed-chunk-30-1.png)![ ](figure/unnamed-chunk-30-2.png)![ ](figure/unnamed-chunk-30-3.png)

# Fitted lines and curves with `ggplot2`


```r
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() + scale_x_log10()
```

![ ](figure/unnamed-chunk-31-1.png)

```r
# Add linear model (lm) smoother
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() + scale_x_log10() +
  geom_smooth(method = "lm")
```

![ ](figure/unnamed-chunk-31-2.png)

```r
# Add local linear model (loess) smoother, span of 0.75 (more smoothed)
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() + scale_x_log10() +
  geom_smooth(method = "loess", span = .75)
```

![ ](figure/unnamed-chunk-31-3.png)

```r
# Add local linear model (loess) smoother, span of 0.25 (less smoothed)
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() + scale_x_log10() +
  geom_smooth(method = "loess", span = .25)
```

![ ](figure/unnamed-chunk-31-4.png)

```r
# Add linear model (lm) smoother, no standard error shading
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() + scale_x_log10() +
  geom_smooth(method = "lm", se = FALSE)
```

![ ](figure/unnamed-chunk-31-5.png)

```r
# Add local linear model (loess) smoother, no standard error shading
ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) + geom_point() + scale_x_log10() +
  geom_smooth(method = "loess", se = FALSE)
```

![ ](figure/unnamed-chunk-31-6.png)

# Combining Multiple Plots

* `ggplot2` graphs can be combined using the *`grid.arrange()`* function in the **`gridExtra`** package


```r
# Initialize gridExtra library
library(gridExtra)

# Create 3 plots to combine in a table
plot1 <- ggplot(data = gap2007, aes(x = gdpPercap, y = lifeExp)) +
  geom_point() + scale_x_log10()
plot2 <- ggplot(data = gap2007, aes(x = pop, y = lifeExp)) +
  geom_point() + scale_x_log10()
plot3 <- ggplot(data = gap, aes(x = year, y = lifeExp)) +
      geom_line(aes(color = country), show.legend = FALSE)


# Call grid.arrange
grid.arrange(plot1, plot2, plot3, nrow=3, ncol = 1)
```

![ ](figure/unnamed-chunk-32-1.png)

# `patchwork`: Combining Multiple `ggplot2` plots

* The `patchwork` package may be used to combine multiple `ggplot2` plots using
  a small set of operators similar to the pipe.
* This requires less syntax than using `gridExtra` and allows complex
  arrangements to be built nearly effortlessly.


```r
# Install and initialize patchwork library
# devtools::install_github("thomasp85/patchwork")
library(patchwork)

# use the patchwork operators
# stack plots horizontally
plot1 + plot2 + plot3
```

![ ](figure/unnamed-chunk-33-1.png)

```r
# stack plots vertically
plot1 / plot2 / plot3
```

![ ](figure/unnamed-chunk-33-2.png)

```r
# side-by-side plots with third plot below
(plot1 | plot2) / plot3
```

![ ](figure/unnamed-chunk-33-3.png)

```r
# side-by-side plots with a space in between, and a third plot below
(plot1 | plot_spacer() | plot2) / plot3
```

![ ](figure/unnamed-chunk-33-4.png)

```r
# stack plots vertically and alter with a single "gg_theme"
(plot1 / plot2 / plot3) & theme_bw()
```

![ ](figure/unnamed-chunk-33-5.png)

Feel free to explore more at [https://github.com/thomasp85/patchwork](https://github.com/thomasp85/patchwork).

# Exporting

Two basic image types:

### **Raster/Bitmap** (.png, .jpeg)

Every pixel of a plot contains its own separate coding; not so great if you want to resize the image


```r
jpeg(filename = "example.jpg", width=, height=)
plot(x,y)
dev.off()
```

### **Vector** (.pdf, .ps)

Every element of a plot is encoded with a function that gives its coding conditional on several factors; great for resizing


```r
# NOT run
pdf(file = "example.pdf", width=, height=)
plot(x,y)
dev.off()
```

### Exporting with `ggplot`


```r
# NOT run

# Assume we saved our plot is an object called `plot1`.

ggsave(filename = "example.pdf", plot = plot1, scale = , width = ,
       height = )
```


# Breakout

These questions ask you to work with the gapminder dataset.

### Basics

1) Plot a histogram of life expectancy. 

2) Plot the gdp per capita against population. Put the x-axis on the log scale.

3) Clean up your scatterplot with a title and axis labels. Output it as a PDF and see if you'd be comfortable with including it in a report/paper.

### Using the ideas

4) Create a trellis plot of life expectancy by gdpPercap scatterplots, one subplot per continent. Use a 2x3 layout of panels in the plot. Now have the size of the points vary with population. Use `scale_x_continuous()` to set the x-axis limits to be in the range from 100 to 50000.

5) Make a boxplot of life expectancy conditional on binned values of gdp per capita.

### Advanced

6) Using the data for 2007, recreate as much as you can of [this famous Gapminder plot](https://s3-eu-west-1.amazonaws.com/static.gapminder.org/GapminderMedia/wp-uploads/20161019161829/screenshot2016.jpg), where the colors are different continents. (Don't worry about the '2015' in the background and ignore the 'play' button at the bottom.)

7) Create a "trellis" plot where, for a given year, each panel uses a) hollow circles to plot lifeExp as a function of log(gdpPercap), and b) a red loess smoother without standard errors to plot the trend. Turn off the grey background. Figure out how to use partially-transparent points to reduce the effect of the overplotting of points.


