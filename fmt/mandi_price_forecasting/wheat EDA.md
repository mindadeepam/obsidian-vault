
The aim of this project is to forecast wheat prices in Shajahanpur mandi, UP.

We take many of our learnings from maize and apply it to wheat. 
- We know top-down approaches won't work. ie complex models (DL) will not compensate for insufficient data.
- We need more data on a daily-frequency (or even weekly) to capture micro-trends.

To that end, we have a lot more data that capture the supply and demand dynamics of the wheat compared to maize in Gulabbagh. Plus, there is a lot more government involvement in wheat. The government keeps stocks of wheat in its reserves and releases it during the off-season to ensure steady supply of wheat and to control prices. 

# Features

- Daily prices -> Target variable
- Daily Arrivals in mandi -> This is the most hyperlocal indicator of supply in that region for short term future.
- OMSS data: The government releases reserved stocks to ensure supply and control prices. This includes the volume released and the prices at which it was released.
- FCI stocks -> Amazingly, we also (will)have the exact amount of stocks maintained by the govt in their warehouses, on a daily frequency. Movement in these stocks might be leading indicators for govt interference.
- Import/Exports -> Monthly export-import data. Due to various reasons, India has not been able to meet its domestic needs in the past couple years(post-covid), and hence banned exports on May 2022. Smaller quantities were still exported since, but are mostly insignificant.  
- Rainfall Data -> Extreme rainfall or absence of it, might affect wheat in various parts of its lifecycle. see [this](https://india.mongabay.com/2023/04/indias-wheat-crop-again-suffered-from-extreme-weather/)  
- Commodity Retail/Wholesale prices -> prices of 15-20 commodities in all states in India. All major producer and consumer states are potentially useful states. Commodities could be wheat and maida, which are the major byproducts of wheat. 

- Heat/Temperature Data -> Extreme heat waves have lowered production in the past(2022). But this affect is very long term. heat in crop season affects marketing season, so might not be useful for us.
- Government schemes -> The government also gives out almost 10-15 percent of wheat yield every year. This affects total reserves but might not be useful for us.
- Consumption and production -> both estimated and actual yearly data.


# Derived features

- Differences between estimated values (either released by govt/other org OR year-on-year differences) and actual values are often good macro indicators for price movement. Much of the news. This includes production, consumption, when new crop arrives in the market, etc.
- 


# Some wheat 
When it comes to consumption, the northern states, including Punjab, Haryana, and Uttar Pradesh, proudly hold the crown for the highest per capita wheat consumption. However, the urban areas of other states are also witnessing a rapid surge in the demand for wheat-based products, reflecting the changing food habits of the population.

https://www.statista.com/topics/10536/wheat-market-in-india/#topicOverview
- The wheat crop is cultivated in the Rabi (winter) season and harvested in the spring or summer season.
- India was one of the major exporters of wheat until May 2022, when the government decided to ban wheat **and its byproducts** export from the country, However, [over two billion U.S. dollars'](https://www.statista.com/statistics/652172/export-value-of-wheat-india/) worth of wheat was exported from India to different countries in 2022. Bangladesh, Sri Lanka, and the United Arab Emirates accounted for the [largest share of wheat exports](https://www.statista.com/statistics/1364728/india-wheat-export-value-by-leading-destination/) from the south Asian country that same year.
  
  ![[Screenshot 2024-05-16 at 3.14.06 PM.png]]


- https://epubs.icar.org.in/index.php/IJAgS/article/download/115880/45259/337421 

 - wheat and
   


- prices vs arrivals vs UP warehouses stock differences(+ means stock in warehouses decreased, low means stocks increased)
   ![[Screenshot 2024-05-17 at 4.32.52 PM.png]]
	decrease in arrivals and govt filling up reserves explain increasing prices form 2023-05,  


## prices seem to decrease before arrivals xome in marketing season, maybe its omss price affecting it??

- **omss me dekh ki current price se kitna neeche nikala hai times the volume released.** -> this explains decrease in 2023 nov/dec. uss samay ke price se 150-200 neche me nikala unhone. 
  
  ![[Screenshot 2024-05-17 at 5.45.45 PM.png]]

- baaki states ka warehouse releases dont show much correlation for this.




## comparison bw mandis


- shahjahapur(blue) and kasganj(purple) lead in early years
  ![[Screenshot 2024-05-20 at 1.03.10 AM.png]]

- kasganj(purple) has most continuous data 
  ![[Screenshot 2024-05-20 at 1.29.38 AM.png]]
  ![[Screenshot 2024-05-20 at 1.16.22 AM.png]]

- kanpur(red) has been very volatile since 2018 
  ![[Screenshot 2024-05-20 at 1.17.34 AM.png]]

- **varanasi(green) always seems to follow** 
  but arrivals nai hai iske??!!! **(next best option is shajahnpur, lag-wise)**
  [[Screenshot 2024-05-20 at 1.27.26 AM.png]]

![[Screenshot 2024-05-20 at 1.26.39 AM.png]]

- Etah and kasganj very similar (they are 30kms apart)

- varanasi no arrivals.
- shahjahanpir arrivals more periodic/seasonal
- delhi mandi should be leading mandi (market research), very volatile, and sortof leads especially pre 2022





## 20th may
- do analysis and report, other sources of information of delhi, like derivatives omss
  ***wheat wholesale (delhi)  vs modal_price***  (atta is similar)
  definitely leads in a few places
  ![[Screenshot 2024-05-20 at 9.32.26 PM.png]]
  
  atta (wholesale) vs modal_price 
  ![[Screenshot 2024-05-21 at 12.11.03 AM.png]]
  gur delhi vs price![[Screenshot 2024-05-21 at 12.17.10 AM.png]]
  
- apply two sided transformation to whole of data and see which smoothening technique works best
  -> why do we need smoothening?? we should do outlier removal only that preserves all the data.
  
- see impact of import export to prices 
  import is negligible compared to exports, high exports were done during war period. need earlier data too?
  ![[Screenshot 2024-05-21 at 2.09.38 AM.png]]

- how to fill modal prices in missing data, maybe train a model on min max prices
  -> shahjahanpur has MAE=8 b/w min-max-avg and modal_price, delhi has 30
  -> lets have modal as the base and fill with min and max?
  -> shahjahanpur missing data from 2010-2014 ..... can we fill this??
  -> shajpur fill from 2010-2014.
  
- use fci warehouse volume data to generate features
  delhi main warehouses vs price 
  ![[Screenshot 2024-05-21 at 2.40.47 PM.png]]
  
  delhi, bihar and MP have historically had negligible stocks compared to other states![[Screenshot 2024-05-21 at 3.55.33 PM.png]]
  
  UP major districts and those which are nearby shahjahanpur. ![[Screenshot 2024-05-21 at 4.13.21 PM.png]]

	***There doesnt seem to be inter-warehouse movement as all stock up and release at similar times.***

for up districts, varanasi has some regular movement in stocks. shajahanpur has recentlty been flat(not a lot of info there).
![[Screenshot 2024-05-21 at 4.20.21 PM.png]]

- check omss data and set a threshold after which data is useable, because data quality was bad in earlier days
   
   *all data seems useful, consecutive identical values have very small volume anyways.*
  omss_price vs org_price ![[Screenshot 2024-05-21 at 12.45.09 AM.png]]
  
  price vs omss_price_diff_scaled_by_qty_feature![[Screenshot 2024-05-21 at 12.47.54 AM.png]] 


## check agmarknet agaisnt po data

- raebareli 34 points in po-data against its mandi!

	![[Screenshot 2024-05-22 at 3.53.42 PM.png]]

- After various comaprisons, 
	- kanpur ncdex against kanpur agmarknet
	- mandi_central against agmarknet prices 
  
  we see a 7-10 day lag in agmarknet data when compared to ncdex OR trader/retailer prices.


## omss different schemes data EDA




## outlier removal

- arrivals is noisy so only extreme ones are removed. using rollling window oif 30 days and zscore of 3.5.
	![[Screenshot 2024-05-23 at 5.35.40 PM.png]]

- threshold used: rolling of [45], and zscore of [3.5], 
  again prices vary +-100 so only extreme ones have been removed
  ![[Screenshot 2024-05-23 at 5.39.14 PM.png]]


## there are also a lot of missing data.
