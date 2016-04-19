# DataIncubator_challenge
My proposal is to analyze how location and the number of bedrooms would affect housing prices over time and forecast housing prices using the Holt-Winters algorithm (Triple exponential smoothing). I analyzed data of the top 10 major cities in the US provided by http://www.trulia.com from 2010-01-01 to 2015-12-31.

Below, I give an example for Median List Housing Prices in New York City from 2013-02-09 to 2014-02-08. I also provide some scripts that I use.

Example
----------------
 The blue curves represent the original data and the red dotted curves represent the predicted data using the Holt-Winters algorithm. 
The figures below are results for properties with different numbers of bedrooms. 

1 Bedroom Properties
![Alt text](1BR_properties.jpg?raw=true "1 BR properties")

4 Bedroom Properties
![Alt text](4BR_properties.jpg?raw=true "4 BR properties")

7 Bedroom Properties
![Alt text](7BR_properties.jpg?raw=true "7 BR properties")

10 Bedroom Properties
![Alt text](10BR_properties.jpg?raw=true "10 BR properties")

The figure below shows median housing prices of properties that have 1 bedroom - 10 bedrooms in NYC from 2013-02-09 to 2014-02-08. The red dotted curve represnets the prediction.
![Alt text](BR1_10_properties.jpg?raw=true "BR1_10 properties")

Scripts
----------------
Script to get data from API 
Run python rest_nyc.py > nyci.csv to save data as csv files. 
```python
import requests
import xml.etree.ElementTree as ET
import csv
import sys


def make_request(url, library, function, city, startDate, endDate, apikey):
    data = [
        ('library', library),
        ('function', function),
        ('city', city),
        ('state', state),
        ('startDate', startDate),
        ('endDate', endDate),
        ('apikey', apikey),
    ]

    url += '?' + '&'.join(['{0}={1}'.format(k, v) for k, v in data])

    resp = requests.get(url)

    return resp.text
    

url = 'http://api.trulia.com/webservices.php'
library = 'TruliaStats'
function = 'getCityStats'
city = 'New York'
state = 'NY'
startDate = '2010-01-01'
endDate = '2015-12-31'
apikey = 'mxvmn5razscxg636662fvhrg'

result = make_request(url, library, function, city, startDate, endDate, apikey)

root = ET.fromstring(result)

listingStats = root.find('response').find('TruliaStats').find('listingStats')

writer = csv.writer(sys.stdout)

# Write out the header.
writer.writerow(['date', 'type', 'numberOfProperties', 'medianListingPrice', 'averageListingPrice'])

for listingStat in listingStats.findall('listingStat'):
    date = listingStat.find('weekEndingDate').text
    for subcategory in listingStat.find('listingPrice').findall('subcategory'):
        type = subcategory.find('type').text
        numberOfProperties = subcategory.find('numberOfProperties').text
        medianListingPrice = subcategory.find('medianListingPrice').text
        averageListingPrice = subcategory.find('averageListingPrice').text

        writer.writerow([date, type, numberOfProperties, medianListingPrice, averageListingPrice])

```
Script to extract data
Run cat nyc.csv | python extract_median_prices.py > median_nyc.csv

```python
import csv
import sys

dict_reader = csv.DictReader(sys.stdin)

all_types = set()
all_dates = set()
median_prices = {}
for row in dict_reader:
    date = row['date']

    all_dates.add(date)

    if date not in median_prices:
        median_prices[date] = {}

    type = row['type']

    if (type != 'All Properties'
        and len(type) != len('14 Bedroom Properties')):

        # This add 0 to front of types like '3 Bedroom Properties'.
        type = '0' + type

    all_types.add(type)

    median_prices[date][type] = row['medianListingPrice']

field_names = ['date'] + sorted(list(all_types))

dates = sorted(list(all_dates))

# Write out the rows you want.
dict_writer = csv.DictWriter(sys.stdout, field_names)
dict_writer.writeheader()

for date in dates:
    groups = median_prices[date]
    groups['date'] = date

    for field in field_names:
        if field not in groups:
            groups[field] = ''

    dict_writer.writerow(groups)
```

Plot/Forecast
```matlab
filename = 'median_nyc.csv';
scrsz = get(0,'ScreenSize');

brtable= csvread(filename,1,1);

L =8; m=8; %for Holt-Winters

% n = number of weeks (2010-01-01 - 2015-12-31)
n = size(brtable, 1); 

% b = max number of bedrooms
br = 10;
% median prices table for 1-10 bedroom properties
brtable = brtable(:,1:br);

% vectoriz the table 
%brtypes = brtable';
%brtypes = reshape(brtypes, n*br, 1);

%plot
fig1 = figure('Position',[1 scrsz(4) scrsz(3)*2/3 scrsz(4)]) ;
colorbar; 
plot(brtable); axis tight
xlabel('Weeks');
ylabel('Properties with bedroom numbers 1-10');
saveas(fig1,'New York.jpg','jpg')



for bs = 1:3
    brb = brtable(:,3*bs-2 );
     holtwinters(brb,8,8);
end
```
