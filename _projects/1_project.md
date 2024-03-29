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

The dataset used in this project can be found here: <a href="https://archive.ics.uci.edu/ml/datasets/online+retail">UCI repository</a>. This dataset, that belongs to an anonymous non-store online retail company, consists of several records with information about the transactions made by approximately 4000 customers between 01/12/2010 and 09/12/2011.

## What is RFM analysis?
RFM stands for recency, frequency and monetary value and it is a behavior-based marketing technique which uses past transactional data to perform customer segmentation into groups of individuals with similar transactional patterns. Other customer segmentation techniques can be based on demographic, psychographic or geographic characteristics; however, any company that keeps a record of its customers transactions can make use of this type of technique. 

To  perform a RFM analysis, we use transactional data to compute the recency (days since last purchase or transaction), frequency (total number of transactions) and monetary value (total money spent) for the considered period of time. In Figure 1 I show a capture of the first five records in the Pandas dataframe. Note that for each transaction we have information such as its unique identifier, the ID of the corresponding customer, date in which the transaction was made, etc. Given this information we can calculate all these quantities for each one of the customers.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/customersDF.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Dataset containing transactional data.
</div>

This calculation can be performed directly using Pandas but in this case I’m going to be using a Python library that is specifically designed for analyzing transactional data. This library is called Lifetimes and the documentation can be found here <a href="https://lifetimes.readthedocs.io/en/latest/">docs</a>. However, before doing such analysis, the first step is cleaning and preprocessing data to exclude missing values or rows that are not suitable for analysis. The detailed cleaning process can be seen in the accompanying notebook <a href="https://github.com/AlbertoR94/Python-RFM-analysis-">RFM</a>, but for the purposes of this report is enough to say that all transaction with no known customer were removed, as well as transactions associated to canceled orders. 

Given the preprocessed dataset we can proceed to perform the calculation of the RFM quantities. This can be done with the following function which accepts as input the whole transactional data an returns a dataframe where each record represents a customer and the corresponding recency, frequency and monetary value. For a definition of these quantities consult the documentation.

{% highlight python linenos %}

import lifetimes as lt

rfm = lt.utils.summary_data_from_transaction_data(df, "CustomerID", "InvoiceDate", 
                                                  "Total", observation_period_end='2011-12-31')
{% endhighlight %}

The resulting dataframe is the following:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/rfm.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Dataframe containing customers information.
</div>

Once we have calculated these measures it is time to split the base of customers into groups with similar purchase patterns. This can be done in many ways, including machine learning techniques such as K-means Clustering or Gaussian Mixtures Models, but in this case we will follow a simpler approach that consists in dividing customers into a set of n quantiles according to the distribution of values for recency, frequency, and monetary value. Here we choose n=4 so there is a total of 4x4x4 possible groups. The interpretation of these groups is straightforward; for example, the customers in group (4,4,4) are the best since they bought recently, frequently and spent the most. On the contrary, customers belonging to group (1,1,1) are the worst since they bought a long time ago, infrequently and spent a little. The following code takes care of the quartile calculation:

{% highlight python linenos %}

rfm['r_category'] = pd.qcut(rfm['recency'], q=5, labels=range(1,5), duplicates='drop').astype(str)
rfm['f_category'] = pd.qcut(rfm['frequency'], q=5, labels=range(1,5), duplicates='drop').astype(str)
rfm['m_category'] = pd.qcut(rfm['total_spent'], q=4, labels=range(1,5)).astype(str)

{% endhighlight %}

Note that the quartile corresponding to each measure is added as a new column for further analysis.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/rfm_2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Dataframe containing customers information.
</div>

## Forecasting customer purchasing patterns. 

In addition to customer segmentation, we can use customers past transactional data to forecast future purchase behavior. An important question at any point of time is whether a customer is still active and if this is the case how much value or revenue we can expect from her in the future. Assuming that past data can give us information about the future behavior of customers we can create probability models that capture the statistical transactional patterns and compute quantities of interest such as the probability of a customer being active, $$ P(\textrm{Active} \lvert \textrm{Data}) $$, or the expected number of transactions $$ X $$ in a given period of time $$ T $$, $$ E(X \lvert \textrm{Data},T) $$. These quantities allow us to assess the potential of each customer in terms of the number of transactions that she will make in the future.

In this report I will not explain the details of these models but present how we can construct them using Python and the Lifetimes library. For a detailed explanation of these type of models the interested reader can consult the references <d-cite key="schmittlein1987counting"></d-cite>. In the following I will use a BG/NBD model which stands for Beta Geometric / Negative Binomial Distribution. The following code shows how we can train the model using past transactional data:

{% highlight python linenos %}

from lifetimes import BetaGeoFitter
# This model require customer information such as frequency and recency
bgf = BetaGeoFitter(penalizer_coef=0.001)
bgf.fit(rfm['frequency'], rfm['recency'], rfm['T'])

{% endhighlight %}

Note that this model only takes into account the recency and frequency measures.

Once we have trained the model, we can visualize the distribution of the probability of still being active or the expected number of transactions as a function of customer recency and frequency.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/graphs.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Dataframe containing customers information.
</div>

Then, for each customer, we can compute the probability of being active and the expected number of transactions and add this information to the dataframe as new columns.

{% highlight python linenos %}

rfm['prob_alive'] = bgf.conditional_probability_alive(rfm['frequency'], rfm['recency'], rfm['T'])
# We can also predict the future number of transactions for the next N time periods
for T in range(1, 11):
    rfm['predicted_purchases_' + str(T)] = bgf.conditional_expected_number_of_purchases_up_to_time(T, rfm['frequency'],                                                      rfm['recency'], rfm['T']).sort_values()
    
{% endhighlight %}

The segmentation of customers into groups and the computation of $$ P(\textrm{Active} \lvert \textrm{Data}) $$ and $$ E(X \lvert \textrm{Data},T) $$ allow us to evaluate how much we can expect from an active customer in the future. The results obtained from this analysis can be used to make informed marketing decisions to better allocate resources and improve revenue. For easy access to this analysis we can load the information contained in the new dataframe in a dashboard. This way, we can easily consult the information associated with each customer. 

## Tableau dashboard
<iframe seamless frameborder="0" src="https://public.tableau.com/views/RFManalysis_16850252617450/Dashboard1?:embed=yes&:display_count=yes&:showVizHome=no" width = '800' height = '650' scrolling='yes' ></iframe>    
