## Street Sense: Learning from Google Street View

Since 2007, Google has been working on regularly taking panoramic images of all the streets in the world. In the West, Google's efforts have been a success: Google's specially designed vehicles have traversed an overwhelming majority of the streets. (See [Wikipedia page devoted to Google Street View coveage](https://en.wikipedia.org/wiki/Coverage_of_Google_Street_View].) In the third-world, however, the coverage is patchy. For instance, as we show below, just about 24.6% of Dhaka's streets are covered by Google Street View.  But Google's coverage of some other big third-world cities isn't too shabby. For instance, it covers 99.9% of Bangkok's streets and 87.2% of Lagos' streets. In all, the coverage is good enough, especially in the West, that people can build a scalable measurement infrastructure on top of it.

But patchy coverage is not the only problem with Google Street View data. The other is that the data are not always current. A large chunk of the data is at least a few years old. But somewhat older data has its value, especially because we expect Google to map those areas again in the future. The data aren't perfect but they are rich and valuable. 

But how do we efficiently capitalize on Google Steet View data? We can download all the data for a city. But doing that is [expensive](https://developers.google.com/maps/documentation/streetview/usage-and-billing). And it may not even be useful. Depending on the question, a large random sample can fill in nicely for a census. For learning the condition of the roads, that is precisely the case.

To efficiently learn about the condition of the streets, sidewalks, and such, from Google Street View data, we devise a new workflow. We start by downloading data on the kinds of roads we are interested from [Open Street Map](https://www.openstreetmap.org) (OSM). We then chunk the roads into half a kilometer segments, and then randomly sample from the segments. (The open source Python package [geo-sampling](https://github.com/geosensing/geo_sampling) implements this workflow.) We then take the starting latitude and longitude of the sampled segments and query the [Google Street View API](https://developers.google.com/maps/documentation/streetview/intro).

### Learning About the Condition of the Roads

To learn the condition of roads in Bangkok, Dhaka, Jakarta, Lagos, and Wayne, MI, in the latter half of 2017, we downloaded data on all the streets from OSM. But we feared that in many of these cities, Google Street View's coverage of neighborhood roads would be patchy. So we decided to focus on [primary](https://wiki.openstreetmap.org/wiki/Tag:highway=primary), [secondary](https://wiki.openstreetmap.org/wiki/Tag:highway=secondary), [tertiary](https://wiki.openstreetmap.org/wiki/Tag:highway=tertiary), [trunk](https://wiki.openstreetmap.org/wiki/Tag:highway=trunk. We used the [geo-sampling package](https://github.com/geosensing/geo_sampling) to take a random sample of primary, secondary, tertiary, and trunk road segments for each location (see [figs/](figs/)). For Bangkok, Dhaka, Jakarta, and Lagos,  we drew a sample of 1,000 segments each. For Wayne, MI, we drew a sample of 5,000 segments. We drew a larger sample for Wayne, MI because we wanted to estimate the relationship between local income and road conditions there. (We chose an American county to estimate the relationship between local income and road conditions because data on local income is readily available for the US.)

Next, we used the Google Street View API to download images at the starting point of each of the random road segment. Sometimes the Google API came back empty. We take the proportion of failed queries as an estimate of Google Street View coverage of the primary, secondary, tertiary, and trunk roads in the respective city. In Dhaka, for instance, just about 24.6\% of [queries were successful](scripts/google_street_view_Mturk-Dhaka.ipynb). Given the low coverage of Dhaka, we dropped Dhaka. In all, we have images of 978 locations for Bangkok, 872 for Jakarta, 999 for Lagos, and 4,828 for Wayne. Each photo captures a small segment of the road. (All the photos are available on [Harvard Dataverse](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/L3HN0K).)

Next, we recruited workers on Amazon's Mechanical Turk (MTurk) to code the images for the condition of the roads. To ensure quality, we only recruited `master' workers. We asked them if the segment of the road in the image had any 1) cracks, and 2) potholes. We also asked them, "if there are any road markings on the road, are they clear?" Lastly, we asked them, if there any litter and if the sidewalks were paved. The final survey for Bangkok, Jakarta, and Wayne, MI was the same (see [data/mturk](data/mturk/)). (We initially got Jakarta's images coded using alternate instrumentation. But we were concerned that this would lead to incommensurability. So we did another round of data collection with the same instrument.) Lagos' survey differed in very minor ways from Bangkok, Jakarta, and Wayne's. We paid MTurkers 5 cents for answering the short survey for each image. To ensure quality, we also checked a few images at random to see if the coding was reasonable. We found one instance where one worker's judgments seemed really off and decided to reject those HITs.

### What did we learn?

Lest the readers miss an obvious point, before we present the results, we would like to draw their attention to it. Differences in the quality of roads across cities do not by default capture the extent of the road network. The extent of road network is easy to compute and regularly cited. Our contribution is  measurement of quality of roads, sidewalks, and litter on the streets efficiently.

The proportion of road segments with potholes is Jakarta is an astonishing .23. The commensurate number for Bangkok, Lagos, and Wayne is between .06--.07. But what does that mean? As we mentioned above, each image captures a small segment of the street. If we assume that a photo captures .5km, the expected number of potholes on a 10 km journey in Jakarta would be 2.3. That would make for a somewhat of a rough ride.

When it comes to cracks in the road, Wayne takes the top spot---the proportion of segments in Wayne with cracks is .62 followed by .44 for Jakarta and .20 and .24 for Bangkok and Lagos respectively. The high proportion is not particularly noteworthy for Wayne given its latitude, but it is noteworthy for Jakarta. 

Jakarta is also the dirtiest of the 4 cities with .21 of the segments containing litter.  Lagos comes second with .15 of the segments with litter. Lagos also takes the bottom spot paved sidewalks---just .30 of the segments have a paved sidewalk.


| city | potholes | cracks | clear road markings | roads w/ markings | litter | paved sidewalk                                               |
| ---- | -------- | ------ | ------------------- | ----------------- | ------ | ------------------------------------------------------------ |
| bangkok | .06 |   .24  |  .81 |   .98 |   .06 |  .51 |
| jakarta | .23 |   .44  |  .34 |   .97 |   .21 |  .49 |
| lagos   | .06 |   .20  |  .23 |   .95 |   .15 |  .30 |
| wayne   | .07 |   .62  |  .60 |   .90 |   .09 |  .67 |

Given there are differences across cities in the proportion of trunk, primary, secondary, and tertiary roads in the road network, we checked if cross-city comparisons are mostly capturing differences in road types than differences in conditions within each type of road. To examine this, we regressed the appropriate variable (whether or not there is a pothole, a crack) on the type of the road and city. Compared to tertiary roads, potholes are more common on primary roads (Diff. = .03), secondary roads (Diff. = .01), and trunk roads (Diff. = .05). But adjusting for the type of road doesn't change the across-city estimates much. For instance, the difference in the proportion of segments with potholes between Wayne and Jakarta is still .16.

Moving to cracks in the road, compared to tertiary roads, primary, secondary, and trunk roads have fewer cracks with differences being -.05, -.03, and -.09 respectively. Like with potholes, adjusting for the kind of roads doesn't seem to make much of a difference for inferences from raw data for cross-city comparison. 

Next, we analyzed the relationship between the condition of the roads and local income. To do that, we used the AskGeo API to get information on per capita income the census tract in which the lat/long lay. And we regressed whether a segment had a crack (or a pothole) or not on income split into quintiles. 

Before we present the results, a caveat. Given that we expect the largest relationship between quality of neighborhood roads and local income, we expect that our subsetting on primary, secondary, tertiary, and trunk roads to lead to smaller coefficients. 

Compared to road segments in tracts with per capita income less than 12k, the proportion of road segments with potholes in tracts with per capita income between 12k and 17k  was -.01 less. The proportion of road segments in tracts with per capita income of 17k to 23k  was -.02 less. For tracts with 23k and 29k, it was -.03 fewer segments with potholes, and for tracts with income between 29k and 83k, -.02 fewer segments had potholes. The relationship between local income and the proportion of segments with cracks was more uneven. The highest quintile had the fewest cracks but roads in the second and third income quintile areas had roughly the same number of cracks.

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
    - [Road Conditions in Wayne by Income, etc.](scripts/wayne_income.ipynb)

* Manuscript:
    - [pdf, .tex, .bib](ms/)

### Authors

Suriyan Laohaprapanon, Kimberly Ortleb, and Gaurav Sood

