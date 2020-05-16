# COVID-19 and Temperature

As we approach the summer months, how might the rate of COVID-19 infections in humans increase or decrease? I aim to find a correlation between the infection rate and the temperature. To accomplish this, I will gather data about various US counties' daily:

- Total number of infections
- Total number of deaths
- Average temperature
- Infection Rate

I wanted to drill in the data down to the county and month in order to leave out rooms for error such as the different temperatures per month and the varied temperatures between counties. This level of specificity should be fairly sufficient since counties generally have the same temperature due to small variations in latitude, longitude, and biomes between them. I also elected to choose the United States only because of the varied temperatures and biomes it has, and also the US had widespread testing insufficiencies. If I included other countries, that would skew the data.

### Datasets

I chose 66 counties out of the BigQuery Public Data COVID-19 USAFacts dataset and obtained the daily deaths, population and weather data of each county.

USAFacts Google COVID Infections Dataset: https://console.cloud.google.com/marketplace/details/bigquery-public-datasets/covid19-dataset-list?filter=solution-type:dataset&filter=category:covid19&id=4a850823-3f83-48f5-92d1-01ba6f8ed81e

```SELECT * FROM `bigquery-public-data.covid19_usafacts.confirmed_cases```

```SELECT * FROM `bigquery-public-data.covid19_usafacts.deaths```

Wunderground weather: https://www.wunderground.com/history

Populations: https://api.census.gov/data/2019/pep/population?get=POP&for=county:*&in=state:*&key=<KEY>

# Cluster Analysis

1. Adequately describe the function you would like to calculate. (specify the domain and range, semantics)
    * Is this a reasonable task?
    * What are its input and output?
2. Choose parameterized functions that are easier to calculate then the original function. (e.g. choosing a neural network, and specified layers)
    * You will compose these "easier to calculate" functions in hopes of recovering an approximation of the original.
    * The proper choice of parametrized functions is extremely important. We want them to reflect properties of the function we are trying to model. This will result in less parameters and easier training.
3. Create/Choose a loss function that captures how well your model does. (e.g., mean squared error, crossentropy, etc..)
    * It helps if the loss is differentiable with respect to the parameters.
4. Given the loss and composition of parameterized functions, choose an optimizer that updates the parameters to minimize the loss.( e.g., sgd, adagrad, adam, ...)
5. Evaluate how well the model did.
    * Can we change any of model parameters to achieve better results,e.g., learning rate, number of composite functions, etc...?
    * Is the model overtrained/undertrained?

### Data

In this project, I chose to apply clustering to my dataset. I tested various feature combinations of the variance of the temperature, average temperature, and infection rate for 66 counties in the United States over 90 days. From it, I hoped to attain some sort of relationship inside each cluster given by the clustering algorithm to make sense of how temperature affects the infection rate of COVID-19. I hypothesized that the higher the temperature the lower infection rate and vice versa. Despite this effort, I do not believe there is much to be extracted from this model based on this hypothesis; though, I do think this task is reasonable in terms of workload. The difficult part was finding, sorting, and aggregating data for the infections, deaths, populations, and temperature. I used the Google BigQuery dataset, US Census API and Wunderground scraping and sanitized and organized the data.

These are the calculations involved with calculating the infection rate $\beta$, sample mean temperature $M_n$, and sample variance $V_n$ of the temperature for each county.

## Infection rate

$\displaystyle \frac{dS}{dt}=-\frac{\beta I S}{S+I+R} \implies \beta = -\frac{dS}{dt} \frac{S + I + R}{IS}$

$\displaystyle \frac{dI}{dt} = \frac{\beta I S}{S + I + R} - \gamma I$

$S(t) = $population

$I(t) = $infected

$R(t) = $dead

## Sample mean

$\displaystyle M_n = \frac{1}{n} \sum_{i = 1}^n T_i$

$T_i =$ temperature per day

## Sample Variance

$\displaystyle V_n = \frac{\sum (T_i - M_n)^2}{n - 1}$

### Modelling

In terms of clustering, the algorithm maps points to a group instead of giving function in a traditional machine learning sense. For example, if a point lands within a certain clustered area, it will return a class corresponding to that clustered area. Additionally, it does not have a loss or optimizer in a traditional sense either. Instead, we have parameters we tweak parameters such as the minimum cluster size that adjusts how many points should be in a cluster at minimum and the metric, or how we want to quantify whether a point belongs into a cluster or not. In my case, I tweaked the minimum cluster size to be five in order to have distinct, spanning clusters instead of many small ones. Also, I used the Euclidian metric to make the distances the metric for clustering.

For each attempt, I graphed the data points and how the algorithm clustered the data. I colored each point according to their cluster. Gray (-1) specifies unclustered data, while other colors (0,1,...) specifies clustered data. I also colored each county according to their cluster.

For the first attempt, I modeled the all three features (variance of temperature, average temperature, and infection rate) and clustered it. It seemed to be clustered based on the variance of the temperature. No immediate or blatant relationships that I can decipher appear. Two clusters were derived.

Furthermore, for the second attempt, I modeled just the variance of the temperature and infection rate and clustered it. This time, no apparent clustering pattern can be seen. There were three clusters.

Finally, I modeled the average temperature and infection rate. This time, it seemed to have grouped based on the temperature, as seen on the map. Two clusters were created.

To make sense of the data, I think more analysis is required to juxtapose the classes against a variety external data, such as seeing whether the counties had stay at home orders. Additionally, to improve the model, increasing the number of data points could help since I only have 66 counties currently. Also, as time continues and summer arrives, I could take into account a varied amount of temperatures. Ultimately, I do not think my hypothesis has been answered, and further analysis is needed.
