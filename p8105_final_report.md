Analyzing trends of organ donor enrollment in New York state
================

| Name          | Uni     |
|---------------|---------|
| Shuliang Deng | sd3258  |
| Nick Williams | ntw2117 |
| Sijia Yue     | sy2824  |
| Jack Yan      | xy2395  |
| Alina Levine  | al3851  |

[Click here for data](https://www.dropbox.com/sh/grlugs9hrksvkqr/AAAwILSg5UyMrW1CD7dWmHPNa?dl=0)

Motivation
----------

Initial questions
-----------------

Data
----

Exploratory analyses
--------------------

Formal analyses
---------------

Formal analyses of the effect of major policy changes were conducted using mixed-effects linear models with a random intercept; counties were treated as the clustering variable. The outcome was analyzed was the proportion of eligible adults registered with one of four organ donor organizations. Four successive models were created: a model only analyzing a linear effect of time; a model including a term for a spline corresponding with a 2012 policy INSERT WHAT THIS POLICY WAS; a model including a term for a spline corresponding with a 2016 policy INSERT WHAT THIS POLICY WAS; a model including a term for a spline corresponding with a 2017 policy that lowered the age from eligible enrollees from 18 to 16 years old; and a model that included all splines. All models were adjusted for the specific organization used in each county; models were compared using likelihood ratio tests.

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.675
</td>
<td style="text-align:right;">
0.008
</td>
<td style="text-align:right;">
2.66
</td>
<td style="text-align:right;">
2.69
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/overall time-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.883
</td>
<td style="text-align:right;">
0.022
</td>
<td style="text-align:right;">
2.840
</td>
<td style="text-align:right;">
2.926
</td>
</tr>
<tr>
<td style="text-align:left;">
2012 spline
</td>
<td style="text-align:right;">
-0.326
</td>
<td style="text-align:right;">
0.032
</td>
<td style="text-align:right;">
-0.389
</td>
<td style="text-align:right;">
-0.262
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/spline 2012-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.563
</td>
<td style="text-align:right;">
0.01
</td>
<td style="text-align:right;">
2.542
</td>
<td style="text-align:right;">
2.583
</td>
</tr>
<tr>
<td style="text-align:left;">
2016 spline
</td>
<td style="text-align:right;">
0.789
</td>
<td style="text-align:right;">
0.05
</td>
<td style="text-align:right;">
0.692
</td>
<td style="text-align:right;">
0.886
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/spline 2016-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
2.561
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
2.543
</td>
<td style="text-align:right;">
2.579
</td>
</tr>
<tr>
<td style="text-align:left;">
2017 spline
</td>
<td style="text-align:right;">
1.700
</td>
<td style="text-align:right;">
0.078
</td>
<td style="text-align:right;">
1.548
</td>
<td style="text-align:right;">
1.852
</td>
</tr>
</tbody>
</table>
<img src="p8105_final_report_files/figure-markdown_github/spline 2017-1.png" width="75%" />

<table class="table table-hover table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Term
</th>
<th style="text-align:right;">
Coefficient
</th>
<th style="text-align:right;">
SE
</th>
<th style="text-align:right;">
Lower bound
</th>
<th style="text-align:right;">
Upper bound
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Time (year)
</td>
<td style="text-align:right;">
3.088
</td>
<td style="text-align:right;">
0.022
</td>
<td style="text-align:right;">
3.045
</td>
<td style="text-align:right;">
3.131
</td>
</tr>
<tr>
<td style="text-align:left;">
2012 spline
</td>
<td style="text-align:right;">
-0.982
</td>
<td style="text-align:right;">
0.041
</td>
<td style="text-align:right;">
-1.063
</td>
<td style="text-align:right;">
-0.902
</td>
</tr>
<tr>
<td style="text-align:left;">
2016 spline
</td>
<td style="text-align:right;">
0.333
</td>
<td style="text-align:right;">
0.150
</td>
<td style="text-align:right;">
0.039
</td>
<td style="text-align:right;">
0.627
</td>
</tr>
<tr>
<td style="text-align:left;">
2017 spline
</td>
<td style="text-align:right;">
2.465
</td>
<td style="text-align:right;">
0.213
</td>
<td style="text-align:right;">
2.048
</td>
<td style="text-align:right;">
2.883
</td>
</tr>
</tbody>
<tfoot>
<tr>
<td style="padding: 0; border: 0;" colspan="100%">
<span style="font-style: italic;">Note: </span>
</td>
</tr>
<tr>
<td style="padding: 0; border: 0;" colspan="100%">
<sup></sup> Splines correspond with dates in which 3 major policy changes were inacted. <br>Estimates are adjusted for organ donor organization
</td>
</tr>
</tfoot>
</table>
<img src="p8105_final_report_files/figure-markdown_github/all splines-1.png" width="75%" />

Discussion
----------
