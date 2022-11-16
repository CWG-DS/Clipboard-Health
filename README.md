# Profit Optimzation

<img src="Images\ClipHealth_Img.png" alt="drawing" width="150"/> 

**Preliminary Analysis**

The main goal of this project is the optimization of a pricing strategy in which potential profits are maximized given the restrictions imposed by the document "Clipboard Health Pricing Case Study" available in this repository.

A ride-hailing service is created and will work for a total duration of 12 months. Potential riders are paired with drivers and charged 30$ per ride. Our main variable of interest is the amount payed to drivers. As such, we will aim to maximize profits by increasing the total amount of rides while reducing as much as possible the amount payed to drivers. 

Drivers are free to accept or not an assignment based on how much they will get payed for it. We are given a sample of 1000 data points which describes their decision criteria (Accepted/Declined) based on the amount of compensation offered. This data can be reviewed in the "driverAcceptanceData.csv" available in this repository. An example of the data is shown below:

|    | PAY   |   Accepted |
|---:|:------:|:---------:|
|  0 | 29.358732  |        0 | 
|  1 | 22.986847  |        0 | 
|  2 | 18.020348  |        0 | 
|  3 | 45.730717  |        1 | 
|  4 | 14.642845  |        0 | 

"PAY" describes the amount offered to the drivers and "Accepted" codifies wether the offer was accepted [1], or declined [0].

Given the data structure we diveded our sample into two subsamples based on if the offers were accepted or declined and plotted them through the use of a boxplot.

<img src="Images\Raw_Data_Distribution.png" alt="drawing"/> 

As it would be expected the distribution pertaining to [Declined] rides are characterized by a lower [PAY] than those of [Accepted] rides as shown in the figure above as well as by their average values (18.62$, and 32.08$ respectively). Given that there are outliers within our sample we decided to remove them through the use of the Interquartile Method, removing data points which exceeded 1.5 times the interquartile range. After which we tested for normality through the use the Shapiro-Wilk Test determining that both destributions are in fact Gaussian/Normal distributions. 

We then proceeded to collapse both distributions into a single plot describing the probability of rejection for every payment point. Given our small sample size we decided to group data points in 1$ intervals. The results are shown bellow:

<img src="Images\Collaps_Probability.png" alt="drawing"/>

As expected there is a reduction in the probability of declined rides as we increase the pay offered to drivers. However, even with a low sensitivity (1$ intervals) we dont have enough data to plot a smooth transition between intervals. As we can see, there is noticeable sudden "jumps" within the (31, 32] and (37,38] intervals as well as no values within the (2,3] interval. In order to solve these problems we increased our sample size by generating normal distributions with the same characteristics as our initial ones. This process is described in the next section.

**Data Generation**

Given that both Decline and Accepted ride request both exhibit a Gaussian/Normal distribution we are able to generate further data points based on the characteristics (mean and standard deviation) of their respective distributions. We will therefore recreate these distributions with a sample size of 100.000.000 data points per condition. This will enable us to generate a more accurate estimate of which rides were accepted/declined for every Driver Pay range as well as increasing our sensitivity as we will be able to create smaller intervals (0.01$ instead of the previously used 1$ range). Our new data distributions are as follows:

<img src="Images\Data_Generation.png" alt="drawing"/>

Note: Values bellow 0 are discarded. Total number of values discarded per contion is 0.05% [Declined] and < 0.001% [Accepted]. No effects are expected from the removal of such a small portion of the sample.

<img src="Images\LikelihoodRejection_NewData.png" alt="drawing"/>

As seen above, our resulting [% of Decline Rides] describes an inverse sigmoid distribution. We can also appreciate some deviance from this distribution at the right tail end caused by extreme values. However, since 30$ marks our break even point it will be irrelevant for our specific case and therefore will have no effect on our analysis.

Now that we have our data cleaned and sorted we will proceed with the pricing strategy portion of our work.

**Fixed Pricing Strategy**

We will first maximize profits for a fixed payment method. This is, Drivers will be payed a fixed amount for every ride they partake in throughout the 12 months duration of the program. To do so we will employ a costum function which will itterate 12 times, once per month, through every possible value [0.01$ - 30.01$] and output the total profit obtained per value input. To make this process as clear as possible we will go through the logic of the function first and then showcase the code it self.

1. Function Set Up

We import the necessary libraries and set up our initial parameters which are:

- Lambda Value: 1
- Number of Riders: 1.000
- Maximun Possible Riders: 10.000
- Exhausted Riders: 0 
- Empty Profit List: []

2. Main For Loop

Since our program will last for a total of 12 months we will itterate the process of calculating our profits 12 times, once per month. There is a trigger set in place to stop the process if the number of Exhausted Riders goes over our preestablished 10.000 mark, this would mean that we have exhausted all of our possible riders and the program ends. 

2.1 Calculating our Poisson Distribution

The number of rides per rider is described by a Poisson Distribution. For our first run, the number of samples of this distribution will be 1.000 and our lambda will be 1. The resulting distribution is saved within TotalLamb_Dist list. This list is then used in order to extract two essential components:
- Unique_Values: A list containing the unique number of ride requests within TotalLamb_Dist.
- NRi_NRe: A list containing the number of riders per unique number of ride requets.

