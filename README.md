# CRM-Analysis
 A comprehensive CRM system for managing customer interactions, sales processes, and analytics. Empowering businesses with scalable solutions for efficient customer relationship management and growth.

# Step 1: Import Libraries

#importing libraies

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Step 2: Load the Data

#saving file path:
file_path = "/Users/dhineshkumar/Downloads/data.csv"

#since we cant encode the data using utf-8 it shows UnicodeDecodeError using 'ISO-8859-1':
data = pd.read_csv(file_path, encoding='ISO-8859-1')

# Step 3: Explore the Data

# Displaying first 3 rows:
data.head(3)

# displaying last 3 rows
data.tail(3)

# No of rows and columns:
data.shape

#To check missing values
data.info()

# Step 4: Data Cleaning

#Identifying missing values

print(data.isnull().sum())

#Before imputation check the number of unique columns:
data['Description'].nunique()

#Before imputation check the number of unique columns:
data['StockCode'].nunique()

# Taking mode of grouping stockcode and description and filling in the missing values if there is no mode use first value
data['Description'].fillna(data.groupby('StockCode')['Description'].transform(lambda x: x.mode().iloc[0] if not x.mode().empty else x.iloc[0]), inplace=True)

data['Description'].isnull().sum()

missing_description_groups = data[data['Description'].isnull()]['StockCode'].unique()
print(missing_description_groups)

data.dropna(subset=['Description'], inplace=True)

print(data.isnull().sum())

data['CustomerID'].nunique()

customer_per_country = data.groupby('Country')['CustomerID'].nunique()
print(customer_per_country)

import matplotlib.pyplot as plt
import seaborn as sns

# Visualize distribution of 'CustomerID' values per country
plt.figure(figsize=(15, 8))
sns.barplot(x=customer_per_country.index, y=customer_per_country.values, palette='viridis')
plt.xticks(rotation=45, ha='right')
plt.title('Distribution of CustomerID Values per Country')
plt.xlabel('Country')
plt.ylabel('Number of Unique CustomerID')
plt.show()

# Check if there are missing 'CustomerID' values in specific countries
missing_customer_id_countries = data[data['CustomerID'].isnull()]['Country'].unique()
print("Countries with Missing CustomerID values:", missing_customer_id_countries)


sns.scatterplot(x='CustomerID', y='Country', data=data)

# Group by 'Country' and calculate the mode 'CustomerID'
def mode_of_group(series):
    mode_values = series.mode()
    return mode_values.iloc[0] if not mode_values.empty else None

country_mode_customer_id = data.groupby('Country')['CustomerID'].agg(mode_of_group)

# Function to impute missing 'CustomerID' based on mode for each country
def impute_customer_id(row):
    if pd.isnull(row['CustomerID']):
        return country_mode_customer_id.get(row['Country'], None)
    else:
        return row['CustomerID']

# Apply the imputation function to fill missing 'CustomerID' values
data['CustomerID'] = data.apply(impute_customer_id, axis=1)


print(data.isnull().sum())

data.dropna(subset=['CustomerID'], inplace=True)

print(data.isnull().sum())

# Step 5: Data Exploration

# to find the summary statistics
data.describe()

import matplotlib.pyplot as plt

# Checking for the distribution of the 'Quantity' column
fig, ax1 = plt.subplots(figsize=(10, 6))

ax1.set_title('Distribution of Quantity', color='brown')
ax1.set_xlabel('Quantity', color='brown')
ax1.set_ylabel('Counts (Quantity)', color='brown')
ax1.grid(axis='y', alpha=0.75)
ax1.hist(data['Quantity'], bins=range(0, 50, 2), rwidth=0.8, color='#607c8e', label='Quantity')
ax1.legend(loc='upper left')

plt.show()


import matplotlib.pyplot as plt

# Checking for the distribution of the 'UnitPrice' column
fig, ax2 = plt.subplots(figsize=(10, 6))

ax2.set_title('Distribution of UnitPrice', color='brown')
ax2.set_xlabel('UnitPrice', color='brown')
ax2.set_ylabel('Counts (UnitPrice)', color='brown')
ax2.grid(axis='y', alpha=0.75)
ax2.hist(data['UnitPrice'], bins=range(0, 50, 2), rwidth=0.8, color='orange', label='UnitPrice')
ax2.legend(loc='upper right')

plt.show()


# Step 6: Feature Engineering

#Adding additional feature to the dataframe for further analysis
data['InvoiceDate'] = pd.to_datetime(data['InvoiceDate'])
data['Month'] = data['InvoiceDate'].dt.month
data['Day'] = data['InvoiceDate'].dt.day
data['Hour'] = data['InvoiceDate'].dt.hour
data['Price'] = data['Quantity'] * data['UnitPrice']

data.columns

# Step 7: Analysis and Visualization

# Total sales per month
monthly_sales = data.groupby('Month')['Quantity'].sum()
monthly_sales.plot(kind='bar')
plt.title('Total Sales per Month')
plt.xlabel('Month')
plt.ylabel('Total Quantity Sold')
plt.show()

import pandas as pd
import matplotlib.pyplot as plt

# Assuming your DataFrame is named 'data' and has a 'Country' and 'CustomerID' column
unique_customers_per_country = data.groupby('Country')['CustomerID'].nunique().sort_values(ascending=False)

plt.figure(figsize=(15, 8))
plt.bar(unique_customers_per_country.index, unique_customers_per_country.values, color='hotpink')
plt.xticks(rotation=45, ha='right')
plt.title('Distribution of Unique CustomerID Values per Country')
plt.xlabel('Country')
plt.ylabel('Number of Unique CustomerID')
plt.show()


import matplotlib.pyplot as plt

# Create a new figure and axis
fig, ax1 = plt.subplots(figsize=(12, 6))

# Plot Quantity on the first y-axis
ax1.plot(data['InvoiceDate'], data['Quantity'], color='b', label='Quantity')
ax1.set_xlabel('Invoice Date')
ax1.set_ylabel('Quantity', color='b')
ax1.tick_params('y', colors='b')

# Create a second y-axis for Price
ax2 = ax1.twinx()
ax2.plot(data['InvoiceDate'], data['Price'], color='r', label='Price')
ax2.set_ylabel('Price', color='r')
ax2.tick_params('y', colors='r')

# Title and legend
plt.title('Time Series Plot of Quantity and Price over Time')
fig.tight_layout()
ax1.legend(loc='upper left')
ax2.legend(loc='upper right')
plt.show()
