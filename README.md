# Rating Decline Analysis

<img src="Images\imdb.png" alt="drawing" width="150"/> 

**Profit Optimization**



Brief Description of the Project:

The main goal of this project is the optimization of a pricing strategy in which potential profits are maximized given the restrictions imposed by the document "Clipboard Health Pricing Case Study" available in this repository.

A ride-hailing service is created and will work for a total duration of 12 months. Potential riders are paired with drivers and charged 30$ per ride. Our main variable of interest is the amount payed to drivers. As such, we will aim to maximize profits by increasing the total amount of rides while reducing as much as possible the amount payed to drivers. 

Drivers are free to accept or not an assignment based on how much they will get payed for it. We are given a sample of 1000 data points which describes their decision criteria (Accepted/Declined) based on the amount of compensation offered. This data can be reviewed in the "driverAcceptanceData.csv" available in this repository. An example of the data is shown below:

|    | PAY   |   Accepted |
|---:|:--------------|---------:|
|  0 | 29.358732  |        0 | 
|  1 | 22.986847  |        0 | 
|  2 | 18.020348  |        0 | 
|  3 | 45.730717  |        1 | 
|  4 | 14.642845  |        0 | 





It is commonly percieved that as a TV Series goes through its Seasons, its 
quality tends to decline. The goal of this project is to test this hypothesis
using statistical analysis. For this purpose we will go through the entire data
analysis pipeline, from data collection to hypothesis testing via univariate means.

In consequence, our null hypothesis (H0) states that there are no significant 
changes in Rating throughout a TV Series life-span. On the other hand, our 
alternative hypothesis (H1) states that there is infact a change in Rating and 
we expect this rating to decline. 
In this project we decided to group Seasons by Stages as follows:
- Early Season Stage: Seasons [1-3]
- Mid Season Stage: Seasons [4-6]
- Late Season Stage: Seasons [7-9]
- Extra Season Stage: Seasons 10 and onward. 

These discreet divisions will enable us to increase our sample size per group,
rather than diluting it through-out each seperate season. These stages will be
our Independent Variable (IV) and Rating our Dependent Variable(DV).

**Data Collection**

In order to create our data set we used IMDB user yohmarit-143-626778's list of
longest running TV Shows created on the 19th of Febuary 2019. With this list in mind
we developed a Selenium-based WebScraper in order to gather relevant information
of each of these series. The variables collected were:

- Series Name
- Season (Number)
- Episode Number
- Episode Name
- Rating
- Number of Votes
- Airdate

Once collected, the data was saved as a CSV file named Series_Data_Final which is
provided with this project. The code of the WebScraper with detailed descriptions can also be found within the DataAnalysis.ipynb file within this repository.

The data consists of 19.656 rows which correspond to the total number of episodes across all series, and 7 columns, one for each variable of interest. 

An extract of the data set can be seen below:

|    | Series Name   |   Season |   Episode Number | Episode Name                      |   Rating |   Number of Votes | Airdate    |
|---:|:--------------|---------:|-----------------:|:----------------------------------|---------:|------------------:|:-----------|
|  0 | The Simpsons  |        1 |                1 | Simpsons Roasting on an Open Fire |      8.1 |              7640 | 1989-12-17 |
|  1 | The Simpsons  |        1 |                3 | Homer's Odyssey                   |      7.3 |              4502 | 1990-01-21 |
|  2 | The Simpsons  |        1 |                5 | Bart the General                  |      7.9 |              4761 | 1990-02-04 |
|  3 | The Simpsons  |        1 |                7 | The Call of the Simpsons          |      7.7 |              4154 | 1990-02-18 |
|  4 | The Simpsons  |        1 |                9 | Life on the Fast Lane             |      7.4 |              3970 | 1990-03-18 |

**Data Filtering**

Once we obtained our raw data we established a minimun threshhold of 80% non-null entries (Episodes) for each TV Series. This minimizes the impact of null values caused by errors derived from the WebScraper. Aditionally we filtered episodes which had less than 10 votes (Number Votes) each, due to the possible impact that a few voters could have on the overall Rating variance. This low entry point was established after a quick overview of the data which concluded that many of the series present within the data have not gardnered the interest of current IMDB users even if they were very influencial when they aired.

Using this procedure we filtered out 13 TV Seires out of the 64 we started with. In conjunction with our additional filter, based on number of votes, we reduced the number of entries from 19.656 to 15.226.

**Descriptive Analysis**

In order to evaluate the consistency of our sample across time we plotted both the Episode Count and Rating by Year.

