# Data Wrangling - Heroes of Pymoli 

## Background

The objective of this assignment is to analyze the data for the most recent fantasy game Heroes of Pymoli.

Like many others in its genre, the game is free-to-play, but players are encouraged to purchase optional items that enhance their playing experience. As a first task, the gaming company would like a report that breaks down the game's purchasing data into meaningful insights.
<hr>

## Final report
For copy of the code, refer to [notebook](HeroesOfPymoli/HeroesOfPymoli_starter.ipynb)

### Data wrangling


```python
# Dependencies and Setup
import pandas as pd
import locale

# File to Load 
file_to_load = "Resources/purchase_data.csv"

# Read Purchasing File and store into Pandas data frame
purchase_data = pd.read_csv(file_to_load)
```
<br>

#### Player Count
```python
# Use the length of list of screen names "SN", for total players.
total_player = len(purchase_data["SN"].unique())

# Create a data frame with total players named player count
total_player_df = pd.DataFrame({"Total Players" : [total_player] })

# Display the dataframe
total_player_df
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td></td>
      <td>Total Players</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>576</td>
    </tr>
  </tbody>
</table>
<br>

#### Purchasing Analysis (Total)
```python
# Calculations for unique items, average price, purchase count, and revenue
unique_items = len(purchase_data["Item ID"].unique())

average_price = round(purchase_data["Price"].mean(),2)

number_of_purchase = purchase_data["Purchase ID"].count()

total_revenue = purchase_data["Price"].sum()

# Create data frame with obtained values
purchase_analysis = pd.DataFrame({"Number of Unique Items": [unique_items],
                                  "Average Price": average_price,
                                  "Number of Purchases": [number_of_purchase],
                                  "Total Revenue": total_revenue})

# Format with currency style
purchase_analysis[["Total Revenue" , "Average Price"]] = \
    purchase_analysis[["Total Revenue" , "Average Price"]].applymap("${:,.2f}".format)

purchase_analysis
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td></td>
      <td>Number of Unique Items</td>
      <td>Average Price</td>
      <td>Number of Purchases</td>
      <td>Total Revenue</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>179</td>
      <td>$3.05</td>
      <td>780</td>
      <td>$2,379.77</td>
    </tr>
  </tbody>
</table>
<br>

### Gender Demographics
```python
# Count total number of player by types
male_players = len(purchase_data.loc[purchase_data["Gender"] == "Male","SN"].unique())
female_players = len(purchase_data.loc[purchase_data["Gender"] == "Female" ,"SN"].unique())
other_players = len(purchase_data.loc[purchase_data["Gender"] == "Other / Non-Disclosed","SN"].unique())

# Create data frame with obtained values
gender_demographics = pd.DataFrame({"Total Count": [male_players,female_players,other_players]},
                                  index = ["Male", "Female", 'Other / Non-Disclosed'])

# Calculate percentage of player by types and format with percentage style
gender_demographics["Percentage of Players"] = \
    round(gender_demographics.loc[:,"Total Count"] / total_player * 100,2)
gender_demographics["Percentage of Players"] = \
    gender_demographics["Percentage of Players"].map("{:.2f}%".format)

# Display gender demographic table
gender_demographics
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td></td>
      <td>Total Count</td>
      <td>Percentage of Players</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Male</td>
      <td>484</td>
      <td>84.03%</td>
    </tr>
    <tr>
      <td>Female</td>
      <td>81</td>
      <td>14.06%</td>
    </tr>
    <tr>
      <td>Other / Non-Disclosed</td>
      <td>11</td>
      <td>1.91%</td>
    </tr>
  </tbody>
</table>
<br>

### Purchasing Analysis (Gender)
```python
# Perform basic calculations
gender_purchase = purchase_data.groupby(["Gender"])

purchase_count = gender_purchase["Purchase ID"].count()
ave_purchase_price = gender_purchase["Price"].mean()
total_purchase = gender_purchase["Price"].sum()

# Create a dataframe
gender_purchase_analysis = pd.DataFrame ({"Purchase Count": purchase_count,
                                          "Average Purchase Price": ave_purchase_price,
                                          "Total Purchase Value": total_purchase})

