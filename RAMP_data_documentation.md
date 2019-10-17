## RAMP Data

### Data Collection

RAMP data are downloaded for participating IR from Google Search Console (GSC) via the Search Console API. The data consist of aggregated information about IR pages which appeared in search result pages (SERP) within Google properties (including web search and Google Scholar). 

Data are downloaded in two sets per participating IR. The first set includes page level statistics about URLs pointing to IR pages and content files. The following fields are downloaded for each URL, with one row per URL:

* __url__: This is returned as a 'page' by the GSC API, and is the URL of the page which was included in an SERP for a Google property.
* __impressions__: The number of times the URL appears within the SERP.
* __clicks__: The number of clicks on a URL which took users to a page outside of the SERP.
* __clickThrough__: Calculated as the number of clicks divided by the number of impressions. 
* __position__: The position of the URL within the SERP.
* __date__: The date of the search.

The second set includes similar information, but instead of being aggregated at the page level, the data are grouped based on the country from which the user submitted the corresponding search, and the type of device used. The following fields are downloaded for combination of country and device, with one row per country/device combination:

* __country__: The country from which the corresponding search originated.
* __device__: The device used for the search.
* __impressions__: The number of times the URL appears within the SERP.
* __clicks__: The number of clicks on a URL which took users to a page outside of the SERP.
* __clickThrough__: Calculated as the number of clicks divided by the number of impressions. 
* __position__: The position of the URL within the SERP.
* __date__: The date of the search.

Note that no personally identifiable information is downloaded by RAMP. Google does not make such information available.

More information about click-through rates, impressions, and position is available from Google's Search Console API documentation: [https://developers.google.com/webmaster-tools/search-console-api-original/v3/searchanalytics/query](https://developers.google.com/webmaster-tools/search-console-api-original/v3/searchanalytics/query) and [https://support.google.com/webmasters/answer/7042828?hl=en](https://support.google.com/webmasters/answer/7042828?hl=en)

## Data Processing

Upon download from GSC, the page level data described above are processed to identify URLs that point to citable content. _Citable content_ is defined within RAMP as any URL which points to any type of non-HTML content file (PDF, CSV, etc.). As part of the daily download of page level statistics from Google Search Console (GSC), URLs are analyzed to determine whether they point to HTML pages or actual content files. URLs that point to content files are flagged as "citable content." In addition to the fields downloaded from GSC described above, following this brief analysis one more field, _citableContent_, is added to the page level data which records whether each page/URL in the GSC data points to citable content. Possible values for the _citableContent_ field are "Yes" and "No."

The data aggregated by the search country of origin and device type do not include URLs. No additional processing is done on these data. Harvested data are passed directly into Elasticsearch.

Processed data are then saved in a series of Elasticsearch indices. Currently, RAMP stores data in two indices per participating IR. One index includes the page level data, the second index includes the country of origin and device type data.

### About Citable Content Downloads

Data visualizations and aggregations in the RAMP interface accessible from [http://ramp.montana.edu/](http://ramp.montana.edu/) present information about citable content downloads, or CCD. As a measure of use of institutional repository content, CCD represent click activity on IR content that _may_ correspond to research use. 

CCD information is summary data calculated on the fly within the RAMP web application. As noted above, data provided by GSC include whether and how many times a URL was clicked by users. Within RAMP, a "click" is counted as a potential download, so a CCD is calculated as the sum of clicks on pages/URLs that are determined to point to citable content (as defined above).

For any specified date range, the steps to calculate CCD are:

1. Filter data to only include rows where "citableContent" is set to "Yes."
2. Sum the value of the "clicks" field on these rows.

### Output to CSV

Published RAMP data are exported from the production Elasticsearch instance and converted to CSV format. The CSV data consist of one "row" for each page or URL from a specific IR which appeared in search result pages (SERP) within Google properties as described above. Also as noted above, daily data are downloaded for each IR in two sets which cannot be combined. One dataset includes the URL. The second dataset is aggrgated by the country from which a search was conducted and the device used.

As a result, two CSV datasets are provided for each month of published data:

__page-clicks__:

The data in these CSV files correspond to the page-level data, and include the following fields:

* __url__: This is returned as a 'page' by the GSC API, and is the URL of the page which was included in an SERP for a Google property.
* __impressions__: The number of times the URL appears within the SERP.
* __clicks__: The number of clicks on a URL which took users to a page outside of the SERP.
* __clickThrough__: Calculated as the number of clicks divided by the number of impressions. 
* __position__: The position of the URL within the SERP.
* __date__: The date of the search.
* __citableContent__: Whether or not the URL points to a content file (ending with pdf, csv, etc.) rather than HTML wrapper pages. Possible values are Yes or No.
* __index__: The Elasticsearch index corresponding to page click data for a single IR. Since the monthly CSV files include data for all participating IR (or all IR included in a subset), index names are needed to extract data for individual IR, or groups of IR.

Filenames for files containing these data end with “page-clicks”. For example, the file named *2019-01\_RAMP\_subset\_page-clicks.csv* contains page level click data for a subset of 35 RAMP participating IR for the month of January, 2019.

__country-device-information__:

The data in these CSV files correspond to the data aggregated by country from which a search was conducted and the device used. These include the following fields:

* __country__: The country from which the corresponding search originated.
* __device__: The device used for the search.
* __impressions__: The number of times the URL appears within the SERP.
* __clicks__: The number of clicks on a URL which took users to a page outside of the SERP.
* __clickThrough__: Calculated as the number of clicks divided by the number of impressions. 
* __position__: The position of the URL within the SERP.
* __date__: The date of the search.
* __index__: The Elasticsearch index corresponding to country and device access information data for a single IR. Since the monthly CSV files include data for all participating IR (or all IR included in a subset), index names are needed to extract data for individual IR, or groups of IR.

Filenames for files containing these data end with “country-device-info”. For example, the file named *2019-01\_RAMP\_subset\_country-device-info.csv* contains country and device data for all participating IR for the month of January, 2019.
