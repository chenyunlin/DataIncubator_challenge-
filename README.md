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
Getting data from API
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
