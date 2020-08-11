
## Final Project Submission

* Student name:                        KEVIN GIROUX
* Student pace:                        FULL TIME
* Scheduled project review date/time:  (TBD)
* Instructor name:                     JAMES IRVING
* Blog post URL:                       (TBD)


# Introduction

##### My overall methodology for organizing and analyzing this data was as follows.  I started by reviewing the provided datasets, grouping them by the type of information they contained, and determining which dataset would serve as the a good starting point for my analysis.  After some online research, I determined the IMDB datasets to be a good place to start for their mergability with each other along IMDB ID tags and for my ability to employe the IMDbPY toolkit to query IMDB's API and fill in any gaps in the data provided (namely, movie financial data).  

##### Once I finished querying all additional features I wanted to loop into my analysis, I merged them all together and cleaned the data down to a set of 551 movies for analysis.  

# Review of provided data; gap analysis


```python
import pandas as pd
import json
import matplotlib.pyplot as plt
import seaborn as sns
```

    /Users/kevingiroux/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/statsmodels/tools/_testing.py:19: FutureWarning: pandas.util.testing is deprecated. Use the functions in the public API at pandas.testing instead.
      import pandas.util.testing as tm



```python
# general movie metadata
imdb_df = pd.read_csv('zippedData/imdb.title.basics.csv.gz')
rt_df = pd.read_csv('zippedData/rt.movie_info.tsv.gz', delimiter='\t')
title_aka_df = pd.read_csv('zippedData/imdb.title.akas.csv.gz')

# ratings and review data
ratings_df = pd.read_csv('zippedData/imdb.title.ratings.csv.gz')
tmdb_ratings_df = pd.read_csv('zippedData/tmdb.movies.csv.gz')
rt_reviews_df = pd.read_csv('zippedData/rt.reviews.tsv.gz', delimiter='\t', encoding='latin1')

# people
ppl_df = pd.read_csv('zippedData/imdb.name.basics.csv.gz')
crew_df = pd.read_csv('zippedData/imdb.title.crew.csv.gz')
principals_df = pd.read_csv('zippedData/imdb.title.principals.csv.gz')

# financials
gross_df = pd.read_csv('zippedData/bom.movie_gross.csv.gz')
budget_df = pd.read_csv('zippedData/tn.movie_budgets.csv.gz')
```

## Qualitative description of data sets:


### a. Movie background info and metadata
- imdb_df:  (IMDB) movie titles (w/ IMDB title id) by release year, runtime, and genre
- rt_df:  (Rotten Tomatoes) synopsis, rating, genre, director, writer, runtime, etc.
- title_aka_df:  (IMDB) movie titles (w/ IMDB title id) in different languages

### b. Reviews and ratings
- ratings_df:  (IMDB) average rating and vote count, by IMDB title id
- tmdb_ratings_df:  (TMDB) popularity score, with vote counts and average vote score, by title 
- rt_reviews_df:  (Rotten Tomatoes) includes review text, numerical rating, and review publisher

### c. People involved
- ppl_df:  (IMDB) info on humans involved, such as birth year, primary profession, and movies they are known for (by title id)
- crew_df:  (IMDB) directors and writers, by movie id (matches title id with name id)
- principals_df:  (IMDB) (title id, name id) categorized list of people by role/job in film production (includes list of characters for actors/actresses)

### d. Financial
- gross_df:  (BOM) domestic and foreign gross rev by movie title, studio, and year
- budget_df: (TN) production budget vs. domestic and worldwide gross rev, by movie title (no movie id!)


##### Below, I will examine each of the above sub-sets of data in more detail and draw conclusions on which data sets/features to use and how

## a. Movie background info and metadata
- imdb_df: 'genres' column contains list of multiple genres separated by comma (no spaces)
- rt_df: no movie titles are included, just an RT movie id;  'rating', 'director', 'writer', 'theater_date', 'studio', 'box_office' are all unique features to RT data
- title_aka_df: not sure I will need this data


##### CONCLUSIONS: 
- Use imdb_df as my base data set
- look for ways to pull in 'rating', 'studio', 'theater_date' and 'box_office' from rt_df or the internet


```python
imdb_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.0</td>
      <td>Biography,Drama</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0069204</td>
      <td>Sabse Bada Sukh</td>
      <td>Sabse Bada Sukh</td>
      <td>2018</td>
      <td>NaN</td>
      <td>Comedy,Drama</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.0</td>
      <td>Comedy,Drama,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
rt_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>synopsis</th>
      <th>rating</th>
      <th>genre</th>
      <th>director</th>
      <th>writer</th>
      <th>theater_date</th>
      <th>dvd_date</th>
      <th>currency</th>
      <th>box_office</th>
      <th>runtime</th>
      <th>studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>This gritty, fast-paced, and innovative police...</td>
      <td>R</td>
      <td>Action and Adventure|Classics|Drama</td>
      <td>William Friedkin</td>
      <td>Ernest Tidyman</td>
      <td>Oct 9, 1971</td>
      <td>Sep 25, 2001</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>104 minutes</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>New York City, not-too-distant-future: Eric Pa...</td>
      <td>R</td>
      <td>Drama|Science Fiction and Fantasy</td>
      <td>David Cronenberg</td>
      <td>David Cronenberg|Don DeLillo</td>
      <td>Aug 17, 2012</td>
      <td>Jan 1, 2013</td>
      <td>$</td>
      <td>600,000</td>
      <td>108 minutes</td>
      <td>Entertainment One</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>Illeana Douglas delivers a superb performance ...</td>
      <td>R</td>
      <td>Drama|Musical and Performing Arts</td>
      <td>Allison Anders</td>
      <td>Allison Anders</td>
      <td>Sep 13, 1996</td>
      <td>Apr 18, 2000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>116 minutes</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6</td>
      <td>Michael Douglas runs afoul of a treacherous su...</td>
      <td>R</td>
      <td>Drama|Mystery and Suspense</td>
      <td>Barry Levinson</td>
      <td>Paul Attanasio|Michael Crichton</td>
      <td>Dec 9, 1994</td>
      <td>Aug 27, 1997</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>128 minutes</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7</td>
      <td>NaN</td>
      <td>NR</td>
      <td>Drama|Romance</td>
      <td>Rodney Bennett</td>
      <td>Giles Cooper</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>200 minutes</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
title_aka_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title_id</th>
      <th>ordering</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0369610</td>
      <td>11</td>
      <td>Jurashikku warudo</td>
      <td>JP</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0369610</td>
      <td>12</td>
      <td>Jurassic World: O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0369610</td>
      <td>13</td>
      <td>O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>short title</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0369610</td>
      <td>14</td>
      <td>Jurassic World</td>
      <td>FR</td>
      <td>NaN</td>
      <td>imdbDisplay</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



## b. Reviews and ratings
- ratings_df: imdb title id, average score and number of votes...needs enrichment
- tmdb_ratings_df: has a lot of cool features like a popularity score, release date, vote_average, vote_count, original_language -- can by joined by 'original_title' or 'title'
- rt_reviews_df: again, no movie titles included, includes a top critic designation which is cool, and a fresh/not fresh binary categorization column, also 'publisher' columns

##### CONCLUSIONS:
- use ratings_df as my starting point for reviews and ratings data, because of easy join-ability with movie metadata via IMDB title id
- find way to enrich this review data via API/web-scraping or by joining some features from the TMDB dataset, which has a lot of features that would be interesting to play with ('original language', 'popularity', 'vote_average', 'vote count')
- RT data has a couple interesting features, but no movie title column will make this data very hard to join with my other rating/review data


```python
ratings_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt10384606</td>
      <td>8.9</td>
      <td>559</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt1043726</td>
      <td>4.2</td>
      <td>50352</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt1060240</td>
      <td>6.5</td>
      <td>21</td>
    </tr>
  </tbody>
</table>
</div>




```python
tmdb_ratings_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>genre_ids</th>
      <th>id</th>
      <th>original_language</th>
      <th>original_title</th>
      <th>popularity</th>
      <th>release_date</th>
      <th>title</th>
      <th>vote_average</th>
      <th>vote_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>[12, 14, 10751]</td>
      <td>12444</td>
      <td>en</td>
      <td>Harry Potter and the Deathly Hallows: Part 1</td>
      <td>33.533</td>
      <td>2010-11-19</td>
      <td>Harry Potter and the Deathly Hallows: Part 1</td>
      <td>7.7</td>
      <td>10788</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>[14, 12, 16, 10751]</td>
      <td>10191</td>
      <td>en</td>
      <td>How to Train Your Dragon</td>
      <td>28.734</td>
      <td>2010-03-26</td>
      <td>How to Train Your Dragon</td>
      <td>7.7</td>
      <td>7610</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>[12, 28, 878]</td>
      <td>10138</td>
      <td>en</td>
      <td>Iron Man 2</td>
      <td>28.515</td>
      <td>2010-05-07</td>
      <td>Iron Man 2</td>
      <td>6.8</td>
      <td>12368</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>[16, 35, 10751]</td>
      <td>862</td>
      <td>en</td>
      <td>Toy Story</td>
      <td>28.005</td>
      <td>1995-11-22</td>
      <td>Toy Story</td>
      <td>7.9</td>
      <td>10174</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>[28, 878, 12]</td>
      <td>27205</td>
      <td>en</td>
      <td>Inception</td>
      <td>27.920</td>
      <td>2010-07-16</td>
      <td>Inception</td>
      <td>8.3</td>
      <td>22186</td>
    </tr>
  </tbody>
</table>
</div>




```python
rt_reviews_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>review</th>
      <th>rating</th>
      <th>fresh</th>
      <th>critic</th>
      <th>top_critic</th>
      <th>publisher</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3</td>
      <td>A distinctly gallows take on contemporary fina...</td>
      <td>3/5</td>
      <td>fresh</td>
      <td>PJ Nabarro</td>
      <td>0</td>
      <td>Patrick Nabarro</td>
      <td>November 10, 2018</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>It's an allegory in search of a meaning that n...</td>
      <td>NaN</td>
      <td>rotten</td>
      <td>Annalee Newitz</td>
      <td>0</td>
      <td>io9.com</td>
      <td>May 23, 2018</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>... life lived in a bubble in financial dealin...</td>
      <td>NaN</td>
      <td>fresh</td>
      <td>Sean Axmaker</td>
      <td>0</td>
      <td>Stream on Demand</td>
      <td>January 4, 2018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Continuing along a line introduced in last yea...</td>
      <td>NaN</td>
      <td>fresh</td>
      <td>Daniel Kasman</td>
      <td>0</td>
      <td>MUBI</td>
      <td>November 16, 2017</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>... a perverse twist on neorealism...</td>
      <td>NaN</td>
      <td>fresh</td>
      <td>NaN</td>
      <td>0</td>
      <td>Cinema Scope</td>
      <td>October 12, 2017</td>
    </tr>
  </tbody>
</table>
</div>



## c. People involved
- ppl_df: full list of people in film industry, with unique imdb name ids, known_for_titles by imdb title id
- crew_df: imdb name ids for directors and writers, associated with imdb film ids
- principals_df: imdb film ids, with imdb name ids of principal resources associated, by job function

##### CONCLUSIONS:
- principals_df could be useful for developing recommendations for specific writers, directors, actors/actresses, etc. to use
- may need to use ppl_df and crew_df to perform any necessary joins between people data and our other data sub-sets (movie metadata, ratings/reviews, and financials)
- all of these are from IMDB and use IMDB's title ID and name ID, which should make for easy joining


```python
ppl_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm0061671</td>
      <td>Mary Ellen Bauder</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,production_manager,producer</td>
      <td>tt0837562,tt2398241,tt0844471,tt0118553</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nm0061865</td>
      <td>Joseph Bauer</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>composer,music_department,sound_department</td>
      <td>tt0896534,tt6791238,tt0287072,tt1682940</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nm0062070</td>
      <td>Bruce Baum</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,actor,writer</td>
      <td>tt1470654,tt0363631,tt0104030,tt0102898</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nm0062195</td>
      <td>Axel Baumann</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>camera_department,cinematographer,art_department</td>
      <td>tt0114371,tt2004304,tt1618448,tt1224387</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nm0062798</td>
      <td>Pete Baxter</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>production_designer,art_department,set_decorator</td>
      <td>tt0452644,tt0452692,tt3458030,tt2178256</td>
    </tr>
  </tbody>
</table>
</div>




```python
crew_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>directors</th>
      <th>writers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0285252</td>
      <td>nm0899854</td>
      <td>nm0899854</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0438973</td>
      <td>NaN</td>
      <td>nm0175726,nm1802864</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0462036</td>
      <td>nm1940585</td>
      <td>nm1940585</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0835418</td>
      <td>nm0151540</td>
      <td>nm0310087,nm0841532</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0878654</td>
      <td>nm0089502,nm2291498,nm2292011</td>
      <td>nm0284943</td>
    </tr>
  </tbody>
</table>
</div>




```python
principals_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>nconst</th>
      <th>category</th>
      <th>job</th>
      <th>characters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0111414</td>
      <td>1</td>
      <td>nm0246005</td>
      <td>actor</td>
      <td>NaN</td>
      <td>["The Man"]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0111414</td>
      <td>2</td>
      <td>nm0398271</td>
      <td>director</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0111414</td>
      <td>3</td>
      <td>nm3739909</td>
      <td>producer</td>
      <td>producer</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0323808</td>
      <td>10</td>
      <td>nm0059247</td>
      <td>editor</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0323808</td>
      <td>1</td>
      <td>nm3579312</td>
      <td>actress</td>
      <td>NaN</td>
      <td>["Beth Boothby"]</td>
    </tr>
  </tbody>
</table>
</div>



## d. Financials
- gross_df: includes a feature for studio abbreviations and a feature for the year the film was released
- budget_df: includes a budget feature, along with revenue data, as well as a full 'release_date' feature

##### CONCLUSIONS:
- this will be the most challenging sub-set of data to join with the rest, as neither of the provided financial datasets is from IMDB
- leaning towards using budget_df exclusively for my financial data at this point, due to inclusion of 'budget' feature and full 'release_date' feature rather than just the release year
- hopefully there will be a more direct way to scrape and integrate these numbers from the internet


```python
gross_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>studio</th>
      <th>domestic_gross</th>
      <th>foreign_gross</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Toy Story 3</td>
      <td>BV</td>
      <td>415000000.0</td>
      <td>652000000</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alice in Wonderland (2010)</td>
      <td>BV</td>
      <td>334200000.0</td>
      <td>691300000</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Harry Potter and the Deathly Hallows Part 1</td>
      <td>WB</td>
      <td>296000000.0</td>
      <td>664300000</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Inception</td>
      <td>WB</td>
      <td>292600000.0</td>
      <td>535700000</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Shrek Forever After</td>
      <td>P/DW</td>
      <td>238700000.0</td>
      <td>513900000</td>
      <td>2010</td>
    </tr>
  </tbody>
</table>
</div>




