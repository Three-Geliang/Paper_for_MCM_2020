# Variables

$P_{ol}$	Indicator of the Polarity of a comment

$S_{ub}$	Indicator of the Subjectivity of a comment

$L_{w}$	Actual quantity of words in a comment

$l_{w}$	Relative quantity of words in a comment

${R}$	Rating of a product

$d$	1-D Euclidian Distance between $P_{ol}$ and the corresponding rating ${R}$

$d_{n}$	Normalized d by applying margin scaling method

$T$	Measure of the identity between $P_{ol}$ of a comment and the corresponding rating ${R}$

$E_{ff}$	Effectiveness of a rating-comment combination

# Q1 Solution

### Basic assumptions

​		To analyze comments, ratings combined with other data, traditionally we can simply use supervised learning algorithms, however, it would require clear labels for each combination in the dataset. Without labels, we must first find a way to understand the information lies within texts, and MOST importantly, extract indicative, quantitative measures for further analysis.

​		From our daily language usage, it is not hard to find that emotional words have a kind of polarity from negative to positive. And more interestingly, similar emotional word tend to cluster. Although emotional transition do exist, it would be easy to distinguish by acknowledge some conjunctions and commas.

​		Moreover, when people are expressing strongly subjective willing, the frequency of using "I" and other emotional words would increase, suggesting that we can tell the subjectivity from contexts.

​		With these two features, it enlightens us that, quantified polarity and subjectivity indicators for each comment could be highly valuable, as shopping comments are basically combinations of customers' emotional expresses and rational analysis of products. 

### Extractions from comments

​		With the assumptions above, we should first find ways to extract the Polarity $P$ and the subjectivity $S$.

​		We acknowledge that there are mature NLP (Natural Language Processing) techniques and multiple Python add-ons have been written. And we choose **TextBlob** as our analyzing tool. In this tool, it would calculate a Word Vector Space and measure these words automatically and give them a polarity value and a subjectivity value in $[0,1]$. We make sums of both measures in every comment, and get our $P$ and $S$ in each dataset.

​		But $P$ and $S$ are not the indicators we desire, as they vary a lot by the length of a comment. Therefore, we calculate the quantities of words for each comment, noted as $L_{w}$. And get our average $P_{avg}$ and $S_{avg}$ as follows:
$$
P_{avg} = \frac{P}{L_{w}}\\
S_{avg} = \frac{S}{L_{W}}
$$
​		As  $P_{avg}$, $S_{avg}$ and $L_{w}$ are all logistic values, we normalize them to $[0,1]$, and note them as $P_{ol}$,  $S_{ub}$ and $l_{w}$ in each dataset. The normalization method is as follows:
$$
l_{w} = \frac{L_{w} - MIN(L_{w})}{MAX(L_{w}) - MIN(L_{w})}
$$

$$
P_{ol} = \frac{P_{avg} - MIN(P_{avg})}{MAX(P_{avg}) - MIN(P_{avg})}
$$

$$
S_{ub} = \frac{S_{avg} - MIN(S_{avg})}{MAX(S_{avg}) - MIN(S_{avg})}
$$

​		Note that $R$ can have only five values in $\{1,2,3,4,5\}$, we also normalize it to make it coincide with other measures, meaning $R$ in the following analyzing process can only take values in $\{0,0.25,0.5,0.75,1\}$

​		The preprocessing of our datasets is complete.

### Trust Indicator

​		The rating-comment system provides both a general summary of experience and detailed textual information. But sometimes people can have inconsistency when making it. For example, there are 5-star ratings combined with less flattering words, maybe out of good will. Some would say "can be better" or "just so-so", but also gave a 5-star. Sometimes the system just records a default comment "five stars", both situations can result in low $P_{ol}$ value but high rating $R$.

​		To solve this problem, we need to design a "Trust Indicator" $T$, to sieve out those inconsistent combinations of ratings and comments. To reach the definition, we first define some supportive measures.

​		One of our teammate got inspired by the sigmoid function by his previous learning of Machine Learning, as it serves quite well when doing the logistic regression problem. The sigmoid function is as follows:
$$
Sigmoid(x) = \frac{1}{1+e^{-x}}
$$
​		And we plot the function:

