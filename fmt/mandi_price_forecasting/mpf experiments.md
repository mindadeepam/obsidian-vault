After weeks of experimentation, what we have learned can be distilled down into one succinct line.
**simple models work**. 

Best performing models have been Linear Regression and Trees.

Linear models give best performance on features like egg_prices, but the marginal improvement with adding more features shoots to 0 very quickly. They also come with a lot of assumptions of our data which are hard to maintain.

Trees have their own caveats, 

- try binning [link](https://www.kaggle.com/code/vikasmalhotra08/feature-binning-using-bayesian-blocks) 
- target transform
- non-consecutive features so that diff dates have diff features
- since a node cant have feature-interaction, we could try to create some extra features.
- ~~some notion of **steepness**, **volatility**, **how long has current micro-trend been going on**~~



1. first make hand feature, each feature should imporve score.
2. check feature importance using permutation importance.
3. use some library to get inter-feature features, and do 2.
4. ~~features like time since rabi, etc etc ie hand engineer the shit pout of it. each one should improve score......~~



already tried
- ~~extraregressor, adaboost regressor, etc.~~
- ~~added **steepness**, **volatility** features, ~~
- some forms of smoothening/binning


### 10/05/24

- 5 day modelling:
	- baseline - price_0
	  ![[Screenshot 2024-05-10 at 5.37.24 PM.png]]
	- past average method -> **price_0 + ewm_of_velocity_or_diffs**
	  overextends local trends. mae worse than lagged. might be useful as feature?
		![[Screenshot 2024-05-10 at 5.36.08 PM.png]]
	- classifying bins 
	
- ***modelling always worse than just n_day lag*** 


## 13/05/24

ideas to try out:
- team is making equations manually, and that seems to be working best. Data therefore is proving difficult to **model** in a way. 
- tried binning targets for trees but that didnt work, but am not convinced yet.
- 