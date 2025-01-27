# Appendix A: Visualization

In this lecture, we will take a look at how to visualize data using the powerful [ggplot2](https://ggplot2.tidyverse.org/) package. We will use `ggplot2` a lot throughout the course! 

## Learning goals

- Take a look at some suboptimal plots, and think about how to make them better.
- Understand the general philosophy behind `ggplot2` -- a grammar of graphics. 
- Understand the mapping from data to geoms in `ggplot2`.
- Create informative figures using grouping and facets. 

## Load packages

Let's first load the packages that we need for this chapter. You can click on the green arrow to execute the code chunk below. 


```r
library("knitr")     # for rendering the RMarkdown file
library("tidyverse") # for plotting (and many more cool things we'll discover later)
```

```
## Warning: package 'ggplot2' was built under R version 4.3.1
```

```
## Warning: package 'dplyr' was built under R version 4.3.1
```

```r
# these options here change the formatting of how comments are rendered
# in RMarkdown 
opts_chunk$set(comment = "",
               fig.show = "hold")
```

The `tidyverse` is a collection of packages that includes `ggplot2`.

## Why visualize data?

<div class="figure">
<img src="figures/hiding_data.png" alt="Are you hiding anything?" width="95%" />
<p class="caption">(\#fig:hiding)Are you hiding anything?</p>
</div>

> The greatest value of a picture is when it forces us to notice what we never expected to see. — John Tukey

> There is no single statistical tool that is as powerful as a well‐chosen graph. [@chambers1983graphical]

> ...make __both__ calculations __and__ graphs. Both sorts of output should be studied; each will contribute to understanding. [@anscombe1973american]

<div class="figure">
<img src="figures/anscombe.png" alt="Anscombe's quartet." width="95%" />
<p class="caption">(\#fig:anscombe)Anscombe's quartet.</p>
</div>

Anscombe's quartet in Figure \@ref(fig:anscombe) (left side) illustrates the importance of visualizing data. Even though the datasets I-IV have the same summary statistics (mean, standard deviation, correlation), they are importantly different from each other. On the right side, we have four data sets with the same summary statistics that are very similar to each other.

<div class="figure">
<img src="figures/correlations.png" alt="The Pearson's $r$ correlation coefficient is the same for all of these datasets. Source: [Data Visualization -- A practical introduction by Kieran Healy](http://socviz.co/lookatdata.html#lookatdata)" width="95%" />
<p class="caption">(\#fig:healy)The Pearson's $r$ correlation coefficient is the same for all of these datasets. Source: [Data Visualization -- A practical introduction by Kieran Healy](http://socviz.co/lookatdata.html#lookatdata)</p>
</div>
All the datasets in Figure \@ref(fig:healy) share the same correlation coefficient. However, again, they are very different from each other.

<div class="figure">
<img src="figures/datasaurus_dozen.png" alt="__The Datasaurus Dozen__. While different in appearance, each dataset has the same summary statistics to two decimal places (mean, standard deviation, and Pearson's correlation)." width="95%" />
<p class="caption">(\#fig:datasaurus)__The Datasaurus Dozen__. While different in appearance, each dataset has the same summary statistics to two decimal places (mean, standard deviation, and Pearson's correlation).</p>
</div>

The data sets in Figure \@ref(fig:datasaurus) all share the same summary statistics. Clearly, the data sets are not the same though.

> __Tip__: Always plot the data first!

[Here](https://www.autodeskresearch.com/publications/samestats) is the paper from which I took Figure \@ref(fig:datasaurus). It explains how the figures were generated and shows more examples for how summary statistics and some kinds of plots are insufficient to get a good sense for what's going on in the data.

## Some basics

### Setting up RStudio

<div class="figure">
<img src="figures/r_preferences_general.png" alt="General preferences." width="50%" />
<p class="caption">(\#fig:unnamed-chunk-2)General preferences.</p>
</div>

__Make sure that__:

- Restore .RData into workspace at startup is _unselected_
- Save workspace to .RData on exit is set to _Never_

This can otherwise cause problems with reproducibility and weird behavior between R sessions because certain things may still be saved in your workspace.

<div class="figure">
<img src="figures/r_preferences_code.png" alt="Code window preferences." width="95%" />
<p class="caption">(\#fig:unnamed-chunk-3)Code window preferences.</p>
</div>

__Make sure that__:

- Soft-wrap R source files is _selected_

This way you don't have to scroll horizontally. At the same time, avoid writing long single lines of code. For example, instead of writing code like so:


```r
ggplot(data = diamonds, mapping = aes(x = cut, y = price)) +
  stat_summary(fun = "mean", geom = "bar", color = "black", fill = "lightblue", width = 0.85) +
  stat_summary(fun.data = "mean_cl_boot", geom = "linerange", size = 1.5) +
  labs(title = "Price as a function of quality of cut", subtitle = "Note: The price is in US dollars", tag = "A", x = "Quality of the cut", y = "Price")
```

You may want to write it this way instead:


```r
ggplot(data = diamonds, 
       mapping = aes(x = cut,
                     y = price)) +
  # display the means
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black",
               fill = "lightblue",
               width = 0.85) +
  # display the error bars
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               size = 1.5) +
  # change labels
  labs(title = "Price as a function of quality of cut",
       subtitle = "Note: The price is in US dollars", # we might want to change this later
       tag = "A",
       x = "Quality of the cut",
       y = "Price")
```

This makes it much easier to see what's going on, and you can easily add comments to individual lines of code.

>__Tip__: If a function has more than two arguments put each argument on a new line.

RStudio makes it easy to write nice code. It figures out where to put the next line of code when you press `ENTER`. And if things ever get messy, just select the code of interest and hit `cmd + i` to re-indent the code.

Here are some more resources with tips for how to write nice code in R:

- [Advanced R style guide](http://adv-r.had.co.nz/Style.html)

>__Tip__: Use a consistent coding style. This makes reading code and debugging much easier! 

### Getting help

There are three simple ways to get help in R. You can either put a `?` in front of the function you'd like to learn more about, or use the `help()` function.


```r
?print
help("print")
```

>__Tip__: To see the help file, hover over a function (or dataset) with the mouse (or select the text) and then press `F1`.

I recommend using `F1` to get to help files -- it's the fastest way!

R help files can sometimes look a little cryptic. Most R help files have the following sections (copied from [here](https://www.dummies.com/programming/r/r-for-dummies-cheat-sheet/)):

---

__Title__: A one-sentence overview of the function.

__Description__: An introduction to the high-level objectives of the function.

__Usage__: A description of the syntax of the function (in other words, how the function is called). This is where you find all the arguments that you can supply to the function, as well as any default values of these arguments.

__Arguments__: A description of each argument. Usually this includes a specification of the class (for example, character, numeric, list, and so on). This section is an important one to understand, because arguments are frequently a cause of errors in R.

__Details__: Extended details about how the function works, provides longer descriptions of the various ways to call the function (if applicable), and a longer discussion of the arguments.

__Value__: A description of the class of the value returned by the function.

__See also__: Links to other relevant functions. In most of the R editors, you can click these links to read the Help files for these functions.

__Examples__: Worked examples of real R code that you can paste into your console and run.

---

Here is the help file for the `print()` function:

<div class="figure">
<img src="figures/help_print.png" alt="Help file for the print() function." width="95%" />
<p class="caption">(\#fig:unnamed-chunk-7)Help file for the print() function.</p>
</div>

### R Markdown infos

An RMarkdown file has four key components: 

1. YAML header 
2. Headings to structure the document
3. Text 
4. Code chunks 

The **YAML** (*Y*et *A*nother *M*arkdown *L*anguage) header specifies general options such as whether you'd like to have a table of content displayed, and in what output format you want to create your report (e.g. html or pdf). Notice that the YAML header cares about indentation, so make sure to get that right!  

**Headings** are very useful for structuring your RMarkdown file. For your reports, it's often a good idea to have one header for each code chunk. The outline viewer here on the right is great for navigating large analysis files. 

**Text** is self-explanatory. 

**Code chunks** is where the coding happens. You can add one via the Insert button above, or via the shortcut `cmd + option + i` (the much cooler way of doing it!)



Code chunks can have code chunk options which we can set by clicking on the cog symbol on the right. You can also give code chunks a name, so that we can refer to it in text. I've named the one above "another-code-chunk". Make sure to have no white space or underscore in a code chunk name. 

### Helpful keyboard shortcuts

- `cmd + enter`: run selected code 
- `cmd + shift + enter`: run code chunk 
- `cmd + i`: re-indent selected code 
- `cmd + shift + c`: comment/uncomment several lines of code 
- `cmd + shift + d`: duplicate line underneath 
- set up your own shortcuts to do useful things like 
  - switch tabs 
  - jump up and down between code chunks 
  - ... 

## Data visualization

We will use the `ggplot2` package to visualize data. By the end of next class, you'll be able to make a figure like this:

<div class="figure">
<img src="figures/combined_plot.png" alt="What a nice figure!" width="95%" />
<p class="caption">(\#fig:unnamed-chunk-8)What a nice figure!</p>
</div>

Now let's figure out (pun intended!) how to get there.

### Setting up a plot

Let's first get some data.


```r
df.diamonds = diamonds
```

The `diamonds` dataset comes with the `ggplot2` package. We can get a description of the dataset by running the following command:


```r
?diamonds
```

Above, we assigned the `diamonds` dataset to the variable `df.diamonds` so that we can see it in the data explorer.

Let's take a look at the full dataset by clicking on it in the explorer.

>__Tip__: You can view a data frame by highlighting the text in the editor (or simply moving the mouse above the text), and then pressing `F2`.

The `df.diamonds` data frame contains information about almost 60,000 diamonds, including their `price`, `carat` value, size, etc. Let's use visualization to get a better sense for this dataset.

We start by setting up the plot. To do so, we pass a data frame to the function `ggplot()` in the following way.


```r
ggplot(data = df.diamonds)
```

<img src="29-visualization_files/figure-html/unnamed-chunk-11-1.png" width="672" />

This, by itself, won't do anything yet. We also need to specify what to plot.

Let's take a look at how much diamonds of different color cost. The help file says that diamonds labeled D have the best color, and diamonds labeled J the worst color. Let's make a bar plot that shows the average price of diamonds for different colors.

We do so via specifying a mapping from the data to the plot aesthetics with the function `aes()`. We need to tell `aes()` what we would like to display on the x-axis, and the y-axis of the plot.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-12-1.png" width="672" />

Here, we specified that we want to plot `color` on the x-axis, and `price` on the y-axis. As you can see, `ggplot2` has already figured out how to label the axes. However, we still need to specify _how_ to plot it. 

### Bar plot

Let's make a __bar graph__:


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price)) +
  stat_summary(fun = "mean",
               geom = "bar")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-13-1.png" width="672" />

Neat! Three lines of code produce an almost-publication-ready plot (to be published in the _Proceedings of Unnecessary Diamonds_)! Note how we used a `+` at the end of the first line of code to specify that there will be more. This is a very powerful idea underlying `ggplot2`. We can start simple and keep adding things to the plot step by step.

We used the `stat_summary()` function to define _what_ we want to plot (the "mean"), and _how_ (as a "bar" chart). Let's take a closer look at that function.


```r
help(stat_summary)
```

Not the the easiest help file ... We supplied two arguments to the function, `fun = ` and `geom = `.

1. The `fun` argument specifies _what_ function we'd like to apply to the data for each value of `x`. Here, we said that we would like to take the `mean` and we specified that as a string.
2. The `geom` (= geometric object) argument specifies _how_ we would like to plot the result, namely as a "bar" plot.

Instead of showing the "mean", we could also show the "median" instead.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price)) +
  stat_summary(fun = "median",
               geom = "bar")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-15-1.png" width="672" />

And instead of making a bar plot, we could plot some points.


```r
ggplot(df.diamonds,
       aes(x = color,
           y = price)) +
  stat_summary(fun = "mean",
               geom = "point")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-16-1.png" width="672" />

>__Tip__: Take a look [here](https://ggplot2.tidyverse.org/reference/#section-layer-geoms) to see what other geoms ggplot2 supports.

Somewhat surprisingly, diamonds with the best color (D) are not the most expensive ones. What's going on here? We'll need to do some more exploration to figure this out.

### Setting the default plot theme

Before moving on, let's set a different default theme for our plots. Personally, I'm not a big fan of the gray background and the white grid lines. Also, the default size of the text should be bigger. We can change the default theme using the `theme_set()` function like so:


```r
theme_set(theme_classic() + # set the theme
            theme(text = element_text(size = 20))) # set the default text size
```

From now on, all our plots will use what's specified in `theme_classic()`, and the default text size will be larger, too. For any individual plot, we can still override these settings.

### Scatter plot

I don't know much about diamonds, but I do know that diamonds with a higher `carat` value tend to be more expensive. `color` was a discrete variable with seven different values. `carat`, however, is a continuous variable. We want to see how the price of diamonds differs as a function of the `carat` value. Since we are interested in the relationship between two continuous variables, plotting a bar graph won't work. Instead, let's make a __scatter plot__. Let's put the `carat` value on the x-axis, and the `price` on the y-axis.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = carat,
                     y = price)) +
  geom_point()
```

<div class="figure">
<img src="29-visualization_files/figure-html/scatter-1.png" alt="Scatterplot." width="672" />
<p class="caption">(\#fig:scatter)Scatterplot.</p>
</div>

Cool! That looks sensible. Diamonds with a higher `carat` value tend to have a higher `price`. Our dataset has 53940 rows. So the plot actually shows 53940 circles even though we can't see all of them since they overlap.

Let's make some progress on trying to figure out why the diamonds with the better color weren't the most expensive ones on average. We'll add some color to the scatter plot in Figure \@ref(fig:scatter). We color each of the points based on the diamond's color. To do so, we pass another argument to the aesthetics of the plot via `aes()`.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = carat,
                     y = price,
                     color = color)) +
  geom_point()
```

<div class="figure">
<img src="29-visualization_files/figure-html/scatter-color-1.png" alt="Scatterplot with color." width="672" />
<p class="caption">(\#fig:scatter-color)Scatterplot with color.</p>
</div>

Aha! Now we've got some color. Notice how in Figure \@ref(fig:scatter-color) `ggplot2` added a legend for us, thanks! We'll see later how to play around with legends. Form just eye-balling the plot, it looks like the diamonds with the best `color` (D) tended to have a lower `carat` value, and the ones with the worst `color` (J), tended to have the highest carat values.

So this is why diamonds with better colors are less expensive -- these diamonds have a lower carat value overall.

There are many other things that we can define in `aes()`. Take a quick look at the vignette:


```r
vignette("ggplot2-specs")
```

#### Practice plot 1

Make a scatter plot that shows the relationship between the variables `depth` (on the x-axis), and `table` (on the y-axis). Take a look at the description for the `diamonds` dataset so you know what these different variables mean. Your plot should look like the one shown in Figure \@ref(fig:practice-plot1).


```r
# make practice plot 1 here
```


```r
include_graphics("figures/vis1_practice_plot1.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot1.png" alt="Practice plot 1." width="95%" />
<p class="caption">(\#fig:practice-plot1)Practice plot 1.</p>
</div>

__Advanced__: A neat trick to get a better sense for the data here is to add transparency. Your plot should look like the one shown in Figure \@ref(fig:practice-plot1a).


```r
# make advanced practice plot 1 here
```


```r
include_graphics("figures/vis1_practice_plot1a.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot1a.png" alt="Practice plot 1." width="95%" />
<p class="caption">(\#fig:practice-plot1a)Practice plot 1.</p>
</div>

### Line plot

What else do we know about the diamonds? We actually know the quality of how they were cut. The `cut` variable ranges from "Fair" to "Ideal". First, let's take a look at the relationship between `cut` and `price`. This time, we'll make a line plot instead of a bar plot (just because we can).


```r
ggplot(data = df.diamonds,
       mapping = aes(x = cut,
                     y = price)) +
  stat_summary(fun = "mean",
               geom = "line")
```

```
`geom_line()`: Each group consists of only one observation.
ℹ Do you need to adjust the group aesthetic?
```

<img src="29-visualization_files/figure-html/unnamed-chunk-21-1.png" width="672" />

Oops! All we did is that we replaced `x = color` with `x = cut`, and `geom = "bar"` with `geom = "line"`. However, the plot doesn't look like expected (i.e. there is no real plot). What happened here? The reason is that the line plot needs to know which points to connect. The error message tells us that each group consists of only one observation. Let's adjust the group aesthetic to fix this.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = cut,
                     y = price,
                     group = 1)) +
  stat_summary(fun = "mean",
               geom = "line")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-22-1.png" width="672" />

By adding the parameter `group = 1` to `mapping = aes()`, we specify that we would like all the levels in `x = cut` to be treated as coming from the same group. The reason for this is that `cut` (our x-axis variable) is a factor (and not a numeric variable), so, by default, `ggplot2` tries to draw a separate line for each factor level. We'll learn more about grouping below (and about factors later).

Interestingly, there is no simple relationship between the quality of the cut and the price of the diamond. In fact, "Ideal" diamonds tend to be cheapest.

### Adding error bars

We often don't just want to show the means but also give a sense for how much the data varies. `ggplot2` has some convenient ways of specifying error bars. Let's take a look at how much `price` varies as a function of `clarity` (another variable in our `diamonds` data frame).


```r
ggplot(data = df.diamonds,
       mapping = aes(x = clarity,
                     y = price)) +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "pointrange")
```

<div class="figure">
<img src="29-visualization_files/figure-html/errorbars-normal-1.png" alt="Relationship between diamond clarity and price. Error bars indicate 95% bootstrapped confidence intervals." width="672" />
<p class="caption">(\#fig:errorbars-normal)Relationship between diamond clarity and price. Error bars indicate 95% bootstrapped confidence intervals.</p>
</div>

Here we have it. The average price of our diamonds for different levels of `clarity` together with bootstrapped 95% confidence intervals. How do we know that we have 95% confidence intervals? That's what `mean_cl_boot()` computes as a default. Let's take a look at that function:


```r
help(mean_cl_boot)
```

Note that I had to use the `fun.data = ` argument here instead of `fun = ` because the `mean_cl_boot()` function produces three data points for each value of the x-axis (the mean, lower, and upper confidence interval). 

### Order matters

The order in which we add geoms to a ggplot matters! Generally, we want to plot error bars before the points that represent the means. To illustrate, let's set the color in which we show the means to "red".


```r
ggplot(data = df.diamonds,
       mapping = aes(x = clarity,
                     y = price)) +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange") +
  stat_summary(fun = "mean",
               geom = "point",
               color = "red")
```

<div class="figure">
<img src="29-visualization_files/figure-html/good-figure-1.png" alt="This figure looks good. Error bars and means are drawn in the correct order." width="672" />
<p class="caption">(\#fig:good-figure)This figure looks good. Error bars and means are drawn in the correct order.</p>
</div>

Figure \@ref(fig:good-figure) looks good.


```r
# I've changed the order in which the means and error bars are drawn.
ggplot(df.diamonds,
       aes(x = clarity,
           y = price)) +
  stat_summary(fun = "mean",
               geom = "point",
               color = "red") +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange")
```

<div class="figure">
<img src="29-visualization_files/figure-html/bad-figure-1.png" alt="This figure looks bad. Error bars and means are drawn in the incorrect order." width="672" />
<p class="caption">(\#fig:bad-figure)This figure looks bad. Error bars and means are drawn in the incorrect order.</p>
</div>

Figure \@ref(fig:bad-figure) doesn't look good. The error bars are on top of the points that represent the means.

One cool feature about using `stat_summary()` is that we did not have to change anything about the data frame that we used to make the plots. We directly used our raw data instead of having to make separate data frames that contain the relevant information (such as the means and the confidence intervals).

You may not remember exactly what confidence intervals actually are. Don't worry! We'll have a recap later in class.

Let's take a look at two more principles for plotting data that are extremely helpful: groups and facets. But before, another practice plot. 

#### Practice plot 2

Make a bar plot that shows the average `price` of diamonds (on the y-axis) as a function of their `clarity` (on the x-axis). Also add error bars. Your plot should look like the one shown in Figure \@ref(fig:practice-plot2).


```r
# make practice plot 2 here
```


```r
include_graphics("figures/vis1_practice_plot2.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot2.png" alt="Practice plot 2." width="95%" />
<p class="caption">(\#fig:practice-plot2)Practice plot 2.</p>
</div>

__Advanced__: Try to make the plot shown in Figure \@ref(fig:practice-plot2a).


```r
# make advanced practice plot 2 here
```


```r
include_graphics("figures/vis1_practice_plot2a.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot2a.png" alt="Practice plot 2." width="95%" />
<p class="caption">(\#fig:practice-plot2a)Practice plot 2.</p>
</div>

### Grouping data

Grouping in `ggplot2` is a very powerful idea. It allows us to plot subsets of the data -- again without the need to make separate data frames first.

Let's make a plot that shows the relationship between `price` and `color` separately for the different qualities of `cut`.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     group = cut)) +
  stat_summary(fun = "mean",
               geom = "line")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-26-1.png" width="672" />

Well, we got some separate lines here but we don't know which line corresponds to which cut. Let's add some color!


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     group = cut,
                     color = cut)) +
  stat_summary(fun = "mean",
               geom = "line",
               size = 2)
```

```
Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
ℹ Please use `linewidth` instead.
This warning is displayed once every 8 hours.
Call `lifecycle::last_lifecycle_warnings()` to see where this warning was generated.
```

<img src="29-visualization_files/figure-html/unnamed-chunk-27-1.png" width="672" />

Nice! In addition to adding color, I've made the lines a little thicker here by setting the `size` argument to 2.

Grouping is very useful for bar plots. Let's take a look at how the average price of diamonds looks like taking into account both `cut` and `color` (I know -- exciting times!). Let's put the `color` on the x-axis and then group by the `cut`.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     group = cut,
                     color = cut)) +
  stat_summary(fun = "mean",
               geom = "bar")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-28-1.png" width="672" />

That's a fail! Several things went wrong here. All the bars are gray and only their outline is colored differently. Instead we want the bars to have a different color. For that we need to specify the `fill` argument rather than the `color` argument! But things are worse. The bars currently are shown on top of each other. Instead, we'd like to put them next to each other. Here is how we can do that:


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     group = cut,
                     fill = cut)) +
  stat_summary(fun = "mean",
               geom = "bar",
               position = position_dodge())
```

<img src="29-visualization_files/figure-html/unnamed-chunk-29-1.png" width="672" />

Neato! We've changed the `color` argument to `fill`, and have added the `position = position_dodge()` argument to the `stat_summary()` call. This argument makes it such that the bars are nicely dodged next to each other. Let's add some error bars just for kicks.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     group = cut,
                     fill = cut)) +
  stat_summary(fun = "mean",
               geom = "bar",
               position = position_dodge(width = 0.9),
               color = "black") +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               position = position_dodge(width = 0.9))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-30-1.png" width="672" />

Voila! Now with error bars. Note that we've added the `width = 0.9` argument to `position_dodge()`. Somehow R was complaining when this was not defined for geom "linerange". I've also added some outline to the bars by including the argument `color = "black"`. I think it looks nicer this way.

So, still somewhat surprisingly, diamonds with the worst color (J) are more expensive than dimanods with the best color (D), and diamonds with better cuts are not necessarily more expensive.

#### Practice plot 3

Recreate the plot shown in Figure \@ref(fig:practice-plot3).


```r
# make practice plot 3 here
```


```r
include_graphics("figures/vis1_practice_plot3.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot3.png" alt="Practice plot 3." width="95%" />
<p class="caption">(\#fig:practice-plot3)Practice plot 3.</p>
</div>

__Advanced__: Try to recreate the plot show in in Figure \@ref(fig:practice-plot3a).


```r
# make advanced practice plot 3 here
```


```r
include_graphics("figures/vis1_practice_plot3a.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot3a.png" alt="Practice plot 3." width="95%" />
<p class="caption">(\#fig:practice-plot3a)Practice plot 3.</p>
</div>

### Making facets

Having too much information in a single plot can be overwhelming. The previous plot is already pretty busy. Facets are a nice way of splitting up plots and showing information in separate panels.

Let's take a look at how wide these diamonds tend to be. The width in mm is given in the `y` column of the diamonds data frame. We'll make a histogram first. To make a histogram, the only aesthetic we needed to specify is `x`.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y)) +
  geom_histogram()
```

```
`stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="29-visualization_files/figure-html/unnamed-chunk-33-1.png" width="672" />

That looks bad! Let's pick a different value for the width of the bins in the histogram.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y)) +
  geom_histogram(binwidth = 0.1)
```

<img src="29-visualization_files/figure-html/unnamed-chunk-34-1.png" width="672" />

Still bad. There seems to be an outlier diamond that happens to be almost 60 mm wide, while most of the rest is much narrower. One option would be to remove the outlier from the data before plotting it. But generally, we don't want to make new data frames. Instead, let's just limit what data we show in the plot.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y)) +
  geom_histogram(binwidth = 0.1) +
  coord_cartesian(xlim = c(3, 10))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-35-1.png" width="672" />

I've used the `coord_cartesian()` function to restrict the range of data to show by passing a minimum and maximum to the `xlim` argument. This looks better now.

Instead of histograms, we can also plot a density fitted to the distribution.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y)) +
  geom_density() +
  coord_cartesian(xlim = c(3, 10))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-36-1.png" width="672" />

Looks pretty similar to our histogram above! Just like we can play around with the binwidth of the histogram, we can change the smoothing bandwidth of the kernel that is used to create the histogram. Here is a histogram with a much wider bandwidth:


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y)) +
  geom_density(bw = 0.5) +
  coord_cartesian(xlim = c(3, 10))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-37-1.png" width="672" />

We'll learn more about how these densities are determined later in class.

I promised that this section was about making facets, right? We're getting there! Let's first take a look at how wide diamonds of different `color` are. We can use grouping to make this happen.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y,
                     group = color,
                     fill = color)) +
  geom_density(bw = 0.2,
               alpha = 0.2) +
  coord_cartesian(xlim = c(3, 10))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-38-1.png" width="672" />

OK! That's a little tricky to tell apart. Notice that I've specified the `alpha` argument in the `geom_density()` function so that the densities in the front don't completely hide the densities in the back. But this plot still looks too busy. Instead of grouping, let's put the densities for the different colors, in separate panels. That's what facetting allows you to do.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y,
                     fill = color)) +
  geom_density(bw = 0.2) +
  facet_grid(cols = vars(color)) +
  coord_cartesian(xlim = c(3, 10))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-39-1.png" width="672" />

Now we have the densities next to each other in separate panels. I've removed the `alpha` argument since the densities aren't overlapping anymore. To make the different panels, I used the `facet_grid()` function and specified that I want separate columns for the different colors (`cols = vars(color)`). What's the deal with `vars()`? Why couldn't we just write `facet_grid(cols = color)` instead? The short answer is: that's what the function wants. The long answer is: long. (We'll learn more about this later in the course.)

To show the facets in different rows instead of columns we simply replace `cols = vars(color)` with `rows = vars(color)`.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = y,
                     fill = color)) +
  geom_density(bw = 0.2) +
  facet_grid(rows = vars(color)) +
  coord_cartesian(xlim = c(3, 10))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-40-1.png" width="672" />

Several aspects about this plot should be improved:

- the y-axis text is overlapping
- having both a legend and separate facet labels is redundant
- having separate fills is not really necessary here

So, what does this plot actually show us? Well, J-colored diamonds tend to be wider than D-colored diamonds. Fascinating!

Of course, we could go completely overboard with facets and groups. So let's do it! Let's look at how the average `price` (somewhat more interesting) varies as a function of `color`, `cut`, and `clarity`. We'll put color on the x-axis, and make separate rows for `cut` and columns for `clarity`.


```r
ggplot(data = df.diamonds,
       mapping = aes(y = price,
                     x = color,
                     fill = color)) +
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black") +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange") +
  facet_grid(rows = vars(cut),
             cols = vars(clarity))
```

```
Warning: Removed 1 rows containing missing values (`geom_segment()`).
```

```
Warning: Removed 3 rows containing missing values (`geom_segment()`).
```

```
Warning: Removed 1 rows containing missing values (`geom_segment()`).
```

<div class="figure">
<img src="29-visualization_files/figure-html/stretching-it-1.png" alt="A figure that is stretching it in terms of information." width="672" />
<p class="caption">(\#fig:stretching-it)A figure that is stretching it in terms of information.</p>
</div>

Figure \@ref(fig:stretching-it) is stretching it in terms of how much information it presents. But it gives you a sense for how to combine the different bits and pieces we've learned so far.

#### Practice plot 4

Recreate the plot shown in Figure \@ref(fig:practice-plot4).


```r
# make practice plot 4 here
```


```r
include_graphics("figures/vis1_practice_plot4.png")
```

<div class="figure" style="text-align: center">
<img src="figures/vis1_practice_plot4.png" alt="Practice plot 4." width="95%" />
<p class="caption">(\#fig:practice-plot4)Practice plot 4.</p>
</div>

### Global, local, and setting `aes()`

`ggplot2` allows you to specify the plot aesthetics in different ways.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = carat,
                     y = price,
                     color = color)) +
  geom_point() +
  geom_smooth(method = "lm",
              se = F)
```

```
`geom_smooth()` using formula = 'y ~ x'
```

<img src="29-visualization_files/figure-html/unnamed-chunk-42-1.png" width="672" />

Here, I've drawn a scatter plot of the relationship between `carat` and `price`, and I have added the best-fitting regression lines via the `geom_smooth(method = "lm")` call. (We will learn more about what these regression lines mean later in class.)

Because I have defined all the aesthetics at the top level (i.e. directly within the `ggplot()` function), the aesthetics apply to all the functions afterwards. Aesthetics defined in the `ggplot()` call are __global__. In this case, the `geom_point()` and the `geom_smooth()` functions. The `geom_smooth()` function produces separate best-fit regression lines for each different color.

But what if we only wanted to show one regression line instead that applies to all the data? Here is one way of doing so:


```r
ggplot(data = df.diamonds,
       mapping = aes(x = carat,
                     y = price)) +
  geom_point(mapping = aes(color = color)) +
  geom_smooth(method = "lm")
```

```
`geom_smooth()` using formula = 'y ~ x'
```

<img src="29-visualization_files/figure-html/unnamed-chunk-43-1.png" width="672" />

Here, I've moved the color aesthetic into the `geom_point()` function call. Now, the `x` and `y` aesthetics still apply to both the `geom_point()` and the `geom_smooth()` function call (they are __global__), but the `color` aesthetic applies only to `geom_point()` (it is __local__). Alternatively, we can simply overwrite global aesthetics within local function calls.


```r
ggplot(data = df.diamonds,
       mapping = aes(x = carat,
                     y = price,
                     color = color)) +
  geom_point() +
  geom_smooth(method = "lm",
              color = "black")
```

```
`geom_smooth()` using formula = 'y ~ x'
```

<img src="29-visualization_files/figure-html/unnamed-chunk-44-1.png" width="672" />

Here, I've set `color = "black"` within the `geom_smooth()` function, and now only one overall regression line is displayed since the global color aesthetic was overwritten in the local function call.

# Visualization 2

In this lecture, we will lift our `ggplot2` skills to the next level! 

## Learning objectives

- Deciding what plot is appropriate for what kind of data.  
- Customizing plots: Take a sad plot and make it better. 
- Saving plots. 
- Making figure panels. 
- Debugging. 
- Making animations. 
- Defining snippets. 

## Install and load packages, load data, set theme

Let's first install the new packages that you might not have yet. 


```r
install.packages(c("gganimate", "gapminder", "ggridges", "devtools", "png", "gifski", "patchwork"))
```

Now, let's load the packages that we need for this chapter. 


```r
library("knitr")     # for rendering the RMarkdown file
library("patchwork") # for making figure panels
library("ggridges")  # for making joyplots 
library("gganimate") # for making animations
library("gapminder") # data available from Gapminder.org 
library("tidyverse") # for plotting (and many more cool things we'll discover later)
```

And set some settings: 


```r
# these options here change the formatting of how comments are rendered
opts_chunk$set(comment = "",
               fig.show = "hold")

# this just suppresses an unnecessary message about grouping 
options(dplyr.summarise.inform = F)

# set the default plotting theme 
theme_set(theme_classic() + #set the theme 
            theme(text = element_text(size = 20))) #set the default text size
```

And let's load the diamonds data again. 


```r
df.diamonds = diamonds
```

## Overview of different plot types for different things

Different plots work best for different kinds of data. Let's take a look at some. 

### Proportions

#### Stacked bar charts


```r
ggplot(data = df.diamonds, 
       mapping = aes(x = cut,
                     fill = color)) +
  geom_bar(color = "black")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-49-1.png" width="672" />

This bar chart shows for the different cuts (x-axis), the number of diamonds of different color. Stacked bar charts give a good general impression of the data. However, it's difficult to precisely compare different proportions. 

#### Pie charts

<div class="figure" style="text-align: center">
<img src="figures/pie_chart.jpg" alt="Finally a pie chart that makes sense." width="90%" />
<p class="caption">(\#fig:unnamed-chunk-50)Finally a pie chart that makes sense.</p>
</div>

Pie charts have a bad reputation. And there are indeed a number of problems with pie charts: 

- proportions are difficult to compare 
- don't look good when there are many categories 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = 1,
                     fill = cut)) +
  geom_bar() +
  coord_polar("y", start = 0) +
  theme_void()
```

<img src="29-visualization_files/figure-html/unnamed-chunk-51-1.png" width="672" />

We can create a pie chart with `ggplot2` by changing the coordinate system using `coord_polar()`.

If we are interested in comparing proportions and we don't have too many data points, then tables are a good alternative to showing figures. 

### Comparisons

Often we want to compare the data from many different conditions. And sometimes, it's also useful to get a sense for what the individual participant data look like. Here is a plot that achieves both. 

#### Means and individual data points


```r
ggplot(data = df.diamonds[1:150, ],
       mapping = aes(x = color,
                     y = price)) +
  # means with confidence intervals 
  stat_summary(fun.data = "mean_cl_boot",
               geom = "pointrange",
               color = "black",
               fill = "yellow",
               shape = 21,
               size = 1) + 
  # individual data points (jittered horizontally)
  geom_point(alpha = 0.2,
             color = "blue",
             position = position_jitter(width = 0.1, height = 0),
             size = 2)
```

<div class="figure">
<img src="29-visualization_files/figure-html/diamonds-price-1.png" alt="Price of differently colored diamonds. Large yellow circles are means, small black circles are individual data poins, and the error bars are 95% bootstrapped confidence intervals." width="672" />
<p class="caption">(\#fig:diamonds-price)Price of differently colored diamonds. Large yellow circles are means, small black circles are individual data poins, and the error bars are 95% bootstrapped confidence intervals.</p>
</div>

Note that I'm only plotting the first 150 entries of the data here by setting `data = df.diamonds[1:150,]` in `gpplot()`. 

This plot shows means, bootstrapped confidence intervals, and individual data points. I've used two tricks to make the individual data points easier to see. 
1. I've set the `alpha` attribute to make the points somewhat transparent.
2. I've used the `position_jitter()` function to jitter the points horizontally.
3. I've used `shape = 21` for displaying the mean. For this circle shape, we can set a `color` and `fill` (see Figure \@ref(fig:plotting-shapes))

<div class="figure">
<img src="29-visualization_files/figure-html/plotting-shapes-1.png" alt="Different shapes that can be used for plotting." width="672" />
<p class="caption">(\#fig:plotting-shapes)Different shapes that can be used for plotting.</p>
</div>

Here is an example of an actual plot that I've made for a paper that I'm working on (using the same techniques). 

<div class="figure" style="text-align: center">
<img src="figures/normality_judgments.png" alt="Participants’ preference for the conjunctive (top) versus dis-junctive (bottom) structure as a function of the explanation (abnormal cause vs. normalcause) and the type of norm (statistical vs. prescriptive). Note: Large circles are groupmeans. Error bars are bootstrapped 95% confidence intervals. Small circles are individualparticipants’ judgments (jittered along the x-axis for visibility)" width="90%" />
<p class="caption">(\#fig:unnamed-chunk-52)Participants’ preference for the conjunctive (top) versus dis-junctive (bottom) structure as a function of the explanation (abnormal cause vs. normalcause) and the type of norm (statistical vs. prescriptive). Note: Large circles are groupmeans. Error bars are bootstrapped 95% confidence intervals. Small circles are individualparticipants’ judgments (jittered along the x-axis for visibility)</p>
</div>


#### Boxplots

Another way to get a sense for the distribution of the data is to use box plots.


```r
ggplot(data = df.diamonds[1:500,],
       mapping = aes(x = color, y = price)) +
  geom_boxplot()
```

<img src="29-visualization_files/figure-html/unnamed-chunk-53-1.png" width="672" />

What do boxplots show? Here adapted from `help(geom_boxplot())`:  

> The boxplots show the median as a horizontal black line. The lower and upper hinges correspond to the first and third quartiles (the 25th and 75th percentiles) of the data. The whiskers (= black vertical lines) extend from the top or bottom of the hinge by at most 1.5 * IQR (where IQR is the inter-quartile range, or distance between the first and third quartiles). Data beyond the end of the whiskers are called "outlying" points and are plotted individually.

Personally, I'm not a big fan of boxplots. Many data sets are consistent with the same boxplot. 

<div class="figure">
<img src="figures/boxplots.gif" alt="Box plot distributions. Source: https://www.autodeskresearch.com/publications/samestats"  />
<p class="caption">(\#fig:box-plot-distributions1)Box plot distributions. Source: https://www.autodeskresearch.com/publications/samestats</p>
</div>

Figure \@ref(fig:box-plot-distributions1) shows three different distributions that each correspond to the same boxplot. 

If there is not too much data, I recommend to plot jittered individual data points instead. If you do have a lot of data points, then violin plots can be helpful. 

<div class="figure">
<img src="figures/box_violin.gif" alt="Boxplot distributions. Source: https://www.autodeskresearch.com/publications/samestats"  />
<p class="caption">(\#fig:box-plot-distributions2)Boxplot distributions. Source: https://www.autodeskresearch.com/publications/samestats</p>
</div>

Figure \@ref(fig:box-plot-distributions2) shows the same raw data represented as jittered dots, boxplots, and violin plots.  

#### Violin plots

We make violin plots like so: 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color, y = price)) +
  geom_violin()
```

<img src="29-visualization_files/figure-html/unnamed-chunk-54-1.png" width="672" />

Violin plots are good for detecting bimodal distributions. They work well when: 

1. You have many data points. 
2. The data is continuous.

Violin plots don't work well for Likert-scale data (e.g. ratings on a discrete scale from 1 to 7). Here is a simple example: 


```r
set.seed(1)
data = tibble(rating = sample(x = 1:7,
                              prob = c(0.1, 0.4, 0.1, 0.1, 0.2, 0, 0.1),
                              size = 500,
                              replace = T))

ggplot(data = data,
       mapping = aes(x = "Likert", y = rating)) +
  geom_violin() + 
  geom_point(position = position_jitter(width = 0.05,
                                        height = 0.1),
             alpha = 0.05)
```

<img src="29-visualization_files/figure-html/unnamed-chunk-55-1.png" width="672" />

This represents a vase much better than it represents the data.

#### Joy plots

We can also show the distributions along the x-axis using the `geom_density_ridges()` function from the `ggridges` package. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = price, y = color)) +
  ggridges::geom_density_ridges(scale = 1.5)
```

```
Picking joint bandwidth of 535
```

<img src="29-visualization_files/figure-html/unnamed-chunk-56-1.png" width="672" />

#### Practice plot 1

Try to make the plot shown in Figure \@ref(fig:visualization2-practice1). Here is a tip: 

- For the data argument in `ggplot()` use: `df.diamonds[1:10000, ]` (this selects the first 10000 rows).


```r
# write your code here
```

<div class="figure">
<img src="figures/vis2_practice_plot1.png" alt="Practice plot 1." width="700" />
<p class="caption">(\#fig:visualization2-practice1)Practice plot 1.</p>
</div>

### Relationships

#### Scatter plots

Scatter plots are great for looking at the relationship between two continuous variables. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = carat,
                     y = price,
                     color = color)) +
  geom_point()
```

<img src="29-visualization_files/figure-html/unnamed-chunk-58-1.png" width="672" />

#### Raster plots

These are useful for looking how a variable of interest varies as a function of two other variables. For example, when we are trying to fit a model with two parameters, we might be interested to see how well the model does for different combinations of these two parameters. Here, we'll plot what `carat` values diamonds of different `color` and `clarity` have. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = clarity,
                     z = carat)) +
  stat_summary_2d(fun = "mean", geom = "tile")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-59-1.png" width="672" />

Not too bad. Let's add a few tweaks to make it look nicer. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = clarity,
                     z = carat)) +
  stat_summary_2d(fun = "mean",
                  geom = "tile",
                  color = "black") +
  scale_fill_gradient(low = "white", high = "black") +
  labs(fill = "carat")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-60-1.png" width="672" />

I've added some outlines to the tiles by specifying `color = "black"` in `geom_tile()` and I've changed the scale for the fill gradient. I've defined the color for the low value to be "white", and for the high value to be "black." Finally, I've changed the lower and upper limit of the scale via the `limits` argument. Looks much better now! We see that diamonds with clarity `I1` and color `J` tend to have the highest `carat` values on average. 

### Temporal data

Line plots are a good choice for temporal data. Here, I'll use the `txhousing` data that comes with the `ggplot2` package. The dataset contains information about housing sales in Texas. 


```r
# ignore this part for now (we'll learn about data wrangling soon)
df.plot = txhousing %>% 
  filter(city %in% c("Dallas", "Fort Worth", "San Antonio", "Houston")) %>% 
  mutate(city = factor(city, levels = c("Dallas", "Houston", "San Antonio", "Fort Worth")))

ggplot(data = df.plot,
       mapping = aes(x = year,
                     y = median,
                     color = city,
                     fill = city)) +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "ribbon",
               alpha = 0.2,
               linetype = 0) +
  stat_summary(fun = "mean", geom = "line") +
  stat_summary(fun = "mean", geom = "point") 
```

<img src="29-visualization_files/figure-html/unnamed-chunk-61-1.png" width="672" />

Ignore the top part where I'm defining `df.plot` for now (we'll look into this in the next class). The other part is fairly straightforward. I've used `stat_summary()` three times: First, to define the confidence interval as a `geom = "ribbon"`. Second, to show the lines connecting the means, and third to put the means as data points points on top of the lines. 

Let's tweak the figure some more to make it look real good. 


```r
df.plot = txhousing %>% 
  filter(city %in% c("Dallas", "Fort Worth", "San Antonio", "Houston")) %>% 
  mutate(city = factor(city, levels = c("Dallas", "Houston", "San Antonio", "Fort Worth")))

df.text = df.plot %>% 
  filter(year == max(year)) %>% 
  group_by(city) %>% 
  summarize(year = mean(year) + 0.2, 
            median = mean(median))

ggplot(data = df.plot,
       mapping = aes(x = year, 
                     y = median,
                     color = city,
                     fill = city)) +
  # draw dashed horizontal lines in the background
  geom_hline(yintercept = seq(from = 100000,
                              to = 250000,
                              by = 50000),
             linetype = 2,
             alpha = 0.2) + 
  # draw ribbon
  stat_summary(fun.data = mean_cl_boot,
               geom = "ribbon",
               alpha = 0.2,
               linetype = 0) +
  # draw lines connecting the means
  stat_summary(fun = "mean", geom = "line") +
  # draw means as points
  stat_summary(fun = "mean", geom = "point") +
  # add the city names
  geom_text(data = df.text,
            mapping = aes(label = city),
            hjust = 0,
            size = 5) + 
  # set the limits for the coordinates
  coord_cartesian(xlim = c(1999, 2015),
                  clip = "off",
                  expand = F) + 
  # set the x-axis labels
  scale_x_continuous(breaks = seq(from = 2000,
                                  to = 2015,
                                  by = 5)) +
  # set the y-axis labels
  scale_y_continuous(breaks = seq(from = 100000,
                                  to = 250000,
                                  by = 50000),
                     labels = str_c("$",
                                    seq(from = 100,
                                        to = 250,
                                        by = 50),
                                    "K")) + 
  # set the plot title and axes titles
  labs(title = "Change of median house sale price in Texas",
       x = "Year",
       y = "Median house sale price",
       fill = "",
       color = "") + 
  theme(title = element_text(size = 16),
        legend.position = "none",
        plot.margin = margin(r = 1, unit = "in"))
```

<img src="29-visualization_files/figure-html/unnamed-chunk-62-1.png" width="672" />

## Customizing plots

So far, we've seen a number of different ways of plotting data. Now, let's look into how to customize the plots. For example, we may want to change the axis labels, add a title, increase the font size. `ggplot2` let's you customize almost anything. 

Let's start simple. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = cut, y = price)) +
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black") +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-63-1.png" width="672" />

This plot shows the average price for diamonds with a different quality of the cut, as well as the bootstrapped confidence intervals. Here are some things we can do to make it look nicer. 


```r
ggplot(data = df.diamonds, 
       mapping = aes(x = cut,
                     y = price)) +
  # change color of the fill, make a little more space between bars by setting their width
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black",
               fill = "lightblue",
               width = 0.85) + 
  # adjust the range of both axes
  coord_cartesian(xlim = c(0.25, 5.75),
                  ylim = c(0, 5000),
                  expand = F) + 
  # make error bars thicker
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               size = 1.5) + 
  # adjust what to show on the y-axis
  scale_y_continuous(breaks = seq(from = 0, to = 4000, by = 2000),
                     labels = seq(from = 0, to = 4000, by = 2000)) + 
  # add a title, subtitle, and changed axis labels 
  labs(title = "Price as a function of quality of cut", 
       subtitle = "Note: The price is in US dollars",
       tag = "A",
       x = "Quality of the cut", 
       y = "Price") + 
  theme(
    # adjust the text size 
    text = element_text(size = 20), 
    # add some space at top of x-title 
    axis.title.x = element_text(margin = margin(t = 0.2, unit = "inch")), 
    # add some space t the right of y-title
    axis.title.y = element_text(margin = margin(r = 0.1, unit = "inch")), 
    # add some space underneath the subtitle and make it gray
    plot.subtitle = element_text(margin = margin(b = 0.3, unit = "inch"),
                                 color = "gray70"),  
    # make the plot tag bold 
    plot.tag = element_text(face = "bold"), 
    # move the plot tag a little
    plot.tag.position = c(0.05, 0.99)
  )
```

<img src="29-visualization_files/figure-html/unnamed-chunk-64-1.png" width="672" />

I've tweaked quite a few things here (and I've added comments to explain what's happening). Take a quick look at the `theme()` function to see all the things you can change. 

### Anatomy of a `ggplot`

I suggest to use this general skeleton for creating a `ggplot`: 


```r
# ggplot call with global aesthetics 
ggplot(data = data,
       mapping = aes(x = cause,
                     y = effect)) +
  # add geometric objects (geoms)
  geom_point() + 
  stat_summary(fun = "mean", geom = "point") + 
  ... + 
  # add text objects 
  geom_text() + 
  annotate() + 
  # adjust axes and coordinates 
  coord_cartesian() + 
  scale_x_continuous() + 
  scale_y_continuous() + 
  # define plot title, and axis titles 
  labs(title = "Title",
       x = "Cause",
       y = "Effect") + 
  # change global aspects of the plot 
  theme(text = element_text(size = 20),
        plot.margin = margin(t = 1, b = 1, l = 0.5, r = 0.5, unit = "cm")) +

# save the plot 
ggsave(filename = "super_nice_plot.pdf",
       width = 8,
       height = 6)
```

### Changing the order of things

Sometimes we don't have a natural ordering of our independent variable. In that case, it's nice to show the data in order. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = reorder(cut, price),
                     y = price)) +
       # mapping = aes(x = cut, y = price)) +
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black",
               fill = "lightblue",
               width = 0.85) +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               size = 1.5) +
  labs(x = "cut")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-66-1.png" width="672" />

The `reorder()` function helps us to do just that. Now, the results are ordered according to price. To show the results in descending order, I would simply need to write `reorder(cut, -price)` instead.

### Dealing with legends

Legends form an important part of many figures. However, it is often better to avoid legends if possible, and directly label the data. This way, the reader doesn't have to look back and forth between the plot and the legend to understand what's going on. 

Here, we'll look into a few aspects that come up quite often. There are two main functions to manipulate legends with ggplot2 
1. `theme()` (there are a number of arguments starting with `legend.`)
2. `guide_legend()`

Let's make a plot with a legend. 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     color = clarity)) +
  stat_summary(fun = "mean",
               geom = "point")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-67-1.png" width="672" />

Let's move the legend to the bottom of the plot: 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     color = clarity)) +
  stat_summary(fun = "mean",
               geom = "point") +
  theme(legend.position = "bottom")
```

<img src="29-visualization_files/figure-html/unnamed-chunk-68-1.png" width="672" />

Let's change a few more things in the legend using the `guides()` function: 

- have 3 rows 
- reverse the legend order 
- make the points in the legend larger 


```r
ggplot(data = df.diamonds,
       mapping = aes(x = color,
                     y = price,
                     color = clarity)) +
  stat_summary(fun = "mean",
               geom = "point",
               size = 2) +
  theme(legend.position = "bottom") +
  guides(color = guide_legend(nrow = 3, # 3 rows 
                              reverse = TRUE, # reversed order 
                              override.aes = list(size = 6))) # point size 
```

<img src="29-visualization_files/figure-html/unnamed-chunk-69-1.png" width="672" />

### Choosing good colors

[Color brewer](http://colorbrewer2.org/) helps with finding colors that are colorblind safe and printfriendly. For more information on how to use color effectively see [here](http://socviz.co/refineplots.html#refineplots). 

### Customizing themes

For a given project, I often want all of my plots to share certain visual features such as the font type, font size, how the axes are displayed, etc. Instead of defining these for each individual plot, I can set a theme at the beginning of my project so that it applies to all the plots in this file. To do so, I use the `theme_set()` command: 


```r
theme_set(theme_classic() + #classic theme
            theme(text = element_text(size = 20))) #text size 
```

Here, I've just defined that I want to use `theme_classic()` for all my plots, and that the text size should be 20. For any individual plot, I can still overwrite any of these defaults. 

## Saving plots

To save plots, use the `ggsave()` command. Personally, I prefer to save my plots as pdf files. This way, the plot looks good no matter what size you need it to be. This means it'll look good both in presentations as well as in a paper. You can save the plot in any format that you like. 

I strongly recommend to use a relative path to specify where the figure should be saved. This way, if you are sharing the project with someone else via Stanford Box, Dropbox, or Github, they will be able to run the code without errors. 

Here is an example for how to save one of the plots that we've created above. 


```r
p1 = ggplot(data = df.diamonds,
            mapping = aes(x = cut, y = price)) +
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black",
               fill = "lightblue") +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               size = 1)
print(p1)

p2 = ggplot(data = df.diamonds,
            mapping = aes(x = cut, y = price)) +
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black",
               fill = "lightblue") +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               size = 1)

ggsave(filename = "figures/diamond_plot.pdf",
       plot = p1,
       width = 8,
       height = 6)
```

<img src="29-visualization_files/figure-html/unnamed-chunk-71-1.png" width="672" />

Here, I'm saving the plot in the `figures` folder and it's name is `diamond_plot.pdf`. I also specify the width and height as the plot in inches (which is the default unit). 

## Creating figure panels

Sometimes, we want to create a figure with several subfigures, each of which is labeled with a), b), etc. We have already learned how to make separate panels using `facet_wrap()` or `facet_grid()`. The R package `patchwork` makes it very easy to combine multiple plots. You can find out more about the package [here](https://patchwork.data-imaginist.com/articles/patchwork.html). 

Let's combine a few plots that we've made above into one. 


```r
# first plot
p1 = ggplot(data = df.diamonds,
            mapping = aes(x = y, fill = color)) +
  geom_density(bw = 0.2,
               show.legend = F) +
  facet_grid(cols = vars(color)) +
  labs(title = "Width of differently colored diamonds") + 
  coord_cartesian(xlim = c(3, 10),
                  expand = F) #setting expand to FALSE removes any padding on x and y axes

# second plot
p2 = ggplot(data = df.diamonds,
            mapping = aes(x = color,
                          y = clarity,
                          z = carat)) +
  stat_summary_2d(fun = "mean",
                  geom = "tile") +
  labs(title = "Carat values",
       subtitle = "For different color and clarity",
       x = "Color")

# third plot
p3 = ggplot(data = df.diamonds,
            mapping = aes(x = cut, y = price)) +
  stat_summary(fun = "mean",
               geom = "bar",
               color = "black",
               fill = "lightblue",
               width = 0.85) +
  stat_summary(fun.data = "mean_cl_boot",
               geom = "linerange",
               size = 1.5) + 
  scale_x_discrete(labels = c("fair", "good", "very\ngood", "premium", "ideal")) +
  labs(title = "Price as a function of cut", 
       subtitle = "Note: The price is in US dollars",
       x = "Quality of the cut", 
       y = "Price") + 
  coord_cartesian(xlim = c(0.25, 5.75),
                  ylim = c(0, 5000),
                  expand = F)

# combine the plots
p1 + (p2 + p3) + 
  plot_layout(ncol = 1) &
  plot_annotation(tag_levels = "A") & 
  theme_classic() &
  theme(plot.tag = element_text(face = "bold", size = 20))

# ggsave("figures/combined_plot.png", width = 10, height = 6)
```

<img src="29-visualization_files/figure-html/unnamed-chunk-72-1.png" width="672" />

Not a perfect plot yet, but you get the idea. To combine the plots, we defined that we would like p2 and p3 to be displayed in the same row using the `()` syntax. And we specified that we only want one column via the `plot_layout()` function. We also applied the same `theme_classic()` to all the plots using the `&` operator, and formatted how the plot tags should be displayed. For more info on how to use `patchwork`, take a look at the [readme](https://github.com/thomasp85/patchwork) on the github page. 

Other packages that provide additional functionality for combining multiple plots into one are 

- [`gridExtra`](https://cran.r-project.org/web/packages/gridExtra/index.html) and 
- [`cowplot`](https://cran.r-project.org/web/packages/cowplot/index.html). You can find more information on how to lay out multiple plots [here](https://cran.r-project.org/web/packages/egg/vignettes/Ecosystem.html).

An alternative way for making these plots is to use Adobe Illustrator, Powerpoint, or Keynote. However, you want to make changing plots as easy as possible. Adobe Illustrator has a feature that allows you to link to files. This way, if you change the plot, the plot within the illustrator file gets updated automatically as well. 

If possible, it's __much__ better to do everything in R though so that your plot can easily be reproduced by someone else. 

## Peeking behind the scenes

Sometimes it can be helpful for debugging to take a look behind the scenes. Silently, `ggplot()` computes a data frame based on the information you pass to it. We can take a look at the data frame that's underlying the plot. 


```r
p = ggplot(data = df.diamonds,
           mapping = aes(x = color,
                         y = clarity,
                         z = carat)) +
  stat_summary_2d(fun = "mean",
                  geom = "tile",
                  color = "black") +
  scale_fill_gradient(low = "white", high = "black")
print(p)

build = ggplot_build(p)
df.plot_info = build$data[[1]]
dim(df.plot_info) # data frame dimensions
```

```
[1] 56 18
```

<img src="29-visualization_files/figure-html/unnamed-chunk-73-1.png" width="672" />

I've called the `ggplot_build()` function on the ggplot2 object that we saved as `p`. I've then printed out the data associated with that plot object. The first thing we note about the data frame is how many entries it has, 56. That's good. This means there is one value for each of the 7 x 8 grids. The columns tell us what color was used for the `fill`, the `value` associated with each row, where each row is being displayed (`x` and `y`), etc.   

If a plot looks weird, it's worth taking a look behind the scenes. For example, something we thing we could have tried is the following (in fact, this is what I tried first): 


```r
p = ggplot(data = df.diamonds,
           mapping = aes(x = color,
                         y = clarity,
                         fill = carat)) +
  geom_tile(color = "black") +
  scale_fill_gradient(low = "white", high = "black")
print(p)

build = ggplot_build(p)
df.plot_info = build$data[[1]]
dim(df.plot_info) # data frame dimensions
```

```
[1] 53940    15
```

<img src="29-visualization_files/figure-html/unnamed-chunk-74-1.png" width="672" />

Why does this plot look different from the one before? What went wrong here? Notice that the data frame associated with the ggplot2 object has 53940 rows. So instead of plotting means here, we plotted all the individual data points. So what we are seeing here is just the top layer of many, many layers. 

## Making animations

Animated plots can be a great way to illustrate your data in presentations. The R package `gganimate` lets you do just that. 

Here is an example showing how to use it. 


```r
ggplot(data = gapminder,
       mapping = aes(x = gdpPercap,
                     y = lifeExp,
                     size = pop,
                     colour = country)) +
  geom_point(alpha = 0.7, show.legend = FALSE) +
  geom_text(data = gapminder %>% 
              filter(country %in% c("United States", "China", "India")), 
            mapping = aes(label = country),
            color = "black",
            vjust = -0.75,
            show.legend = FALSE) +
  scale_colour_manual(values = country_colors) +
  scale_size(range = c(2, 12)) +
  scale_x_log10(breaks = c(1e3, 1e4, 1e5),
                labels = c("1,000", "10,000", "100,000")) +
  theme_classic() +
  theme(text = element_text(size = 23)) +
  # Here come the gganimate specific bits
  labs(title = "Year: {frame_time}", x = "GDP per capita", y = "life expectancy") +
  transition_time(year) +
  ease_aes("linear")
```

```
Warning: No renderer available. Please install the gifski, av, or magick
package to create animated output
```

```
NULL
```

```r
# anim_save(filename = "figures/life_gdp_animation.gif") # to save the animation
```

This takes a while to run but it's worth the wait. The plot shows the relationship between GDP per capita (on the x-axis) and life expectancy (on the y-axis) changes across different years for the countries of different continents. The size of each dot represents the population size of the respective country. And different countries are shown in different colors. This animation is not super useful yet in that we don't know which continents and countries the different dots represent. I've added a label to the United States, China, and India. 

Note how little is required to define the `gganimate`-specific information! The `{frame_time}` variable changes the title for each frame. The `transition_time()` variable is set to `year`, and the kind of transition is set as 'linear' in `ease_aes()`. I've saved the animation as a gif in the figures folder. 
We won't have time to go into more detail here but I encourage you to play around with `gganimate`. It's fun, looks cool, and (if done well) makes for a great slide in your next presentation! 

## Shiny apps

The package [`shiny`](https://shiny.rstudio.com/) makes it relatively easy to create interactive plots that can be hosted online. Here is a [gallery](https://shiny.rstudio.com/gallery/) with some examples. 

## Defining snippets

Often, we want to create similar plots over and over again. One way to achieve this is by finding the original plot, copy and pasting it, and changing the bits that need changing. Another more flexible and faster way to do this is by using snippets. Snippets are short pieces of code that 

Here are some snippets I use: 


```r
snippet sngg
	ggplot(data = ${1:data},
	       mapping = aes(${2:aes})) +
		${0}

snippet sndf
	${1:data} = ${1:data} %>% 
		${0}
```

To make a bar plot, I now only need to type `snbar` and then hit TAB to activate the snippet. I can then cycle through the bits in the code that are marked with `${Number:word}` by hitting TAB again. 

In RStudio, you can change and add snippets by going to Tools --> Global Options... --> Code --> Edit Snippets. Make sure to set the tick mark in front of Enable Code Snippets (see Figure \@ref(fig:code-snippets)). 
). 

<div class="figure">
<img src="figures/snippets.png" alt="Enable code snippets." width="591" />
<p class="caption">(\#fig:code-snippets)Enable code snippets.</p>
</div>

To edit code snippets faster, run this command from the `usethis` package. Make sure to install the package first if you don't have it yet. 


```r
# install.packages("usethis")
usethis::edit_rstudio_snippets()
```

This command opens up a separate tab in RStudio called `r.snippets` so that you can make new snippets and adapt old ones more quickly. Take a look at the snippets that RStudio already comes with. And then, make some new ones! By using snippets you will be able to avoid typing the same code over and over again, and you won't have to memorize as much, too. 

## Additional resources

### Cheatsheets

- [shiny](figures/shiny.pdf) --> interactive plots 
- [RStudio IDE](figures/rstudio-ide.pdf) --> information about RStudio
- [RMarkdown](figures/rmarkdown.pdf) --> information about writing in RMarkdown
- [RMarkdown reference](figures/rmarkdown-reference.pdf) --> RMarkdown reference sheet
- [Data visualization](figures/visualization-principles.pdf) --> general principles of effective graphic design
- [ggplot2](figures/data-visualization.pdf) --> specific information about ggplot

### Datacamp courses

- [shiny](https://www.datacamp.com/courses/building-web-applications-in-r-with-shiny-case-studies)
- [Introduction to R](https://www.datacamp.com/courses/free-introduction-to-r)
- [ggplot (intro)](https://learn.datacamp.com/courses/introduction-to-data-visualization-with-ggplot2)
- [Reporting](https://www.datacamp.com/courses/communicating-with-data-in-the-tidyverse)
- [visualization best practices](https://www.datacamp.com/courses/visualization-best-practices-in-r)

### Books and chapters

- [ggplot2 book](https://ggplot2-book.org/) 
- [R for Data Science book](http://r4ds.had.co.nz/)
	+ [Data visualization](http://r4ds.had.co.nz/data-visualisation.html)
	+ [Graphics for communication](http://r4ds.had.co.nz/graphics-for-communication.html)
- [Data Visualization -- A practical introduction (by Kieran Healy)](http://socviz.co/)
  + [Look at data](http://socviz.co/lookatdata.html#lookatdata)
  + [Make a plot](http://socviz.co/makeplot.html#makeplot)
  + [Show the right numbers](http://socviz.co/groupfacettx.html#groupfacettx)
  + [Refine your plots](http://socviz.co/refineplots.html#refineplots)
- [Fundamentals of Data Visualization](https://serialmentor.com/dataviz/) --> very nice resource that goes beyond basic functionality of `ggplot` and focuses on how to make good figures (e.g. how to choose colors, axes, ...)

### Misc

- [nice online ggplot tutorial](https://evamaerey.github.io/ggplot2_grammar_guide/about)
- [how to read R help files](https://socviz.co/appendix.html#a-little-more-about-r)
- [ggplot2 extensions](https://exts.ggplot2.tidyverse.org/gallery/) --> gallery of ggplot2 extension packages
- [ggplot2 extensions](https://z3tt.github.io/exciting-extensions/slides.html?s=09#/layers) --> gallery of ggplot2 extension packages 
- [ggplot2 gui](https://github.com/dreamRs/esquisse) --> ggplot2 extension package 
- [ggplot2 visualizations with code](http://r-statistics.co/Top50-Ggplot2-Visualizations-MasterList-R-Code.html) --> gallery of plots with code
- [Color brewer](http://colorbrewer2.org/) --> for finding colors 
- [shiny apps examples](https://sites.psu.edu/shinyapps/) --> shiny apps examples that focus on statistics teaching (made by students at PennState) 

## Session info


```
R version 4.3.0 (2023-04-21)
Platform: aarch64-apple-darwin20 (64-bit)
Running under: macOS 14.1.1

Matrix products: default
BLAS:   /Library/Frameworks/R.framework/Versions/4.3-arm64/Resources/lib/libRblas.0.dylib 
LAPACK: /Library/Frameworks/R.framework/Versions/4.3-arm64/Resources/lib/libRlapack.dylib;  LAPACK version 3.11.0

locale:
[1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

time zone: America/Chicago
tzcode source: internal

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
 [1] gapminder_1.0.0 gganimate_1.0.8 ggridges_0.5.4  patchwork_1.1.3
 [5] lubridate_1.9.2 forcats_1.0.0   stringr_1.5.0   dplyr_1.1.4    
 [9] purrr_1.0.2     readr_2.1.4     tidyr_1.3.0     tibble_3.2.1   
[13] ggplot2_3.4.4   tidyverse_2.0.0 knitr_1.42     

loaded via a namespace (and not attached):
 [1] gtable_0.3.3      xfun_0.39         bslib_0.4.2       htmlwidgets_1.6.2
 [5] lattice_0.21-8    tzdb_0.4.0        vctrs_0.6.5       tools_4.3.0      
 [9] generics_0.1.3    fansi_1.0.4       highr_0.10        cluster_2.1.4    
[13] pkgconfig_2.0.3   Matrix_1.6-4      data.table_1.14.8 checkmate_2.2.0  
[17] lifecycle_1.0.3   compiler_4.3.0    farver_2.1.1      textshaping_0.3.6
[21] progress_1.2.2    munsell_0.5.0     htmltools_0.5.5   sass_0.4.6       
[25] yaml_2.3.7        htmlTable_2.4.2   Formula_1.2-5     pillar_1.9.0     
[29] crayon_1.5.2      jquerylib_0.1.4   cachem_1.0.8      Hmisc_5.1-1      
[33] rpart_4.1.19      nlme_3.1-162      tidyselect_1.2.0  digest_0.6.31    
[37] stringi_1.7.12    bookdown_0.34     labeling_0.4.2    splines_4.3.0    
[41] fastmap_1.1.1     grid_4.3.0        colorspace_2.1-0  cli_3.6.1        
[45] magrittr_2.0.3    base64enc_0.1-3   utf8_1.2.3        foreign_0.8-84   
[49] withr_2.5.0       prettyunits_1.1.1 scales_1.3.0      backports_1.4.1  
[53] timechange_0.2.0  rmarkdown_2.21    nnet_7.3-18       gridExtra_2.3    
[57] ragg_1.2.5        png_0.1-8         hms_1.1.3         evaluate_0.21    
[61] viridisLite_0.4.2 mgcv_1.8-42       rlang_1.1.1       glue_1.6.2       
[65] tweenr_2.0.2      rstudioapi_0.14   jsonlite_1.8.4    R6_2.5.1         
[69] systemfonts_1.0.4
```

<div class="figure">
<img src="figures/reproducibility_court.jpg" alt="Defense at the reproducibility court (graphic by [Allison Horst](https://github.com/allisonhorst))." width="95%" />
<p class="caption">(\#fig:unnamed-chunk-79)Defense at the reproducibility court (graphic by [Allison Horst](https://github.com/allisonhorst)).</p>
</div>
