
> [!info] target is (shahjahanpur mandi prices) until said otherwise

#### @18/06/2024

**current feature pipeline**
projected sentiments for each buyer_loc_quality is calculated, avg it for variety then for state/location,
now we have only buyer name and their projected sentiments for next 6/7 days. 
from this calculate 
- *percent up*
- *percent stable*
- *total* 
- *weighted mean*, where weights for a buyer are: `sqrt(len(buyer_prices))/ (abs(std(buyer_prices))+1)`

**ideas**
if we group this by state, major state sentiments in one number?

**experiments**
- if we only average it for variety and not for state, ie treat diff buyers in diff locations as diff MAE increases. this might show that there is noise in more specificity.
- if we provide state specific sentiment ka means, MAE decreases by 1,
	- taking weighted means instead of means doesnt help.
	- more states? (helps in MAE but graph is worse, model becomes too conservative)
	  \['MH_sentiment' 'UP_sentiment' 'MP_sentiment' 'KA_sentiment'
		'UK_sentiment' 'BR_sentiment' 'DL_sentiment' : 0.114 0.0421 0.109 0.045 0.0557 0.0882 0.0797]
- playing with threshold for stable/up/down: not much improvement in graph but numbers improve by 0.5/1
- 



#### @24/05/2024

-> target  has 35% null values, removed outliers(20, odd)
-> using interpolate.limit(7) gets percent_null down from 35.8 to 7.5.  

-> ~~as seen in [[wheat EDA]],~~
-> trees with only prices and arrivals, predicting and using differences only. along with timestep features. 
	one common issue is going to be feature selection (especially seeing which lags and differences make sense. it think using model_feature_importances are the best way to do that, rather than correlation)


#### @27/05/2024

-> cleaning data using other mandi prices. 
`'modal_price_Agra_Agra','modal_price_Etah_Etah','modal_price_Kanpur_Choubepur','modal_price_Kanpur_Kanpur(Grain)','modal_price_Kasganj_Kasganj','modal_price_Prayagraj_Allahabad','modal_price_Saharanpur_Saharanpur','modal_price_Varanasi_Varanasi' + target lags + timestep features` gives 15 mae in val

> [!info] shuffle rows for trees, koi acha CV method chaiye (since we arent predicting for future this isnt wrong here. actual forecasting me thora sochna hoga kya kar sakte hai)

-> hyperparameter tuning will give a mae boost of 2-3 nothing much else.
	study.best_value, study.best_params 
	`(12.52312865711273, {'n_estimators': 758, 'learning_rate': 0.010174521274960066, 'num_leaves': 96, 'max_depth': 9, 'min_data_in_leaf': 10, 'feature_fraction': 0.9060600674732261, 'bagging_fraction': 0.9429170582127037, 'bagging_freq': 10, 'lambda_l1': 6.292753304304085e-05, 'lambda_l2': 0.00839101327366978})`

-> adding more mandis data, starting with delhi (not much difference)

-> added other mandi lags and target lags. added leading values too. 13-14 mae. very volatile graph still.
->adding interpolated prices of lag1 as feature brings down volatiliy and mae to 7-8. still not as clean as we want.

###### another approach
-> find nearest mandis and use their mean or some basic/weightred aggregation?.
-> within 100km radius: `['Bareilly', 'Etah', 'Firozabad', 'Hardoi', 'Kasganj', 'Mainpuri', 'Rampur', 'Shahjahanpur', 'Sitapur']`


`{'Bareilly': ['Anwala', 'Bahedi', 'Bareilly', 'Richha'], 'Etah': ['Aliganj', 'Awagarh', 'Etah'], 'Firozabad': ['Firozabad', 'Shikohabad', 'Sirsaganj', 'Tundla'], 'Hardoi': ['Hardoi', 'Madhoganj', 'Sandi', 'Sandila', 'Shahabad(New Mandi)'], 'Kasganj': ['Kasganj'], 'Mainpuri': ['Bewar', 'Ghiraur', 'Mainpuri'], 'Rampur': ['Milak', 'Rampur', 'Shahabad', 'Tanda(Rampur)', 'Vilaspur'], 'Shahjahanpur': ['Badda', 'Jalalabad', 'Puwaha', 'Shahjahanpur', 'Tilhar'], 'Sitapur': ['Hargaon (Laharpur)', 'Mehmoodabad', 'Sitapur', 'Viswan']}`


%% kanpur ka ncdex se 23-24 train karke backfill?? %%
- kanpur ka 23-24 ncdex is target. kanpur ko 7 day shift karna hoga, then train to predict ncdex test on 23-24. 
- then use it to get before??? 



- **we can use nearby mandis to fill data too** -> 
  get agmarknet data, clean it, and preprocess it. 




