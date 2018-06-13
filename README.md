## Street Sense: Learning from Randomly Sampled Locations on Google Street View

What is the condition of the streets? Are the streets paved? Do the streets have proper traffic signs and road markings? Is there litter on the streets? What proportion of vehicles on the streets is two-wheeled? And what proportion is man-powered, e.g., rickshaws? These are some of many the questions we can answer with [Google Street View](https://www.google.com/streetview/). 

But Google Street View has two deficiencies, especially in the third-world. The data aren't always current. And the coverage is [patchy](https://en.wikipedia.org/wiki/Coverage_of_Google_Street_View). For instance, as we show below, just about 24.6% of Dhaka's streets are covered by Google Street View. (Some of Google's estimates of its coverage are either wrong or have become outdated as the road network continues to grow.)

But somewhat older data has its value, especially because we expect Google to map those areas again in the future. And Google's coverage of some other big third-world cities isn't too shabby. For instance, it covers 99.9% of Bangkok's streets and 87.2% of Lagos' streets.

So, how do we capitalize on Google Steet View data? We can download all the data for a city. But doing that is [expensive](https://developers.google.com/maps/documentation/streetview/usage-and-billing). And it may not even be useful. Depending on the question, a large random sample can fill in nicely for a census. What we need then is a way to randomly sample locations on the streets. 

[geo_sampling](https://github.com/soodoku/geo_sampling/) solves that exact problem. The package implements the following workflow: it downloads the Open Street Maps data for the city, chunks the roads into .5km segments, and randomly samples from the segments. We pair [geo_sampling](https://github.com/soodoku/geo_sampling/) with Google Street View to do shed light on the condition of roads in a few cities, and to learn the association between income and condition of the roads.

### Learning About the Condition of the Roads

We began the project nearly a year ago by downloading a random sample of 1,000 road segments of primary, secondary, tertiary, and trunk roads for four cities: Bangkok, Dhaka, Jakarta, Lagos. (See [here](data/geo_sample_out)). And to estimate the relationship between income and road conditions, we also downloaded a larger random sample of 5,000 road segments for one county, Wayne, Michigan. (We chose an American county because of the ready availability of data on income at the local level.)

Next, we used the Google Street View API to download images at the starting point of each of the random road segment. Sometimes the Google API came back empty. We take the proportion of failed queries as an estimate of Google Street View coverage of the primary, secondary, tertiary, and trunk roads in the respective city. In Dhaka, for instance, just about 24.6% of [queries were successful](scripts/google_street_view_Mturk-Dhaka.ipynb). Given the low coverage of Dhaka, we dropped Dhaka. 

Next, we got MTurkers to code the images for the condition of the roads. We asked MTurkers if the road shown in the image had any cracks. We also asked them if the road had any potholes. For Lagos, Bangkok, and Wayne, we also asked MTurkers if they saw any litter and if the sidewalks were paved. Lastly, we asked Mturkers if there were road markings on the road, were they clear. We paid Mturkers 5 cents for each image. We only invited Master Mturkers for the survey. And we checked a few images at random to judge quality.

In all, we have data on 978 photos for Bangkok, 872 for Jakarta, 999 for Lagos, and 4,828 for Wayne. Each photo captures a small segment of the road.

### What did we learn?

The proportion of road segments with potholes is Jakarta is an astonishing .39. The commensurate number for Bangkok, Lagos, and Wayne is between .06--.07. But what does that mean? As we mentioned above, each image captures a small segment of the street. If we assume that a photo captures .5km, the expected number of potholes on a 10 km journey in Jakarta would be 3.9. That would make for a somewhat of a rough ride.

When it comes to cracks in the road, Wayne takes the top spot---the proportion of segments in Wayne with cracks is .62 followed by early to mid-twenties for the rest of the three cities. This is not particularly surprising given Wayne's latitude. 

Lagos is the dirtiest of the 4 cities with .15 of the segments containing litter.  Lagos also takes the bottom spot for city with the fewest paved sidewalks---just .30 of the segments have a paved sidewalk.


| city | potholes | cracks | clear road markings | roads w/ markings | litter | paved sidewalk                                               |
| ---- | -------- | ------ | ------------------- | ----------------- | ------ | ------------------------------------------------------------ |
| bangkok | .06 |   .24  |  .81 |   .98 |   .06 |  .51 |
| jakarta | .39 |   .22  |  .22 |   .46 |   -  |  -  |
| lagos   | .06 |   .20  |  .23 |   .95 |   .15 |  .30 |
| wayne   | .07 |   .62  |  .60 |   .90 |   .09 |  .67 |

We also investigated if these vary by type of the road and if cross-city comparisons are compromised by that variation. To study this, we regressed the appropriate variable (potholes, cracks) on type of the road and city. Compared to tertiary roads, potholes are more common on primary roads (Diff. = .03), Secondary roads (Diff. = .02), and Trunk roads (Diff. = .04). And after adjusting for the type of road, compared to Wayne, Bangkok and Jakarta have fewer potholes (-.01 for both) and Jakarta has many many more (Diff. = .32).

Moving to cracks in the road, compared to tertiary roads, primary, secondary, and trunk roads have fewer cracks with differences being -.04, -.03, and -.06 respectively. Like with potholes, adjusting for kind of roads doesn't seem to make much of a difference for inferences from raw data for cross-city comparison. 

Next, we analyzed the relationship between condition of the roads and local income. To do that, we used the AskGeo API to get information on per capita income the census tract in which the lat/long lay. Given that we subset on primary, secondary, tertiary, and trunk roads, ignoring neighborhood and local roads, we do not expect the relationship between road quality and income to be large. And that is indeed what we find. Compared to road segments in tracts with per capita income less than 12k, proportion of road segments with potholes in tracts with per capita income between 12k and 17k  was -.01 less. Proportion of road segments in tracts with with per capita income of 17k to 23k  was -.02 less. For tracts with 23k and 29k, it was -.03 fewer segments with potholes, and for tracts with income between 29k and 83k, -.02 fewer segments had potholes.

### Scripts and Data

* CSVs of initial random sample of segments generated using geo_sampling:  
    - [Bangkok](data/geo_sample_out/bangkok-roads-s1k.csv)
    - [Dhaka](data/geo_sample_out/dhaka-roads-s1k.csv)
    - [Jakarta](data/geo_sample_out/jakarta-roads-s1k.csv)
    - [Lagos](data/geo_sample_out/lagos-roads-s1k.csv)
    - [Wayne](data/geo_sample_out/wayne2-roads-s5k.csv)

* Get Google Street View Data:  
    - Jupyter Notebooks:  
        + [Bangkok](scripts/google_street_view_Mturk-Bangkok.ipynb)
        + [Dhaka](scripts/google_street_view_Mturk-Dhaka.ipynb)
        + [Jakarta](scripts/google_street_view_Mturk-Jakarta.ipynb)
        + [Lagos](scripts/google_street_view_Mturk-Lagos.ipynb)
        + [Wayne](scripts/google_street_view_Mturk-Wayne-5k.ipynb)

    - Data:  
        + [Photos for Bangkok, Dhaka, Lagos, Jakarta, and Wayne](https://doi.org/10.7910/DVN/L3HN0K) are on the Harvard Dataverse.
        + [CSV with meta data](data/google_street_view_metadata/): There are 4 columns "url_img0", "url_img90", "url_img180" and "url_img270" in the CSV file for each row corresponding to images taken at 4 different angles.

* Code the images on Mturk
    - Screenshots of Surveys on Mturk to code images  
        + [Bangkok](data/mturk/bangkok_mturk_screenshot.png)
        + [Jakarta](data/mturk/jakarta_mturk_screenshot.png)
        + [Lagos and Wayne](data/mturk/lagos_mturk_screenshot.png)

    - Resulting Data
        + [Bangkok](data/mturk/bangkok_mturk_2018_06_02.csv)
        + [Jakarta](data/mturk/jakarta_mturk_2018_06_02.csv)
        + [Lagos](data/mturk/lagos_mturk_2018_06_08.csv)
        + [Wayne](data/mturk/wayne_mturk_2018_06_11_18.csv)

* [Geocode Wayne via AskGeo](scripts/wayne2-askgeo.ipynb)

* Analysis:  
    - [Cross-city comparison](scripts/mturk_streetview.ipynb)
    - [Road Conditions in Wayne by Income, etc.](scripts/wayne_by_income.ipynb)

### Authors

Suriyan Laohaprapanon, Kimberly Ortleb, and Gaurav Sood

