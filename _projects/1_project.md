---
layout: page
title: RFM analysis and customer segmentation with Python
description: This is a project that uses RFM analysis for customer segmentation and statistical models to forecast customer future purchase behavior.
img: assets/img/e_commerce.png
importance: 1
category: work
---

## Introduction

It is clear that customers are one of the most important assets of any business and therefore is very important to understand their needs and preferences. If a company has access to relevant information about customer behavior it makes sense to use this information for marketing purposes. For example, we can split customers into different groups according to their purchase patterns; then, use this groups to make special offers or make other business-related decisions. Also, we can use past data to try to forecast the future behavior of customer in order to calculate the customer lifetime value. This will be important if we want to know whether we should spend effort in trying to retain a customer or not.

In this project I present a RFM analysis that aims to segment a collection of customers into groups of similar purchase patterns. Also I apply a probability model to forecast the future expected behavior (in term of transactions) which can be used as an estimate of customer lifetime value. Finally I present the results in an interactive dashboard. 

The dataset used in this project can be found here. This dataset, that belongs to an anonymous non-store online retail company, consists of several records with information about the transactions made by approximately 4000 customers between 01/12/2010 and 09/12/2011.

## What is RFM analysis?
RFM stands for recency, frequency and monetary value and it is a behavior-based marketing technique which uses past transactional data to perform customer segmentation into groups of individuals with similar transactional patterns. Other customer segmentation techniques can be based on demographic, psychographic or geographic characteristics; however, any company that keeps a record of its customers transactions can make use of this type of technique. 

To  perform a RFM analysis, we use transactional data to compute the recency (days since last purchase or transaction), frequency (total number of transactions) and monetary value (total money spend) for the considered period of time. In Figure 1 I show a capture of the first five records in the Pandas dataframe. Note that for each transaction we have information such as its unique identifier, the ID of the corresponding customer, date in which the transaction was made, etc. Given this information we can calculate all these quantities for each one the customers.

This calculation could be performed directly using Pandas but in this case I’m going to be using a Python library that is specifically designed for analyzing transactional data. This library is called Lifetimes and the documentation can be found here. However, before doing such analysis, the first step is cleaning and preprocessing data to exclude missing values or rows that are not suitable for analysis. The detailed cleaning process can be seen in the accompanying notebook (link), but for the purposes of this report is enough to say that all transaction with no known customer were removed, as well as transactions associated to canceled orders. 

Given the preprocessed dataset we can proceed to perform the calculation of the RFM quantities. This can be done with the following function which accepts as input the whole transactional data an returns a dataframe where each record represents a customer and the corresponding recency, frequency and monetary value. For a definition of these quantities consult the documentation.

Once we have calculated these measures it is time to split the base of customers into groups with similar purchase patterns. This can be done in many ways, including machine learning techniques such as K-means Clustering or Gaussian Mixtures Models, but in this case we will follow a simpler approach that consists in dividing customers into a set of n quantiles according to the distribution of values for recency, frequency, and monetary value. Here we choose n=4 so there is a total of 4x4x4 possible groups. The interpretation of these groups is straightforward; for example, the customers in group (4,4,4) are the best since they bought recently, frequently and spent the most. On the contrary, customers belonging to group (1,1,1) are the worst since they bought a long time ago, infrequently and spent a little. The following code takes care of the quartile calculation:

(code)

Note that the quartile corresponding to each measure is added as a new column for further analysis.

## Forecasting customer purchasing patterns. 

In addition to customer segmentation, we can use customers past transactional data to forecast future purchase behavior. An important question at any point of time is whether a customer is still active and if this is the case how much value or revenue we can expect from her in the future. Assuming that past data can give us information about the future behavior of customers we can create probability models that capture the statistical transactional patterns and compute quantities of interest such as the probability of a customer being active, \(P(Active|D)\), or the expected number of transactions $X$ in a given period of time $T$, \(E(X|Data,T)\). These quantities allow us to assess the potential of each customer in terms of the number of transactions that she will make in the future.

In this report I will not explain the details of these models but present how we can construct them using Python and the Lifetimes library. For a detailed explanation of these type of models the interested reader can consult the references [1] and [2]. In the following I will use a BG/NBD model [1] which stands for Beta Geometric / Negative Binomial Distribution. The following code shows how we can train the model using past transactional data: 

Note that this model only takes into account the recency and frequency measures.

Once we have trained the model, we can visualize the distribution of the probability of still being active or the expected number of transactions as a function of customer recency and frequency.

Then, for each customer, we can compute the probability of being active and the expected number of transactions and add this information to the dataframe as new columns.

The segmentation of customers into groups and the computation of $$ P(Active|D) $$ and $$ E(X|D) $$ allow us to evaluate how much we can expect from an active customer in the future. The results obtained from this analysis can be used to make informed marketing decisions to better allocate resources and improve revenue. For easy access to this analysis we can load the information contained in the new dataframe in a dashboard. This way, we can easily consult the information associated with each customer. 

Every project has a beautiful feature showcase page.
It's easy to include images in a flexible 3-column grid format.
Make your photos 1/3, 2/3, or full width.

To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: project
    description: a project with a background image
    img: /assets/img/12.jpg
    ---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images.
Say you wanted to write a little bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, *bled* for your project, and then... you reveal its glory in the next row of images.


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>


The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}
```html
<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
```
{% endraw %}

<iframe seamless frameborder="0" src="https://public.tableau.com/views/RFManalysis_16850252617450/Dashboard1?:embed=yes&:display_count=yes&:showVizHome=no" width = '650' height = '450' scrolling='yes' ></iframe>    
