
```python

def drop_na_duplicates(df, subset, na_columns):

	def keep_least_na(group):
	
		non_na_counts = group[na_columns].notna().sum(axis=1)
		max_non_na = non_na_counts.max()
		if max_non_na > 0:
			return group.loc[non_na_counts.idxmax()]
		return group.iloc[0] # If all rows have NA, keep the first row
			
		  
	
	return (
		df.groupby(subset, group_keys=False).apply(keep_least_na).reset_index(drop=True)
	)
```


