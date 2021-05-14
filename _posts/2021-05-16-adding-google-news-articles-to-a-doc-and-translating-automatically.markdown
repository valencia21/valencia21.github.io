---
layout: post
title:  "Adding Google News Articles to a Doc and Translating Automatically"
date:   2021-05-14 18:00:00 +0900
categories:
---

As of April 2021, I have taken on a freelance project for a [*consultancy*](https://prtimes.jp/main/html/rd/p/000000002.000076032.html) that helps both Japanese corporations switch to using cage-free eggs, and cage-free egg producers in Japan increase their production capacity and acquire new customers.

As part of this project, I have been asked to gather any relevant industry news in Japanese, add it to a doc somewhere and roughly translate the content.

This job is regular, manual, and will be required in regions outside Japan - a perfect candidate for automation.

## Program Features

Makes use of [*SerpAPI*](https://serpapi.com/news-results) ($50 a month, up to 5,000 requests).

- Grabs 50 Google News results for a list of Japanese keywords you input - keyword list and number of results can be changed.
- Adds them to a Google Sheet in order of latest to oldest.
- Provides an automatic English translation of the results.

View program [*here*](https://colab.research.google.com/drive/1Fo6Skv90STXM72Xb6zD2wwqJkT7oAju1?usp=sharing) in Colab (API key removed).
Example output [*here*](https://docs.google.com/spreadsheets/d/11btKCI4iF_5eGFnqyXB2Ch3IrXgKYK_nHzSanrWfzVA/edit#gid=1539352199) in Google Sheets.

## Flow

- Pull Google News results for a list of Japanese keywords using SerpAPI.
- Create a separate Pandas dataframe using the JSON from each request (while ensuring there is no mojibake from Japanese results).
- Add columns to each dataframe for automatic translation in Google Sheets.
- Create new Google Sheets doc in Google Drive, add worksheets for each keyword using the gspread library.
- Populate worksheets with dataframes using gspread_dataframe library.

## Challenges

### Google News provides a date value for articles less than 1 month old in the format (3 weeks ago, 2 days ago etc.)

So the results could be sorted by latest to oldest, I needed to parse these values using regex, substract the time period from the time of executing the program, and finally convert to datetime.

The function below was applied to each date value if it contained the string "ago".

{% highlight python %}
import regex as re
from dateutil.relativedelta import relativedelta
from datetime import datetime

def ago_do_date(ago):
    value, unit = re.search(r'(\d+) (\w+) ago', ago).groups()
    if not unit.endswith('s'):
      unit += 's'
    if unit == "mins":
      unit == "minutes"
    delta = relativedelta(**{unit: int(value)})
    return datetime.strftime(datetime.now() - delta, "%b %d, %Y")
{% endhighlight %}

### Adding translations automatically

Translations were provided using the Google Translate functionality, accessible in Google Sheets using the =GOOGLETRANSLATE() formula.
Since the automation program creates a new file, whoever uses this sheet should not have to add translation cells to each worksheet manually.

This was resolved by adding the formula directly to a Pandas dataframe, including the cells that should be translated.

{% highlight python %}
for i in range(1,number_of_results + 1):
  title_EN.append("=GOOGLETRANSLATE(C%s)" % i)

source_EN = []

for i in range(1,number_of_results + 1):
  source_EN.append("=GOOGLETRANSLATE(D%s)" % i)

snippet_EN = []

for i in range(1,number_of_results + 1):
  snippet_EN.append("=GOOGLETRANSLATE(E%s)" % i)

df["title_EN"], df["source_EN"], df["snippet_EN"] = title_EN, source_EN, snippet_EN  
{% endhighlight %}

### Adding a new worksheet for each keyword

The gspread Python library is a wrapper for the Google Sheets API. After creating a new Google Sheets file in Google Drive, I added a new worksheet to the file for each keyword to be searched in Google News.

The content of each new worksheet is then populated with the dataframe pertaining to each keyword.

{% highlight python %}
sh = gc.create('Industry News (%s)' % date.today().strftime("%b %d, %Y"))

for i in range(len(keywords)):
  sh.add_worksheet(title=keywords[i], rows="100", cols="20")

for i in range(len(keywords)):
  worksheet = sh.worksheet(keywords[i])
  set_with_dataframe(worksheet, list_of_dfs[i], include_index=True, include_column_header=False)
{% endhighlight %}