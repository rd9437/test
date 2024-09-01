# Predictive Analysis for Parking Management

## Data Cleaning

We already know that one thing we need to do is change **LastUpdated** to be a datetime type.

After doing so, there are a few additional features to add that will help with our analysis:

- **PercentOccupied** - Ratio of Occupancy to Capacity. We will want to make sure these values go from 0%-100%
- **Date**- Only the date component of LastUpdated. Used for checking the data.
- **DayOfWee**k - An integer representing the day of the week. (1 = Monday)
- **Date_Time_HalfHour** - this field will round each time to the nearest half hour. This will help when we aggregate accross all parking areas.
- **Time** - Only the time component of the Date_Time_HalfHour. Used for checking the data.


```
df['LastUpdated'] = pd.to_datetime(df['LastUpdated'], format='%d-%m-%Y %H:%M')
df['PercentOccupied'] = df['Occupancy'] / df['Capacity'] * 100
df['Date'] = df['LastUpdated'].dt.date
df['DayOfWeek'] = df['LastUpdated'].dt.dayofweek + 1
df['Date_Time_HalfHour'] = df['LastUpdated'].dt.round('30min')
df['Time'] = df['Date_Time_HalfHour'].dt.time
```

In case there are any duplicates, we will use pandas ``.drop_duplicates()`` method to automatically remove any.

```
original_length = len(df)
df.drop_duplicates(inplace=True)
duplicates_dropped = original_length - len(df)
print(f"Number of duplicates dropped: {duplicates_dropped}")
```

```
It happens that there are 207 duplicates removed.
```

We also wanted to make sure that the Occupancy ranged from a minimum value of 0 to a maximum value of Capacity. We can check along all of the observations by seeing whether PercentOccupied is between 0% and 100%.

```
print('Minimum Percent Occupied: {:.2%}'.format(df['PercentOccupied'].min()))
print('Maximum Percent Occupied: {:.2%}'.format(df['PercentOccupied'].max()))
```

```
Minimum PercentOccupied value: -1.67%
Maximum PercentOccupied value: 104.13%
```

There were some values outside the range (probably due to malfunction equipment), so we will manually limit the data to be between 0 and the Capacity value. After doing so, we do another check to make sure we are now ranging from 0% to 100% in the PercentOccupied field.

```
# Check if PercentOccupied values range from 0% to 100%
percent_min = df['PercentOccupied'].min()
percent_max = df['PercentOccupied'].max()
print(f"Minimum PercentOccupied value: {percent_min}%")
print(f"Maximum PercentOccupied value: {percent_max}%")
```

```
Minimum PercentOccupied value: 0.0%
Maximum PercentOccupied value: 100.0%
```

Let's take a look at a few graphs for each location to see what the trend in parking occupancy rates looks like.

![Figure_1](https://github.com/rd9437/test/assets/143277515/5d650540-ac5a-4e79-a2dc-ae9e6e372a64)


There seems to be a pattern to the parking occupancy, which makes sense. 

+ People may use certain parking areas more often during lunch hours if there are places to eat nearby.
  ![Screenshot 2024-03-31 195129](https://github.com/rd9437/test/assets/143277515/81a80a94-7dd7-4493-81c1-ea5bf7e4ed08)

+ Most areas will also see higher parking rates during weekdays.

## Look for Seasonality and Test for Stationarity

Graphs a plot of autocorrelation and partial correlation:

![image](https://github.com/user-attachments/assets/fa15dc2d-59f0-4bf3-9b67-4300fe77f747)

One necessity for time series is to have stationarity. This means that the mean and standard deviation in the endogenous variable are stable through time.

In order to achieve stationarity, we must remove the patterns in the data.

To start with, we know that the pattern repeats every 18 periods, so we use that as the period in the function below.

The Dickey-Fuller test is used to test for stationarity. As shown below, the stationarity condition is met when using 18 periods, but we can probably do better.

```
Results of Dickey-Fuller Test:
Test Statistic                -1.148817e+01
p-value                        4.808321e-21
#Lags Used                     1.700000e+01
Number of Observations Used    3.490000e+02
Critical Value (1%)           -3.449227e+00
Critical Value (5%)           -2.869857e+00
Critical Value (10%)          -2.571201e+00
dtype: float64

```

Given that the test statistic is much more negative than the critical values and the p-value is extremely low, you can confidently conclude that your time series is stationary after applying the seasonal differencing with a period of 18. This means that the mean and variance of the series are stable over time, which is a crucial property for effective modeling and forecasting.

