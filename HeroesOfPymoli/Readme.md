# Heroes Of Pymoli Data Analysis

* Of the 1163 active players, the vast majority are male (82%). There also exists, a smaller, but notable proportion of female players (16%).

* Our peak age demographic falls between 20-24 (42%) with secondary groups falling between 15-19 (17.80%) and 25-29 (15.48%).

* Our players are putting in significant cash during the lifetime of their gameplay. Across all major age and gender demographics, the average purchase for a user is roughly $491.   

- - -

```python
# Dependencies and Setup
import pandas as pd
import numpy as np

# File to Load (Remember to Change These)
file_to_load_json = "raw_data/purchase_data.json"

# Read Purchasing File and store into Pandas data frame
purchase_data = pd.read_json(file_to_load_json, orient="records")
```

## Player Count

```python
# Calculate the Number of Unique Players
player_demographics = purchase_data.loc[:, ["Gender", "SN", "Age"]]
player_demographics = player_demographics.drop_duplicates()
num_players = player_demographics.count()[0]

# Display the total number of players
pd.DataFrame({"Total Players": [num_players]})
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1163</td>
    </tr>
  </tbody>
</table>
</div>

## Purchasing Analysis (Total)

```python
# Run basic calculations
average_item_price = purchase_data["Price"].mean()
total_purchase_value = purchase_data["Price"].sum()
purchase_count = purchase_data["Price"].count()
item_count = len(purchase_data["Item ID"].unique())

# Create a DataFrame to hold results
summary_table = pd.DataFrame({"Number of Unique Items": item_count,
                              "Total Revenue": [total_purchase_value],
                              "Number of Purchases": [purchase_count],
                              "Average Price": [average_item_price]})

# Minor Data Munging
summary_table = summary_table.round(2)
summary_table ["Average Price"] = summary_table["Average Price"].map("${:,.2f}".format)
summary_table ["Number of Purchases"] = summary_table["Number of Purchases"].map("{:,}".format)
summary_table ["Total Revenue"] = summary_table["Total Revenue"].map("${:,.2f}".format)
summary_table = summary_table.loc[:,["Number of Unique Items", "Average Price", "Number of Purchases", "Total Revenue"]]

# Display the summary_table
summary_table
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number of Unique Items</th>
      <th>Average Price</th>
      <th>Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>184</td>
      <td>$2.98</td>
      <td>192,056</td>
      <td>$571,654.35</td>
    </tr>
  </tbody>
</table>
</div>

## Gender Demographics

```python
# Calculate the Number and Percentage by Gender
gender_demographics_totals = player_demographics["Gender"].value_counts()
gender_demographics_percents = gender_demographics_totals / num_players * 100
gender_demographics = pd.DataFrame({"Total Count": gender_demographics_totals, "Percentage of Players": gender_demographics_percents})

# Minor Data Munging
gender_demographics = gender_demographics.round(2)

gender_demographics
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percentage of Players</th>
      <th>Total Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Male</th>
      <td>81.94</td>
      <td>953</td>
    </tr>
    <tr>
      <th>Female</th>
      <td>16.08</td>
      <td>187</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>1.98</td>
      <td>23</td>
    </tr>
  </tbody>
</table>
</div>

## Purchasing Analysis (Gender)

```python
# Run basic calculations
gender_purchase_total = purchase_data.groupby(["Gender"]).sum()["Price"].rename("Total Purchase Value")
gender_average = purchase_data.groupby(["Gender"]).mean()["Price"].rename("Average Purchase Price")
gender_counts = purchase_data.groupby(["Gender"]).count()["Price"].rename("Purchase Count")

# Calculate Normalized Purchasing
normalized_total = gender_purchase_total / gender_demographics["Total Count"]

# Convert to DataFrame
gender_data = pd.DataFrame({"Purchase Count": gender_counts, "Average Purchase Price": gender_average, "Total Purchase Value": gender_purchase_total, "Normalized Totals": normalized_total})

# Minor Data Munging
gender_data["Average Purchase Price"] = gender_data["Average Purchase Price"].map("${:,.2f}".format)
gender_data["Total Purchase Value"] = gender_data["Total Purchase Value"].map("${:,.2f}".format)
gender_data ["Purchase Count"] = gender_data["Purchase Count"].map("{:,}".format)
gender_data["Normalized Totals"] = gender_data["Normalized Totals"].map("${:,.2f}".format)
gender_data = gender_data.loc[:, ["Purchase Count", "Average Purchase Price", "Total Purchase Value", "Normalized Totals"]]