# Calculate the average total purchase per player
gender_purchase_analysis["Avg Total Purchase per Person"] = gender_purchase_analysis["Total Purchase Value"]\
                                    / gender_demographics["Total Count"]

# Apply format
gender_purchase_analysis[["Average Purchase Price" , "Total Purchase Value" , "Avg Total Purchase per Person"]] = \
    gender_purchase_analysis[["Average Purchase Price" , "Total Purchase Value" , "Avg Total Purchase per Person"]]\
    .applymap("${:,.2f}".format)

# Display the gender purchase table
gender_purchase_analysis
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td>Gender</td>
      <td>Purchase Count</td>
      <td>Average Purchase Price</td>
      <td>Total Purchase Value</td>
      <td>Avg Total Purchase per Person</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Female</td>
      <td>113</td>
      <td>$3.20</td>
      <td>$361.94</td>
      <td>$4.47</td>
    </tr>
    <tr>
      <td>Male</td>
      <td>652</td>
      <td>$3.02</td>
      <td>$1,967.64</td>
      <td>$4.07</td>
    </tr>
    <tr>
      <td>Other / Non-Disclosed</td>
      <td>15</td>
      <td>$3.35</td>
      <td>$50.19</td>
      <td>$4.56</td>
    </tr>
  </tbody>
</table>
<br>

### Age Demographics
```python
# Establish bins for ages
labels = ["<10" , "10-14" , "15-19" , "20-24" , "25-29" , "30-34" , "35-39" , "40+"]
bins = [0 , 9 , 14 , 19 , 24 , 29 , 34 , 39 , 200 ]

# Segment and sort age values into bins established above
purchase_data["Age Ranges"] = pd.cut(purchase_data["Age"], bins, labels = labels)

# Create new data frame with the added "Age Range" and group it
player_df = purchase_data.groupby("SN").first()
total_count_age = player_df.groupby("Age Ranges")["Age"].count()

# Count total players by age category
age_df = pd.DataFrame({"Total Count": total_count_age})

# Calculate percentages by age category 
age_df["Percentage of Players"] = age_df["Total Count"] / total_player * 100

# Format percentage with percentage and two decimal places 
age_df["Percentage of Players"] = age_df["Percentage of Players"].map("{:.2f}%".format)

# Display age demographic table
age_df
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td>Age Ranges</td>
      <td>Total Count</td>
      <td>Percentage of Players</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>&lt;10</td>
      <td>17</td>
      <td>2.95%</td>
    </tr>
    <tr>
      <td>10-14</td>
      <td>22</td>
      <td>3.82%</td>
    </tr>
    <tr>
      <td>15-19</td>
      <td>107</td>
      <td>18.58%</td>
    </tr>
    <tr>
      <td>20-24</td>
      <td>258</td>
      <td>44.79%</td>
    </tr>
    <tr>
      <td>25-29</td>
      <td>77</td>
      <td>13.37%</td>
    </tr>
    <tr>
      <td>30-34</td>
      <td>52</td>
      <td>9.03%</td>
    </tr>
    <tr>
      <td>35-39</td>
      <td>31</td>
      <td>5.38%</td>
    </tr>
    <tr>
      <td>40+</td>
      <td>12</td>
      <td>2.08%</td>
    </tr>
  </tbody>
</table>
<br>

### Purchasing Analysis (age)
```python
# Group data by age group
pur_age = purchase_data.groupby("Age Ranges")

# Count purchases by age group
purchase_count_age = pur_age["Price"].count()

# Obtain average purchase price by age group 
ave_pur_price = pur_age["Price"].mean()

# Calculate total purchase value by age group 
total_purchase = pur_age["Price"].sum()

# Create data frame with obtained values
pur_age_df = pd.DataFrame({"Purchase Count" : purchase_count_age,
                          "Average Purchase Price" : ave_pur_price,
                          "Total Purchase Value" : total_purchase })

# Calculate the average purchase per person in the age group 
pur_age_df["Avg Total Purchase per Person"] = pur_age_df["Total Purchase Value"] /  age_df["Total Count"]