Example:

        Unique_Values = [0, 1, 2, 3, 4, 6]
        NRi_NRe = [300, 200, 100, 50, 10, 1]

        300 Riders have requested 0 Rides
        200 Riders have requested 1 Ride
        100 Riders have requested 2 Rides
        50  Riders have requested 3 Rides
        10  Riders have Requested 4 Rides
        1   Rider  has  Requested 6 Rides

2.2 Number of Requests Accepted per Number of Requests Made

We then proceed to calculate the number of requests accepted per Unique_Value. This is achieved through our first indented for loop which requieres our previous lists as inputs. The for loop begins by dumping the initial element of each list as it corresponds with the number of users who have not requests any rides. Then it goes through each element within our NRi_NRe list and calculates the number of accepted rides per rider per number of requests given the probability of acceptance based on the Driver Pay amount as follows:

        Given a 20% Acceptance rate and
        Lambda_Values = [1, 2, 3, 4, 6]
        NRi_NRe_Values = [200, 100, 50, 10, 1]

        Results:
        200*0.2^1 = 40.00; Out of the 200 people which requested a ride 1 time 40 were Accepted

        100*0.2^1 = 20.00 
        100*0.2^2 =  4.00

        20 - 4    = 16.00; Out of the 100 people which requested a ride 2 times 16 were Accepted 1 time
                     4.00; Out of the 100 people which requested a ride 2 times  4 were Accepted 2 times
    
        50*0.2^1  = 10.00
        50*0.2^2  =  2.00
        50*0.2^3  =  0.04

        10 - 2    =  8.00; Out of the  50 people which requested a ride 3 times 8 were accepted 1 time
        2 - 0.04  =  1.96; Out of the  50 people which requested a ride 3 times 1.96 were accepted 2 times
                     0.04; Out of the  50 people which requested a ride 3 times 0.04 were accepted 3 times
        (...)

Once the function goes through each element it groups riders by the amount of requests accepted, regardless of the amount of requests made and rounded to the closest whole number. Finally, it drops values which are equal to zero. The final result is a list of accepted riders (Rider_Lamb) whose element position indicates the number of rides accepted, much like our previous Unique_Values and NRi_NRe List.

Example for x = 20:

        Rider_Lamb = [113.0, 9.0, 1.0]

        113 riders have been Accepted 1 time
          9 riders have been Accepted 2 times
          1 rider  has  been Accpeted 3 times

2.3 Calculating Profit

Now that we have a list which describes the number of accepted rides we can easily calculate our profit as the difference between our earnings, defined as number of total rides accepted x 30, and costs, defined as total of rides accepted x Driver Pay.

2.4 Next Loop Set Up

Our final step is to determine the parameters for our next loop which are:

Exhausted Riders: Number of riders who exit the program because either they did not use the service or were not accepted once. 

New Rider Pool: Established by our Accepted Ride list (Rider_Lamb) previously described and an additional 1000 new users added to the initial element of the list.

New Lambda Values: A list which contains the values of our new lambda values which is determined by the element positions of our Accepted Ride list. 

After these elements are established the program itterates the process another 11 times before outputing a final profit value which is equal to the sum of the profits generated throughout the 12 months.

The code itself is shown bellow:

        def profit(x):

            import numpy as np
            from scipy.stats import poisson
            import itertools

            lamb = [1]                                                                           
            Ri_Retention = [1000]                                                               
            Max_Riders = 10000                                                                   

            Exhausted_Riders = 0                                                                 
            Profit = []                                                                          


            for month in range(0, 12):
                if Max_Riders <= Exhausted_Riders:
                break

                Total_LambDist = []                                                                  
                for ele in range(0, len(lamb)):
                    Total_LambDist.append(poisson.rvs(mu=lamb[ele], size=int(round(Ri_Retention[ele],0))))

                Total_LambDist = [item for sublist in Total_LambDist for item in sublist]            
                Unique_Values = list(set(Total_LambDist))                                           

    
                NRi_NRe = []                                                                         
                for i in range(0, len(Unique_Values)):                                               
                    NRi_NRe.append(np.count_nonzero(Total_LambDist == Unique_Values[i]))            

    
                    Lambda_Values = Unique_Values[1:]
                    NRi_NRe_Values = NRi_NRe[1:] 

                    Probability = []

                    for i in range(0, len(NRi_NRe_Values)):
                        Inner_List = []
                        Exp = Lambda_Values[i]
                        for e in range(1, 1+Exp):
                            Inner_List.append(NRi_NRe_Values[i]*Acceptance_Rate(x)**e)
                        for u in range(0, len(Inner_List)-1):
                            Inner_List[u] = Inner_List[u] - Inner_List[u+1]
                        Probability.append(Inner_List)

                Rider_Lamb = [round(sum(i),0) for i in itertools.zip_longest(*Probability, fillvalue=0)]

                Rider_Lamb = [i for i in Rider_Lamb if i != 0]

                Earn = []
                for i in Rider_Lamb:
                    Earn.append(i * (Rider_Lamb.index(i) + 1) * 30)
                Earn = sum(Earn)

                Spen = []
                for i in Rider_Lamb:
                    Spen.append(i * (Rider_Lamb.index(i) + 1) * x)
                Spen = sum(Spen)

                Profit.append(Earn - Spen)

                Attrition = sum(Ri_Retention) - sum(Rider_Lamb)
    
                Ri_Retention = Rider_Lamb
                if  len(Ri_Retention) == 0:
                    Ri_Retention = [0] 
                Ri_Retention[0] = Ri_Retention[0] + 1000

                lamb = list(range(1, len(Rider_Lamb)+1))

                Exhausted_Riders = Exhausted_Riders + Attrition                 
            return sum(Profit)

