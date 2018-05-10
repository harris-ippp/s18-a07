# Assignment 7: Web Scraping
In this assignment we will use `requests` and `BeautifulSoup` to scrape Wikipedia's [List of accidents and incidents involving commercial aircraft](https://en.wikipedia.org/wiki/List_of_accidents_and_incidents_involving_commercial_aircraft) and analyze the data. Put your scraping code in a script called `scrape.py` and put your solutions to part B in scripts called `q1.py,...,q3.py`.

## A: Scrape
Here we will write code to scrape the list of flights along with the following characteristics for each flight (located in the "infobox" on the right of each accident's page):

 - Date
 - Operator
 - Flight origin
 - Destination
 - Fatalities

Note that not all of the accident infoboxes contain all of these attributes, so some data will be missing in our results. After you have all of the data, put it into a pandas DataFrame and write that to a CSV file that we can load and analyze in part B. (16 points)

There are many different ways to perform this scraping. Here is an outline of one of them:

1. Get the [Wikipedia article](https://en.wikipedia.org/wiki/List_of_accidents_and_incidents_involving_commercial_aircraft) and turn it into a `BeautifulSoup`.

2. Find a way to identify all of the accident links. I noticed that all of the links are bold, i.e. wrapped in `<b>...</b>` tags. So you can start with `find_all('b')` but beware that not all bold text in the article has a link to an accident. More on that below.
    
3. Iterate over the bold tags and request the page for each link and turn it into a `BeautifulSoup`. As mentioned above, not all of them have links. In fact one of the bold links isn't to an accident:
    
    ```html
    <b><a href="#cite_ref-1">^</a></b>
    ```

    which has a link but not to an accident. It's the very last one so if you call the list of bold tags `bolds` then you can ignore this one by simply using `bolds[:-1]`.
        
    More annoyingly, there are some bold tags that don't contain links. Some are accidents without articles, others are not accidents. If you try to `find()` a tag with BeautifulSoup that doesn't exist it will return `None`. Thus, you can test whether a bold tag `b` contains a link by checking whether `b.find('a') is None`.

4. For each accident article, extract from the infobox the attributes listed above into a dictionary. Also include the name of the accident, which you can get from the text of the original link. For example the first accident would produce the following dictionary:

    ```python
    {'Date': 'July 21, 1919',
     'Destination': 'White City amusement park, Chicago, Illinois',
     'Fatalities': '13 (2 passengers, 1 crew, 10 on ground)',
     'Flight origin': 'Grant Park, Chicago, Illinois',
     'Name': 'Goodyear dirigible Wingfoot Air Express catches fire and crashes',
     'Operator': 'Goodyear Tire and Rubber Company'}
    ```
    
    By looking at the HTML source for an accident article note that each datum is the text of the table cell (`<td>`) immediately after a table header (`<th>`) that describes it. For example, the date of the [first accident](https://en.wikipedia.org/wiki/Wingfoot_Air_Express_crash):
    
    ```html
    <tr>
    <th scope="row" style="line-height:1.3em; padding-right:1.0em; white-space:nowrap;">Date</th>
    <td style="line-height:1.3em;">July 21, 1919</td>
    </tr>
    ```
    
    In class we saw that `find('div', class_='nytint-detainee-fullcol')` finds the `<div>` whose `class="nytint-detainee-fullcol"`. We can similarly find a tag with a particular text with the `text` argument to `find()`. So to find the Date table header you can use `a_page.find('th', text='Date')`. Then you'll need to find the next `<td>` and get its text.
    
    Put this code into a function calld `get_table_data(a_page, header)` that takes as its argument an accident page and the name of a header (e.g. `'Date'`, `'Flight origin'`, etc.) and returns the corresponding value. Not every page has every piece of data. In those cases `get_data()` can just return `None`. To identify these cases you can check if the result of your `find('th', text=header)` `is None`.
    
    Finally, iterate over the different header names and get each piece of data and put it into a dictionary. Append that dictionary to a big list of dictionaries, one for each accident. I recommend testing this process on a single accident page first and then on a small number before scraping all 1000+ accidents.
    
5. Turn that list of dictionaries into a DataFrame simply by passing it to `pd.DataFrame()`. This is an alternative way to construct a DataFrame, e.g.:

    ```python
    >>> pd.DataFrame([{'a':1, 'b':2}, {'a':3, 'b':4}])   a  b
    0  1  2
    1  3  4
    ```
    
    You'll also want to call `drop_duplicates()` on your DataFrame because there are a few accidents that were linked multiple times in the original list article. Finally, write your results to a CSV file called `accidents.csv`.

## B: Analyze
Now we will analyze the data from A. In case you have trouble getting your scraper to work, I have posted the the data [here](https://raw.githubusercontent.com/harris-ippp/s18-a07/master/accidents.csv). Thus you can get partial credit by proceeding with those results.

1. What is the most common origin for the flight accidents? (2 points)

2. Which operator has the most accidents? (2 points)

3. Extract the number of fatalities from each accident into a column called `'Fatalities count'`. Save this as `accidents2.csv` for use below. (4 points)

    Hint: The fatalities can be strings like `'13 (2 passengers, 1 crew, 10 on ground)'`. We'll assume that the first number in each string is the total number of fatalities and use a regex to extract it. 

    Write a function called `get_first_number(text)` that takes a string and uses a regular expression to return the first number in it. If the text is null or contains no numbers, return `None`. You can check if `text` is null using the `pd.isnull()` function. Apply this to `df['Fatalities']`.

3. Find the flight accident with the most fatalities. (2 points)

4. Which air operator with the most fatalities. Hint: Use `groupby`. (4 points)
    
5. Make a histogram of accident years. (4 points)

    Hint: Some of the dates are missing or formatted poorly so you can pass the argument `errors='coerce'` to [`pd.to_datetime()`](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.to_datetime.html) to simply convert them to `NaT` ("Not a Time",  the equivalent for times of `NaN` for numbers).