# Format with currency style
pur_age_df[["Average Purchase Price","Total Purchase Value","Avg Total Purchase per Person"]] = \
    pur_age_df[["Average Purchase Price","Total Purchase Value","Avg Total Purchase per Person"]]\
    .applymap("${:,.2f}".format) 

# Display purchase by age table
pur_age_df
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td>Age Ranges</td>
      <td>Purchase Count</td>
      <td>Average Purchase Price</td>
      <td>Total Purchase Value</td>
      <td>Avg Total Purchase per Person</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>&lt;10</td>
      <td>23</td>
      <td>$3.35</td>
      <td>$77.13</td>
      <td>$4.54</td>
    </tr>
    <tr>
      <td>10-14</td>
      <td>28</td>
      <td>$2.96</td>
      <td>$82.78</td>
      <td>$3.76</td>
    </tr>
    <tr>
      <td>15-19</td>
      <td>136</td>
      <td>$3.04</td>
      <td>$412.89</td>
      <td>$3.86</td>
    </tr>
    <tr>
      <td>20-24</td>
      <td>365</td>
      <td>$3.05</td>
      <td>$1,114.06</td>
      <td>$4.32</td>
    </tr>
    <tr>
      <td>25-29</td>
      <td>101</td>
      <td>$2.90</td>
      <td>$293.00</td>
      <td>$3.81</td>
    </tr>
    <tr>
      <td>30-34</td>
      <td>73</td>
      <td>$2.93</td>
      <td>$214.00</td>
      <td>$4.12</td>
    </tr>
    <tr>
      <td>35-39</td>
      <td>41</td>
      <td>$3.60</td>
      <td>$147.67</td>
      <td>$4.76</td>
    </tr>
    <tr>
      <td>40+</td>
      <td>13</td>
      <td>$2.94</td>
      <td>$38.24</td>
      <td>$3.19</td>
    </tr>
  </tbody>
</table>
<br>

Identify the the top 5 spenders in the game by total purchase value, then list (in a table):
```python
# Group purchase data by screen names
spenders_gb = purchase_data.groupby(["SN"])

# Count the total purchases by name
purchase_count_spender = spenders_gb["Price"].count()

# Calculate the average purchase by name 
ave_pur_price_spender = spenders_gb["Price"].mean()

# Calculate purchase total 
total_purchase_spender = spenders_gb["Price"].sum()

# Create data frame with obtained values
spenders_df = pd.DataFrame({"Purchase Count" : purchase_count_spender,
                          "Average Purchase Price" : ave_pur_price_spender,
                          "Total Purchase Value" : total_purchase_spender })

# Sort in descending order to obtain top 5 spender names 
spenders_df = spenders_df.sort_values("Total Purchase Value", ascending = False)

# Format with currency style
spenders_df[["Average Purchase Price","Total Purchase Value"]] = \
        spenders_df[["Average Purchase Price","Total Purchase Value"]].applymap("${:.2f}".format)

# Display top spenders
spenders_df.head()
```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td>SN</td>
      <td>Purchase Count</td>
      <td>Average Purchase Price</td>
      <td>Total Purchase Value</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Lisosia93</td>
      <td>5</td>
      <td>$3.79</td>
      <td>$18.96</td>
    </tr>
    <tr>
      <td>Idastidru52</td>
      <td>4</td>
      <td>$3.86</td>
      <td>$15.45</td>
    </tr>
    <tr>
      <td>Chamjask73</td>
      <td>3</td>
      <td>$4.61</td>
      <td>$13.83</td>
    </tr>
    <tr>
      <td>Iral74</td>
      <td>4</td>
      <td>$3.40</td>
      <td>$13.62</td>
    </tr>
    <tr>
      <td>Iskadarya95</td>
      <td>3</td>
      <td>$4.37</td>
      <td>$13.10</td>
    </tr>
  </tbody>
</table>
<br>

