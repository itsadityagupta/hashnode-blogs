---
title: "Leetcode 355: Design Twitter"
datePublished: Sun May 23 2021 06:17:07 GMT+0000 (Coordinated Universal Time)
cuid: ckp0sg91f0icxdss1d57v3f7k
slug: leetcode-355-design-twitter
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1621750383256/rZA7un9Un.png
tags: python, computer-science, coding, competitive-programming, technical-interview

---

### Problem Introduction 📖
>Design a simplified version of Twitter where users can post tweets, follow/unfollow another user, and is able to see the 10 most recent tweets in the user's news feed.

>Implement the **Twitter** class:

>**Twitter()** Initializes your twitter object.

>**void postTweet(int userId, int tweetId)** Composes a new tweet with ID **tweetId** by the user **userId**. Each call to this function will be made with a unique **tweetId**.

>**List<Integer> getNewsFeed(int userId)** Retrieves the **10** most recent tweet IDs in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user themself. Tweets must be **ordered from most recent to least recent**.

>**void follow(int followerId, int followeeId)** The user with ID followerId started following the user with ID **followeeId**.

>**void unfollow(int followerId, int followeeId)** The user with ID **followerId** started unfollowing the user with ID **followeeId**.

>For examples, visit  [this](https://leetcode.com/problems/design-twitter/)  link.

### Understanding the problem 💡

At first glance, the problem seems to be quite straightforward. Just store the mapping between users and their tweets, and then between followers and those being followed. However, the crux of the question lies in the *getNewsFeed* function. Here, we have to return 10 recent tweets of the user and its followers combined. The "**recent tweets**" gives us an idea that we need to have an order for the tweets coming in (like a timestamp or a counter). And "**10 most recent tweets**" indicates that probably heaps can help us here.

Hence, we can have a counter starting from 1, considering it as a timestamp. Instead of storing a tweet, we'll store a tuple in the form *(timestamp, tweet)*, incrementing the timestamp after every addition of a tweet. Also, a separate mapping needs to be done to store the following users.


> An additional optimization can be to retain only the 10 recent tweets for each user and remove the least recent tweet as soon as the 11th tweet for a user comes in. This means that instead of having a list of tweets for every user, we can have a heap of size 10. This will save us memory because as per our problem, we need to return a maximum of 10 tweets. Hence, if a user has no followers, we need his 10 recent tweets only.

### Algorithm ⚙️


* Initialise the dictionaries for tweets and following, along with a timestamp.
* For **postTweet**:

>* Create a heap corresponding to a userId, in tweets dictionary, and add a tuple of (timestamp, tweetId) to that heap.

>* Increment the timestamp.

>* If the heap size is greater than 10, delete the least recent tweet for that user in the heap.

* For **follow**, add the followeeId to the list corresponding to followerId, in the following dictionary.
* For **unfollow**, if the follower follows the followee, then delete the followee from follower's list in following dictionary.
* For **getNewsFeed**:
>* Get all the users followed by userId from tweets dictionary.
>* Add userId itself to the list, as we have to consider it also, for returning the feed.
>* Get all the tweets from all the users in the above list.
>* Heapify the tweets
>* Get 10 recent tweets. (n-largest elements in a heap)
>* Separate timestamp from tweets.
>* Remove duplicate tweetIds.
>* Return the result

### Implementation 🖥️

```
from collections import defaultdict, OrderedDict
import heapq

class Twitter:

    def __init__(self):
        
        #initialization
        self.following = defaultdict(list)
        self.tweets = defaultdict(list)
        self.timestamp = 1

    def postTweet(self, userId: int, tweetId: int) -> None:
        
        #insert tweet to the heap
        heapq.heappush(self.tweets[userId], (self.timestamp, tweetId))
        self.timestamp += 1
        
        #if heap has more than 10 elements, remove the least recent tweet
        if len(self.tweets[userId]) > 10:
            heapq.heappop(self.tweets[userId])
        

    def getNewsFeed(self, userId: int) -> List[int]:
        
        #get all the followers of given user
        all_users = self.following[userId].copy()
        
        #add user itself to the list
        all_users.append(userId)
        tweets = []
        
        #get tweets from all the followers and user itself
        for user in all_users:
            tweets.extend(self.tweets[user])
        
        #heapify tweets
        heapq.heapify(tweets)
        
        #get 10 recent tweets
        recent_tweets = heapq.nlargest(10, tweets)
        
        #seperate timestamp from tweets
        recent_tweets = [t for x, t in recent_tweets]
        
        #remove duplicates from tweets by using dictionary
        recent_tweets = list(OrderedDict.fromkeys(recent_tweets))
        
        #return the result
        return recent_tweets

    def follow(self, followerId: int, followeeId: int) -> None:
        
        #add entry in following table
        self.following[followerId].append(followeeId)
        

    def unfollow(self, followerId: int, followeeId: int) -> None:
        
        #if given follower follows followee, then only remove followee
        #from follower's list
        if len(self.following[followerId]) != 0:
            if followeeId in self.following[followerId]:
                self.following[followerId].remove(followeeId)
```

### Complexity Analysis✔️

**postTweet**: Even though the complexity of adding an element to a heap is O(logN), we will have an insertion complexity of **O(1)**. This is because we are maintaining a heap size of 10 which is a constant.

**getNewsFeed**: Heapifying *recent_tweets* list takes O(logN) time. Finding nlargest elements takes O(N + klogN) where k= 10 => O(N). Operations after that will take O(1) time as *recent_tweets* will always have maximum size of 10. Hence, overall complexity will be O(logN + N + 1) => **O(N)**.

**follow**: We are simply adding an element to a list. So, time complexity will be **O(1)**.

**unfollow**: As we are searching for the followeeId in the list of followees, it's time complexity will be O(N)

### Alternate approaches / Feedback ? 📝

Thanks for reading the article! If you have any other approaches, comments, or feedback, do let me know in the comments!