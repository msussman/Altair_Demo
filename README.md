
# Altair Choropleth Walkthrough

## Background
I recently completed the [Data Science Graduation Certificate program at Georgetown University](https://scs.georgetown.edu/programs/375/certificate-in-data-science/) where I led a team Capstone project that tried to determine if the District of Columbia's new dockless bikeshare pilot is impacting demand it's traditional bikeshare system, [Capital Bikeshare](https://capitalbikeshare.com).  You can learn more about my team's Capstone on our [Github](https://github.com/georgetown-analytics/DC-Bikeshare).

Being able to visualize data geographically was paramount for my team's Capstone project and after being introduced to [Altair](https://altair-viz.github.io) in the program's visualization course, I wanted to explore the GIS capabilites of this lesser known, but up and coming Python visualization package.

In addition to Altair's excellent [documentation](https://altair-viz.github.io) There are a good number of general Altair walkthroughs that I highly recommend you browse prior to this walkthrough, as I will be focusing soley on how to build Choropleth maps from scratch in Altair.  Below are some recommended Altair tutorials:

+ [Practical Business Python](http://pbpython.com/altair-intro.html)
+ [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2017/12/introduction-to-altair-a-declarative-visualization-in-python/)
+ [Altair Notebooks](https://github.com/altair-viz/altair_notebooks)

## Required Packages

In addition to Altair, which I recommend you follow the installation instructions [here](https://altair-viz.github.io/getting_started/installation.html#installation-notebook), we'll be using the packages below for this demostration.

+ Requests - In order to pull the DC geopolitical GeoJSON from the [Open Data DC](http://opendata.dc.gov/) website. 
+ Pandas - In order to read in the cleaned DC population data that we'll add as the choropleth layer on our map
+ Geopandas - In order to join the DC population and GeoJSON data together
+ JSON - In order to convert the Geopandas dataframe into a JSON, which is required by Altair.  More context on Altair Geopandas incompatibility can be found [here](https://github.com/altair-viz/altair/issues/588).

A full requirements file is located on my GitHub [here](https://github.com/msussman/Altair_Demo/blob/master/requirements.txt).




```python
import altair as alt
import requests
import pandas as pd
import geopandas as gpd
import json
```

## Altair Settings

In order to render Altair plots in Jupyter Noteboook, you must enable the "alt.renderers.enable('notebook')".

If you're interested in saving your maps or any other plot as a JPEG, it's highly recommended that you also enable the 'opaque' theme, as the default Altair theme is transparent. The defauly theme will cause a your JPEG to have a checkered background.

Note that while Jupyter notebook is fully supported by Altair, the developers recommend using Jupyterlab for a better experience.



```python
alt.renderers.enable('notebook')
alt.themes.enable('opaque')
```




    ThemeRegistry.enable('opaque')



## Create Baselayer of Map from GeoJSON

For this demostration, we'll leverage the GeoJSON for the DC's Advisory Neighborhood Commision district provided by [Open Data DC](https://dc.gov/page/open-data).  There is a more recent version of this GeoJSON, but the 2000 and 2010 Census population data we'll be adding later is based on this interation of the ANC geopolitical districts.

First, we use the "download_json" function to download the ANC GeoJSON from the opendata website.  We can then create the base layer of our map by passing the GeoJSON directly to Altair and marking the geoshape accordingly as shown in the "gen_base" function.


```python
def download_json():
    '''Downloads ANC JSON from Open Data DC'''
    url = "https://opendata.arcgis.com/datasets/bfe6977cfd574c2b894cd67cf6a787c3_2.geojson"
    resp = requests.get(url)
    return resp.json()

def gen_base(geojson):
    '''Generates baselayer of DC ANC map'''
    base = alt.Chart(alt.Data(values=geojson)).mark_geoshape(
        stroke='black',
        strokeWidth=1
    ).encode(
    ).properties(
        width=400,
        height=400
    )
    return base

anc_json = download_json()
base_layer = gen_base(geojson=anc_json)
base_layer

```

![png](markdown_images/Altair_Demo_6_3.png)


## Convert to Geopandas Dataframe

Next, we'll convert the GeoJSON used to create the base layer of the map to a Geopandas dataframe in order to join on the ANC specific population data and make some additional data manipulations.  Geopandas dataframes function almost exactly like standard Pandas dataframe, except they have additional functionality for geographic geometry like points and polygons.


```python
# Convert GeoJSON to Geopandas Dataframe 
gdf = gpd.GeoDataFrame.from_features((anc_json))
gdf.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANC_ID</th>
      <th>NAME</th>
      <th>OBJECTID</th>
      <th>SHAPE_Area</th>
      <th>SHAPE_Length</th>
      <th>WEB_URL</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4C</td>
      <td>ANC 4C</td>
      <td>1</td>
      <td>3.344032e+06</td>
      <td>10341.699937</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=4&amp;office=C</td>
      <td>POLYGON ((-77.02801250848198 38.9612668573881,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4D</td>
      <td>ANC 4D</td>
      <td>2</td>
      <td>1.842719e+06</td>
      <td>6421.433702</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=4&amp;office=D</td>
      <td>POLYGON ((-77.01787596864135 38.95766774981323...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1B</td>
      <td>ANC 1B</td>
      <td>3</td>
      <td>2.747224e+06</td>
      <td>7418.871696</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=1&amp;office=B</td>
      <td>POLYGON ((-77.01824302717323 38.92852061963107...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2B</td>
      <td>ANC 2B</td>
      <td>4</td>
      <td>2.160268e+06</td>
      <td>7713.349828</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=2&amp;office=B</td>
      <td>POLYGON ((-77.03847471278722 38.91701117992299...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6B</td>
      <td>ANC 6B</td>
      <td>5</td>
      <td>4.899464e+06</td>
      <td>10778.799866</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=6&amp;office=B</td>
      <td>POLYGON ((-77.00591806003315 38.88981961048005...</td>
    </tr>
  </tbody>
</table>
</div>



## Add Population Data to Geopandas Dataframe

Now that we have a Geopandas Dataframe, we can join on our 2000 and 2010 population data that comes from the [DC Office of Planning](https://planning.dc.gov/publication/anc-population-change-census-2000-and-2010).  This data is in PDF format, which we could leverage here directly, but to simplifly the process I've provided a CSV of this data in my [Github](https://github.com/msussman/Altair_Demo/blob/master/data/anc_population.csv).  We will read this CSV directly into a dataframe and join onto our Geopandas dataframe.


```python
pop_df = pd.read_csv('../data/anc_population.csv')
gdf = gdf.merge(pop_df, on='ANC_ID', how='inner')
gdf.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANC_ID</th>
      <th>NAME</th>
      <th>OBJECTID</th>
      <th>SHAPE_Area</th>
      <th>SHAPE_Length</th>
      <th>WEB_URL</th>
      <th>geometry</th>
      <th>pop_2000</th>
      <th>pop_2010</th>
      <th>pop_diff</th>
      <th>pop_diff_perc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4C</td>
      <td>ANC 4C</td>
      <td>1</td>
      <td>3.344032e+06</td>
      <td>10341.699937</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=4&amp;office=C</td>
      <td>POLYGON ((-77.02801250848198 38.9612668573881,...</td>
      <td>19579</td>
      <td>20330</td>
      <td>751</td>
      <td>0.038</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4D</td>
      <td>ANC 4D</td>
      <td>2</td>
      <td>1.842719e+06</td>
      <td>6421.433702</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=4&amp;office=D</td>
      <td>POLYGON ((-77.01787596864135 38.95766774981323...</td>
      <td>12341</td>
      <td>12463</td>
      <td>122</td>
      <td>0.010</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1B</td>
      <td>ANC 1B</td>
      <td>3</td>
      <td>2.747224e+06</td>
      <td>7418.871696</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=1&amp;office=B</td>
      <td>POLYGON ((-77.01824302717323 38.92852061963107...</td>
      <td>21640</td>
      <td>25111</td>
      <td>3,471</td>
      <td>0.160</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2B</td>
      <td>ANC 2B</td>
      <td>4</td>
      <td>2.160268e+06</td>
      <td>7713.349828</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=2&amp;office=B</td>
      <td>POLYGON ((-77.03847471278722 38.91701117992299...</td>
      <td>17867</td>
      <td>18117</td>
      <td>249</td>
      <td>0.014</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6B</td>
      <td>ANC 6B</td>
      <td>5</td>
      <td>4.899464e+06</td>
      <td>10778.799866</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=6&amp;office=B</td>
      <td>POLYGON ((-77.00591806003315 38.88981961048005...</td>
      <td>21364</td>
      <td>23847</td>
      <td>2,483</td>
      <td>0.116</td>
    </tr>
  </tbody>
</table>
</div>



## Determine Center of Each ANC Polygon

For the next data preparation step, we'll calculate the centroid (center) coordinates of each ANC polygon in order later add centered ANC labels to each geographic ANC polygon.  The Geopandas centroid method makes this calculation easy.


```python
gdf['centroid_lon'] = gdf['geometry'].centroid.x
gdf['centroid_lat'] = gdf['geometry'].centroid.y
gdf.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ANC_ID</th>
      <th>NAME</th>
      <th>OBJECTID</th>
      <th>SHAPE_Area</th>
      <th>SHAPE_Length</th>
      <th>WEB_URL</th>
      <th>geometry</th>
      <th>pop_2000</th>
      <th>pop_2010</th>
      <th>pop_diff</th>
      <th>pop_diff_perc</th>
      <th>centroid_lon</th>
      <th>centroid_lat</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4C</td>
      <td>ANC 4C</td>
      <td>1</td>
      <td>3.344032e+06</td>
      <td>10341.699937</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=4&amp;office=C</td>
      <td>POLYGON ((-77.02801250848198 38.9612668573881,...</td>
      <td>19579</td>
      <td>20330</td>
      <td>751</td>
      <td>0.038</td>
      <td>-77.027911</td>
      <td>38.945261</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4D</td>
      <td>ANC 4D</td>
      <td>2</td>
      <td>1.842719e+06</td>
      <td>6421.433702</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=4&amp;office=D</td>
      <td>POLYGON ((-77.01787596864135 38.95766774981323...</td>
      <td>12341</td>
      <td>12463</td>
      <td>122</td>
      <td>0.010</td>
      <td>-77.018234</td>
      <td>38.951656</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1B</td>
      <td>ANC 1B</td>
      <td>3</td>
      <td>2.747224e+06</td>
      <td>7418.871696</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=1&amp;office=B</td>
      <td>POLYGON ((-77.01824302717323 38.92852061963107...</td>
      <td>21640</td>
      <td>25111</td>
      <td>3,471</td>
      <td>0.160</td>
      <td>-77.024453</td>
      <td>38.921205</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2B</td>
      <td>ANC 2B</td>
      <td>4</td>
      <td>2.160268e+06</td>
      <td>7713.349828</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=2&amp;office=B</td>
      <td>POLYGON ((-77.03847471278722 38.91701117992299...</td>
      <td>17867</td>
      <td>18117</td>
      <td>249</td>
      <td>0.014</td>
      <td>-77.040645</td>
      <td>38.908462</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6B</td>
      <td>ANC 6B</td>
      <td>5</td>
      <td>4.899464e+06</td>
      <td>10778.799866</td>
      <td>http://app.anc.dc.gov/wards.asp?ward=6&amp;office=B</td>
      <td>POLYGON ((-77.00591806003315 38.88981961048005...</td>
      <td>21364</td>
      <td>23847</td>
      <td>2,483</td>
      <td>0.116</td>
      <td>-76.987266</td>
      <td>38.883534</td>
    </tr>
  </tbody>
</table>
</div>



## Convert Geopandas Dataframe back to GeoJSON

Now that we have all the data we need to create my map, we can convert the Geopandas dataframe back to a GeoJSON and render the features from the GeoJSON into Altair.


```python
choro_json = json.loads(gdf.to_json())
choro_data = alt.Data(values=choro_json['features'])
```

## Add Choropleth and Label Layers to the Map

Having now compiled all the data we need into a GeoJSON, we can expand the "gen_base" function from before to add all three layers to my map:

1. Base
2. Choropeth
3. ANC Labels

with the ANC Population in 2000 choropleth map exactly how we want it, here are some finer items to focus on in the 'gen_map' function.

+ **Color Scheme**:  The color scheme is explicitly defined 'bluegreen' as a parameter of the Scale method when encoding the Choropleth layer.  Since Altaier is built on Vega, the available color schemes are predefined by what's [available in Vega](https://vega.github.io/vega/docs/schemes/)
+ **Specifying Data Types**: In the labels layer, the data typesare explicitly defined as quantitative ":Q" and ordinal ":O".  This is necessary because we're passing a JSON, not a dataframe into the Altair Chart method, so the data types are cannot be communicated to Altair.  Most Altair plots leverage a dataframe, so this step isn't generally necessary, but is a good habit to ensure altair is rendering the data as intended.
+ **Adding Layers**: In the return statement, we use the "+" to add the layers on top of each other, which highlights the elegant simplicity that separates Altair from other visualization packages.


```python
def gen_map(geodata, color_column, title):
    '''Generates DC ANC map with population choropleth and ANC labels'''
    # Add Base Layer
    base = alt.Chart(geodata, title = title).mark_geoshape(
        stroke='black',
        strokeWidth=1
    ).encode(
    ).properties(
        width=400,
        height=400
    )
    # Add Choropleth Layer
    choro = alt.Chart(geodata).mark_geoshape(
        fill='lightgray',
        stroke='black'
    ).encode(
        alt.Color(color_column, 
                  type='quantitative', 
                  scale=alt.Scale(scheme='bluegreen'),
                  title = "DC Population")
    )
    # Add Labels Layer
    labels = alt.Chart(geodata).mark_text(baseline='top'
     ).properties(
        width=400,
        height=400
     ).encode(
         longitude='properties.centroid_lon:Q',
         latitude='properties.centroid_lat:Q',
         text='properties.ANC_ID:O',
         size=alt.value(8),
         opacity=alt.value(1)
     )

    return base + choro + labels

pop_2000_map = gen_map(geodata=choro_data, color_column='properties.pop_2000', title='2000')
pop_2000_map
```


<div class="vega-embed" id="aca360ef-3c53-4047-887d-40df2c290a4a"></div>

<style>
.vega-embed .vega-actions > a {
    transition: opacity 200ms ease-in;
    opacity: 0.3;
    margin-right: 0.6em;
    color: #444;
    text-decoration: none;
}

.vega-embed .vega-actions > a:hover {
    color: #000;
    text-decoration: underline;
}

.vega-embed:hover .vega-actions > a {
    opacity: 1;
    transition: 0s;
}

.vega-embed .error p {
    color: firebrick;
    font-size: 1.2em;
}
</style>








    




![png](markdown_images/Altair_Demo_16_3.png)


## Add 2010 Population to Map

Lastly, we generate a second choropleth for 2010 population using the "gen_map" function and concentate the two maps into one plot.  In Altair, the "|" operator adds concentates two plots horizontally and the "&" operator vertically.  Again, highlighting Altair's ease of use.

By concatentating these two maps together, the color scale automatically adjusts to span both maps.


```python
pop_2010_map = gen_map(geodata=choro_data, color_column='properties.pop_2010', title='2010')
pop_2000_map | pop_2010_map
```


<div class="vega-embed" id="36ea744d-2032-4f8c-af8b-bc113593d959"></div>

<style>
.vega-embed .vega-actions > a {
    transition: opacity 200ms ease-in;
    opacity: 0.3;
    margin-right: 0.6em;
    color: #444;
    text-decoration: none;
}

.vega-embed .vega-actions > a:hover {
    color: #000;
    text-decoration: underline;
}

.vega-embed:hover .vega-actions > a {
    opacity: 1;
    transition: 0s;
}

.vega-embed .error p {
    color: firebrick;
    font-size: 1.2em;
}
</style>








    




![png](markdown_images/Altair_Demo_18_3.png)

## Observations on Final Maps

Now that we can view the two maps side-by-side, some trends jump out immediately:

+ ANCs 5A, 5B and 5C are both the largest ANCs by size and population, which is why they were cut up during the 2013 redistricting.
+ Migration to center of the district can be seen by concentration gains ANCs 1B, 5C, 6B, and 6C
+ Wards 7 and 8 are largely losing population in both absolute and relative terms

I hope this walkthrough has peaked your interest in Altair.  You can find the notebook that this post is based on [here](https://github.com/msussman/Altair_Demo/blob/master/notebooks/Altair_Demo.ipynb)