```python
budget_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
    </tr>
  </tbody>
</table>
</div>



## Gap analysis

- Use imdb_df as my base data set for movie metadata
    - look for ways to pull in 'rating', 'studio', 'theater_date' and 'box_office' from rt_df or from the internet


- Use ratings_df as my starting point for reviews and ratings data, because of easy join-ability with movie metadata via IMDB title id
    - join in 'original language', 'popularity', 'vote_average', 'vote count' from tmdb_ratings_df or internet
    - Unlikely to use RT data due to lack of movie title feature


- Use principals_df for developing recommendations on specific writers, directors, actors/actresses, etc. to use (join with metadata via IMDB ID)
    - may need to use ppl_df and crew_df to complete the joins with final combined dataset


- sources of financial provided are NOT from IMDB, so cannot join financials via IMDB ID as above
    - budget_df includes 'budget' along with gross totals, so this is the better of the two provided sets of data
    - finding ways to scrape financials from web will be a top priority 

# Cleaning and tidying


```python
# I know these data sets are messy, so before attempting to clean I jumped online and found a Python toolkit called
# IMDbPY that was specifically designed to query IMDb using python.  I will import below and see what gaps I can fill
# in imdb_df using the API before attempting any joins.
```

## Exploring IMDbPY Python toolkit


```python
import imdb
ia = imdb.IMDb()
```


```python
# List of the sets of features we can import using IMDbPY
ia.get_movie_infoset()
```




    ['airing',
     'akas',
     'alternate versions',
     'awards',
     'connections',
     'crazy credits',
     'critic reviews',
     'episodes',
     'external reviews',
     'external sites',
     'faqs',
     'full credits',
     'goofs',
     'keywords',
     'list',
     'locations',
     'main',
     'misc sites',
     'news',
     'official sites',
     'parents guide',
     'photo sites',
     'plot',
     'quotes',
     'recommendations',
     'release dates',
     'release info',
     'reviews',
     'sound clips',
     'soundtrack',
     'synopsis',
     'taglines',
     'technical',
     'trivia',
     'tv schedule',
     'video clips',
     'vote details']




```python
# List of features within the feature set 'main'
# NOTE Moonlight (2016) aka 'tt4975722' used as placeholder for all following queries of IMDbPY
ia.get_movie('4975722', info='main').keys()
```




    ['original title',
     'cast',
     'genres',
     'runtimes',
     'countries',
     'country codes',
     'language codes',
     'color info',
     'aspect ratio',
     'sound mix',
     'box office',
     'certificates',
     'original air date',
     'rating',
     'votes',
     'cover url',
     'imdbID',
     'plot outline',
     'languages',
     'title',
     'year',
     'kind',
     'directors',
     'writers',
     'producers',
     'composers',
     'cinematographers',
     'editors',
     'editorial department',
     'casting directors',
     'production designers',
     'art directors',
     'set decorators',
     'costume designers',
     'make up department',
     'production managers',
     'assistant directors',
     'art department',
     'sound department',
     'visual effects',
     'stunts',
     'camera department',
     'casting department',
     'costume departmen',
     'location management',
     'music department',
     'script department',
     'miscellaneous',
     'thanks',
     'akas',
     'writer',
     'director',
     'production companies',
     'distributors',
     'special effects',
     'other companies',
     'canonical title',
     'long imdb title',
     'long imdb canonical title',
     'smart canonical title',
     'smart long imdb canonical title',
     'full-size cover url']



### Programmatic exploration of toolkit
#### a. Movie metadata
- movie rating:  IMDbPY 'main' 'certificates' (this lists theater rating for all countries, need to isolate US)
- studio:  IMDbPY 'main' 'production companies'
- theater date:  IMDbPY 'main' 'original air date' AND 'year'
- PLUS original language:  this could be pulled in from TMDB_ratings_df as well

#### b. Ratings and reviews
- IMDbPY 'vote details' 'demographics' for demographic info about top voters of movies)
- PLUS still need to join in TMDB_ratings_df for 'popularity', 'vote_average', 'vote_count'

#### c. People
- FIRST I want to explore principals_df, but if that proves to be too complicated for joining....
- if not, I can at least pull 'writer' and 'director' data from IMDbPY

#### d. Financials
- IMDbPY 'main' 'box office' (includes budget, opening weekend gross, domestic gross, global gross)


```python
ia.get_movie('4975722', info='main')['certificates']
```




    ['Argentina:16',
     'Australia:M',
     'Austria:16',
     'Brazil:16',
     'Canada:14A::(Alberta/British Columbia/Manitoba)',
     'Canada:13+::(Quebec)',
     'Chile:14',
     'Colombia:12',
     'Denmark:11',
     'Finland:K-12',
     'France:Tous publics',
     'Germany:12',
     'Greece:K-16',
     'Hong Kong:IIB',
     'Hungary:16',
     'India:A',
     'Indonesia:17+',
     'Ireland:15A',
     'Italy:T',
     'Japan:R15+',
     'Lithuania:N-13',
     'Malaysia:(Banned)',
     'Malta:15',
     'Mexico:B15',
     'Netherlands:12',
     'New Zealand:M',
     'Norway:12',
     'Philippines:R-16',
     'Portugal:M/16',
     'Singapore:M18',
     'South Africa:13',
     'South Korea:15',
     'Spain:16',
     'Sweden:11',
     'Switzerland:14',
     'Taiwan:R-15',
     'Turkey:15+',
     'United Kingdom:15',
     'United States:R',
     'Ukraine:16']




```python
ia.get_movie('4975722', info='main')['production companies']
```




    [<Company id:0390816[http] name:_A24_>,
     <Company id:0641956[http] name:_PASTEL_>,
     <Company id:0136967[http] name:_Plan B Entertainment_>]




```python
ia.get_movie('4975722', info='main')['box office']
```




    {'Budget': '$1,500,000 (estimated)',
     'Opening Weekend United States': '$1,488,740, 18 Nov 2016',
     'Cumulative Worldwide Gross': '$55,561,162, 20 Mar 2017'}




```python
ia.get_movie('4975722', info='vote details')['demographics']
```




    {'imdb users': {'votes': 261847, 'rating': 7.4},
     'aged under 18': {'votes': 295, 'rating': 7.9},
     'aged 18 29': {'votes': 68709, 'rating': 7.6},
     'aged 30 44': {'votes': 84075, 'rating': 7.2},
     'aged 45 plus': {'votes': 20049, 'rating': 7.2},
     'males': {'votes': 147071, 'rating': 7.3},
     'males aged under 18': {'votes': 202, 'rating': 8.0},
     'males aged 18 29': {'votes': 50751, 'rating': 7.6},
     'males aged 30 44': {'votes': 67279, 'rating': 7.2},
     'males aged 45 plus': {'votes': 15888, 'rating': 7.1},
     'females': {'votes': 39010, 'rating': 7.6},
     'females aged under 18': {'votes': 67, 'rating': 7.8},
     'females aged 18 29': {'votes': 16082, 'rating': 7.7},
     'females aged 30 44': {'votes': 15275, 'rating': 7.5},
     'females aged 45 plus': {'votes': 3710, 'rating': 7.5},
     'top 1000 voters': {'votes': 566, 'rating': 6.7},
     'us users': {'votes': 32784, 'rating': 7.8},
     'non us users': {'votes': 105173, 'rating': 7.3}}




```python
ia.get_movie('4975722', info='main')['writer']
```




    [<Person id:1503575[http] name:_Barry Jenkins_>,
     <Person id:4144120[http] name:_Tarell Alvin McCraney_>]




```python
ia.get_movie('4975722', info='main')['director']
```




    [<Person id:1503575[http] name:_Barry Jenkins_>]



### CONCLUSIONS

- I will join imdb_df, ratings_df, and principals_df to start forming a fully combined dataframe for analysis
- almost all interesting missing metadata can be pulled in from IMDbPY, including financials
- TMDB_ratings_df is the only non-IMDB dataset I need to focus on joining in

## Joining IMDB datasets

### Metadata + ratings = full_df


```python
# First I will left-join imdb_df and ratings_df on 'tconst' to form full_df, my main data set for analysis

imdb_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.0</td>
      <td>Biography,Drama</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0069204</td>
      <td>Sabse Bada Sukh</td>
      <td>Sabse Bada Sukh</td>
      <td>2018</td>
      <td>NaN</td>
      <td>Comedy,Drama</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.0</td>
      <td>Comedy,Drama,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>




```python
ratings_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt10384606</td>
      <td>8.9</td>
      <td>559</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt1042974</td>
      <td>6.4</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt1043726</td>
      <td>4.2</td>
      <td>50352</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt1060240</td>
      <td>6.5</td>
      <td>21</td>
    </tr>
  </tbody>
</table>
</div>




```python
full_df = pd.merge(imdb_df, ratings_df, on='tconst', how='left')
```

### full_df += people data


```python
# Now I will examine principals_df to see how I can join this info into full_df

principals_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>nconst</th>
      <th>category</th>
      <th>job</th>
      <th>characters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0111414</td>
      <td>1</td>
      <td>nm0246005</td>
      <td>actor</td>
      <td>NaN</td>
      <td>["The Man"]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0111414</td>
      <td>2</td>
      <td>nm0398271</td>
      <td>director</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0111414</td>
      <td>3</td>
      <td>nm3739909</td>
      <td>producer</td>
      <td>producer</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0323808</td>
      <td>10</td>
      <td>nm0059247</td>
      <td>editor</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0323808</td>
      <td>1</td>
      <td>nm3579312</td>
      <td>actress</td>
      <td>NaN</td>
      <td>["Beth Boothby"]</td>
    </tr>
  </tbody>
</table>
</div>




```python
# I will need to unmelt principals_df to get one single row for each film with separate columns 
# for each of the below categories containing the nconst ID where applicable

principals_df.category.value_counts()
```




    actor                  256718
    director               146393
    actress                146208
    producer               113724
    cinematographer         80091
    composer                77063
    writer                  74357
    self                    65424
    editor                  55512
    production_designer      9373
    archive_footage          3307
    archive_sound              16
    Name: category, dtype: int64




```python
# I will drop the unwanted columns 'ordering', 'job', and 'characters'
principals_df2 = principals_df.copy()
principals_df2.drop(['ordering', 'job', 'characters'], axis = 1, inplace=True)
principals_df2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>nconst</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0111414</td>
      <td>nm0246005</td>
      <td>actor</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0111414</td>
      <td>nm0398271</td>
      <td>director</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0111414</td>
      <td>nm3739909</td>
      <td>producer</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0323808</td>
      <td>nm0059247</td>
      <td>editor</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0323808</td>
      <td>nm3579312</td>
      <td>actress</td>
    </tr>
  </tbody>
</table>
</div>




```python
principals_df2.pivot(index='tconst', columns='category', values='nconst')
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-31-c84d35c3207a> in <module>
    ----> 1 principals_df2.pivot(index='tconst', columns='category', values='nconst')
    

    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/frame.py in pivot(self, index, columns, values)
       5921         from pandas.core.reshape.pivot import pivot
       5922 
    -> 5923         return pivot(self, index=index, columns=columns, values=values)
       5924 
       5925     _shared_docs[


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/reshape/pivot.py in pivot(data, index, columns, values)
        448         else:
        449             indexed = data._constructor_sliced(data[values].values, index=index)
    --> 450     return indexed.unstack(columns)
        451 
        452 


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/series.py in unstack(self, level, fill_value)
       3548         from pandas.core.reshape.reshape import unstack
       3549 
    -> 3550         return unstack(self, level, fill_value)
       3551 
       3552     # ----------------------------------------------------------------------


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/reshape/reshape.py in unstack(obj, level, fill_value)
        417             level=level,
        418             fill_value=fill_value,
    --> 419             constructor=obj._constructor_expanddim,
        420         )
        421         return unstacker.get_result()


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/reshape/reshape.py in __init__(self, values, index, level, value_columns, fill_value, constructor)
        139 
        140         self._make_sorted_values_labels()
    --> 141         self._make_selectors()
        142 
        143     def _make_sorted_values_labels(self):


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/reshape/reshape.py in _make_selectors(self)
        177 
        178         if mask.sum() < len(self.index):
    --> 179             raise ValueError("Index contains duplicate entries, cannot reshape")
        180 
        181         self.group_index = comp_index


    ValueError: Index contains duplicate entries, cannot reshape



```python
# Pivoting on the above dataframe produces a duplicate entries error, which I suspect means there are duplicate 
# tconst - category pairings for certain tconst ID's (i.e. multiple actors listed, each as its own row, for the 
# same movie)

# I will first drop all categories that I'm not interested in joining with full_df
# Then I will investigate the duplicates issue

principals_df2 = principals_df2[(principals_df2.category == 'actor') | 
                                (principals_df2.category == 'director') | 
                                (principals_df2.category == 'producer') | 
                                (principals_df2.category == 'editor') | 
                                (principals_df2.category == 'actress') | 
                                (principals_df2.category == 'writer') | 
                                (principals_df2.category == 'composer')]
```


```python
# Unfortunately, I was correct about the duplicates issue described above, based on below example
principals_df3 = principals_df2.drop('nconst', axis=1)
principals_df3[principals_df3.tconst == 'tt0323808']
```


```python
# To resolve the duplicate row problem, I will group principals_df2 by tconst and category, and aggregate 
# the matching nconst values into a single cell (when there are multiple), in order to execute a pivot

principals_df4 = principals_df2.groupby(['tconst', 'category'], as_index=False).agg({'nconst':lambda x: ', '.join(tuple(x.tolist()))})

```


```python
# Nice - this worked, we have combined duplicate category items into a single cell for each tconst ID
principals_df4.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>category</th>
      <th>nconst</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>actor</td>
      <td>nm0474801, nm0756379, nm0474876</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0063540</td>
      <td>actress</td>
      <td>nm0904537</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0063540</td>
      <td>composer</td>
      <td>nm0006210</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0063540</td>
      <td>director</td>
      <td>nm0712540</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0063540</td>
      <td>writer</td>
      <td>nm0023551, nm1194313, nm0347899, nm1391276</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Pivoting on the above, I can now join each of the below "people" columns with full_df by using tconst
principals_piv = principals_df4.pivot(index='tconst', columns='category', values='nconst')
principals_piv = principals_piv.reset_index()
principals_piv.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>category</th>
      <th>tconst</th>
      <th>actor</th>
      <th>actress</th>
      <th>archive_footage</th>
      <th>archive_sound</th>
      <th>cinematographer</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>production_designer</th>
      <th>self</th>
      <th>writer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>nm0474801, nm0756379, nm0474876</td>
      <td>nm0904537</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0006210</td>
      <td>nm0712540</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0023551, nm1194313, nm0347899, nm1391276</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0066787</td>
      <td>nm0451809, nm0794511</td>
      <td>nm0045119, nm0754829</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0536728</td>
      <td>nm2600399</td>
      <td>nm0002411</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0069049</td>
      <td>nm0001379, nm0000953</td>
      <td>nm0462648, nm0001782</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0004372</td>
      <td>nm0006166</td>
      <td>nm0000080</td>
      <td>nm0613657</td>
      <td>nm0550881, nm1475059</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0069204</td>
      <td>nm0315917, nm0037026, nm2147526, nm0244884</td>
      <td>nm1027929, nm1875977, nm1877741</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0006005</td>
      <td>nm0611531</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0347899</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0100275</td>
      <td>nm0016013, nm0721280</td>
      <td>nm0728971, nm1415193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0005948</td>
      <td>nm0749914, nm0765384</td>
      <td>NaN</td>
      <td>nm0462571, nm1856792</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm1360635</td>
    </tr>
  </tbody>
</table>
</div>




```python
full_df = full_df.merge(principals_piv, on='tconst', how='left')
```


```python
full_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>actor</th>
      <th>actress</th>
      <th>archive_footage</th>
      <th>archive_sound</th>
      <th>cinematographer</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>production_designer</th>
      <th>self</th>
      <th>writer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
      <td>7.0</td>
      <td>77.0</td>
      <td>nm0474801, nm0756379, nm0474876</td>
      <td>nm0904537</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0006210</td>
      <td>nm0712540</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0023551, nm1194313, nm0347899, nm1391276</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.0</td>
      <td>Biography,Drama</td>
      <td>7.2</td>
      <td>43.0</td>
      <td>nm0451809, nm0794511</td>
      <td>nm0045119, nm0754829</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0536728</td>
      <td>nm2600399</td>
      <td>nm0002411</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
      <td>6.9</td>
      <td>4517.0</td>
      <td>nm0001379, nm0000953</td>
      <td>nm0462648, nm0001782</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0004372</td>
      <td>nm0006166</td>
      <td>nm0000080</td>
      <td>nm0613657</td>
      <td>nm0550881, nm1475059</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0069204</td>
      <td>Sabse Bada Sukh</td>
      <td>Sabse Bada Sukh</td>
      <td>2018</td>
      <td>NaN</td>
      <td>Comedy,Drama</td>
      <td>6.1</td>
      <td>13.0</td>
      <td>nm0315917, nm0037026, nm2147526, nm0244884</td>
      <td>nm1027929, nm1875977, nm1877741</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0006005</td>
      <td>nm0611531</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0347899</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.0</td>
      <td>Comedy,Drama,Fantasy</td>
      <td>6.5</td>
      <td>119.0</td>
      <td>nm0016013, nm0721280</td>
      <td>nm0728971, nm1415193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0005948</td>
      <td>nm0749914, nm0765384</td>
      <td>NaN</td>
      <td>nm0462571, nm1856792</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm1360635</td>
    </tr>
  </tbody>