​		**\*\*图1-Sigmoid 图像\*\***

​		Sigmoid function has a very good features, it maps $\R$ to $(0,1)$, and because it climbs fast near 0, it has the tendency to classify data on $\R$ into two separate parts, 0 or 1. Therefore, sigmoid function would be perfect for us to tell if a data is trustworthy or not.

​		But we still need to consider what it should be the "$x$" of the $Sigmoid(x)$. As described above, the $P_{ol}$ shows the emotional preference of a comment, therefore the inconsistency can be described by $P_{ol}$ and $R$. We define the 1D-Euclidian distance $d$ as:
$$
d = \vert P_{ol} - R\vert
$$
​		Directly fit "$d$" as our "$x$" is clearly not a good idea, as an ideal $x$ should fit on $\R$. To reach a good classification performance, we should map our $d$ to $\R$ before we send our $x$ into the function. Here we choose $tan(x)$. Practical calculation shows that $d$ usually not fit the range $[0,1]$ perfectly, they close to each other a bit, so we apply the same normalization method as we have done in the preprocessing. In each dataset, we take $d_{n}$ as the normalized $d$:
$$
d_{n} = \frac{d - MIN(d)}{MAX(d) - MIN(d)}
$$
​		Then we map $d_{n}$ to $\R$, result noted as $t(d_{n})$:
$$
t(d_{n}) =k* tan(\pi * (0.5-d_{n}))
$$
​		Now we get $t(d_{n})$ as our desired "$x$", and we send it into $Sigmoid(x)$, considering the edge points, we can define our $T$ as follows:
$$
{T} = \left\{\begin{matrix}
1 &d=0\\ 
\frac{1}{1+e^{-t(d_{n})}} & 0<d<1\\ 
0 & d=1
\end{matrix}\right.
$$
​		To be specific, $k$ is a positive constant to adjust how fast it can be when the curve climbs as our $d_{n}$ is near 0.5. If $k$ is too large, few points get a middle middle value of $T$; otherwise we reach a bad classification performance, which we would not desire. The $T-d_{n}$ relation graph is as follows:

<img src="Q1 Solution.assets/Q1dnT.svg" alt="Q1dnT"  />

​		By testing for multiple times, we find choosing $k$ in $(1,2)$ can reach a balance between the diversity of $T$ and classification performance.

### Evaluation of the Trust Indicator

​		We plot the $T - P_{ol}$ Graph as follows:

<img src="Q1 Solution.assets/T-Pol.jpg" alt="T-Pol"  />

​		For products with lower ratings, our $T$ increases as  $P_{ol}$ drop, and vice versa for those with higher ratings. For middle-rated products to have higher $T$, because the average $P_{ol}$ is around 0.4, and 1 & 2 star correspond to 0.25 & 0.5 in normalized $R$, which naturally result in smaller $d$. 

​		Although this property means $T$ cannot serve as perfect indicator of whether a comment can be trusted, it is still needed for further analysis, as it weakens comments with extreme ratings, specifically 5-Star and 1-Star, helping us pick out really useful comments.

​		When $k = 1.2$, about 12% of all the data we have satisfies $T > 0.8$.

### Effectiveness Indicator

​		When we are shopping online, some comments of a relatively longer length with objective descriptions could sometimes inspire or wave our willing. Although voting on comments can indicate whether they are useful or not, it does not work when a new product just come online. Moreover, voting can be faked or can simply be an expression of emotion. Therefore, voting can be referable, but not reliable.

​		Since we already have $T$, we should use it along with other data we have to design a much more reliable indicator to help us find effective comments.

​		Relating to our daily life, we think a effective comment should be:

- With sufficient textual information
- Mostly objective
- Trustworthy

​		Relatively seen, some comments are much longer than the others, we would also like to see comments of medium size which can provide us useful information. We define our Effectiveness Indicator as $E_{ff}$:
$$
E_{ff}=(1-S_{ub}) * \sqrt{l_{w}}*T
$$
​		We tested this by looking at the top 100 comments with the highest $E_{ff}$, useful comments with or without votes both appeared in our view, meaning this indicator do can help us pick out effective comments.