We then itterated the previously described function for every Driver Pay value of interest [0.01, 30.01] and plotted the Profit outputs per Driver Pay with the following results:

<img src="Images\Profit_DriverPay.png" alt="drawing"/>

The figure above seems to represent a log normal distribution. However, this is taken from just one sample per Driver Pay. We could further itterate throughout the whole range of intervals but since we are only interested in maximizing our profits we can set a cut of point at 25.000$. We then obtain the min and max intervals that satisfy this condition and itterate over this smaller sample, saving computing time. By doing so we narrow down our optimal pricing strategy to the Driver Pay interval [22.68, 27.83] with an average profit margin of 28375.12$. However, as seen by the figure bellow, we can narrow our interval even further and itterate our costum function 100 times in order to achieve greater accuracy in our estimate.

<img src="Images\Initial_Estimate.png" alt="drawing"/>

Bellow are the final results of our fixed pricing optimization. We can expect to achieve on average 29758.48$ by paying drivers a stardard amount anywhere between 24.15$ and 26.82$.

<img src="Images\Final_Fix.png" alt="drawing"/>

**Variable Pricing Strategy**

In order to explore the viability of a variable price strategy we studied the progression of ride requests based on 4 set Driver Pay amounts [25, 30, 35, 40]. These amounts correspond to:

        25$: A value within our maximun profit interval using a fixed pricing strategy.
        30$: Our breakeven point.
        35$: A value equidistant over the breakeven point.
        40$: An "extreme" value over the breakeven point. 

 These amounts will give us a good understanding of how the userbase interacts with our service over different price points and how will a change in pricing affect said interaction. We focused on three different variables in order to evaluate the fisability of a variable pricing strategy: Number of Rides Accepted per month, Percentual Increase in Rides per month and Profit Generated per month. 
 For a variable pricing strategy to achieve a greater performance than our previously discussed fixed strategy it will have to significantly outperform our fixed pricing strategy by increasing the number of accepted rides and mantain them to an extent when the pricing is changed in order to offset initial losses. 

In order to set a baseline we modified our previous costum function so it outputs our desired variables. The results are shown bellow:

<img src="Images\Var_ST.png" alt="drawing"/>

As expected, there is an increase of Rides Accepted as we increase Driver Pay. It is not supprising due to our inverse sigmoid Ride Decline distribution. The higher the Driver Pay is, the fewer rides are declined. This factor in addition to the way riders request rides as defined by our Poissons distributions perfectly explains this effect. However, this progresive increase in the amount of rides requested does taper and eventually levels to a constant, much like a logarithmic function. Our second figure better illustrates this progressive decrease in growth for each subsequent month. Lastly, we can observe the profit balance per month which, as expected, is closely related to our first subplot within the figure, albeit inverted for values above 30$. Due to the high increase of losses we would suffer and the greatest increase of Rides Accepted taking place early on, we estimate that a change of price would be most beneficial between the second and forth month.

We then explored the same parameters but changing Driver Pay to 25$, as it has been established as one of the most protifable Driver Pay ranges, on the fourth month with the following results:

<img src="Images\Var_ST_2.png" alt="drawing"/>

Note: Due Driver Pay 25$ suffering from high user Attrition rate, users are exhausted on month 11, therefore there are no values present in the final month. 

The resulting figure is identical to the former in the initial 3 months. However we can clearly see the effects of the pricing change with a rapid decline of rides accepted which converges with the 25$ plot. Our third subplot shows an alarming insight with price reversal not showing a proportional gain in profit to the losses incurred in the initial months. This effect points towards the unfiseability of employing a variable pricing strategy. After further analysis varying the month of implementation of our price change we can conclude that the earlier the change is established the better the outcome. However the pattern shown in the above figure repeats it self. Thus, we conclude that no variable pricing strategy is able to surpass our fixed pricing strategy.

**Conclusion**

After a deep exploration of the data and restrictions given to us by the problem we conclude that the most profitable strategy is a fixed pricing strategy where drivers are payed an amount between 24.15$ and 26.82$ with an expected result of 29758.48$ in profits. Additional analysis using bootstrap methodology will be able to give us a confidence interval to our results. Replication of this project might find deviations in the pricing range in the cent range due to variations in the inverse sigmoid distribution caused by slight differences in the accepted/declined data generation. For future projects a seed will be attached to the generation of data in order for better replication of our findings. 



