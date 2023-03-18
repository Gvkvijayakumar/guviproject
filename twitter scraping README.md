# guviproject
#Twitter scraping 
%%writefile streamlitapp.py
import streamlit as st
import pandas as pd
from datetime import date
import pymongo
import json
import snscrape.modules.twitter as sntwitter
from PIL import Image
st.title("Twitter Scraping")
today=str(date.today())
def Scrapingdata(Hashtag,start_date,End_date,tweets_count):
      tweets_list=[]
      for i,tweet in enumerate(sntwitter.TwitterSearchScraper(f'{Hashtag} since:{start_date} until:{End_date}').get_items()):
           if i>=tweets_count:
               break
           tweets_list.append([tweet.date, tweet.id, tweet.content, tweet.user.username, tweet.url, tweet.retweetCount, tweet.source, tweet.likeCount,tweet.replyCount,tweet.lang])
      return(tweets_list)
menu=["Home","About","Search","Display","Download"]
choice=st.sidebar.selectbox("menu",menu)
if choice=="Home": 
    st.markdown("_This app is a Twitter Scraping web app created using streamlit. It scrapes the twitter data for the given hashtag/keyword for the given period. The tweets are uploaded in Mongodb and can be download as csv or json file._")
    img=Image.open("/content/elonmusk twitter.jpg")
    st.image(img)
elif choice=="About":
    with st.expander("Twitter Scraping"):
        st.write("Twitter Scraper will scrap the data from Public Twitter Profiles.It will collect the data about***date,id,url,tweet content,reply count,retweet,count,language,like count,followers and lot more information***t to gather the real facts about the Tweets")
    with st.expander("Snscrape"):
        st.write("Snscrape is a scraper for social media services like ** twitter, facebook, instagram** and so on. It scrapes **user profiles, hashtages, other tweet information** and returns the discovered items")
    with st.expander("Mongodb"):
        st.write("mongodb is an open source document database used for storing unstructured data. It is used by developers to work easily with real time data analysis,content management and lot of other web applications")
    with st.expander("Streamlit"):
        st.write("Streamlit is a awesome opensource framwork used for building highly interactive sharable web applications in python language. Its easy to share machine learning and datascience web apps using streamlit. It allows the app to load the large set of dates from web for manipulation and performing expensive computations.")
elif choice=="Search":
    Hashtag=st.text_input("Enter Hashtag or Keyword")
    start_date=st.date_input("Enter starting date:(YYYY-MM-DD)")
    End_date=st.date_input("Enter end date:(YYYY-MM-DD)")
    tweets_count=st.number_input("Enter Tweet count",min_value=1,max_value=1500,step=2)
    if Hashtag:
        submit=st.checkbox("***Scraped TWeet***")
        if submit:

             data=Scrapingdata(Hashtag,start_date,End_date,tweets_count)
             st.success("Data scraped successfully")
             def data_frame(data):
                    return pd.DataFrame(data,columns=['Datetime', 'Tweet Id', 'Text', 'Username',"url","retweetCount","source","like_count","replycount","lan"])
             df=data_frame(data)
             client = pymongo.MongoClient("mongodb+srv://vijayakumarG:vijay12345@cluster0.mzuhccl.mongodb.net/?retryWrites=true&w=majority")
             db = client.Twitter
             records=db.scrapping
             scr_data={"Scraped_word":Hashtag,"Scraped_date":today,"Scraped_data":df.to_dict("list")}
             records.delete_many({})
             records.insert_one(scr_data)
             st.success("upload to mongodb Successful")
             

    else:
         st.checkbox("scrapedtweet",disabled=True)
elif choice=="Display":
    Hashtag=st.sidebar.text_input("Enter Hashtag or Keyword")
    start_date=st.sidebar.date_input("Enter starting date:(YYYY-MM-DD)")
    End_date=st.sidebar.date_input("Enter end date:(YYYY-MM-DD)")
    tweets_count=st.sidebar.number_input("Enter Tweet count",min_value=1,max_value=1500,step=2)
    list_tweet=Scrapingdata(Hashtag,start_date,End_date,tweets_count)
    def data_frame(data):
         return(pd.DataFrame(data,columns=['Datetime', 'Tweet Id', 'Text', 'Username',"url","retweetCount","source","like_count","replycount","lan"]))
    df=data_frame(list_tweet)
    submit=st.checkbox("View Dataframe") 
    if submit:
          st.success("Dataframe")
          st.write(df) 
elif choice=="Download":
    Hashtag=st.sidebar.text_input("Enter Hashtag or Keyword")
    start_date=st.sidebar.date_input("Enter starting date:(YYYY-MM-DD)")
    End_date=st.sidebar.date_input("Enter end date:(YYYY-MM-DD)")
    tweets_count=st.sidebar.number_input("Enter Tweet count",min_value=1,max_value=1500,step=2)
    tweetlist=Scrapingdata(Hashtag,start_date,End_date,tweets_count)
    def data_frame(data):
         return(pd.DataFrame(data,columns=['Datetime', 'Tweet Id', 'Text', 'Username',"url","retweetCount","source","like_count","replycount","lan"]))
    
    def convert_csv(df_input_csv):
          return df_input_csv.to_csv().encode("utf-8")

    def convert_json(j):
          return j.to_json(orient="index")

    df=data_frame(tweetlist)
    csv=convert_csv(df)
    json = convert_json(df)
    st.download_button(label="Download csv",data=csv,file_name="file.csv",mime="text/csv") 
    st.download_button(label="Download json",data=json,file_name="file.json",mime="text/csv")