# Display the Gender Table
gender_data
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>30,894</td>
      <td>$2.98</td>
      <td>$92,130.15</td>
      <td>$492.67</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>157,426</td>
      <td>$2.98</td>
      <td>$468,404.32</td>
      <td>$491.51</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>3,736</td>
      <td>$2.98</td>
      <td>$11,119.88</td>
      <td>$483.47</td>
    </tr>
  </tbody>
</table>
</div>

## Age Demographics

```python
# Establish the bins 
age_bins = [0, 9.90, 14.90, 19.90, 24.90, 29.90, 34.90, 39.90, 99999]
group_names = ["<10", "10-14", "15-19", "20-24", "25-29", "30-34", "35-39", "40+"]

# Categorize the existing players using the age bins
player_demographics["Age Ranges"] = pd.cut(player_demographics["Age"], age_bins, labels=group_names)

# Calculate the Numbers and Percentages by Age Group
age_demographics_totals = player_demographics["Age Ranges"].value_counts()
age_demographics_percents = age_demographics_totals / num_players * 100
age_demographics = pd.DataFrame({"Total Count": age_demographics_totals, "Percentage of Players": age_demographics_percents})

# Minor Data Munging
age_demographics = age_demographics.round(2)

# Display Age Demographics Table
age_demographics.sort_index()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percentage of Players</th>
      <th>Total Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>3.96</td>
      <td>46</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>4.99</td>
      <td>58</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>17.80</td>
      <td>207</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>42.05</td>
      <td>489</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>15.48</td>
      <td>180</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>8.77</td>
      <td>102</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>5.42</td>
      <td>63</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>1.55</td>
      <td>18</td>
    </tr>
  </tbody>
</table>
</div>

## Purchasing Analysis (Age)

```python
# Bin the Purchasing Data
purchase_data["Age Ranges"] = pd.cut(purchase_data["Age"], age_bins, labels=group_names)

# Run basic calculations
age_purchase_total = purchase_data.groupby(["Age Ranges"]).sum()["Price"].rename("Total Purchase Value")
age_average = purchase_data.groupby(["Age Ranges"]).mean()["Price"].rename("Average Purchase Price")
age_counts = purchase_data.groupby(["Age Ranges"]).count()["Price"].rename("Purchase Count")

# Calculate Normalized Purchasing
normalized_total = age_purchase_total / age_demographics["Total Count"]

# Convert to DataFrame
age_data = pd.DataFrame({"Purchase Count": age_counts, "Average Purchase Price": age_average, "Total Purchase Value": age_purchase_total, "Normalized Totals": normalized_total})

# Minor Data Munging
age_data["Average Purchase Price"] = age_data["Average Purchase Price"].map("${:,.2f}".format)
age_data["Total Purchase Value"] = age_data["Total Purchase Value"].map("${:,.2f}".format)
age_data ["Purchase Count"] = age_data["Purchase Count"].map("{:,}".format)
age_data["Normalized Totals"] = age_data["Normalized Totals"].map("${:,.2f}".format)
age_data = age_data.loc[:, ["Purchase Count", "Average Purchase Price", "Total Purchase Value", "Normalized Totals"]]

# Display the Age Table
age_data
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10-14</th>
      <td>9,595</td>
      <td>$2.99</td>
      <td>$28,677.10</td>
      <td>$494.43</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>34,149</td>
      <td>$2.98</td>
      <td>$101,921.28</td>
      <td>$492.37</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>81,029</td>
      <td>$2.97</td>
      <td>$240,883.91</td>
      <td>$492.61</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>29,560</td>
      <td>$2.97</td>
      <td>$87,867.54</td>
      <td>$488.15</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>16,863</td>
      <td>$2.98</td>
      <td>$50,205.29</td>
      <td>$492.21</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>10,402</td>
      <td>$2.98</td>
      <td>$30,970.57</td>
      <td>$491.60</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>2,870</td>
      <td>$2.97</td>
      <td>$8,515.10</td>
      <td>$473.06</td>
    </tr>
    <tr>
      <th>&lt;10</th>
      <td>7,588</td>
      <td>$2.98</td>
      <td>$22,613.56</td>
      <td>$491.60</td>
    </tr>
  </tbody>
</table>
</div>

## Top Spenders

