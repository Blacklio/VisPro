# Introduction

My project researched and visualized how lyrics and associated data of popular songs have evolved since 1950. I grabbed the top 100 songs on Billboard for each year, and used natural language processing to analyze a variety of metrics. Users can interactively choose a year/genre range they are interested in to get a closer look at subtleties.

**A joint project between Elias E. Mammo and [Kevin Schaich](https://kevinschaich.io)**

# Data

## Crawling/Analysis

### Billboard Top 100 Songs

Our only initial dataset comes from Billboard's Top 100. We grabbed a [**CSV file**](http://ryanrishi.com/files/billboard-top-100-1950-2015.tar.gz) of the top 100 songs for the years 1950 - 2015 from reddit's r/datasets.

### Lyrics

Using [**Wikia Lyrics**](http://lyrics.wikia.com/wiki/Lyrics_Wiki), as well as its Python counterpart on Github, [**Heroku API**](http://lyric-api.herokuapp.com/api/) we scraped each song's title/artist combination and downloaded the song's full lyrics.

Please note that this data is not intended to be a full set, as we ran into some problems along the way with slight inconsistencies in our dataset's naming schemes vs. the API's request structure. Many lyrics for older songs in the 1950-60's are less readily available online as well, however we have about 80-90% coverage for all the songs on Billboard's list in our year range.

### Genres/Tags

Using the [**MusicBrainz API**](https://beta.musicbrainz.org) as well as the Python interface [**Musicbrainzng**](https://github.com/alastair/python-musicbrainzngs), we scraped each song artist's associated genre tags. These tags are quite numerous and extensive, so we came up with a total of 15 '*aggregate genres*' based on their total occurrence rate in all our songs to represent the aggregate of our data and to keep our visualization clean. A minified sample of these aggregates can be found below:

```
aggregate_genres = [
{"rock" = ["pop rock", "jazz-rock", "heartland rock", ...]},
{"alternative/indie" = [...]},
{"electronic/dance" = [...]},
{"soul" = [...]},
{"classical/soundtrack" = [...]},
{"pop" = [...]},
{"hip-hop/rnb" = [...]},
{"disco" = [...]},
{"swing" = [...]},
{"folk" = [...]},
{"country" = [...]},
{"jazz" = [...]},
{"religious" = [...]},
{"blues" = [...]},
{"reggae" = [...]},
]
```

### Sentiment Analysis

Using the [**Natural Language Toolkit (NLTK)**](http://www.nltk.org/) for Python, we used the [**VADER model**](http://comp.social.gatech.edu/papers/icwsm14.vader.hutto.pdf) for parsimonious rule-based sentiment analysis of each song's lyrics. Each song was run through a sentiment analyzer and output an object with data about its sentiment:

```
"sentiment": {
    "neg": [float],             # Negativity assoc. w/ lyrics. (between 0-1 inclusive, 1 being 100% negative).
    "neu": [float],             # Neutrality assoc. w/ lyrics. (between 0-1 inclusive, 1 being 100% neutral).
    "pos": [float],             # Positivity assoc. w/ lyrics. (between 0-1 inclusive, 1 being 100% positive).
    "compound": [float]
}
```

The `pos`, `neg`, and `neu` are the three interesting values. Each value represents the percentage probability that the song is associated with a positive, negative, or neutral connotation and sentiment, respectively. Using the positive and negative values, we are able to tell whether a song's lyrics lean towards *"happy"* or *"sad"* in demeanor.

### Readability Metrics

Using the [**textstat**](https://github.com/shivam5992/textstat) package for Python, we calculated a number of aggregate readability metrics associated with each song's lyrics:

```
"num_words": [int],             # Number of words in lyrics.
"num_lines": [int],             # Number of lines in lyrics.
"num_syllables": [int],         # Number of syllables in lyrics.
"difficult_words": [int],       # Number of words not on the Dale–Chall "easy" word list.
"fog_index": [float],           # Gunning-Fog readability index.
"flesch_index": [float],        # Flesch reading ease score.
"f_k_grade": [float],           # Flesch–Kincaid grade level of lyrics.
```

While the top few are most explanatory, the [**Gunning-Fog Index**](https://en.wikipedia.org/wiki/Gunning_fog_index) and [**Flesh-Kincaid Grade Level**](https://en.wikipedia.org/wiki/Flesch%E2%80%93Kincaid_readability_tests#Flesch.E2.80.93Kincaid_grade_level) are the most powerful. Both of these metrics use a variety of linguistics data like average sentence length, word, length, and complexity/number of syllables to determine the readability of a text. These metrics allows us to graph the trend over time for specific genres, i.e. you would need to be in 2nd grade to understand the average pop song from 1972.

### Repetition

For each song, we count the number of duplicate lines that appear in the lyrics. This can be used as a rough measure of repetition in the song content, i.e. the more duplicate lines in the lyrics, the more repetitive a song is.

## Data Summary

Our aggregate data JSON file includes all of the following metrics:

```
{
    "title": [string],              # Title of the song.       
    "artist": [string],             # Artist of the song.
    "year": [int],                  # Release year of the song.
    "pos": [int],                   # Position of Billboard's Top 100 for year [year].
    "lyrics": [string],             # Lyrics of the song.
    "tags": [string array],         # Genre tags associated with artist of the song.
    "sentiment": {
        "neg": [float],             # Negativity assoc. w/ lyrics. (between 0-1 inclusive, 1 being 100% negative).
        "neu": [float],             # Neutrality assoc. w/ lyrics. (between 0-1 inclusive, 1 being 100% neutral).
        "pos": [float],             # Positivity assoc. w/ lyrics. (between 0-1 inclusive, 1 being 100% positive).
        "compound": [float]
    },
    "f_k_grade": [float],           # Flesch–Kincaid grade level of lyrics.
    "flesch_index": [float],        # Flesch reading ease score.
    "fog_index": [float],           # Gunning-Fog readability index.
    "difficult_words": [int],       # Number of words not on the Dale–Chall "easy" word list.
    "num_syllables": [int],         # Number of syllables in lyrics.
    "num_words": [int],             # Number of words in lyrics.
    "num_lines": [int],             # Number of lines in lyrics.
    "num_dupes": [int]              # Number of duplicate (repetitive) lines in lyrics.
}
```

## Data Aggregation/filtering

Another python script aggregates our data by year for easy filtering in real-time in JavaScript. Our output JSON file looks similar to the above song format with the following structure:

```
{
    {
        "year": 1950,
        songs = {
            {song object 1},
            song object 2}},
            ...
    }
    {
        "year": 1951,
        songs = {
            {song object 1},
            song object 2}},
            ...
    }
    ...
}
```

This structure is cleaned and minimized and the original lyrics are removed to keep our file size under 2MB for nearly 5000 songs. Using [underscore.js](http://underscorejs.org/), we are able to utilize functional programming in JavaScript to very quickly filter and sort through our data. Using the above JSON notation with year-oriented objects allows us to filter through nearly 5000 songs in real-time on the user side in fractions of a second.

## License

MIT © [Kevin Schaich](https://kevinschaich.io)
