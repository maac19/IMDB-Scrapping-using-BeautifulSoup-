# IMDB-Scrapping-using-BeautifulSoup-
Scrapping IMDB's top 250 movie data using Beautiful Soup

#importing required Libraries
import pandas as pd 
import requests
from bs4 import BeautifulSoup
import numpy as np

#creating empty list 
movie_name = []
year = []
rating = []
time = []
metascore = []
votes = []
gross = []

movie_no=1
while movie_no <= 201:
    
    url = "https://www.imdb.com/search/title/?groups=top_250&sort=user_rating,desc&start='+movie_no+'&ref_=adv_nxt"
    response = requests.get(url)
    movie_no += 50
    soup = BeautifulSoup(response.content, 'html.parser')
    movie_data = soup.findAll('div',attrs ={'class': 'lister-item mode-advanced'})
    for store in movie_data:    
        name = store.h3.a.text
        movie_name.append(name)
        
        
        year_of_release = store.h3.find('span', class_ = 'lister-item-year text-muted unbold').text.replace('(', '').replace(')', '')
        year.append(year_of_release)
    
        runtime = store.p.find('span', class_ = 'runtime').text.replace(' min', '')
        time.append(runtime)

        rate = store.find('div', class_ = 'inline-block ratings-imdb-rating').text.replace('\n', '')
        rating.append(rate)

        meta  = store.find('span', class_ = 'metascore').text.replace(' ', '') if store.find('span', class_ = 'metascore') else '^^^^^^'
        metascore.append(meta)
        
        value = store.find_all('span', attrs = {'name': 'nv'})

        vote = value[0].text
        votes.append(vote)

        grosses = value[1].text if len(value) >1 else '*****'
        gross.append(grosses)
        
        #creating a dataframe using pandas library
movie_DF = pd.DataFrame({'Name of movie': movie_name, 'Year of relase': year, 'Watchtime': time, 'Movie Rating': rating, 'Metascore': metascore, 'Votes': votes, 'Gross collection': gross})

movie_DF.head(250)

#importing required libraries and making connections
from sqlalchemy import inspect, create_engine
from sqlalchemy import event

alchemy_driver = 'postgresql+psycopg2'
postgres_user = 'postgres'
postgres_password = '123456789'
postgres_host = '127.0.0.1'
postgres_db = 'imdb_data'
postgres_schema = 'public'


postgres_conn = create_engine(f'{alchemy_driver}://{postgres_user}:{postgres_password}@{postgres_host}:5432/{postgres_db}')
print("Postgres connection set")

movie_DF.to_sql(name='imdb_movies', con=postgres_conn, schema=postgres_schema, if_exists='replace', index=False)