</table>
</div>



## Initial IMDB data clean


```python
# Before I query IMDbPY, I want to thin out the data included in full_df, to minimize query execution time
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 146144 entries, 0 to 146143
    Data columns (total 20 columns):
     #   Column               Non-Null Count   Dtype  
    ---  ------               --------------   -----  
     0   tconst               146144 non-null  object 
     1   primary_title        146144 non-null  object 
     2   original_title       146123 non-null  object 
     3   start_year           146144 non-null  int64  
     4   runtime_minutes      114405 non-null  float64
     5   genres               140736 non-null  object 
     6   averagerating        73856 non-null   float64
     7   numvotes             73856 non-null   float64
     8   actor                95162 non-null   object 
     9   actress              77333 non-null   object 
     10  archive_footage      1714 non-null    object 
     11  archive_sound        15 non-null      object 
     12  cinematographer      67928 non-null   object 
     13  composer             63521 non-null   object 
     14  director             128436 non-null  object 
     15  editor               48950 non-null   object 
     16  producer             70319 non-null   object 
     17  production_designer  8965 non-null    object 
     18  self                 19863 non-null   object 
     19  writer               49297 non-null   object 
    dtypes: float64(3), int64(1), object(16)
    memory usage: 23.4+ MB



```python
# To start, I will drop any rows that don't have a value for "averagerating" or "numvotes"
full_df.dropna(subset=['numvotes'], axis=0, inplace=True)
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 73856 entries, 0 to 146134
    Data columns (total 20 columns):
     #   Column               Non-Null Count  Dtype  
    ---  ------               --------------  -----  
     0   tconst               73856 non-null  object 
     1   primary_title        73856 non-null  object 
     2   original_title       73856 non-null  object 
     3   start_year           73856 non-null  int64  
     4   runtime_minutes      66236 non-null  float64
     5   genres               73052 non-null  object 
     6   averagerating        73856 non-null  float64
     7   numvotes             73856 non-null  float64
     8   actor                58001 non-null  object 
     9   actress              49952 non-null  object 
     10  archive_footage      1151 non-null   object 
     11  archive_sound        11 non-null     object 
     12  cinematographer      40662 non-null  object 
     13  composer             41547 non-null  object 
     14  director             67757 non-null  object 
     15  editor               28664 non-null  object 
     16  producer             46674 non-null  object 
     17  production_designer  6746 non-null   object 
     18  self                 9870 non-null   object 
     19  writer               32894 non-null  object 
    dtypes: float64(3), int64(1), object(16)
    memory usage: 11.8+ MB



```python
# Now I will drop any rows without runtime_minutes 
full_df.dropna(subset=['runtime_minutes'], axis=0, inplace=True)
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 66236 entries, 0 to 146134
    Data columns (total 20 columns):
     #   Column               Non-Null Count  Dtype  
    ---  ------               --------------  -----  
     0   tconst               66236 non-null  object 
     1   primary_title        66236 non-null  object 
     2   original_title       66236 non-null  object 
     3   start_year           66236 non-null  int64  
     4   runtime_minutes      66236 non-null  float64
     5   genres               65720 non-null  object 
     6   averagerating        66236 non-null  float64
     7   numvotes             66236 non-null  float64
     8   actor                51825 non-null  object 
     9   actress              44617 non-null  object 
     10  archive_footage      1090 non-null   object 
     11  archive_sound        11 non-null     object 
     12  cinematographer      36810 non-null  object 
     13  composer             38200 non-null  object 
     14  director             60983 non-null  object 
     15  editor               25983 non-null  object 
     16  producer             42762 non-null  object 
     17  production_designer  6210 non-null   object 
     18  self                 9115 non-null   object 
     19  writer               29812 non-null  object 
    dtypes: float64(3), int64(1), object(16)
    memory usage: 10.6+ MB



```python
full_df.numvotes.value_counts()
```




    6.0         2281
    5.0         2134
    7.0         1962
    8.0         1739
    9.0         1551
                ... 
    6380.0         1
    9167.0         1
    3843.0         1
    179453.0       1
    4176.0         1
    Name: numvotes, Length: 7349, dtype: int64




```python
# To further shave down the data set, I will remove all titles that received fewer than 100 votes
# so we are only getting those titles that have the most engagement with fans on IMDB
full_df = full_df[(full_df.numvotes >= 100)]
```


```python
# Finally, I will drop all movies from 2014 and earlier, so any insights we derive are based on only the most
# recent information
full_df.start_year.value_counts()
```




    2017    3402
    2016    3369
    2015    3276
    2014    3258
    2013    3103
    2012    2895
    2018    2809
    2011    2720
    2010    2540
    2019     606
    Name: start_year, dtype: int64




```python
full_df = full_df[(full_df.start_year >= 2015)]
```


```python
full_df = full_df.reset_index()
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 13462 entries, 0 to 13461
    Data columns (total 21 columns):
     #   Column               Non-Null Count  Dtype  
    ---  ------               --------------  -----  
     0   index                13462 non-null  int64  
     1   tconst               13462 non-null  object 
     2   primary_title        13462 non-null  object 
     3   original_title       13462 non-null  object 
     4   start_year           13462 non-null  int64  
     5   runtime_minutes      13462 non-null  float64
     6   genres               13451 non-null  object 
     7   averagerating        13462 non-null  float64
     8   numvotes             13462 non-null  float64
     9   actor                11877 non-null  object 
     10  actress              10775 non-null  object 
     11  archive_footage      238 non-null    object 
     12  archive_sound        3 non-null      object 
     13  cinematographer      7279 non-null   object 
     14  composer             8352 non-null   object 
     15  director             12740 non-null  object 
     16  editor               4723 non-null   object 
     17  producer             10686 non-null  object 
     18  production_designer  1507 non-null   object 
     19  self                 1238 non-null   object 
     20  writer               7930 non-null   object 
    dtypes: float64(3), int64(2), object(16)
    memory usage: 2.2+ MB


## IMDbPY query:  Financials

### Performing the query


```python
# SEE OTHER PROJECT FILE
```

### Joining and cleaning


```python
## found this code online at:
## https://stackoverflow.com/questions/55424733/how-can-i-reformat-a-json-file-to-contain-an-array

json_filename = 'imdb_financials.json'

with open(json_filename) as file:
    array = {'foo': []}
    foo_list = array['foo']
    for line in file:
        obj = json.loads(line)
        foo_list.append(obj)
```


```python
len(foo_list)
```




    13462




```python
full_df['financials'] = foo_list
```


```python
full_df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>actor</th>
      <th>actress</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>writer</th>
      <th>financials</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.0</td>
      <td>Drama</td>
      <td>6.9</td>
      <td>4517.0</td>
      <td>nm0001379, nm0000953</td>
      <td>nm0462648, nm0001782</td>
      <td>nm0006166</td>
      <td>nm0000080</td>
      <td>nm0613657</td>
      <td>nm0550881, nm1475059</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.0</td>
      <td>Comedy,Drama,Fantasy</td>
      <td>6.5</td>
      <td>119.0</td>
      <td>nm0016013, nm0721280</td>
      <td>nm0728971, nm1415193</td>
      <td>nm0005948</td>
      <td>nm0749914, nm0765384</td>
      <td>NaN</td>
      <td>nm0462571, nm1856792</td>
      <td>nm1360635</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7</td>
      <td>tt0137204</td>
      <td>Joe Finds Grace</td>
      <td>Joe Finds Grace</td>
      <td>2017</td>
      <td>83.0</td>
      <td>Adventure,Animation,Comedy</td>
      <td>8.1</td>
      <td>263.0</td>
      <td>nm0365480, nm0003210</td>
      <td>nm0367762, nm0186322</td>
      <td>nm1930572</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm0153581, nm0448515, nm0908708</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>38</td>
      <td>tt0315642</td>
      <td>Wazir</td>
      <td>Wazir</td>
      <td>2016</td>
      <td>103.0</td>
      <td>Action,Crime,Drama</td>
      <td>7.1</td>
      <td>15378.0</td>
      <td>nm0000821, nm1027719, nm1303433</td>
      <td>nm2390814</td>
      <td>NaN</td>
      <td>nm2349060</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm3871075, nm0006765, nm0430785, nm5241801, nm...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>47</td>
      <td>tt0331314</td>
      <td>Bunyan and Babe</td>
      <td>Bunyan and Babe</td>
      <td>2017</td>
      <td>84.0</td>
      <td>Adventure,Animation,Comedy</td>
      <td>5.0</td>
      <td>302.0</td>
      <td>nm0000422, nm0289344, nm0001288, nm2259477</td>
      <td>NaN</td>
      <td>nm1356349, nm0688953</td>
      <td>nm8625898</td>
      <td>NaN</td>
      <td>nm1240210</td>
      <td>nm0630057, nm0908438</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>50</td>
      <td>tt0337926</td>
      <td>Chatô - The King of Brazil</td>
      <td>Chatô: O Rei do Brasil</td>
      <td>2015</td>
      <td>102.0</td>
      <td>Biography</td>
      <td>6.0</td>
      <td>558.0</td>
      <td>nm0723142, nm0079343</td>
      <td>nm0069683, nm0754542</td>
      <td>NaN</td>
      <td>nm0285071</td>
      <td>nm0479884</td>
      <td>NaN</td>
      <td>nm0138676, nm0730422, nm1412474</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>51</td>
      <td>tt0339736</td>
      <td>The Evil Within</td>
      <td>The Evil Within</td>
      <td>2017</td>
      <td>98.0</td>
      <td>Horror</td>
      <td>5.6</td>
      <td>2420.0</td>
      <td>nm0001218, nm0462735, nm0077720</td>
      <td>nm0000539</td>
      <td>nm0339704</td>
      <td>nm1274189</td>
      <td>NaN</td>
      <td>nm0524451, nm0823123, nm2999181</td>
      <td>NaN</td>
      <td>{'Budget': '$6,000,000 (estimated)'}</td>
    </tr>
    <tr>
      <th>7</th>
      <td>55</td>
      <td>tt0360556</td>
      <td>Fahrenheit 451</td>
      <td>Fahrenheit 451</td>
      <td>2018</td>
      <td>100.0</td>
      <td>Drama,Sci-Fi,Thriller</td>
      <td>4.9</td>
      <td>14469.0</td>
      <td>nm0430107, nm2210323, nm0788335</td>
      <td>nm0441654</td>
      <td>nm1902248, nm0664020</td>
      <td>nm1023919</td>
      <td>NaN</td>
      <td>nm0167708</td>
      <td>nm0001969, nm0618881</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>57</td>
      <td>tt0365545</td>
      <td>Nappily Ever After</td>
      <td>Nappily Ever After</td>
      <td>2018</td>
      <td>98.0</td>
      <td>Comedy,Drama,Romance</td>
      <td>6.4</td>
      <td>6287.0</td>
      <td>nm1340638, nm0072713</td>
      <td>nm0005125, nm0005551</td>
      <td>NaN</td>
      <td>nm2223783</td>
      <td>NaN</td>
      <td>nm1545176, nm0082894</td>
      <td>nm0111845, nm10021676, nm1946260</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>60</td>
      <td>tt0369610</td>
      <td>Jurassic World</td>
      <td>Jurassic World</td>
      <td>2015</td>
      <td>124.0</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>7.0</td>
      <td>539338.0</td>
      <td>nm0695435, nm1339223</td>
      <td>nm0397171, nm0339460</td>
      <td>NaN</td>
      <td>nm1119880</td>
      <td>NaN</td>
      <td>nm0189777</td>
      <td>nm0415425, nm0798646, nm2081046, nm0000341</td>
      <td>{'Budget': '$150,000,000 (estimated)', 'Openin...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Getting the value_counts() for the new financials column returns an error, as the non-zero entries
# are all dictionaries

full_df['financials'].value_counts()
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.map_locations()


    TypeError: unhashable type: 'dict'


    Exception ignored in: 'pandas._libs.index.IndexEngine._call_map_locations'
    Traceback (most recent call last):
      File "pandas/_libs/hashtable_class_helper.pxi", line 1653, in pandas._libs.hashtable.PyObjectHashTable.map_locations
    TypeError: unhashable type: 'dict'





    0                                                                                                                                                 8961
    {'Budget': '$1,000,000 (estimated)'}                                                                                                               104
    {'Budget': '$500,000 (estimated)'}                                                                                                                  59
    {'Budget': '$2,000,000 (estimated)'}                                                                                                                58
    {'Budget': '$3,000,000 (estimated)'}                                                                                                                57
                                                                                                                                                      ... 
    {'Budget': '$16,000,000 (estimated)', 'Opening Weekend United States': '$6,870,740, 17 Jun 2018'}                                                    1
    {'Budget': '$12,000,000 (estimated)', 'Opening Weekend United States': '$8,500,000, 01 Feb 2015', 'Cumulative Worldwide Gross': '$33,213,241'}       1
    {'Budget': '$65,000,000 (estimated)', 'Opening Weekend United States': '$29,085,719, 07 Jun 2015'}                                                   1
    {'Opening Weekend Italy': 'EUR75,948, 07 May 2017'}                                                                                                  1
    {'Budget': 'CAD13,000,000 (estimated)'}                                                                                                              1
    Name: financials, Length: 2934, dtype: int64




```python
# The results indicate 8961 items in full_df don't have associated financial data - so I will drop these 
# rows below

