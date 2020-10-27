# Exchange Now

Tracking exchange data from https://openexchangerates.org using Github action.

## Workflows: Scrape latest exchange rates

![Scrape latest exchange rate data](https://github.com/KwangYeol/exchange-now/workflows/Scrape%20latest%20exchange%20rate%20data/badge.svg)

* Scrape daily: `4 0 * * *`
* Create csv file monthly: `rates/{{ YYYYMMDD }}.csv`

## Workflow: Report exchange rate <WIP>


## Historical data (for backfill)

This code block runs only for Mac OS X. 

```bash
# Set the target month
APP_ID={{ YOUR APPLICATION ID }}
MONTH='2020-01'

# !<-- do not edit code manually 
BEGIN_DT="${MONTH}-01"
NEXT_MONTH=$(date -j -f "%Y-%m-%d" -v+32d "${BEGIN_DT}" +%Y-%m)
NUM_DAYS=$(date -j -f "%Y-%m-%d" -v-1d "${NEXT_MONTH}-01" +%d)
CSV_MONTHLY="rates/$(date -j -f "%Y-%m" $MONTH +'%Y%m').csv"

for i in `seq 1 $NUM_DAYS`;
do
  DT=$(date -j -f "%Y-%m-%d" -v+${$((i-1))}d $BEGIN_DT +%Y-%m-%d)
  echo "get $DT"
  curl -s "https://openexchangerates.org/api/historical/${DT}.json?app_id=$APP_ID" | jq -r '. as $R | $R.timestamp | strftime("%Y-%m-%d") as $ts | ($R.rates | to_entries[] | [$ts, .key, .value]) | @csv' > curr.csv
  sleep 1
  cat $CSV_MONTHLY curr.csv | awk -F, '!a[$1$2]++' | tee mon.csv
  mv mon.csv $CSV_MONTHLY
  sleep 1
  rm curr.csv
done;
# -->!
```

To validate csv file

```
R -e 'df=read.csv("rates/202001.csv", header=F)'
```