# Scrape-Twitter-Data-Using-Python

#!/usr/bin/env python
# encoding: utf-8

import tweepy #https://github.com/tweepy/tweepy
import csv



#Twitter API credentials
consumer_key = "IC2MiUjOszYm8zDKl1TChksPp"
consumer_secret = "c5YV7oDhNGSkdEVBmlpz3aRF6V91IxslvPOMZuBSaEor5MXdia"
access_key = "61221956-FP0kvLEQNfnIkBljJFaGa7Ilh1WJQrX0M07M8M8x1"
access_secret = "lp0PJafD1vSt2vQH1uqnm1napGgPOU4Pz2Ppb0lrmfBq4"


def get_all_tweets(screen_name):
    #Twitter only allows access to a users most recent 3240 tweets with this method
    
    #authorize twitter, initialize tweepy
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_key, access_secret)
    api = tweepy.API(auth)
    
    #initialize a list to hold all the tweepy Tweets
    alltweets = []    
    
    #make initial request for most recent tweets (200 is the maximum allowed count)
    new_tweets = api.user_timeline(screen_name = screen_name,count=300,tweet_mode="extended")

      
    #save most recent tweets
    alltweets.extend(new_tweets)
    
    #save the id of the oldest tweet less one
    oldest = alltweets[-1].id - 1
    
    #keep grabbing tweets until there are no tweets left to grab
    while len(new_tweets) > 0:
        print "getting tweets before %s" % (oldest)
        
        #all subsiquent requests use the max_id param to prevent duplicates
        new_tweets = api.user_timeline(screen_name = screen_name,count=300,max_id=oldest,tweet_mode="extended")
        
        #save most recent tweets
        alltweets.extend(new_tweets)
        
        #update the id of the oldest tweet less one
        oldest = alltweets[-1].id - 1
        
        print "...%s tweets downloaded so far" % (len(alltweets))
        
    
    #transform the tweepy tweets into a 2D array that will populate the csv    
    outtweets = [[tweet.id_str, 
              tweet.created_at, 
              tweet.full_text.encode("utf-8"),
              tweet.retweet_count,
              tweet.favorite_count] for tweet in alltweets]
    
    #write the csv    
    with open('%s_tweets.csv' % screen_name, 'wb') as f:
        writer = csv.writer(f)
        writer.writerow(["id","created_at","text","retweet_count","favorite_count"])
        writer.writerows(outtweets)
    
    pass


if __name__ == '__main__':
    #pass in the username of the account you want to download
    get_all_tweets("jokowi")