Our first figure (Top) describes the distribution of the number of episodes across time, grouped by Year. As we can observe it takes the form of a bimodal distribution with the first peak ranging between the years [1950-1970] and the second one between the years [1990-2020]. This type of distribution would entail some problems as it is not evenly distributed across time and thus time could become a factor that influences our results. However, as shown by our second figure (Bottom) which describes averaged rating distribution across time grouped by Year, it does not seem to affect Rating in any drastic maner as it stays well within the [6-8] point range. Further analysis should be taken into consideration in order to rule out possible effects of time.


<img src="Images\Time_Plot.png" alt="drawing"/>


**Sample Distribution Across Seasons**

To evaluate our sample distribution across Seasons we plotted Episode Count and Series Count across Seasons. 

Our first figure  describes the number of episodes within our sample across seasons. At first glance it seems clear that our distribution is right-skewed, with values remaining fairly consistent within the initial nine seasons and then decreasing in an logarithmic fashion.

<img src="Images\Episode_CountxSeason_Plot.png" alt="drawing"/>

 This trend is also present when comparing the number of series within each season as seen in the next figure. We can then infer that the decline in episode count is probably caused by the decline in the number of series with each succesive season after number 9. 
 
 <img src="Images\Season_CountxSeason_Plot.png" alt="drawing"/>
 
 To back up this inference we performed a Pearson correlation between Episode Count and Series Count across Seasons obtaining a nearly perfect correlation (r = 0.99) and then plotted said correlation onto a scatterplot, as seen below, once values where normalized.

<img src="Images\Scatterplot.png" alt="drawing"/>

Lastly, we grouped Episode Count by the Series Stages we previously defined (Early[1-3], Mid[4-6], Late[7-9] and Extra[10, 34]) as shown bellow. 

<img src="Images\Episode_CountxStage_Plot.png" alt="drawing"/>

As expected, these groupings distribute our sample into similar amounts which will enable us to analyze Averaged Rating distribution between them and determine if there are significant differences between Series Stages. However, as our Extra Series stage is comprised of only a couple of series, it will inevitably only reflect their Averaged Rating distribution and not the sample as a whole. Therefore, any conclusions drawn from it should remain anecdotal at best.

**Statistical Analysis**

Once we ascertained the validity of our sample we proceeded onto the actual statistical analysis. We first plotted Rating across seasons in order to get a sense of its progression. 

<img src="Images\RatingxSeason_Plot.png" alt="drawing"/>

As shown in the figure above, at first glance there does not seem to be any significant change of Episode Rating across Seasons [1-9], with the only real noticeable change starting around Season 20. 

We then grouped Episode Rating by the pre-established Season Stages as shown bellow. 

<img src="Images\RatingxStage_Plot.png" alt="drawing"/>

By doing so we uncover a large amount of outliers which will have to be dealt with before we can employ an Independent Sample Student's T-Test to reveal any statistically significant change in Episode Rating across Season Stages. 

Outliers were removed sample-wise. The criteria used was any data point that deviated 1.5 times the interquartile range.

<img src="Images\RatingxStage_OutlierRemoved_Plot.png" alt="drawing"/>

**Testing for Statistical Significance**

Due to our main goal being determining if a Series quality **declines** over time expressed as a reduction in Episode Rating across Season Stages we need to employ a One-Sided Student's T-Test pairing each of our defined Stages. The results are shown below. 

|             |   statistic |   pvalue |
|:------------|------------:|---------:|
| Early_Mid   |       -0.46 |     0.68 |
| Early_Late  |        9.32 |     0    |
| Early_Extra |       15.12 |     0    |
| Mid_Late    |        9.69 |     0    |
| Mid_Extra   |       15.49 |     0    |
| Late_Extra  |        5.39 |     0    |


Based on these results it seems that there is no significant decline between our Early Stage[1-3] and Mid Stage[4-6]. However, there is a statisticaly significant decline in Episode Rating between Early and both Late and Extra stages, as well as between our Mid and both, Late and Extra stages. Finally, there is also a significant decline between our Late and Extra stages, however as noted above, this should only be treated as anecdotal due to a lack of validity within the final stage caused by uneven sampling.

**Final Result Plot**

In order to express our final results we once again plotted Episode Rating by Season Stage, this time dropping our Extra Stage due to its anecdotal nature.
The corresponding box plot can be seen below:

<img src="Images\RatingxStage_Significance.png" alt="drawing"/>

**Conclusion**

Given our sample of 51 series and 15.000 plus data entries, it seems that there is indeed a significant decline in a TV Series quality during its final seasons. 

It must be stated that while many different factors come in to play when it comes to a series cancelation, its quality as measured by its rating is certainly one. 

Going forward it would be interesting to increase our sample and include TV Series with different life spans and aim to determine the critical rating threshold that predicts its cancelation. One possible approach to this calculation is by using a sigmoid dosage mortality curve, an approach widely use in pharmacology. Here we would aim to determine the rating at which 50% of TV Series get canceled. 
