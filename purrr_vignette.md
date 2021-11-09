---
title: "Tidyverse CREATE - Alec"
author: "Alec"
date: "10/24/2021"
output: 
  prettydoc::html_pretty:
    theme: cosmo
    highlight: github
    keep_md: true
    toc: true
    toc_depth: 2
    df_print: paged
---



## Introduction to Purrr

This vignette has been created to provide information on the tidyverse package: purrr. Purrr is a great package for analyzing and manipulating json data.

We will be using a League of Legends dataset, provided by Santiago Torres in project 2.

## Loading the Data


```r
library(rjson)
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──
```

```
## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
## ✓ tibble  3.1.4     ✓ dplyr   1.0.7
## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
## ✓ readr   2.0.1     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(stringr)
library(jsonlite)
```

```
## 
## Attaching package: 'jsonlite'
```

```
## The following object is masked from 'package:purrr':
## 
##     flatten
```

```
## The following objects are masked from 'package:rjson':
## 
##     fromJSON, toJSON
```

```r
library(purrr)
```

# Load Data

Data is hosted on ddragon.leagueoflegends.com. We can use rjson::fromJSON to read and parse directly from the URL.

fromJSON will return a nested list structure containing all of the available json data


```r
champion_json <- rjson::fromJSON(file="https://ddragon.leagueoflegends.com/cdn/11.19.1/data/en_US/champion.json")
```


The champ_list is a list of 157 json objects. Each json object contains information about an available character in the online MOBA League of Legends. For instance, these json objects may hold the name of the character, descriptions of the character, or stats of the character. 


```r
champ_list <- champion_json$data
```

# Demonstrate value of Purrr package

We will be using the tidyverse "purrr" package to demonstrate how working with JSONs doesn't always need to be a chore.

## Extracting nested data - single values

One of the cool functions available in the purrr package is map(). Map() takes in two arguments:

- a list of jsons
- a key-value to extract

Below, we will extract all of the champion names into a new character vector called "champ_names"


```r
champ_names <- purrr::map(champ_list, "name")

champ_names[1:5]
```

```
## $Aatrox
## [1] "Aatrox"
## 
## $Ahri
## [1] "Ahri"
## 
## $Akali
## [1] "Akali"
## 
## $Akshan
## [1] "Akshan"
## 
## $Alistar
## [1] "Alistar"
```

## Extracting nested data - multiple values

What if we wanted to extract more than just the name of a champion? With purrr, we can do that as well. 

Say for instance we wanted to extract all champions' name, title, and blurb from the champion json. If we wanted to do this for a single json object, we could accomplish it like so:


```r
champ_list[[1]][c("name","title","blurb")]
```

```
## $name
## [1] "Aatrox"
## 
## $title
## [1] "the Darkin Blade"
## 
## $blurb
## [1] "Once honored defenders of Shurima against the Void, Aatrox and his brethren would eventually become an even greater threat to Runeterra, and were defeated only by cunning mortal sorcery. But after centuries of imprisonment, Aatrox was the first to find..."
```

In the above example, the square brackets surrounding the character vector is essentially a functional call, indexing the champlist object with the provided character vector. We can use this directly in purrr::map(.x, .f, ...). The function in this case is actually `[`!