full_df = full_df[(full_df['financials'] != 0)]
```


```python
len(full_df)
```




    4501




```python
full_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>actor</th>
      <th>actress</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>writer</th>
      <th>financials</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>51</td>
      <td>tt0339736</td>
      <td>The Evil Within</td>
      <td>The Evil Within</td>
      <td>2017</td>
      <td>98.0</td>
      <td>Horror</td>
      <td>5.6</td>
      <td>2420.0</td>
      <td>nm0001218, nm0462735, nm0077720</td>
      <td>nm0000539</td>
      <td>nm0339704</td>
      <td>nm1274189</td>
      <td>NaN</td>
      <td>nm0524451, nm0823123, nm2999181</td>
      <td>NaN</td>
      <td>{'Budget': '$6,000,000 (estimated)'}</td>
    </tr>
    <tr>
      <th>9</th>
      <td>60</td>
      <td>tt0369610</td>
      <td>Jurassic World</td>
      <td>Jurassic World</td>
      <td>2015</td>
      <td>124.0</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>7.0</td>
      <td>539338.0</td>
      <td>nm0695435, nm1339223</td>
      <td>nm0397171, nm0339460</td>
      <td>NaN</td>
      <td>nm1119880</td>
      <td>NaN</td>
      <td>nm0189777</td>
      <td>nm0415425, nm0798646, nm2081046, nm0000341</td>
      <td>{'Budget': '$150,000,000 (estimated)', 'Openin...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>86</td>
      <td>tt0420293</td>
      <td>The Stanford Prison Experiment</td>
      <td>The Stanford Prison Experiment</td>
      <td>2015</td>
      <td>122.0</td>
      <td>Biography,Drama,History</td>
      <td>6.9</td>
      <td>32591.0</td>
      <td>nm3009232, nm4446467, nm0001082</td>
      <td>nm1880888</td>
      <td>NaN</td>
      <td>nm1547859</td>
      <td>NaN</td>
      <td>nm0295288, nm2886189, nm0256283</td>
      <td>nm0848003, nm1674354</td>
      <td>{'Opening Weekend United States': '$37,514, 19...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>107</td>
      <td>tt0437086</td>
      <td>Alita: Battle Angel</td>
      <td>Alita: Battle Angel</td>
      <td>2019</td>
      <td>122.0</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>7.5</td>
      <td>88207.0</td>
      <td>nm0910607, nm0991810</td>
      <td>nm4023073, nm0000124</td>
      <td>nm0432725</td>
      <td>nm0001675</td>
      <td>NaN</td>
      <td>nm0484457</td>
      <td>nm0000116, nm0436164, nm1738737</td>
      <td>{'Budget': '$170,000,000 (estimated)', 'Openin...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>118</td>
      <td>tt0443533</td>
      <td>The History of Love</td>
      <td>The History of Love</td>
      <td>2016</td>
      <td>134.0</td>
      <td>Drama,Romance,War</td>
      <td>6.3</td>
      <td>1024.0</td>
      <td>nm0001394, nm0001285</td>
      <td>nm2605345, nm4563869</td>
      <td>nm0023940</td>
      <td>nm0586123</td>
      <td>NaN</td>
      <td>nm2275877, nm2274042</td>
      <td>nm0738925, nm1842569</td>
      <td>{'Budget': 'EUR15,000,000 (estimated)'}</td>
    </tr>
  </tbody>
</table>
</div>




```python
full_df.reset_index(inplace=True)
full_df = full_df.drop(['level_0','index'], axis = 1)
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4501 entries, 0 to 4500
    Data columns (total 16 columns):
     #   Column           Non-Null Count  Dtype  
    ---  ------           --------------  -----  
     0   tconst           4501 non-null   object 
     1   primary_title    4501 non-null   object 
     2   original_title   4501 non-null   object 
     3   start_year       4501 non-null   int64  
     4   runtime_minutes  4501 non-null   float64
     5   genres           4499 non-null   object 
     6   averagerating    4501 non-null   float64
     7   numvotes         4501 non-null   float64
     8   actor            4101 non-null   object 
     9   actress          3676 non-null   object 
     10  composer         2611 non-null   object 
     11  director         4244 non-null   object 
     12  editor           1294 non-null   object 
     13  producer         3691 non-null   object 
     14  writer           2967 non-null   object 
     15  financials       4501 non-null   object 
    dtypes: float64(3), int64(1), object(12)
    memory usage: 562.8+ KB



```python
full_df.financials
```




    0                    {'Budget': '$6,000,000 (estimated)'}
    1       {'Budget': '$150,000,000 (estimated)', 'Openin...
    2       {'Opening Weekend United States': '$37,514, 19...
    3       {'Budget': '$170,000,000 (estimated)', 'Openin...
    4                 {'Budget': 'EUR15,000,000 (estimated)'}
                                  ...                        
    4496                 {'Budget': '$1,500,000 (estimated)'}
    4497                 {'Budget': '$3,000,000 (estimated)'}
    4498    {'Budget': 'MYR20,000,000 (estimated)', 'Cumul...
    4499               {'Budget': 'INR4,000,000 (estimated)'}
    4500              {'Budget': 'INR10,000,000 (estimated)'}
    Name: financials, Length: 4501, dtype: object




```python
# Found code online to extract multiple dict items into their own appropriately named columns:
# https://codereview.stackexchange.com/questions/93923/extracting-contents-of-dictionary-contained-in-pandas-dataframe-to-make-new-data

def unpack(df, column, fillna=None):
    ret = None
    if fillna is None:
        ret = pd.concat([df, pd.DataFrame((d for idx, d in df[column].iteritems()))], axis=1)
        del ret[column]
    else:
        ret = pd.concat([df, pd.DataFrame((d for idx, d in df[column].iteritems())).fillna(fillna)], axis=1)
        del ret[column]
    return ret
```


```python
full_df = unpack(full_df, 'financials', 0)
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4501 entries, 0 to 4500
    Data columns (total 59 columns):
     #   Column                                Non-Null Count  Dtype  
    ---  ------                                --------------  -----  
     0   tconst                                4501 non-null   object 
     1   primary_title                         4501 non-null   object 
     2   original_title                        4501 non-null   object 
     3   start_year                            4501 non-null   int64  
     4   runtime_minutes                       4501 non-null   float64
     5   genres                                4499 non-null   object 
     6   averagerating                         4501 non-null   float64
     7   numvotes                              4501 non-null   float64
     8   actor                                 4101 non-null   object 
     9   actress                               3676 non-null   object 
     10  composer                              2611 non-null   object 
     11  director                              4244 non-null   object 
     12  editor                                1294 non-null   object 
     13  producer                              3691 non-null   object 
     14  writer                                2967 non-null   object 
     15  Budget                                4501 non-null   object 
     16  Opening Weekend United States         4501 non-null   object 
     17  Cumulative Worldwide Gross            4501 non-null   object 
     18  Opening Weekend United Kingdom        4501 non-null   object 
     19  Opening Weekend Turkey                4501 non-null   object 
     20  Opening Weekend Italy                 4501 non-null   object 
     21  Opening Weekend Netherlands           4501 non-null   object 
     22  Opening Weekend Spain                 4501 non-null   object 
     23  Opening Weekend France                4501 non-null   object 
     24  Opening Weekend India                 4501 non-null   object 
     25  Opening Weekend China                 4501 non-null   object 
     26  Opening Weekend Japan                 4501 non-null   object 
     27  Opening Weekend Russia                4501 non-null   object 
     28  Opening Weekend Brazil                4501 non-null   object 
     29  Opening Weekend Pakistan              4501 non-null   object 
     30  Opening Weekend Sweden                4501 non-null   object 
     31  Opening Weekend Mexico                4501 non-null   object 
     32  Opening Weekend Germany               4501 non-null   object 
     33  Opening Weekend Greece                4501 non-null   object 
     34  Opening Weekend South Korea           4501 non-null   object 
     35  Opening Weekend Belgium               4501 non-null   object 
     36  Opening Weekend Finland               4501 non-null   object 
     37  Opening Weekend Ireland               4501 non-null   object 
     38  Opening Weekend Chile                 4501 non-null   object 
     39  Opening Weekend Egypt                 4501 non-null   object 
     40  Opening Weekend Vietnam               4501 non-null   object 
     41  Opening Weekend New Zealand           4501 non-null   object 
     42  Opening Weekend Philippines           4501 non-null   object 
     43  Opening Weekend Hungary               4501 non-null   object 
     44  Opening Weekend Hong Kong             4501 non-null   object 
     45  Opening Weekend Serbia                4501 non-null   object 
     46  Opening Weekend United Arab Emirates  4501 non-null   object 
     47  Opening Weekend Romania               4501 non-null   object 
     48  Opening Weekend Poland                4501 non-null   object 
     49  Opening Weekend Malaysia              4501 non-null   object 
     50  Opening Weekend Argentina             4501 non-null   object 
     51  Opening Weekend Iran                  4501 non-null   object 
     52  Opening Weekend South Africa          4501 non-null   object 
     53  Opening Weekend Canada                4501 non-null   object 
     54  Opening Weekend Portugal              4501 non-null   object 
     55  Opening Weekend Ukraine               4501 non-null   object 
     56  Opening Weekend Bangladesh            4501 non-null   object 
     57  Opening Weekend Indonesia             4501 non-null   object 
     58  Opening Weekend Yemen                 4501 non-null   object 
    dtypes: float64(3), int64(1), object(55)
    memory usage: 2.0+ MB



```python
# Now I will drop columns 18 to the end, so I am left with just Budget, Opening Wkdn US, and Worldwide Gross 
full_df = full_df.iloc[:,0:18]
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4501 entries, 0 to 4500
    Data columns (total 18 columns):
     #   Column                         Non-Null Count  Dtype  
    ---  ------                         --------------  -----  
     0   tconst                         4501 non-null   object 
     1   primary_title                  4501 non-null   object 
     2   original_title                 4501 non-null   object 
     3   start_year                     4501 non-null   int64  
     4   runtime_minutes                4501 non-null   float64
     5   genres                         4499 non-null   object 
     6   averagerating                  4501 non-null   float64
     7   numvotes                       4501 non-null   float64
     8   actor                          4101 non-null   object 
     9   actress                        3676 non-null   object 
     10  composer                       2611 non-null   object 
     11  director                       4244 non-null   object 
     12  editor                         1294 non-null   object 
     13  producer                       3691 non-null   object 
     14  writer                         2967 non-null   object 
     15  Budget                         4501 non-null   object 
     16  Opening Weekend United States  4501 non-null   object 
     17  Cumulative Worldwide Gross     4501 non-null   object 
    dtypes: float64(3), int64(1), object(14)
    memory usage: 633.1+ KB



```python
full_df['Cumulative Worldwide Gross'].value_counts()
```




    0                           3570
    $90,000,000                    2
    INR520,000,000                 2
    $44,328,624, 21 Aug 2017       1
    $522,236,223                   1
                                ... 
    $48,277,588                    1
    $269,920, 19 Mar 2017          1
    $125,186,461                   1
    $31,882,724, 06 Dec 2018       1
    $34,742,348, 10 Jan 2019       1
    Name: Cumulative Worldwide Gross, Length: 930, dtype: int64




```python
# My primary measure of financial success will be via the 'Cumulative Worldwide Gross' column
# so I will drop all rows where there is no measure for this value

full_df = full_df[(full_df['Cumulative Worldwide Gross'] != 0)]
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 931 entries, 1 to 4498
    Data columns (total 18 columns):
     #   Column                         Non-Null Count  Dtype  
    ---  ------                         --------------  -----  
     0   tconst                         931 non-null    object 
     1   primary_title                  931 non-null    object 
     2   original_title                 931 non-null    object 
     3   start_year                     931 non-null    int64  
     4   runtime_minutes                931 non-null    float64
     5   genres                         931 non-null    object 
     6   averagerating                  931 non-null    float64
     7   numvotes                       931 non-null    float64
     8   actor                          882 non-null    object 
     9   actress                        802 non-null    object 
     10  composer                       478 non-null    object 
     11  director                       898 non-null    object 
     12  editor                         165 non-null    object 
     13  producer                       777 non-null    object 
     14  writer                         737 non-null    object 
     15  Budget                         931 non-null    object 
     16  Opening Weekend United States  931 non-null    object 
     17  Cumulative Worldwide Gross     931 non-null    object 
    dtypes: float64(3), int64(1), object(14)
    memory usage: 138.2+ KB



```python
# rename the columns
full_df.rename(columns = {'Budget':'budget', 
                          'Opening Weekend United States':'usa_opening_wknd', 
                          'Cumulative Worldwide Gross':'global_gross'}, inplace=True)
```


```python
# It looked like some values in global_gross are not in USD, so I will try to find and drop those rows too
# There also appear to be some items with a date included, so I will need to isolate just the dollar values
full_df.global_gross.value_counts()
```




    $90,000,000                    2
    INR520,000,000                 2
    $11,692,444, 27 Jan 2017       1
    $259,344,059                   1
    $5,900,000, 10 Feb 2017        1
                                  ..
    $183,887,723                   1
    $3,500,000, 26 Jan 2018        1
    $3,450,000, 23 Sep 2018        1
    INR130,900,000, 03 Sep 2019    1
    $96,881                        1
    Name: global_gross, Length: 929, dtype: int64




```python
# splitting items with dates included
full_df['globalgross'] = full_df['global_gross'].map(lambda x: x.split(', '))
```


```python
full_df.globalgross.value_counts
```




    <bound method IndexOpsMixin.value_counts of 1       [$1,648,854,864, 04 Mar 2016]
    3                      [$404,852,543]
    5                      [$365,971,656]
    6                      [$821,763,408]
    7             [$494,123, 20 Aug 2017]
                        ...              
    4475    [PHP600,000,000, 30 Jan 2019]
    4485                    [$94,951,615]
    4488    [PHP370,000,000, 13 Mar 2019]
    4491                   [$255,863,112]
    4498     [MYR25,280,000, 27 May 2019]
    Name: globalgross, Length: 931, dtype: object>




```python
# Any items that the split was applied to were turned into 2-item lists
# I need to keep just the monetary value component of those items and drop the date

globgross = []

for i in full_df.globalgross:
        globgross.append(str(i[0]))
```


```python
len(full_df['global_gross']) == len(globgross)
```




    True




```python
# now I will apply my cleaned column values within the globgross list to my full_df dataframe

full_df['global_gross'] = globgross
```


```python
full_df = full_df.drop('globalgross', axis=1)
```


```python
full_df['global_gross'].value_counts()
```




    INR200,000,000    4
    $3,500,000        3
    INR520,000,000    3
    $1,000,000        2
    $90,000,000       2
                     ..
    CNY9,904,000      1
    $28,000,000       1
    $180,613,180      1
    $119,100,758      1
    $96,881           1
    Name: global_gross, Length: 919, dtype: int64




```python
# Using regex, I will filter for all rows that start with a non-alphanumeric character
full_df = full_df[(full_df.global_gross.str.match('(\W)')==True)]
```


```python
full_df.reset_index(inplace=True)
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 833 entries, 0 to 832
    Data columns (total 19 columns):
     #   Column            Non-Null Count  Dtype  
    ---  ------            --------------  -----  
     0   index             833 non-null    int64  
     1   tconst            833 non-null    object 
     2   primary_title     833 non-null    object 
     3   original_title    833 non-null    object 
     4   start_year        833 non-null    int64  
     5   runtime_minutes   833 non-null    float64
     6   genres            833 non-null    object 
     7   averagerating     833 non-null    float64
     8   numvotes          833 non-null    float64
     9   actor             785 non-null    object 
     10  actress           716 non-null    object 
     11  composer          396 non-null    object 
     12  director          806 non-null    object 
     13  editor            122 non-null    object 
     14  producer          694 non-null    object 
     15  writer            675 non-null    object 
     16  budget            833 non-null    object 
     17  usa_opening_wknd  833 non-null    object 
     18  global_gross      833 non-null    object 
    dtypes: float64(3), int64(2), object(14)
    memory usage: 123.8+ KB



```python
full_df.global_gross.value_counts()
```




    $3,500,000      3
    $1,000,000      2
    $90,000,000     2
    $20,000         2
    $10,200,000     1
                   ..
    $119,100,758    1
    $544,068,574    1
    $473,990,832    1
    $3,700,000      1
    $96,881         1
    Name: global_gross, Length: 828, dtype: int64




```python
full_df.to_csv('full_df_3.5_883rows')
```

## IMDbPY query: Add'l metadata

### Performing the query


```python
# Per section 3.1, I will pull the following additional metadata and rating features using IMDbPY

# 1. MOVIE RATING: IMDbPY 'main', 'certificates'
# 2. PRODUCTION STUDIO: IMDbPY 'main', 'production companies'
# 3. VOTING DEMOGRAPHICS: 'vote details', 'demographics'
```


```python
# SEE OTHER PROJECT FILE
```

### Joining & cleaning: Ratings


```python
with open(r"imdb_ratings.json", "r") as read_file:
    ratings_json = json.load(read_file)
```


```python
# Quick exploration of the file imdb_ratings.json
ratings_json[0]
```




    ['Argentina:13::(with warning)',
     'Australia:M',
     'Austria:12',
     'Brazil:12',
     'Canada:PG::(British Columbia)',
     'Canada:PG::(tv rating)',
     'Chile:TE+7',
     'Colombia:T',
     'Denmark:11',
     'Finland:K-12',
     'France:Tous publics::(with warning)',
     'Germany:12',
     'Greece:K-13',
     'Hong Kong:IIA',
     'Hungary:12',
     'Hungary:16',
     'Iceland:12',
     'India:UA',
     'Ireland:12A',
     'Ireland:12',
     'Italy:T',
     'Japan:G',
     'Lithuania:N-13',
     'Malaysia:P13',
     'Maldives:15+',
     'Mexico:B',
     'Netherlands:12',
     'New Zealand:M',
     'Norway:12',
     'Norway:12::(cinema rating)',
     'Philippines:PG-13',
     'Poland:12',
     'Portugal:M/12',
     'Russia:12+',
     'Russia:16+::(DVD rating)',
     'Singapore:PG13',
     'South Africa:10-12',
     'South Korea:12',
     'Spain:12',
     'Spain:16::(Movistar+)',
     'Sweden:11',
     'Switzerland:12',
     'Thailand:G',
     'Thailand:13::(DVD rating)',
     'United Kingdom:12A',
     'United Kingdom:12',
     'United States:TV-14::(LV)',
     'United States:PG-13',
     'Vietnam:P::(2015, self-applied)']




```python
ratings_json[0][0]
```




    'Argentina:13::(with warning)'




```python
# Code that iterates through imdb_rating, finds and extracts the USA rating for each film
# Some films have two ratings listed for the USA, so code needs to associate both of these with the same row #

countries = []
ratings = []
rownums = []

for x, i in enumerate(ratings_json):
    if i == 0:
        continue
    else:
        for item in i:
            country = str(item).split(':')[0]
            if country != "United States":
                continue
            else:
                countries.append(item.split(":")[0])
                ratings.append(item.split(":")[1])
            rownums.append(x)
```


```python
# Zipping row numbers and ratings together, so each rating is associated with the correct row in full_df
ratingdata = list(zip(rownums, ratings))
```


```python
ratingdata[:10]
```




    [(0, 'TV-14'),
     (0, 'PG-13'),
     (1, 'PG-13'),
     (2, 'PG-13'),
     (3, 'PG-13'),
     (4, 'R'),
     (5, 'PG-13'),
     (6, 'PG-13'),
     (7, 'R'),
     (8, 'PG-13')]




```python
# Remove duplicate ratings for any film that had both TV and movie ratings listed

movieratings = []
for item in ratingdata:
    if item[1].split('-')[0] == 'TV':
        continue
    else:
        movieratings.append(item)
```


```python
# Split row number and rating into their own separate lists, in prep for conversion to dataframe
row = []
rating = []

for i in movieratings:
    row.append(i[0])
    rating.append(i[1])

seriesrow = pd.Series(row)
seriesratings = pd.Series(rating)
```


```python
#convert to dictionary, for two column dataframe
ratingdict = {'rownum':row, 'movrating':rating}
```


```python
df_movieratings = pd.DataFrame(ratingdict)
```


```python
df_movieratings.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 731 entries, 0 to 730
    Data columns (total 2 columns):
     #   Column     Non-Null Count  Dtype 
    ---  ------     --------------  ----- 
     0   rownum     731 non-null    int64 
     1   movrating  731 non-null    object
    dtypes: int64(1), object(1)
    memory usage: 11.5+ KB



```python
df_movieratings.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rownum</th>
      <th>movrating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>726</th>
      <td>823</td>
      <td>R</td>
    </tr>
    <tr>
      <th>727</th>
      <td>825</td>
      <td>G</td>
    </tr>
    <tr>
      <th>728</th>
      <td>826</td>
      <td>Not Rated</td>
    </tr>
    <tr>
      <th>729</th>
      <td>828</td>
      <td>PG</td>
    </tr>
    <tr>
      <th>730</th>
      <td>829</td>
      <td>Unrated</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_movieratings['rownum'].duplicated().sum()
```




    11




```python
len(df_movieratings[~df_movieratings['rownum'].duplicated()])
```




    720




```python
df_movieratings = df_movieratings[~df_movieratings['rownum'].duplicated()]
```


```python
df_movieratings['rownum'].value_counts()
```




    829    1
    251    1
    249    1
    248    1
    247    1
          ..
    508    1
    507    1
    506    1
    505    1
    0      1
    Name: rownum, Length: 720, dtype: int64




```python
full_df.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>actor</th>
      <th>actress</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>writer</th>
      <th>budget</th>
      <th>usa_opening_wknd</th>
      <th>global_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>828</th>
      <td>4441</td>
      <td>tt8969332</td>
      <td>The Biggest Little Farm</td>
      <td>The Biggest Little Farm</td>
      <td>2018</td>
      <td>91.0</td>
      <td>Documentary</td>
      <td>8.0</td>
      <td>421.0</td>
      <td>nm10726590</td>
      <td>NaN</td>
      <td>nm0063618</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm3537480</td>
      <td>nm0598531</td>
      <td>0</td>
      <td>$110,492, 12 May 2019</td>
      <td>$5,293,304</td>
    </tr>
    <tr>
      <th>829</th>
      <td>4442</td>
      <td>tt8991268</td>
      <td>Honeyland</td>
      <td>Honeyland</td>
      <td>2019</td>
      <td>87.0</td>
      <td>Documentary</td>
      <td>8.5</td>
      <td>158.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm3334929</td>
      <td>nm9393813, nm10118100</td>
      <td>NaN</td>
      <td>nm1389493</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>$1,140,879</td>
    </tr>
    <tr>
      <th>830</th>
      <td>4456</td>
      <td>tt9081562</td>
      <td>More Than Blue</td>
      <td>Bi bei shang geng bei shang de gu shi</td>
      <td>2018</td>
      <td>105.0</td>
      <td>Romance</td>
      <td>5.6</td>
      <td>566.0</td>
      <td>nm6630432, nm3214105</td>
      <td>nm2442121, nm5397122</td>
      <td>nm3871161, nm3928207</td>
      <td>nm4341096</td>
      <td>NaN</td>
      <td>nm7599343</td>
      <td>nm7250425</td>
      <td>0</td>
      <td>0</td>
      <td>$153,000,000</td>
    </tr>
    <tr>
      <th>831</th>
      <td>4485</td>
      <td>tt9408490</td>
      <td>Kill Mobile</td>
      <td>Shoujikuang xiang</td>
      <td>2018</td>
      <td>90.0</td>
      <td>Drama</td>
      <td>5.7</td>
      <td>198.0</td>
      <td>nm1291827, nm4596259</td>
      <td>nm4454684, nm2356251</td>
      <td>NaN</td>
      <td>nm6785661</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm2601560, nm3798121, nm0182499, nm0002645, nm...</td>
      <td>0</td>
      <td>0</td>
      <td>$94,951,615</td>
    </tr>
    <tr>
      <th>832</th>
      <td>4491</td>
      <td>tt9597190</td>
      <td>Pegasus</td>
      <td>Fei chi ren sheng</td>
      <td>2019</td>
      <td>98.0</td>
      <td>Comedy,Sport</td>
      <td>6.4</td>
      <td>817.0</td>
      <td>nm7613067, nm8361677, nm7613069, nm8288748</td>
      <td>NaN</td>
      <td>nm0150989</td>
      <td>nm3954274</td>
      <td>NaN</td>
      <td>nm10464217, nm7815808, nm7815809, nm10464218</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>$255,863,112</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Replacing index column with a new index column from 0 to 832, in order to join in the ratings data
full_df = full_df.drop('index', axis=1)
full_df = full_df.reset_index()
```


```python
full_df.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>actor</th>
      <th>actress</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>writer</th>
      <th>budget</th>
      <th>usa_opening_wknd</th>
      <th>global_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>828</th>
      <td>828</td>
      <td>tt8969332</td>
      <td>The Biggest Little Farm</td>
      <td>The Biggest Little Farm</td>
      <td>2018</td>
      <td>91.0</td>
      <td>Documentary</td>
      <td>8.0</td>
      <td>421.0</td>
      <td>nm10726590</td>
      <td>NaN</td>
      <td>nm0063618</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm3537480</td>
      <td>nm0598531</td>
      <td>0</td>
      <td>$110,492, 12 May 2019</td>
      <td>$5,293,304</td>
    </tr>
    <tr>
      <th>829</th>
      <td>829</td>
      <td>tt8991268</td>
      <td>Honeyland</td>
      <td>Honeyland</td>
      <td>2019</td>
      <td>87.0</td>
      <td>Documentary</td>
      <td>8.5</td>
      <td>158.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm3334929</td>
      <td>nm9393813, nm10118100</td>
      <td>NaN</td>
      <td>nm1389493</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>$1,140,879</td>
    </tr>
    <tr>
      <th>830</th>
      <td>830</td>
      <td>tt9081562</td>
      <td>More Than Blue</td>
      <td>Bi bei shang geng bei shang de gu shi</td>
      <td>2018</td>
      <td>105.0</td>
      <td>Romance</td>
      <td>5.6</td>
      <td>566.0</td>
      <td>nm6630432, nm3214105</td>
      <td>nm2442121, nm5397122</td>
      <td>nm3871161, nm3928207</td>
      <td>nm4341096</td>
      <td>NaN</td>
      <td>nm7599343</td>
      <td>nm7250425</td>
      <td>0</td>
      <td>0</td>
      <td>$153,000,000</td>
    </tr>
    <tr>
      <th>831</th>
      <td>831</td>
      <td>tt9408490</td>
      <td>Kill Mobile</td>
      <td>Shoujikuang xiang</td>
      <td>2018</td>
      <td>90.0</td>
      <td>Drama</td>
      <td>5.7</td>
      <td>198.0</td>
      <td>nm1291827, nm4596259</td>
      <td>nm4454684, nm2356251</td>
      <td>NaN</td>
      <td>nm6785661</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm2601560, nm3798121, nm0182499, nm0002645, nm...</td>
      <td>0</td>
      <td>0</td>
      <td>$94,951,615</td>
    </tr>
    <tr>
      <th>832</th>
      <td>832</td>
      <td>tt9597190</td>
      <td>Pegasus</td>
      <td>Fei chi ren sheng</td>
      <td>2019</td>
      <td>98.0</td>
      <td>Comedy,Sport</td>
      <td>6.4</td>
      <td>817.0</td>
      <td>nm7613067, nm8361677, nm7613069, nm8288748</td>
      <td>NaN</td>
      <td>nm0150989</td>
      <td>nm3954274</td>
      <td>NaN</td>
      <td>nm10464217, nm7815808, nm7815809, nm10464218</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>$255,863,112</td>
    </tr>
  </tbody>
</table>
</div>




```python
full_df = full_df.merge(df_movieratings, left_on='index', right_on='rownum', how='left')
```


```python
full_df.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>actor</th>
      <th>...</th>
      <th>composer</th>
      <th>director</th>
      <th>editor</th>
      <th>producer</th>
      <th>writer</th>
      <th>budget</th>
      <th>usa_opening_wknd</th>
      <th>global_gross</th>
      <th>rownum</th>
      <th>movrating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>828</th>
      <td>828</td>
      <td>tt8969332</td>
      <td>The Biggest Little Farm</td>
      <td>The Biggest Little Farm</td>
      <td>2018</td>
      <td>91.0</td>
      <td>Documentary</td>
      <td>8.0</td>
      <td>421.0</td>
      <td>nm10726590</td>
      <td>...</td>
      <td>nm0063618</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm3537480</td>
      <td>nm0598531</td>
      <td>0</td>
      <td>$110,492, 12 May 2019</td>
      <td>$5,293,304</td>
      <td>828.0</td>
      <td>PG</td>
    </tr>
    <tr>
      <th>829</th>
      <td>829</td>
      <td>tt8991268</td>
      <td>Honeyland</td>
      <td>Honeyland</td>
      <td>2019</td>
      <td>87.0</td>
      <td>Documentary</td>
      <td>8.5</td>
      <td>158.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>nm3334929</td>
      <td>nm9393813, nm10118100</td>
      <td>NaN</td>
      <td>nm1389493</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>$1,140,879</td>
      <td>829.0</td>
      <td>Unrated</td>
    </tr>
    <tr>
      <th>830</th>
      <td>830</td>
      <td>tt9081562</td>
      <td>More Than Blue</td>
      <td>Bi bei shang geng bei shang de gu shi</td>
      <td>2018</td>
      <td>105.0</td>
      <td>Romance</td>
      <td>5.6</td>
      <td>566.0</td>
      <td>nm6630432, nm3214105</td>
      <td>...</td>
      <td>nm3871161, nm3928207</td>
      <td>nm4341096</td>
      <td>NaN</td>
      <td>nm7599343</td>
      <td>nm7250425</td>
      <td>0</td>
      <td>0</td>
      <td>$153,000,000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>831</th>
      <td>831</td>
      <td>tt9408490</td>
      <td>Kill Mobile</td>
      <td>Shoujikuang xiang</td>
      <td>2018</td>
      <td>90.0</td>
      <td>Drama</td>
      <td>5.7</td>
      <td>198.0</td>
      <td>nm1291827, nm4596259</td>
      <td>...</td>
      <td>NaN</td>
      <td>nm6785661</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>nm2601560, nm3798121, nm0182499, nm0002645, nm...</td>
      <td>0</td>
      <td>0</td>
      <td>$94,951,615</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>832</th>
      <td>832</td>
      <td>tt9597190</td>
      <td>Pegasus</td>
      <td>Fei chi ren sheng</td>
      <td>2019</td>
      <td>98.0</td>
      <td>Comedy,Sport</td>
      <td>6.4</td>
      <td>817.0</td>
      <td>nm7613067, nm8361677, nm7613069, nm8288748</td>
      <td>...</td>
      <td>nm0150989</td>
      <td>nm3954274</td>
      <td>NaN</td>
      <td>nm10464217, nm7815808, nm7815809, nm10464218</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>$255,863,112</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
full_df = full_df.drop('rownum', axis=1)
```


```python
full_df.to_csv('full_df_ratings.csv')
```

### Joining & cleaning: Voter demographics


```python
with open(r"imdb_voterdemos.json", "r") as read_file:
    voters_json = json.load(read_file)
```


```python
len(voters_json)
```




    833




```python
pd.DataFrame.from_records(voters_json[0])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>aged 18 29</th>
      <th>aged 30 44</th>
      <th>aged 45 plus</th>
      <th>aged under 18</th>
      <th>females</th>
      <th>females aged 18 29</th>
      <th>females aged 30 44</th>
      <th>females aged 45 plus</th>
      <th>females aged under 18</th>
      <th>imdb users</th>
      <th>males</th>
      <th>males aged 18 29</th>
      <th>males aged 30 44</th>
      <th>males aged 45 plus</th>
      <th>males aged under 18</th>
      <th>non us users</th>
      <th>top 1000 voters</th>
      <th>us users</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>votes</th>
      <td>132311.0</td>
      <td>205346.0</td>
      <td>42891.0</td>
      <td>576.0</td>
      <td>73321.0</td>
      <td>27500.0</td>
      <td>31940.0</td>
      <td>6320.0</td>
      <td>84.0</td>
      <td>567540.0</td>
      <td>339750.0</td>
      <td>102321.0</td>
      <td>170164.0</td>
      <td>35794.0</td>
      <td>424.0</td>
      <td>216868.0</td>
      <td>760.0</td>
      <td>72690.0</td>
    </tr>
    <tr>
      <th>rating</th>
      <td>7.1</td>
      <td>6.9</td>
      <td>6.9</td>
      <td>7.0</td>
      <td>7.3</td>
      <td>7.4</td>
      <td>7.2</td>
      <td>7.3</td>
      <td>7.4</td>
      <td>7.0</td>
      <td>6.9</td>
      <td>7.0</td>
      <td>6.8</td>
      <td>6.8</td>
      <td>6.9</td>
      <td>6.8</td>
      <td>6.8</td>
      <td>7.1</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_voters = pd.DataFrame(voters_json)[0]
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/indexes/base.py in get_loc(self, key, method, tolerance)
       2645             try:
    -> 2646                 return self._engine.get_loc(key)
       2647             except KeyError:


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    KeyError: 0

    
    During handling of the above exception, another exception occurred:


    KeyError                                  Traceback (most recent call last)

    <ipython-input-7-68a65b8b7437> in <module>
    ----> 1 df_voters = pd.DataFrame(voters_json)[0]
    

    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/frame.py in __getitem__(self, key)
       2798             if self.columns.nlevels > 1:
       2799                 return self._getitem_multilevel(key)
    -> 2800             indexer = self.columns.get_loc(key)
       2801             if is_integer(indexer):
       2802                 indexer = [indexer]


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/indexes/base.py in get_loc(self, key, method, tolerance)
       2646                 return self._engine.get_loc(key)
       2647             except KeyError:
    -> 2648                 return self._engine.get_loc(self._maybe_cast_indexer(key))
       2649         indexer = self.get_indexer([key], method=method, tolerance=tolerance)
       2650         if indexer.ndim > 1 or indexer.size > 1:


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    KeyError: 0



```python
df_voters.head()
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-4-f3302ee162b3> in <module>
    ----> 1 df_voters.head()
    

    NameError: name 'df_voters' is not defined



```python
# GO OVER THIS CELL IN THE MOD REVIEW!!  I wanted to do this programatically

columns = list(df_voters)

#df_demovotes = pd.DataFrame()
#df_demoratings = pd.DataFrame()

votes = []
rating = []

for i in columns:
    for row in df_voters['"'+ i + '"']:
        v = row.keys()
        r = row.values()
        votes.append(v)
        rating.append(r)
    df_demovotes[('v ' + i)] = votes
    df_demoratings[('r ' + i)] = rating
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/indexes/base.py in get_loc(self, key, method, tolerance)
       2645             try:
    -> 2646                 return self._engine.get_loc(key)
       2647             except KeyError:


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    KeyError: '"imdb users"'

    
    During handling of the above exception, another exception occurred:


    KeyError                                  Traceback (most recent call last)

    <ipython-input-166-ad6b1c09f3ac> in <module>
         10 
         11 for i in columns:
    ---> 12     for row in df_voters['"'+ i + '"']:
         13         v = row.keys()
         14         r = row.values()


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/frame.py in __getitem__(self, key)
       2798             if self.columns.nlevels > 1:
       2799                 return self._getitem_multilevel(key)
    -> 2800             indexer = self.columns.get_loc(key)
       2801             if is_integer(indexer):
       2802                 indexer = [indexer]


    ~/opt/anaconda3/envs/learn-env/lib/python3.6/site-packages/pandas/core/indexes/base.py in get_loc(self, key, method, tolerance)
       2646                 return self._engine.get_loc(key)
       2647             except KeyError:
    -> 2648                 return self._engine.get_loc(self._maybe_cast_indexer(key))
       2649         indexer = self.get_indexer([key], method=method, tolerance=tolerance)
       2650         if indexer.ndim > 1 or indexer.size > 1:


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.get_item()


    KeyError: '"imdb users"'



```python
# test extraction with one column of df_voters to set up DataFrame for collecting the rest of the results

votes = []
ratings = []

for i in df_voters['males']:
    votes.append(i.get('votes'))
    ratings.append(i.get('rating'))
```


```python
# set up dataframes to hold the vote counts and average ratings, respectively

df_demovotes = pd.DataFrame(data = votes, columns = ['v males'])
df_demoratings = pd.DataFrame(data = ratings, columns = ['r males'])
```


```python
df_demovotes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>v males</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>339750</td>
    </tr>
    <tr>
      <th>1</th>
      <td>132554</td>
    </tr>
    <tr>
      <th>2</th>
      <td>139504</td>
    </tr>
    <tr>
      <th>3</th>
      <td>299525</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5792</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_demoratings.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>r males</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6.9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7.3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
print(len(df_demovotes))
print(len(df_demoratings))
```

    833
    833



```python
# define a function that extracts the contents of a column when the column name is input as a parameter

def extract(column):
    
    votes = []
    rating = []
    
    for row in df_voters[column]:
        v = row.get('votes')
        r = row.get('rating')
        votes.append(v)
        rating.append(r)
    
    df_demovotes[('v ' + column)] = votes
    df_demoratings[('r ' + column)] = rating
```


```python
extract('females')
```


```python
df_demovotes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>v males</th>
      <th>v females</th>
      <th>v imdb users</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>339750</td>
      <td>73321</td>
      <td>567540</td>
    </tr>
    <tr>
      <th>1</th>
      <td>132554</td>
      <td>17230</td>
      <td>214649</td>
    </tr>
    <tr>
      <th>2</th>
      <td>139504</td>
      <td>21572</td>
      <td>237700</td>
    </tr>
    <tr>
      <th>3</th>
      <td>299525</td>
      <td>72851</td>
      <td>532866</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5792</td>
      <td>2318</td>
      <td>10721</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_demoratings.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>r males</th>
      <th>r females</th>
      <th>r imdb users</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6.9</td>
      <td>7.3</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7.3</td>
      <td>7.3</td>
      <td>7.3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.0</td>
      <td>7.2</td>
      <td>7.1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7.3</td>
      <td>7.8</td>
      <td>7.4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6.3</td>
      <td>6.4</td>
      <td>6.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
#columns = list(df_voters)

#for i in columns:
#    extract(i)
```


```python
columns = list(df_voters)

for i in columns:
    print('extract("' + i + '")')
```

    extract("imdb users")
    extract("aged under 18")
    extract("aged 18 29")
    extract("aged 30 44")
    extract("aged 45 plus")
    extract("males")
    extract("males aged under 18")
    extract("males aged 18 29")
    extract("males aged 30 44")
    extract("males aged 45 plus")
    extract("females")
    extract("females aged under 18")
    extract("females aged 18 29")
    extract("females aged 30 44")
    extract("females aged 45 plus")
    extract("top 1000 voters")
    extract("us users")
    extract("non us users")



```python
df_voters['aged under 18'][0].get('votes')
```




    576




```python
extract("imdb users")
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-169-160748ad0749> in <module>
          1 extract("imdb users")
    ----> 2 extract("aged under 18")
          3 extract("aged 18 29")
          4 extract("aged 30 44")
          5 extract("aged 45 plus")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("aged under 18")
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-173-af70d562111c> in <module>
    ----> 1 extract("aged under 18")
    

    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("aged 18 29")
extract("aged 30 44")
extract("aged 45 plus")
extract("males")
extract("males aged under 18")
extract("males aged 18 29")
extract("males aged 30 44")
extract("males aged 45 plus")
extract("females")
extract("females aged under 18")
extract("females aged 18 29")
extract("females aged 30 44")
extract("females aged 45 plus")
extract("top 1000 voters")
extract("us users")
extract("non us users")
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-174-38c6f3f88958> in <module>
          1 extract("aged 18 29")
          2 extract("aged 30 44")
    ----> 3 extract("aged 45 plus")
          4 extract("males")
          5 extract("males aged under 18")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("males")
extract("males aged under 18")

```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-175-964d501159d8> in <module>
          1 extract("males")
    ----> 2 extract("males aged under 18")
          3 extract("males aged 18 29")
          4 extract("males aged 30 44")
          5 extract("males aged 45 plus")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("males aged 18 29")
extract("males aged 30 44")
extract("males aged 45 plus")

```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-176-46d2e57802f9> in <module>
          1 extract("males aged 18 29")
          2 extract("males aged 30 44")
    ----> 3 extract("males aged 45 plus")
          4 extract("females")
          5 extract("females aged under 18")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("females")
extract("females aged under 18")

```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-177-9504fcc1d3db> in <module>
          1 extract("females")
    ----> 2 extract("females aged under 18")
          3 extract("females aged 18 29")
          4 extract("females aged 30 44")
          5 extract("females aged 45 plus")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("females aged 18 29")
extract("females aged 30 44")

```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-178-6ba207b63ce1> in <module>
          1 extract("females aged 18 29")
    ----> 2 extract("females aged 30 44")
          3 extract("females aged 45 plus")
          4 extract("top 1000 voters")
          5 extract("us users")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("females aged 45 plus")

```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-179-d4770fa5aadc> in <module>
    ----> 1 extract("females aged 45 plus")
          2 extract("top 1000 voters")
          3 extract("us users")
          4 extract("non us users")


    <ipython-input-156-a36474d5bcf8> in extract(column)
          5 
          6     for row in df_voters[column]:
    ----> 7         v = row.get('votes')
          8         r = row.get('rating')
          9         votes.append(v)


    AttributeError: 'float' object has no attribute 'get'



```python
extract("top 1000 voters")
extract("us users")
extract("non us users")
```


```python
df_demovotes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>v males</th>
      <th>v females</th>
      <th>v imdb users</th>
      <th>v aged 18 29</th>
      <th>v aged 30 44</th>
      <th>v males aged 18 29</th>
      <th>v males aged 30 44</th>
      <th>v females aged 18 29</th>
      <th>v top 1000 voters</th>
      <th>v us users</th>
      <th>v non us users</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>339750</td>
      <td>73321</td>
      <td>567540</td>
      <td>132311</td>
      <td>205346</td>
      <td>102321</td>
      <td>170164</td>
      <td>27500</td>
      <td>760</td>
      <td>72690</td>
      <td>216868</td>
    </tr>
    <tr>
      <th>1</th>
      <td>132554</td>
      <td>17230</td>
      <td>214649</td>
      <td>41605</td>
      <td>75292</td>
      <td>33366</td>
      <td>65612</td>
      <td>5677</td>
      <td>437</td>
      <td>22435</td>
      <td>82891</td>
    </tr>
    <tr>
      <th>2</th>
      <td>139504</td>
      <td>21572</td>
      <td>237700</td>
      <td>55420</td>
      <td>74585</td>
      <td>44182</td>
      <td>63708</td>
      <td>7895</td>
      <td>520</td>
      <td>28026</td>
      <td>84745</td>
    </tr>
    <tr>
      <th>3</th>
      <td>299525</td>
      <td>72851</td>
      <td>532866</td>
      <td>122368</td>
      <td>168568</td>
      <td>91663</td>
      <td>136770</td>
      <td>27532</td>
      <td>724</td>
      <td>67417</td>
      <td>189546</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5792</td>
      <td>2318</td>
      <td>10721</td>
      <td>1997</td>
      <td>3934</td>
      <td>1253</td>
      <td>2783</td>
      <td>714</td>
      <td>119</td>
      <td>1401</td>
      <td>4733</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_demoratings.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>r males</th>
      <th>r females</th>
      <th>r imdb users</th>
      <th>r aged 18 29</th>
      <th>r aged 30 44</th>
      <th>r males aged 18 29</th>
      <th>r males aged 30 44</th>
      <th>r females aged 18 29</th>
      <th>r top 1000 voters</th>
      <th>r us users</th>
      <th>r non us users</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6.9</td>
      <td>7.3</td>
      <td>7.0</td>
      <td>7.1</td>
      <td>6.9</td>
      <td>7.0</td>
      <td>6.8</td>
      <td>7.4</td>
      <td>6.8</td>
      <td>7.1</td>
      <td>6.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7.3</td>
      <td>7.3</td>
      <td>7.3</td>
      <td>7.2</td>
      <td>7.3</td>
      <td>7.2</td>
      <td>7.3</td>
      <td>7.2</td>
      <td>6.8</td>
      <td>7.3</td>
      <td>7.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.0</td>
      <td>7.2</td>
      <td>7.1</td>
      <td>7.2</td>
      <td>6.9</td>
      <td>7.1</td>
      <td>6.9</td>
      <td>7.3</td>
      <td>6.8</td>
      <td>7.3</td>
      <td>6.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7.3</td>
      <td>7.8</td>
      <td>7.4</td>
      <td>7.5</td>
      <td>7.3</td>
      <td>7.5</td>
      <td>7.2</td>
      <td>7.9</td>
      <td>7.0</td>
      <td>7.7</td>
      <td>7.2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6.3</td>
      <td>6.4</td>
      <td>6.3</td>
      <td>6.3</td>
      <td>6.3</td>
      <td>6.2</td>
      <td>6.2</td>
      <td>6.5</td>
      <td>5.0</td>
      <td>6.3</td>
      <td>6.2</td>
    </tr>
  </tbody>
</table>
</div>




```python
print(len(df_demoratings))
print(len(df_demovotes))
```

    833
    833



```python
print(len(full_df))
```

    833



```python
full_df = full_df.join(df_demoratings)
```


```python
full_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 833 entries, 0 to 832
    Data columns (total 31 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   index                 833 non-null    int64  
     1   tconst                833 non-null    object 
     2   primary_title         833 non-null    object 
     3   original_title        833 non-null    object 
     4   start_year            833 non-null    int64  
     5   runtime_minutes       833 non-null    float64
     6   genres                833 non-null    object 
     7   averagerating         833 non-null    float64
     8   numvotes              833 non-null    float64
     9   actor                 785 non-null    object 
     10  actress               716 non-null    object 
     11  composer              396 non-null    object 
     12  director              806 non-null    object 
     13  editor                122 non-null    object 
     14  producer              694 non-null    object 
     15  writer                675 non-null    object 
     16  budget                833 non-null    object 
     17  usa_opening_wknd      833 non-null    object 
     18  global_gross          833 non-null    object 
     19  movrating             720 non-null    object 
     20  r males               833 non-null    float64
     21  r females             833 non-null    float64
     22  r imdb users          833 non-null    float64
     23  r aged 18 29          833 non-null    float64
     24  r aged 30 44          833 non-null    float64
     25  r males aged 18 29    833 non-null    float64
     26  r males aged 30 44    833 non-null    float64
     27  r females aged 18 29  833 non-null    float64
     28  r top 1000 voters     833 non-null    float64
     29  r us users            833 non-null    float64
     30  r non us users        833 non-null    float64
    dtypes: float64(14), int64(2), object(15)
    memory usage: 248.2+ KB



```python
full_df.to_csv('full_df_backup.csv')
```

## Final review and cleaning


```python
# drop rows that dont contain a value in column 19: movrating
# drop the index, averagerating, and numvotes columns (index unneeded post-joins, avgrating/numvotes are dups)

final_df = full_df.drop(['index', 'averagerating', 'numvotes'], axis = 1)

final_df = final_df[final_df.movrating.notnull()]
```


```python
final_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 720 entries, 0 to 829
    Data columns (total 28 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   tconst                720 non-null    object 
     1   primary_title         720 non-null    object 
     2   original_title        720 non-null    object 
     3   start_year            720 non-null    int64  
     4   runtime_minutes       720 non-null    float64
     5   genres                720 non-null    object 
     6   actor                 680 non-null    object 
     7   actress               619 non-null    object 
     8   composer              324 non-null    object 
     9   director              701 non-null    object 
     10  editor                91 non-null     object 
     11  producer              612 non-null    object 
     12  writer                599 non-null    object 
     13  budget                720 non-null    object 
     14  usa_opening_wknd      720 non-null    object 
     15  global_gross          720 non-null    object 
     16  movrating             720 non-null    object 
     17  r males               720 non-null    float64
     18  r females             720 non-null    float64
     19  r imdb users          720 non-null    float64
     20  r aged 18 29          720 non-null    float64
     21  r aged 30 44          720 non-null    float64
     22  r males aged 18 29    720 non-null    float64
     23  r males aged 30 44    720 non-null    float64
     24  r females aged 18 29  720 non-null    float64
     25  r top 1000 voters     720 non-null    float64
     26  r us users            720 non-null    float64
     27  r non us users        720 non-null    float64
    dtypes: float64(12), int64(1), object(15)
    memory usage: 163.1+ KB



```python
# remove dollar signs, commas and other text from budget, usa_opening_wknd, and global_gross columns
# convert budget, usa_opening_wknd, and global_gross columns to float
final_df.global_gross.value_counts()
```




    $3,500,000      2
    $90,000,000     2
    $23,000,000     1
    $106,380,000    1
    $387,663,547    1
                   ..
    $11,430,025     1
    $773,512,274    1
    $65,146,020     1
    $15,871,398     1
    $101,134,059    1
    Name: global_gross, Length: 718, dtype: int64




```python
final_df[['budget', 'usa_opening_wknd', 'global_gross']] = final_df[['budget', 'usa_opening_wknd', 'global_gross']].astype('str')
```


```python
final_df['budg'] = final_df['budget'].map(lambda x: x.replace(',',''))
final_df['usa_open'] = final_df['usa_opening_wknd'].map(lambda x: x.replace(',',''))
final_df['glogross'] = final_df['global_gross'].map(lambda x: x.replace(',',''))
```


```python
final_df[['budg', 'usa_open', 'glogross']].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>budg</th>
      <th>usa_open</th>
      <th>glogross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>$150000000 (estimated)</td>
      <td>$208806270 12 Jun 2015</td>
      <td>$1648854864</td>
    </tr>
    <tr>
      <th>1</th>
      <td>$170000000 (estimated)</td>
      <td>$28525613 17 Feb 2019</td>
      <td>$404852543</td>
    </tr>
    <tr>
      <th>2</th>
      <td>$100000000 (estimated)</td>
      <td>$53505326 07 Apr 2019</td>
      <td>$365971656</td>
    </tr>
    <tr>
      <th>3</th>
      <td>$149000000 (estimated)</td>
      <td>$103251471 04 Jun 2017</td>
      <td>$821763408</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>$54458 13 Aug 2017</td>
      <td>$494123</td>
    </tr>
  </tbody>
</table>
</div>




```python
final_df['budget'] = final_df['budg'].map(lambda x: x.replace('$',''))
final_df['usa_opening_wknd'] = final_df['usa_open'].map(lambda x: x.replace('$',''))
final_df['global_gross'] = final_df['glogross'].map(lambda x: x.replace('$',''))
```


```python
final_df[['budget', 'usa_opening_wknd', 'global_gross']].head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>budget</th>
      <th>usa_opening_wknd</th>
      <th>global_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>150000000 (estimated)</td>
      <td>208806270 12 Jun 2015</td>
      <td>1648854864</td>
    </tr>
    <tr>
      <th>1</th>
      <td>170000000 (estimated)</td>
      <td>28525613 17 Feb 2019</td>
      <td>404852543</td>
    </tr>
    <tr>
      <th>2</th>
      <td>100000000 (estimated)</td>
      <td>53505326 07 Apr 2019</td>
      <td>365971656</td>
    </tr>
    <tr>
      <th>3</th>
      <td>149000000 (estimated)</td>
      <td>103251471 04 Jun 2017</td>
      <td>821763408</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>54458 13 Aug 2017</td>
      <td>494123</td>
    </tr>
  </tbody>
</table>
</div>




```python
final_df['budg'] = final_df['budget'].map(lambda x: x.replace(' (estimated)',''))
```


```python
final_df['budg'].value_counts()
```




    0                132
    20000000          27
    30000000          22
    40000000          21
    35000000          18
                    ... 
    14800000           1
    INR200000000       1
    155000000          1
    129000000          1
    INR1800000000      1
    Name: budg, Length: 199, dtype: int64




```python
# dropping rows where budgets aren't in US dollars -- just to be safe re: global gross values
final_df = final_df[(final_df.budg.str.match('(\d)')==True)]
```


```python
final_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 683 entries, 0 to 829
    Data columns (total 31 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   tconst                683 non-null    object 
     1   primary_title         683 non-null    object 
     2   original_title        683 non-null    object 
     3   start_year            683 non-null    int64  
     4   runtime_minutes       683 non-null    float64
     5   genres                683 non-null    object 
     6   actor                 646 non-null    object 
     7   actress               587 non-null    object 
     8   composer              303 non-null    object 
     9   director              664 non-null    object 
     10  editor                84 non-null     object 
     11  producer              577 non-null    object 
     12  writer                571 non-null    object 
     13  budget                683 non-null    object 
     14  usa_opening_wknd      683 non-null    object 
     15  global_gross          683 non-null    object 
     16  movrating             683 non-null    object 
     17  r males               683 non-null    float64
     18  r females             683 non-null    float64
     19  r imdb users          683 non-null    float64
     20  r aged 18 29          683 non-null    float64
     21  r aged 30 44          683 non-null    float64
     22  r males aged 18 29    683 non-null    float64
     23  r males aged 30 44    683 non-null    float64
     24  r females aged 18 29  683 non-null    float64
     25  r top 1000 voters     683 non-null    float64
     26  r us users            683 non-null    float64
     27  r non us users        683 non-null    float64
     28  budg                  683 non-null    object 
     29  usa_open              683 non-null    object 
     30  glogross              683 non-null    object 
    dtypes: float64(12), int64(1), object(18)
    memory usage: 170.8+ KB



```python
final_df = final_df.drop('budget', axis=1)
final_df.rename(columns={'budg':'budget'}, inplace=True)
```


```python
# remove date info from usa opening weekend column
final_df['usa_open'] = final_df['usa_opening_wknd'].map(lambda x: x[:-12])
```


```python
final_df['usa_open'].value_counts()
```




                297
    5988370       1
    9740064       1
    10845330      1
    26608020      1
               ... 
    33600000      1
    9497665       1
    46607250      1
    11756244      1
    9445456       1
    Name: usa_open, Length: 387, dtype: int64




```python
final_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 683 entries, 0 to 829
    Data columns (total 30 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   tconst                683 non-null    object 
     1   primary_title         683 non-null    object 
     2   original_title        683 non-null    object 
     3   start_year            683 non-null    int64  
     4   runtime_minutes       683 non-null    float64
     5   genres                683 non-null    object 
     6   actor                 646 non-null    object 
     7   actress               587 non-null    object 
     8   composer              303 non-null    object 
     9   director              664 non-null    object 
     10  editor                84 non-null     object 
     11  producer              577 non-null    object 
     12  writer                571 non-null    object 
     13  usa_opening_wknd      683 non-null    object 
     14  global_gross          683 non-null    object 
     15  movrating             683 non-null    object 
     16  r males               683 non-null    float64
     17  r females             683 non-null    float64
     18  r imdb users          683 non-null    float64
     19  r aged 18 29          683 non-null    float64
     20  r aged 30 44          683 non-null    float64
     21  r males aged 18 29    683 non-null    float64
     22  r males aged 30 44    683 non-null    float64
     23  r females aged 18 29  683 non-null    float64
     24  r top 1000 voters     683 non-null    float64
     25  r us users            683 non-null    float64
     26  r non us users        683 non-null    float64
     27  budget                683 non-null    object 
     28  usa_open              683 non-null    object 
     29  glogross              683 non-null    object 
    dtypes: float64(12), int64(1), object(17)
    memory usage: 165.4+ KB



```python
final_df = final_df.drop('usa_opening_wknd', axis=1)
final_df.rename(columns={'usa_open':'usa_opening_wknd'}, inplace=True)
```


```python
final_df['glogross'] = final_df['global_gross'].map(lambda x: x)
final_df = final_df.drop('global_gross', axis=1)
final_df.rename(columns={'glogross':'global_gross'}, inplace=True)
```


```python
final_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 683 entries, 0 to 829
    Data columns (total 28 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   tconst                683 non-null    object 
     1   primary_title         683 non-null    object 
     2   original_title        683 non-null    object 
     3   start_year            683 non-null    int64  
     4   runtime_minutes       683 non-null    float64
     5   genres                683 non-null    object 
     6   actor                 646 non-null    object 
     7   actress               587 non-null    object 
     8   composer              303 non-null    object 
     9   director              664 non-null    object 
     10  editor                84 non-null     object 
     11  producer              577 non-null    object 
     12  writer                571 non-null    object 
     13  movrating             683 non-null    object 
     14  r males               683 non-null    float64
     15  r females             683 non-null    float64
     16  r imdb users          683 non-null    float64
     17  r aged 18 29          683 non-null    float64
     18  r aged 30 44          683 non-null    float64
     19  r males aged 18 29    683 non-null    float64
     20  r males aged 30 44    683 non-null    float64
     21  r females aged 18 29  683 non-null    float64
     22  r top 1000 voters     683 non-null    float64
     23  r us users            683 non-null    float64
     24  r non us users        683 non-null    float64
     25  budget                683 non-null    object 
     26  usa_opening_wknd      683 non-null    object 
     27  global_gross          683 non-null    object 
    dtypes: float64(12), int64(1), object(15)
    memory usage: 154.7+ KB



```python
final_df['global_gross'] = final_df['global_gross'].astype('int64')
```


```python
final_df['budget'] = final_df['budget'].astype('int64')
```


```python
#skipping over usa_opening_wknd for now
final_df = final_df.drop('usa_opening_wknd', axis=1)
```


```python
final_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 683 entries, 0 to 829
    Data columns (total 27 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   tconst                683 non-null    object 
     1   primary_title         683 non-null    object 
     2   original_title        683 non-null    object 
     3   start_year            683 non-null    int64  
     4   runtime_minutes       683 non-null    float64
     5   genres                683 non-null    object 
     6   actor                 646 non-null    object 
     7   actress               587 non-null    object 
     8   composer              303 non-null    object 
     9   director              664 non-null    object 
     10  editor                84 non-null     object 
     11  producer              577 non-null    object 
     12  writer                571 non-null    object 
     13  movrating             683 non-null    object 
     14  r males               683 non-null    float64
     15  r females             683 non-null    float64
     16  r imdb users          683 non-null    float64
     17  r aged 18 29          683 non-null    float64
     18  r aged 30 44          683 non-null    float64
     19  r males aged 18 29    683 non-null    float64
     20  r males aged 30 44    683 non-null    float64
     21  r females aged 18 29  683 non-null    float64
     22  r top 1000 voters     683 non-null    float64
     23  r us users            683 non-null    float64
     24  r non us users        683 non-null    float64
     25  budget                683 non-null    int64  
     26  global_gross          683 non-null    int64  
    dtypes: float64(12), int64(3), object(12)
    memory usage: 149.4+ KB



```python
final_df.to_csv('final_df.csv')
```


```python
# drop all rows that don't contain budget information
final_df = final_df[final_df['budget'] != 0]
```


```python
final_df = final_df.reset_index()
final_df = final_df.drop('index', axis=1)
```

# Analysis


```python
# First I will perform some exploratory analysis to get a better understanding of the dataset
# Then I will pose and answer a series of questions to come up with my recommendations for Microsoft
```

## Exploring the data


```python
df = pd.read_csv('final_df.csv')
```


```python
df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>r males</th>
      <th>r females</th>
      <th>r imdb users</th>
      <th>r aged 18 29</th>
      <th>r aged 30 44</th>
      <th>r males aged 18 29</th>
      <th>r males aged 30 44</th>
      <th>r females aged 18 29</th>
      <th>r top 1000 voters</th>
      <th>r us users</th>
      <th>r non us users</th>
      <th>budget</th>
      <th>global_gross</th>
      <th>budget_inmil</th>
      <th>gross_inmil</th>
      <th>profit_ratio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>5.510000e+02</td>
      <td>5.510000e+02</td>
      <td>551.000000</td>
      <td>551.000000</td>
      <td>551.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>275.000000</td>
      <td>2016.695100</td>
      <td>111.408348</td>
      <td>6.370962</td>
      <td>6.623956</td>
      <td>6.445917</td>
      <td>6.525771</td>
      <td>6.355535</td>
      <td>6.460799</td>
      <td>6.308530</td>
      <td>6.680581</td>
      <td>5.863339</td>
      <td>6.463702</td>
      <td>6.338113</td>
      <td>5.794136e+07</td>
      <td>2.191605e+08</td>
      <td>57.941363</td>
      <td>219.160530</td>
      <td>4.648959</td>
    </tr>
    <tr>
      <th>std</th>
      <td>159.204271</td>
      <td>1.252472</td>
      <td>17.354126</td>
      <td>0.991413</td>
      <td>0.917173</td>
      <td>0.964429</td>
      <td>1.031640</td>
      <td>0.945593</td>
      <td>1.072290</td>
      <td>0.969001</td>
      <td>0.952129</td>
      <td>0.959317</td>
      <td>1.000958</td>
      <td>0.965345</td>
      <td>6.181918e+07</td>
      <td>3.169000e+08</td>
      <td>61.819179</td>
      <td>316.900005</td>
      <td>6.261870</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>2015.000000</td>
      <td>74.000000</td>
      <td>2.900000</td>
      <td>2.500000</td>
      <td>3.200000</td>
      <td>2.700000</td>
      <td>3.200000</td>
      <td>2.700000</td>
      <td>3.300000</td>
      <td>2.400000</td>
      <td>2.400000</td>
      <td>2.600000</td>
      <td>3.100000</td>
      <td>5.000000e+04</td>
      <td>3.909100e+04</td>
      <td>0.050000</td>
      <td>0.039091</td>
      <td>0.011169</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>137.500000</td>
      <td>2016.000000</td>
      <td>98.000000</td>
      <td>5.800000</td>
      <td>6.100000</td>
      <td>5.900000</td>
      <td>5.900000</td>
      <td>5.750000</td>
      <td>5.800000</td>
      <td>5.700000</td>
      <td>6.100000</td>
      <td>5.300000</td>
      <td>5.850000</td>
      <td>5.700000</td>
      <td>1.490000e+07</td>
      <td>3.478921e+07</td>
      <td>14.900000</td>
      <td>34.789214</td>
      <td>1.475801</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>275.000000</td>
      <td>2017.000000</td>
      <td>110.000000</td>
      <td>6.400000</td>
      <td>6.700000</td>
      <td>6.500000</td>
      <td>6.600000</td>
      <td>6.400000</td>
      <td>6.600000</td>
      <td>6.400000</td>
      <td>6.700000</td>
      <td>5.900000</td>
      <td>6.500000</td>
      <td>6.400000</td>
      <td>3.500000e+07</td>
      <td>1.043995e+08</td>
      <td>35.000000</td>
      <td>104.399548</td>
      <td>3.129662</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>412.500000</td>
      <td>2018.000000</td>
      <td>122.000000</td>
      <td>7.100000</td>
      <td>7.300000</td>
      <td>7.150000</td>
      <td>7.300000</td>
      <td>7.100000</td>
      <td>7.200000</td>
      <td>7.000000</td>
      <td>7.400000</td>
      <td>6.600000</td>
      <td>7.200000</td>
      <td>7.100000</td>
      <td>8.000000e+07</td>
      <td>2.558271e+08</td>
      <td>80.000000</td>
      <td>255.827129</td>
      <td>5.216859</td>
    </tr>
    <tr>
      <th>max</th>
      <td>550.000000</td>
      <td>2019.000000</td>
      <td>181.000000</td>
      <td>8.400000</td>
      <td>9.300000</td>
      <td>8.400000</td>
      <td>8.700000</td>
      <td>8.300000</td>
      <td>8.700000</td>
      <td>8.300000</td>
      <td>9.200000</td>
      <td>7.900000</td>
      <td>8.900000</td>
      <td>8.300000</td>
      <td>3.560000e+08</td>
      <td>2.797801e+09</td>
      <td>356.000000</td>
      <td>2797.800564</td>
      <td>85.752315</td>
    </tr>
  </tbody>
</table>
</div>



### Release year


```python
# Our dataset only includes films from the years 2015 through 2019, to ensure the relevancy of the results.
```


```python
sns.countplot(df.start_year)
plt.xlabel('Release Year')
plt.ylabel('Movie Count')
plt.title('Count of Movies by Release Year');
```


![png](output_197_0.png)


### Runtime


```python
# Per the charts below, most films in the dataset run between approx. 80 and 150 minutes, 
# with a handful of outliers running beyond the 150 minute mark.
```


```python
sns.distplot(df["runtime_minutes"], kde=False);
plt.title('Distribution of Movie Runtimes')
plt.ylabel('Film Count')
plt.xlabel('Runtime (min)');
```


![png](output_200_0.png)



```python
sns.boxplot(data=df, x="runtime_minutes")
plt.xlabel('Runtime (min)')
plt.title('Distribution of Movie Runtimes');
```


![png](output_201_0.png)


### Audience rating


```python
# This dataset includes films of all ratings, however it may contain overrepresentative samples of PG-13 and
# R-rated films.
```


```python
df['movrating'].value_counts()
```




    PG-13        232
    R            196
    PG            88
    Not Rated     29
    G              3
    Unrated        2
    Approved       1
    Name: movrating, dtype: int64




```python
df.loc[df["movrating"] == 'Unrated', ["movrating"]]  = 'Not Rated'
```


```python
rateorder = ['G','PG','PG-13','R','Not Rated','Approved']
```


```python
sns.countplot(df.movrating, order = rateorder)
plt.xlabel('Rating')
plt.ylabel('Movie Count')
plt.title('Count of Movies by Audience ');
```


![png](output_207_0.png)


### Production budget


```python
# The average budget in our dataset was approx. $57 million, but extended as high as #350 million (though
# anything with a budget of over approx $175 million was an outlier).
```


```python
# creating a budget column in units of millions of USD
df['budget_inmil'] = df.budget.map(lambda x: x/1000000)
```


```python
df['budget_inmil'].mean()
```




    57.94136324500908




```python
sns.distplot(df["budget_inmil"], kde=False)
plt.xlabel('Budget (in millions USD)')
plt.ylabel('Movie Count')
plt.title('Distribution of Movie Budgets (in mil)')
plt.xticks(np.linspace(start=0, stop=400, num=5));
```


![png](output_212_0.png)



```python
sns.boxplot(data=df, x="runtime_minutes")
plt.xlabel('Runtime (min)')
plt.title('Distribution of Movie Runtimes');
```


![png](output_213_0.png)


### Gross revenue


```python
# The average gross revenue in our dataset was approx. $219 million, with most films in the dataset grossing 
# between $0 and approximately $600 million.  Additionally, outliers in the dataset grossed up to $3 billion.
```


```python
# creating a global gross revenue column in units of millions of USD
df['gross_inmil'] = df['global_gross'].map(lambda x: x/1000000)
```


```python
df['gross_inmil'].mean()
```




    219.16052985480948




```python
sns.distplot(df["gross_inmil"], kde=False);
plt.xlabel('Gross Revenue (in millions USD)')
plt.ylabel('Movie Count')
plt.title('Distribution of Gross Movie Revenue (in mil)');
```


![png](output_218_0.png)



```python
sns.boxplot(data=df, x = "gross_inmil");
plt.title('Distribution of Gross Movie Revenue (in mil)')
plt.xlabel('Gross Revenue (in millions USD)');
```


![png](output_219_0.png)


### Profitability


```python
# The average film in the dataset returned 4.65x on its budget; 
```


```python
# create a calulated column for profitability

df['profit_ratio'] = df['gross_inmil'] / df['budget_inmil']
```


```python
df['profit_ratio'].mean()
```




    4.648958516336651




```python
sns.distplot(df["profit_ratio"], kde=False)
plt.xlabel('Return on Budget')
plt.ylabel('Movie Count')
plt.title('Distribution of Return on Budget');
```


![png](output_224_0.png)



```python
sns.boxplot(data=df, x="profit_ratio")
plt.xlabel('Return on Budget (as a multiple of Budget)')
plt.title('Distribution of Return on Budget');
```


![png](output_225_0.png)



```python
# I want to categorize all movies in the dataset by their profitability:

df['profit_ratio'].describe()
```




    count    551.000000
    mean       4.648959
    std        6.261870
    min        0.011169
    25%        1.475801
    50%        3.129662
    75%        5.216859
    max       85.752315
    Name: profit_ratio, dtype: float64




```python
# Splitting data into groups 1 through 4, where 1 is least profitable quartile of movies in the data set
# and 4 is most profitable quartile

df['profitability'] = df['profit_ratio'].map(lambda x: x)

df.loc[df["profit_ratio"] >= 0, ["profitability"]]  = '1'
df.loc[df["profit_ratio"] >= 1.5, ["profitability"]] = '2'
df.loc[df["profit_ratio"] >= 3.1, ["profitability"]] = '3'
df.loc[df["profit_ratio"] >= 5.2, ["profitability"]] = '4'
```


```python
sns.countplot(df['profitability'])
plt.title('Profitability Quartiles')
plt.xlabel('Quartiles (1 = least profitable)')
plt.ylabel('Count of Movies');
```


![png](output_228_0.png)


### Genre


```python
# Since most movies are tagged with multiple genres, I am uncoupling these groupings to get a count for 
# how many times each individual genre appears in our data

genrelist = []

for i in df['genres']:
    genrelist.extend(i.split(','))
```


```python
from collections import Counter
c = dict(Counter(genrelist))
```


```python
df_c = pd.Series(c).sort_values(ascending=False)
```


```python
df_c.index
```




    Index(['Drama', 'Action', 'Adventure', 'Comedy', 'Thriller', 'Crime', 'Horror',
           'Animation', 'Biography', 'Sci-Fi', 'Fantasy', 'Mystery', 'Romance',
           'Family', 'History', 'Music', 'Sport', 'War', 'Documentary', 'Musical',
           'Western', 'News'],
          dtype='object')




```python
fig, ax = plt.subplots(figsize=(12,6))
sns.countplot(genrelist, order=list(df_c.index))
degrees=45
plt.xticks(rotation='vertical')
plt.xlabel('Genres')
plt.ylabel('Count of Movies')
plt.title('Distribution of Movies by Genre Tag');
```


![png](output_234_0.png)


## Analysis

### Which audience rating in our dataset is the most profitable?


```python
# Based on the below scatter plot, we can see that the highest earning films in our data set are 
# rated either PG-13 or R 

# strip plot, swarm plot, 

sns.stripplot(data = df, x = 'movrating', y = 'profit_ratio', order = rateorder);
plt.xticks(rotation = 30)
plt.xlabel('Film Rating')
plt.ylabel('Return')
plt.title('Return as a multiple of Budget, by Film Rating');
```


![png](output_237_0.png)



```python
ratexprof = pd.DataFrame(df.groupby('profitability')['movrating'].value_counts())
ratexprof = ratexprof.rename(columns={'movrating':'count'})
ratexprof = ratexprof.reset_index()
ratexprof.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>profitability</th>
      <th>movrating</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>R</td>
      <td>67</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>PG-13</td>
      <td>52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>PG</td>
      <td>17</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>Not Rated</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>Approved</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# The below chart indicates that most of the least profitable movies are R-rated, however most of the movies
# in ALL categories are either R or PG-13, so this is not the most informative plot.

sns.barplot(x=ratexprof['profitability'], y=ratexprof['count'], hue=ratexprof['movrating'])
plt.title('Distribution of Movie Ratings, by Profitability Quartile')
plt.xlabel('Profitability (1 = least profitable)')
plt.ylabel('Count of Movies');
```


![png](output_239_0.png)



```python
# Grouping the same chart by rating instead of profitability quartile, it becomes more clear that R-rated 
# movies are the most likely to flop (i.e. Profitability =1), whereas PG-13 and PG films tend to 
# perform better.

sns.barplot(x=ratexprof['movrating'], y=ratexprof['count'], hue=ratexprof['profitability'], order=rateorder)
plt.title('Distribution of Profitability, by Movie Rating')
plt.xlabel('Movie Rating')
plt.ylabel('Count of Movies');
```


![png](output_240_0.png)



```python
ratingavgprof = pd.DataFrame(df.groupby('movrating')['profit_ratio'].mean())
ratingavgprof = ratingavgprof.reset_index()
ratingavgprof.sort_values('profit_ratio', ascending=False, inplace=True)

sns.barplot(x=ratingavgprof['movrating'], y=ratingavgprof['profit_ratio'])
plt.title('Average return, by film rating')
plt.xlabel('Rating')
plt.ylabel('Return (as a mult. of Budget)');
```


![png](output_241_0.png)



```python
# According to the above chart, G-rated movies earn the most return on average, however our n for 
# G-rated films in this data set is only 3, making these results somewhat unreliable
```

#### BIZ REC:  Target audience rating - PG-13
- Though R and PG-13 films both earned some of the highest returns in our dataset, R-rated films are more likely to land in the least profitable quartile of films than in any other profitability quartile, making PG-13 films the less risky choice for Microsoft.

### Which genre combinations performed the best and the worst financially?


```python
genreavgprof = pd.DataFrame(df.groupby('genres')['profit_ratio'].mean())
genreavgprof = genreavgprof.reset_index()
genreavgprof.sort_values('profit_ratio', ascending=False, inplace=True)
genreavgprof = genreavgprof.head(10)

sns.barplot(x=genreavgprof['genres'], y=genreavgprof['profit_ratio'])
tilt=45
plt.xticks(rotation='vertical')
plt.xlabel('Film Genres')
plt.ylabel('Return on Investment')
plt.title('Top Earning Movie Genre Combos');
```


![png](output_245_0.png)



```python
genreavgprof2 = pd.DataFrame(df.groupby('genres')['profit_ratio'].mean())
genreavgprof2 = genreavgprof2.reset_index()
genreavgprof2.sort_values('profit_ratio', inplace=True)
genreavgprof2 = genreavgprof2.head(10)

sns.barplot(x=genreavgprof2['genres'], y=genreavgprof2['profit_ratio'])
tilt=45
plt.xticks(rotation='vertical')
plt.xlabel('Film Genres')
plt.ylabel('Return on Investment')
plt.title('Lowest Earning Movie Genre Combos');
```


![png](output_246_0.png)


#### BIZ REC:  Target film genre - Thriller
- With the genre combo "Drama, Mystery, Thriller" as the most profitable genre combo on average within our dataset and the high frequency of Drama and Thriller in the genre groupings that make up the most profitable films on average, one of these genres would be a strategic choice for Microsofts first film.  Given the high level of saturation in the Drama category, as indicated by our analysis in section 4.1.7, Thriller has been identified as the better choice for Microsoft between the two.

### Which composer should do the soundtrack?


```python
# Grouping data by composer and calculating mean profit_ratio value earned across all of their films

comps = pd.DataFrame(df.groupby('composer')['profit_ratio'].mean())
comps = comps.reset_index()
comps = comps.sort_values('profit_ratio', ascending=False)
```


```python
# I need to use ppl_df to pull in the actual names of the composers based on their nconst id

ppl_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm0061671</td>
      <td>Mary Ellen Bauder</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,production_manager,producer</td>
      <td>tt0837562,tt2398241,tt0844471,tt0118553</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nm0061865</td>
      <td>Joseph Bauer</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>composer,music_department,sound_department</td>
      <td>tt0896534,tt6791238,tt0287072,tt1682940</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nm0062070</td>
      <td>Bruce Baum</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,actor,writer</td>
      <td>tt1470654,tt0363631,tt0104030,tt0102898</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nm0062195</td>
      <td>Axel Baumann</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>camera_department,cinematographer,art_department</td>
      <td>tt0114371,tt2004304,tt1618448,tt1224387</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nm0062798</td>
      <td>Pete Baxter</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>production_designer,art_department,set_decorator</td>
      <td>tt0452644,tt0452692,tt3458030,tt2178256</td>
    </tr>
  </tbody>
</table>
</div>




```python
merge_df = ppl_df.iloc[:,:2]
merge_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm0061671</td>
      <td>Mary Ellen Bauder</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nm0061865</td>
      <td>Joseph Bauer</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nm0062070</td>
      <td>Bruce Baum</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nm0062195</td>
      <td>Axel Baumann</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nm0062798</td>
      <td>Pete Baxter</td>
    </tr>
  </tbody>
</table>
</div>




```python
# It looks like these 5 composers have the most profitable films on average based on our dataset

comps = comps.merge(merge_df, left_on='composer', right_on='nconst', how='left').head()
comps
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>composer</th>
      <th>profit_ratio</th>
      <th>nconst</th>
      <th>primary_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm3788293</td>
      <td>85.752315</td>
      <td>nm3788293</td>
      <td>Torin Borrowdale</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nm1615109</td>
      <td>37.040775</td>
      <td>nm1615109</td>
      <td>Nicholas Britell</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nm8752173</td>
      <td>31.949963</td>
      <td>nm8752173</td>
      <td>Michael Abels</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nm1853865</td>
      <td>27.237284</td>
      <td>nm1853865</td>
      <td>Matthew Margeson</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nm0590141</td>
      <td>24.658413</td>
      <td>nm0590141</td>
      <td>Paul Mills</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.barplot(x=comps['primary_name'], y=comps['profit_ratio'])
plt.xticks(rotation='vertical')
plt.title('Average Return on Film Budget, by Composer')
plt.xlabel('Composer name')
plt.ylabel('Avg. Return on Budget')
```




    Text(0, 0.5, 'Avg. Return on Budget')




![png](output_253_1.png)



```python
# Looking into the top composer, Torin Borrowdale, whose films achieved an almost 86x return on average
# I want to understand what films were included in this calculation and to confirm that my numbers are right

ppl_df[ppl_df['nconst'] == 'nm3788293']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>281746</th>
      <td>nm3788293</td>
      <td>Torin Borrowdale</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>composer,music_department,sound_department</td>
      <td>tt3685586,tt7293920,tt4057632,tt7668870</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Searching my data for all the title ids in Torin's "known_for_titles" column above, I found just one movie of
# his in my final dataset.  This movie had a budget of $880k and grossed over $75 mil globally, resulting in
# the 86x return mentioned above.  So everything looks correct.  But......

df[df['tconst'] == 'tt7668870']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>actor</th>
      <th>actress</th>
      <th>composer</th>
      <th>...</th>
      <th>r females aged 18 29</th>
      <th>r top 1000 voters</th>
      <th>r us users</th>
      <th>r non us users</th>
      <th>budget</th>
      <th>global_gross</th>
      <th>budget_inmil</th>
      <th>gross_inmil</th>
      <th>profit_ratio</th>
      <th>profitability</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>536</th>
      <td>536</td>
      <td>tt7668870</td>
      <td>Searching</td>
      <td>Searching</td>
      <td>2018</td>
      <td>102.0</td>
      <td>Drama,Mystery,Thriller</td>
      <td>nm0158626, nm4334711</td>
      <td>nm0005226, nm8045046</td>
      <td>nm3788293</td>
      <td>...</td>
      <td>7.8</td>
      <td>6.9</td>
      <td>7.7</td>
      <td>7.6</td>
      <td>880000</td>
      <td>75462037</td>
      <td>0.88</td>
      <td>75.462037</td>
      <td>85.752315</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 32 columns</p>
</div>




```python
# I'm also curious who the highest grossing composers are - especially since the results for the most profitable
# composers did not turn up any household names.

comps2 = pd.DataFrame(df.groupby('composer')['gross_inmil'].mean())
comps2 = comps2.reset_index()
comps2 = comps2.sort_values('gross_inmil', ascending=False)
```


```python
comps2 = comps2.merge(merge_df, left_on='composer', right_on='nconst', how='left').head(10)
```


```python
comps2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>composer</th>
      <th>gross_inmil</th>
      <th>nconst</th>
      <th>primary_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm0003911</td>
      <td>1377.405338</td>
      <td>nm0003911</td>
      <td>Brian Tyler</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nm3234869</td>
      <td>1347.071259</td>
      <td>nm3234869</td>
      <td>Ludwig Göransson</td>
    </tr>
    <tr>
      <th>2</th>
      <td>nm0579678</td>
      <td>1050.693953</td>
      <td>nm0579678</td>
      <td>Alan Menken</td>
    </tr>
    <tr>
      <th>3</th>
      <td>nm0002201</td>
      <td>966.550600</td>
      <td>nm0002201</td>
      <td>John Debney</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nm0653211</td>
      <td>903.655259</td>
      <td>nm0653211</td>
      <td>John Ottman</td>
    </tr>
    <tr>
      <th>5</th>
      <td>nm0847926</td>
      <td>634.151679</td>
      <td>nm0847926</td>
      <td>Joby Talbot</td>
    </tr>
    <tr>
      <th>6</th>
      <td>nm0673137</td>
      <td>600.641132</td>
      <td>nm0673137</td>
      <td>Heitor Pereira</td>
    </tr>
    <tr>
      <th>7</th>
      <td>nm0002354</td>
      <td>570.738190</td>
      <td>nm0002354</td>
      <td>John Williams</td>
    </tr>
    <tr>
      <th>8</th>
      <td>nm0002353</td>
      <td>559.477598</td>
      <td>nm0002353</td>
      <td>Thomas Newman</td>
    </tr>
    <tr>
      <th>9</th>
      <td>nm0006133</td>
      <td>544.443318</td>
      <td>nm0006133</td>
      <td>James Newton Howard</td>
    </tr>
  </tbody>
</table>
</div>




```python
sns.barplot(x=comps2['primary_name'], y=comps2['gross_inmil'])
plt.xticks(rotation='vertical')
plt.title('Average Gross Revenue (per film), by Composer')
plt.xlabel('Composer name')
plt.ylabel('Avg. Gross Revenue per Film')
```




    Text(0, 0.5, 'Avg. Gross Revenue per Film')




![png](output_259_1.png)



```python
# SO, though we determined that films with music by Torin Borrowdale profit the most on average, 
# films scored by any of the above 10 composers gross the most global revenue on average (irrespective
# of budget)
```

#### BIZ REC:  Target composer: Torin Borrowdale
- Films with scores composed by Torin Borrowdale had the highest returns on average (as a multiple of budget.  Though Torin only scored one film in our data set, it was a Thriller which aligns with our previous recommendation for Microsoft's film genre.  Additionally, with his up-and-coming status, we believe Microsoft will be able to negotiate more favorable contract terms with Torin than with a more famous composer such as Alan Menken or John Williams.



```python

```