Identify the 5 most popular items by purchase count, then list (in a table):
```python
# Group the item data by item id and item name 
item_gb = purchase_data.groupby(["Item ID","Item Name"])

# Count the number of times an item has been purchased 
purchase_count_item = item_gb["Price"].count()

# Calcualte the purchase value per item 
ave_pur_price_item = item_gb["Price"].mean()

# Calcualte the purchase value per item 
total_purchase_item = item_gb["Price"].sum()

# Create data frame with obtained values
item_df = pd.DataFrame({"Purchase count" : purchase_count_item,
                        "Item Price" : ave_pur_price_item,
                        "Total Purchase Value" : total_purchase_item })

# Sort in descending order to obtain top spender names and provide top 5 item names
item_df_c = item_df.sort_values("Purchase count",  ascending = False)

# Format with currency style
item_df_c[["Item Price","Total Purchase Value"]] = item_df_c[["Item Price","Total Purchase Value"]].applymap("${:.2f}".format)

# Display the most 5 popular items
item_df_c.head()

```
<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td>Item ID</td>
      <td>Item Name</td>
      <td>Purchase count</td>
      <td>Item Price</td>
      <td>Total Purchase Value</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>92</td>
      <td>Final Critic</td>
      <td>13</td>
      <td>$4.61</td>
      <td>$59.99</td>
    </tr>
    <tr>
      <td>178</td>
      <td>Oathbreaker, Last Hope of the Breaking Storm</td>
      <td>12</td>
      <td>$4.23</td>
      <td>$50.76</td>
    </tr>
    <tr>
      <td>145</td>
      <td>Fiery Glass Crusader</td>
      <td>9</td>
      <td>$4.58</td>
      <td>$41.22</td>
    </tr>
    <tr>
      <td>132</td>
      <td>Persuasion</td>
      <td>9</td>
      <td>$3.22</td>
      <td>$28.99</td>
    </tr>
    <tr>
      <td>108</td>
      <td>Extraction, Quickblade Of Trembling Hands</td>
      <td>9</td>
      <td>$3.53</td>
      <td>$31.77</td>
    </tr>
  </tbody>
</table>
<br>

Identify the 5 most profitable items by total purchase value, then list (in a table):

```python
# Take the most_popular items data frame and change the sorting to find highest total purchase value
item_df = item_df.sort_values("Total Purchase Value",  ascending = False)

# Format with currency style
item_df[["Item Price","Total Purchase Value"]] = item_df[["Item Price","Total Purchase Value"]].applymap("${:.2f}".format)

# Display the 5 most profitable items
item_df.head()
```

<table class="dataframe" border="1">
  <thead>
    <tr style="text-align: right;">
      <td>Purchase count</td>
      <td>Item Price</td>
      <td>Total Purchase Value</td>
      <td>Item ID</td>
      <td>Item Name</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>92</td>
      <td>Final Critic</td>
      <td>13</td>
      <td>$4.61</td>
      <td>$59.99</td>
    </tr>
    <tr>
      <td>178</td>
      <td>Oathbreaker, Last Hope of the Breaking Storm</td>
      <td>12</td>
      <td>$4.23</td>
      <td>$50.76</td>
    </tr>
    <tr>
      <td>82</td>
      <td>Nirvana</td>
      <td>9</td>
      <td>$4.90</td>
      <td>$44.10</td>
    </tr>
    <tr>
      <td>145</td>
      <td>Fiery Glass Crusader</td>
      <td>9</td>
      <td>$4.58</td>
      <td>$41.22</td>
    </tr>
    <tr>
      <td>103</td>
      <td>Singed Scalpel</td>
      <td>8</td>
      <td>$4.35</td>
      <td>$34.80</td>
    </tr>
  </tbody>
</table>

### Conclusions

1. Majority of players purchased game items are male players (84%). This gender group brought highest revenue to the company $1,967, although average purchase per person ($3.02) is lowest compared to the other groups.

2. Almost half of the players are within 20 to 24 years old. This age group also purchased more items and brought highest revenue to the gaming company.

3. Some items are priced lower than less popular items. For example, Final Critic is the most popular item with 13 sale transactions, and it was priced lower than the less popular item - Narvana. The company should price the items based on its popularity. Higher demand items should be more expensive to bring the company more profit.