```python
# Basic Calculations
user_total = purchase_data.groupby(["SN"]).sum()["Price"].rename("Total Purchase Value")
user_average = purchase_data.groupby(["SN"]).mean()["Price"].rename("Average Purchase Price")
user_count = purchase_data.groupby(["SN"]).count()["Price"].rename("Purchase Count")

# Convert to DataFrame
user_data = pd.DataFrame({"Total Purchase Value": user_total, "Average Purchase Price": user_average, "Purchase Count": user_count})

# Minor Data Munging
user_data["Average Purchase Price"] = user_data["Average Purchase Price"].map("${:,.2f}".format)
user_data["Total Purchase Value"] = user_data["Total Purchase Value"].map("${:,.2f}".format)
user_data = user_data.loc[:,["Purchase Count", "Average Purchase Price", "Total Purchase Value"]]


# Display Table
user_data.sort_values("Total Purchase Value", ascending=False).head(5)
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Lisosia93</th>
      <td>325</td>
      <td>$2.96</td>
      <td>$962.08</td>
    </tr>
    <tr>
      <th>Yasur85</th>
      <td>204</td>
      <td>$2.97</td>
      <td>$604.87</td>
    </tr>
    <tr>
      <th>Lisilsa68</th>
      <td>201</td>
      <td>$3.01</td>
      <td>$604.66</td>
    </tr>
    <tr>
      <th>Lirtossa84</th>
      <td>192</td>
      <td>$3.13</td>
      <td>$601.16</td>
    </tr>
    <tr>
      <th>Hyaduesu61</th>
      <td>198</td>
      <td>$3.03</td>
      <td>$600.43</td>
    </tr>
  </tbody>
</table>
</div>

## Most Popular Items

```python
# Extract item Data
item_data = purchase_data.loc[:,["Item ID", "Item Name", "Price"]]

# Perform basic calculations
total_item_purchase = item_data.groupby(["Item ID", "Item Name"]).sum()["Price"].rename("Total Purchase Value")
average_item_purchase = item_data.groupby(["Item ID", "Item Name"]).mean()["Price"]
item_count = item_data.groupby(["Item ID", "Item Name"]).count()["Price"].rename("Purchase Count")

# Minor Data Munging
item_data_pd = pd.DataFrame({"Total Purchase Value": total_item_purchase, "Item Price": average_item_purchase, "Purchase Count": item_count})
item_data_pd["Item Price"] = item_data_pd["Item Price"].map("${:,.2f}".format)
item_data_pd ["Purchase Count"] = item_data_pd["Purchase Count"].map("{:,}".format)
item_data_pd["Total Purchase Value"] = item_data_pd["Total Purchase Value"].map("${:,.2f}".format)
item_data_pd = item_data_pd.loc[:,["Purchase Count", "Item Price", "Total Purchase Value"]]

# Display the Item Table
item_data_pd.sort_values("Purchase Count", ascending=False).head(5)
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>126</th>
      <th>Exiled Mithril Longsword</th>
      <td>997</td>
      <td>$3.55</td>
      <td>$3,539.35</td>
    </tr>
    <tr>
      <th>9</th>
      <th>Thorn, Conqueror of the Corrupted</th>
      <td>996</td>
      <td>$3.30</td>
      <td>$3,286.80</td>
    </tr>
    <tr>
      <th>182</th>
      <th>Toothpick</th>
      <td>996</td>
      <td>$4.16</td>
      <td>$4,143.36</td>
    </tr>
    <tr>
      <th>13</th>
      <th>Serenity</th>
      <td>995</td>
      <td>$4.13</td>
      <td>$4,109.35</td>
    </tr>
    <tr>
      <th>109</th>
      <th>Downfall, Scalpel Of The Emperor</th>
      <td>994</td>
      <td>$3.80</td>
      <td>$3,777.20</td>
    </tr>
  </tbody>
</table>
</div>

## Most Profitable Items

```python
# Minor Data Munging

# Display the Item Table (Sorted by Total Purchase Value)
item_data_pd.sort_values("Total Purchase Value", ascending=False).head(5)
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>153</th>
      <th>Mercenary Sabre</th>
      <td>1,105</td>
      <td>$4.89</td>
      <td>$5,403.45</td>
    </tr>
    <tr>
      <th>15</th>
      <th>Soul Infused Crystal</th>
      <td>1,091</td>
      <td>$4.94</td>
      <td>$5,389.54</td>
    </tr>
    <tr>
      <th>2</th>
      <th>Verdict</th>
      <td>1,060</td>
      <td>$4.99</td>
      <td>$5,289.40</td>
    </tr>
    <tr>
      <th>149</th>
      <th>Tranquility, Razor of Black Magic</th>
      <td>1,068</td>
      <td>$4.86</td>
      <td>$5,190.48</td>
    </tr>
    <tr>
      <th>121</th>
      <th>Massacre</th>
      <td>1,054</td>
      <td>$4.91</td>
      <td>$5,175.14</td>
    </tr>
  </tbody>
</table>
</div>
