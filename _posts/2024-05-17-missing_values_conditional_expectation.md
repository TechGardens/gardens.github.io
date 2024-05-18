---
layout: distill
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

During my recent study of machine learning concepts in book by C. Bishop <d-cite key="bishop2006pattern"></d-cite>, I came across the concept of conditional expectation.This sparked a realization: why not leverage this method for handling missing data in datasets? Thus, this article came to fruition.

Handling missing values is a common challenge in real-world datasets, where empty cells can disrupt analysis. These “missing values” need to be addressed before we can effectively use the data. To bridge these gaps, we often turn to strategies like using the mean or median of a feature, or simply removing the incomplete records. Using the mean or median is a straightforward solution, especially when features are uncorrelated. By calculating the global mean of a specific feature and filling in the missing values, we can quickly patch the dataset. However, this method might compromise the data’s integrity.

For this reason, another method for determining missing values is needed. In this article, we will explore and test conditional expectation as an alternative approach. Conditional expectation takes into account the relationships between features, providing a potentially more accurate and context-aware way to handle missing data. By leveraging the dependencies among variables, we aim to fill in the gaps more effectively and preserve the integrity of the dataset.

Before we dive in, it's important to note that while I'm excited to share my discoveries, I want to acknowledge that there may already exist prior scientific research on this topic. This article stands as my independent investigation, aiming to validate and demonstrate the effectiveness of conditional expectation through practical experimentation.

### Why data's integrity suffers? - short example

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

Although there's no code here, you can check out the behind-the-scenes work in the notebook on repository <d-cite key="repo"></d-cite>.


## Data characteristics

### Information about dataset 
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

### Relationship between weight and height visualised

The relationship between weight and height is visualized in the images below. The scatter plot on the left shows the original data, while the plot on the right represents the data fitted into a Multivariate Normal Distribution. These plots indicate that **the weight and height features are correlated**.  Later, we'll leverage this correlation to estimate missing weight values based on the known heights using the conditional expectation method.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/2024-05-17-missing_values_conditional_expectation/scatter_plot.png" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/2024-05-17-missing_values_conditional_expectation/distribution_plot.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Data scatter and distribution plots.
</div>


## Baseline method - mean value replacement

First, we calculated the mean value of weight and used it to replace each missing record. The mean weight is 38.25 kg. The table with the replaced records is shown below. These calculations were performed behind the scenes using the Python notebook <d-cite key="repo"></d-cite>.

The errors for this method, also computed in the notebook, are as follows:

- Mean Value Method - RMSE: 35.60
- Mean Value Method - Maximum Absolute Difference: 148.75

These metrics will serve as a baseline to compare with the more sophisticated conditional expectation approach.


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

### Motivation and quick math

Given the correlation between height and weight, we can use the known height to better estimate the missing weight values. This can be achieved using the conditional expectation method. But how do we calculate the conditional expectation of weight given the height?

Assume we have two random variables, $$X$$ (weight) and $$Y$$ (height), and the height value is already known $$Y=y$$. We can calculate the conditional probability as a weighted sum of $$X$$ values and their conditional probabilities $$p(X | Y)$$. Mathematically, this can be expressed as <d-cite key="tabogaconditionalexpectation"></d-cite>, <d-cite key="bishop2006pattern"></d-cite>:

$$
E[X|Y = y] = \sum_{x}p(x|y)\cdot x
$$

This equation tells us that the expected value of weight, given a specific height, is a sum of all possible weights weighted by their conditional probabilities given the height. This approach leverages the relationship between the variables to provide a more accurate estimation than simple mean replacement.

### Conditional probability - interactive plot

The plot below illustrates how the expected value of weight changes concerning a given height. The red slice on the 3D plot and the 2D plot on the right show the weight's conditional probability. As the height increases, the weight also tends to increase. You can interact with the plot using a slider to see these changes dynamically.

This method aims to provide a more nuanced and accurate way of handling missing data by utilizing the inherent relationships within the dataset.

<div class="l-page">
  <iframe src="{{ '/assets/2024-05-17-missing_values_conditional_expectation/interact_marginal_dist.html' | relative_url }}" frameborder='0' scrolling='no' height="600px" width="100%" style="border: 1px dashed grey;"></iframe>
</div>

## Replacing missing values with conditional expectation

Now that we've calculated the conditional expectation of weight based on height, we can replace the missing values in the dataset with these estimated values. The table below displays only the records that were initially missing data. As you explore this table, you'll notice that the weight values have been set proportionally to the height, reflecting the relationship we observed between these features.

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

After estimating the weight values using both the mean and conditional expectation methods, we can now calculate the errors between our guesses and the ground truth. These errors have been computed in a Python notebook <d-cite key="repo"></d-cite> and are presented in the table below. As shown, the guessing errors are lower when using the conditional expectation method compared to the mean value approach. This improvement suggests that leveraging the relationship between height and weight through conditional expectation leads to more accurate estimations.

<a id="tab1">Table 1.</a> Guessing errors for mean and conditional expectation methods

| Guessing Method | RMSE |  Maximum absolute difference  |
|---|---|---| --- |
| Mean value | 35.60 kg  | 148.75 kg  |
| Conditional expectation | **27.61** kg  | **95.81** kg | 

These results demonstrate the effectiveness of incorporating conditional expectation analysis into handling missing data. Additionally, further improvements could be achieved by integrating additional features, such as obesity labels, into our analysis.

## Conclusions


The article highlights the importance of addressing missing values in datasets and the potential drawbacks of using simplistic methods like mean value replacement. In response, it introduces conditional expectation as a more sophisticated approach, leveraging the context provided by correlated variables to fill in missing data. Through empirical testing, it demonstrates that conditional expectation can effectively reduce guessing errors compared to mean value replacement, thereby enhancing the integrity of the dataset.

For those interested in delving deeper into this concept, the Python notebook available in the repository <d-cite key="repo"></d-cite> provides a comprehensive exploration of the methodology and its implementation. By embracing methods like conditional expectation, analysts can better preserve the integrity of their data and extract more accurate insights from complex datasets.