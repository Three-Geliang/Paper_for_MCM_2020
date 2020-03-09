# Variables

$R_{epu}$				Indicator of product reputation

$t$						Time increment measured by days since the earliest date on a specific dataset

$P_{ol\_hist}(t)$		Indicator of historical polarity until day $t$

$T_{hist}(t)$			Indicator of historical $T$ until day $t$

$R_{hist}(t)$			Indicator of historical rating until day $t$

$R_{epu\_hist}(t)$	Indicator of historical reputation until day $t$

$S(t)$				Quantity of all items on a dataset before day $t$

# Q2 Solution

### Statistics on a timeline

​		To talk about this problem, we first need to define what a timeline is on each dataset. Considering the variance of data, we cannot assure that the earliest items of all three datasets take place on the same day. Therefore we use relative increment to represent the shift of time.

​		On a specific dataset, we take $D_{current}$ as the current date and $D_{earliest}$ as the earliest date in the dataset, then we define the time increment $t$ as follows:
$$
t = D_{current} - D_{earliest}
$$
​		This is easy to calculate on a modern computer as date is also stored as time increment since Jan. 1st, 1970.

​		With a timeline, we can define some useful statistical indicators based on days:
$$
P_{ol\_hist}(t) = \frac{\sum_{0}^{t} P_{ol}}{S(t)}
$$

$$
T_{hist}(t) = \frac{\sum_{0}^{t} T}{S(t)}
$$

$$
R_{hist}(t) = \frac{\sum_{0}^{t} R}{S(t)}
$$

​		We use the pacifier dataset to calculate indicators above, then plot the $P_{ol\_hist}(t)-t$ and the $R_{hist}(t)-t$ graph here:

​		Note that in both graphs, $t$ is normalized.

<img src="Q2_Solution.assets/Pol_hist_R_hist-t.jpg" alt="Pol_hist_R_hist-t"  />

​		Turning points are very important because we may potentially find useful information, like the adjustments made to advertising, to products and so on; however, we may notice turning points in one graph vary from those in another. Therefore we start to think about the necessity of a new indicator which reconciles both two important indicators about customers' experience.

​		We call it Reputation Indicator, noted as $R_{epu}$. We used the weakening feature of $T$ to lower the impact of $P_{ol}$ on our $R_{epu}$.
$$
R_{epu} = T \times P_{ol} + R
$$
​		And correspondingly, we have its historical average:
$$
R_{epu\_hist}(t) = \frac{\sum_{0}^{t} R_{epu}}{S(t)}
$$
​		We plot the $T_{hist}(t)-t$ and the $R_{epu\_hist}(t)-t$ graph on the pacifier dataset as follow:

​		Note that in both graphs, $t$ is normalized.

![T_hist_Repu_hist-t](Q2_Solution.assets/T_hist_Repu_hist-t.jpg)

​		From approximately $t = 0.45$, when the data points got denser since then, $T$ slightly drops from 0.39 to lower than 0.36. However, our $R_{epu\_hist}(t)$ still remained the increasing trend, meaning it making obvious progress on the pacifier product.

​		Talking about the turning points in the $R_{epu\_hist}(t)-t$ graph, it is noticeable that the turning points of the TWO graphs above mostly appeared on this one, though we have a more smoothly increasing curve.