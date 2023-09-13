# RFM_Analysis_OnlineRetail
#The Online Retail II dataset contains sales from an online retail store based in the United Kingdom.
#Data Understanding
pip install pandas
import pandas as pd

pd.set_option('display.max_columns', None)
pd.set_option('display.float_format', lambda x: '%.3f' % x)
#pd.options.mode.chained_assignment = None

df_ = pd.read_csv("PROJECT/OnlineRetail.csv", encoding="ISO-8859-1")
df = df_.copy()
df.head()

df.shape

#Deleting missing values in the data
df.isnull().sum()

#Unique product count
df["Description"].nunique()

#How many of each product were sold?
df["Description"].value_counts().head()

#Most ordered product
df.groupby("Description").agg({"Quantity": "sum"}).head()
df.groupby("Description").agg( {"Quantity": "sum"} ).sort_values("Quantity", ascending=False ).head()

#Unique invoice
df["InvoiceNo"].nunique()

df["TotalPrice"]=df["Quantity"]* df["UnitPrice"]
df.groupby("InvoiceNo").agg({"TotalPrice": "sum"}).head()

#Data Preparation
df.isnull().sum()
df.dropna(inplace=True)
df.shape

df.describe().T

df = df[~df["InvoiceNo"]. str.contains("C", na=False)]

#Calculating RFM metrics
df["InvoiceDate"].max() #Finding the deadline in the list

df.head()

import datetime as dt
today_date= dt.datetime(2011, 12, 11)
type(today_date) #Here it shows that the variable is a variable in time form.

df['InvoiceDate'] = pd.to_datetime(df['InvoiceDate'])

rfm = df.groupby('CustomerID').agg({
    'InvoiceDate': lambda InvoiceDate: (today_date - InvoiceDate.max()).days,
    'InvoiceNo': lambda InvoiceNo: InvoiceNo.nunique(),
    'TotalPrice': lambda TotalPrice: TotalPrice.sum()
})

rfm.head()

rfm.columns = ['recency', 'frequency', 'monetary']

rfm.describe().T

rfm= rfm[rfm ["monetary"] > 0]

rfm.shape

#Calculating RFM Scores
#Recency score
rfm["recency_score"] = pd.qcut(rfm['recency'], 5, labels=[5,4,3,2,1])

#Monetary score
rfm["monetary_score"] = pd.qcut(rfm['monetary'], 5, labels=[1,2,3,4,5])

#Frequency score
rfm["frequency_score"] = pd.qcut(rfm['frequency'].rank(method="first"), 5, labels=[1,2,3,4,5])

rfm["RFM_SCORE"] = (rfm['recency_score'].astype(str) + rfm['frequency_score'].astype(str))

rfm.describe().T

#Creating & Analysing RFM Segments
#Naming
seg_map = {r'[1-2][1-2]': 'hibernating',
           r'[1-2][3-4]': 'at_risk',
           r'[1-2]5': 'cant_loose',
           r'3[1-2]': 'about_to_sleep',
           r'33': 'need_attention',
           r'[3-4][4-5]': 'loyal_customers',
           r'41': 'promising',
           r'51': 'new_customers',
           r'[4-5][2-3]': 'potential_loyalists',
           r'5[4-5]': 'champions'
}

rfm['segment'] = rfm['RFM_SCORE'].replace(seg_map, regex=True)
rfm [["segment", "recency", "frequency", "monetary"]] .groupby("segment"). agg(["mean", "count"])

rfm[rfm["segment"] == "cant_loose"].head()
rfm[rfm["segment"] == "cant_loose"].index   
new_df= pd.DataFrame()
new_df["new_customer_id"]=rfm[rfm["segment"] == "new_customers"].index
new_df

new_df["new_customer_id"] = new_df["new_customer_id"]. astype(int)

new_df. to_csv("new_customers.csv")
rfm.to_csv("rfm.csv")
