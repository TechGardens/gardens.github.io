---
layout: post
title: Conditional expectation method for missing values replacement
description: Context-based missing value estimation
tags: machine-learning data-preparation missing-values
giscus_comments: true
pretty_table: true
date: 2024-05-16
featured: true

authors:
  - name: Piotr Durawa
    affiliations:
      name: Freelance

bibliography: 2024-05-17-missing_values_conditional_expectation.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Citations
  - name: Footnotes
  - name: Code Blocks
  - name: Interactive Plots
  - name: Layouts
  - name: Other Typography?

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---


## Introduction 


Handling missing values is a common challenge in real-world datasets, where empty cells can disrupt analysis. These "missing values" need to be addressed before we can effectively use the data. To bridge these gaps, we often turn to strategies like using the mean or median of a feature, or simply removing the incomplete records. 
Using the mean or median is a straightforward solution, especially when features are uncorrelated. By calculating the global mean of a specific feature and filling in the missing values, we can quickly patch the dataset. However, this method might compromise the data's integrity.

## Short example

Let's delve into a practical example: obesity classification. Imagine our dataset includes height, weight, and an obesity label. Now, suppose some weight entries are missing. We want to retain these records, so we need to estimate the missing weights. If we choose to fill in these gaps with the average weight—say, 70 kg—we might end up with distorted data. Table [1](#tab1) illustrates this with three artificial records where the weight is replaced by the mean value of 70 kg. The obesity column shows the true labels, while the model prediction column displays the predictions from a well-trained model. 


<a id="tab1">Table 1.</a> Sample records with weight replaced by mean value.

| Weight [kg]|  Height [cm] |  Obesity  | Model prediction |
|---|---|---| --- |
| 70 | 140  | Normal weight  | Obese |
| 70 | 170  | Overweight | Normal weight |
| 70 | 210  | Normal weight | Underweight |


Observations:
- For the first person, the mean weight is too high, leading the model to classify them as "Obese".
- For the second person, the mean value is too low, causing the model to predict "Normal weight" despite the ground truth being "Overweight". The missing value should be higher.
- The third person is very tall with a "Normal weight" label. However, the mean value is too low, and the model predicts "Underweight".

Clearly, data integrity suffers when missing values are replaced with the mean.

An intuitive observation is that weight and height are correlated. Therefore, it might be possible to estimate an unknown weight based on the known height value. Additionally, considering the obesity label could further improve the accuracy of these estimates.

In this article, we will attempt to solve this problem using the conditional expectation method. By leveraging the relationship between height and weight, we aim to more accurately estimate the missing weight values and preserve the integrity of our dataset. 

Although there's no code here, you can check out the behind-the-scenes work in the notebook on repository <d-cite key="techgardensconditionalexpectationrepo"></d-cite>.


## Dataset

For our testing, we'll use The Complete Pokemon Dataset  <d-cite key="completepokedataset"></d-cite>, focusing on the height and weight features. We'll simulate missing data by assuming some weight values are lost, creating gaps in the dataset. First, we'll fill these gaps using the mean values as a baseline method. Then, we'll use conditional expectation for a more sophisticated approach. Finally, we'll compare the errors from each method to see which one performs better.

<table
  data-height="460"
  data-pagination="true"
  data-url="{{ 'assets/json/2024-05-17-missing_values_conditional_expectation/dataset.json' | relative_url }}">
  <thead>
    <tr>
      <th data-field="weight_kg" data-halign="left" data-align="center" data-sortable="true">Weight</th>
      <th data-field="height_m" data-halign="center" data-align="right" data-sortable="true">Height</th>
    </tr>
  </thead>
</table>

## Baseline method - mean value replacement

Firstly, we will calculate mean value of weight and then replace each of the missing records with this value.
Mean value of weigth is 38.25 kg. The table with replaced records is shown below.


Errors for this method are:
Mean value method - RMSE: 35.60
Mean value method - Maximum absolute difference: 148.75


<table
  data-height="460"
  data-pagination="true"
  data-url="{{ 'assets/json/2024-05-17-missing_values_conditional_expectation/dataset_mean_replacement.json' | relative_url }}">
  <thead>
    <tr>
      <th data-field="weight_kg" data-halign="left" data-align="center" data-sortable="true">Weight</th>
      <th data-field="height_m" data-halign="center" data-align="right" data-sortable="true">Height</th>
    </tr>
  </thead>
</table>

## Conditional expectation with interactive plot

We know that height and weight features are correlated. Thus, we can use information about the height to better guess the weight missing value. We will use conditional expectation method. How we can calculate conditional expectation of weight knowing the value of height?

Assuming there are two random variables $$X$$ (weight) and $$Y$$ (height) and height value is already known $$Y=y$$ we can calculate conditional probability as a weighted sum of X values and their conditional probability <d-cite key="tabogaconditionalexpectation"></d-cite>, <d-cite key="bishop2006pattern"></d-cite>.

$$
E[X|Y = y] = \sum_{x}p(x|y)\cdot x
$$

The plot below shows how expected value of weight changes with respect to given height. You can see weight conditional probability as a red slice on a 3d plot and 2d plot on the right. When height rises, weight also rises. You can play with the plot using a slider.

<div class="l-page">
  <iframe src="{{ '/assets/2024-05-17-missing_values_conditional_expectation/interact_marginal_dist.html' | relative_url }}" frameborder='0' scrolling='no' height="600px" width="100%" style="border: 1px dashed grey;"></iframe>
</div>

## Replacing missing values with conditional expecatation

Now, when we calculated conditional expectation of weight, we can replace blank spots in a dataset with these values. The table belows shows only the records that were missing data. You can explore this table and see that weight values has been set proportionaly to height.

<table
  data-height="460"
  data-pagination="true"
  data-url="{{ 'assets/json/2024-05-17-missing_values_conditional_expectation/ce_replacement_data.json' | relative_url }}">
  <thead>
    <tr>
      <th data-field="weight_kg" data-halign="left" data-align="center" data-sortable="true">Weight</th>
      <th data-field="height_m" data-halign="center" data-align="right" data-sortable="true">Height</th>
    </tr>
  </thead>
</table>

## Errors

Now when we guessed weight values, we can calculate errors between our guess and ground truth. I've calculated these values behind the scenes in jupyter notebook. These errors are shown in table below. As you can see we obtained lower guessing errors. These errors could be further reduced by introducing other features like obesity label into our conditional expectation analysis.

<a id="tab1">Table 1.</a> Guessing errors for mean and conditional expectation methods

| Guessing Method | RMSE |  Maximum absolute difference  |
|---|---|---| --- |
| Mean value | 35.60 kg  | 148.75 kg  |
| Conditional expectation | **27.61** kg  | **95.81** kg | 