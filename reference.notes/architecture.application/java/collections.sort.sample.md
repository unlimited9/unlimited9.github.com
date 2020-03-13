### Collections.sort with multiple fields

#### sample
```
List<DailyAdvrtsStatsData> aggregatedList = (List<DailyAdvrtsStatsData>) commonStatsUtils.aggregate(list, "adverId", "corpName", "adverId,corpName,statsDttm,homePage","adverCnt");

Comparator<DailyAdvrtsStatsData> comparator = (d1, d2) -> {
	int sortAdverId = Boolean.compare(d1.getAdverId().equals("total"), d2.getAdverId().equals("total"));
	return sortAdverId != 0 ? sortAdverId : Double.compare(Double.parseDouble(d1.getAdvrtsAmt()), Double.parseDouble(d2.getAdvrtsAmt()));
};
aggregatedList.sort(searchConditionData.getOrderBy().equals(SearchConditionData.OrderBy.ASC) ? comparator : comparator.reversed());
```