```r
champ_list %>%
  purrr::map(`[`,c("name","title","blurb")) %>%
  .[2:3]
```

```
## $Ahri
## $Ahri$name
## [1] "Ahri"
## 
## $Ahri$title
## [1] "the Nine-Tailed Fox"
## 
## $Ahri$blurb
## [1] "Innately connected to the latent power of Runeterra, Ahri is a vastaya who can reshape magic into orbs of raw energy. She revels in toying with her prey by manipulating their emotions before devouring their life essence. Despite her predatory nature..."
## 
## 
## $Akali
## $Akali$name
## [1] "Akali"
## 
## $Akali$title
## [1] "the Rogue Assassin"
## 
## $Akali$blurb
## [1] "Abandoning the Kinkou Order and her title of the Fist of Shadow, Akali now strikes alone, ready to be the deadly weapon her people need. Though she holds onto all she learned from her master Shen, she has pledged to defend Ionia from its enemies, one..."
```

Make sure that when you enter the `[` you are NOT using quotes.


## Extracting data into a dataframe

map() is great for extracting data, but ultimately we likely will need to include that data into a more readable format than another list! That is where map_dfr (map dataframe) comes into play. Map_dfr() automatically converts the extracted nested list and converts to a dataframe


```r
champ_list %>%
  purrr::map_dfr(`[`,c("name","title","blurb")) %>%
  head()
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["name"],"name":[1],"type":["chr"],"align":["left"]},{"label":["title"],"name":[2],"type":["chr"],"align":["left"]},{"label":["blurb"],"name":[3],"type":["chr"],"align":["left"]}],"data":[{"1":"Aatrox","2":"the Darkin Blade","3":"Once honored defenders of Shurima against the Void, Aatrox and his brethren would eventually become an even greater threat to Runeterra, and were defeated only by cunning mortal sorcery. But after centuries of imprisonment, Aatrox was the first to find..."},{"1":"Ahri","2":"the Nine-Tailed Fox","3":"Innately connected to the latent power of Runeterra, Ahri is a vastaya who can reshape magic into orbs of raw energy. She revels in toying with her prey by manipulating their emotions before devouring their life essence. Despite her predatory nature..."},{"1":"Akali","2":"the Rogue Assassin","3":"Abandoning the Kinkou Order and her title of the Fist of Shadow, Akali now strikes alone, ready to be the deadly weapon her people need. Though she holds onto all she learned from her master Shen, she has pledged to defend Ionia from its enemies, one..."},{"1":"Akshan","2":"the Rogue Sentinel","3":"Raising an eyebrow in the face of danger, Akshan fights evil with dashing charisma, righteous vengeance, and a conspicuous lack of shirts. He is highly skilled in the art of stealth combat, able to evade the eyes of his enemies and reappear when they..."},{"1":"Alistar","2":"the Minotaur","3":"Always a mighty warrior with a fearsome reputation, Alistar seeks revenge for the death of his clan at the hands of the Noxian empire. Though he was enslaved and forced into the life of a gladiator, his unbreakable will was what kept him from truly..."},{"1":"Amumu","2":"the Sad Mummy","3":"Legend claims that Amumu is a lonely and melancholy soul from ancient Shurima, roaming the world in search of a friend. Doomed by an ancient curse to remain alone forever, his touch is death, his affection ruin. Those who claim to have seen him describe..."}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


## Extracting nested data, more than one level deep

Json objects may contain nested json objects. This nesting can theoretically continue down to many levels below the surface. In our dataset example, champions indeed have secondary levels to their data, including sub-lists such as "info", "image", and "stats".

In the below example, we will use the function map_chr() to extract the "HP" stat for each champion, which reside under the "stat" list for each champion json.


```r
champ_list %>%
  map_chr(c("stats", "hp")) %>%
  .[1:5]
```

```
##       Aatrox         Ahri        Akali       Akshan      Alistar 
## "580.000000" "526.000000" "500.000000" "560.000000" "600.000000"
```

## using keep(), the select_if for lists

Say you only wanted to analyze list objects based on some condition. purrr::keep() allows you to do this. You simply need to provide a list, and a conditional to follow.

In the below example, we will focus on analyzing only the champs that have a "HP" value above 500


```r
champ_list %>%
  map(c("stats","hp")) %>%
  keep(~ .x > 500) %>%
  .[1:5]
```

```
## $Aatrox
## [1] 580
## 
## $Ahri
## [1] 526
## 
## $Akshan
## [1] 560
## 
## $Alistar
## [1] 600
## 
## $Amumu
## [1] 615
```

## Reversing functions with negate()

This is a bit different than what we've explored so far, but purrr package also provides some useful functionality around customization of functions. 

Take for example the is.null() function. To create the opposite of this, we can use negate()


```r
lst <- list("a", 3, 22, NULL, "q", NULL)

is_not_null <- negate(is.null)

map_lgl(lst, is_not_null)
```

```
## [1]  TRUE  TRUE  TRUE FALSE  TRUE FALSE
```




