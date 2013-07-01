---
layout: post
title: "Using ServiceStack.Redis Part 2 Sets and Hashes"
date: 2013-06-12 14:11
comments: true
categories: 
---

## ServiceStack.Redis Sets and Hashses

In my last post about redis I talked about getting set up with Redis and using the ServiceStack.Redis IRedisList to empower yourself as a developer.

This post is a quick overview of the IRedisSet<T> and IRedisHash<TKey, T> that are available in ServiceStack.Redis. I'm not going to spend much time talking about the non-generic implementation of both of these interfaces since for the most part the function like calling them with T as string

Set and Hash are much more powerful data models for manipulating your data in Redis directly

### Sets

If you take a quick glance at redis sets they don't look that different from lists. In fact when you get data back from `ServiceStack.Redis` you get it as an `IList` which is fantastic, but there are a few important things to know.

If you're using the redis set operations, (sort, intersect, etc) you will find that you will have problems with types that have over loaded equaility. This is because these operations are actually being done in redis, and you are comparing them byte to byte (in most cases).So Sets are great if you are using a primative type that exists in redis.

PROTIP: Set's are great but by nature they are going to take up more memory than just a list, this is something to think about if you don't really need the power in the set.

How do you make a set and manipulate it:

    //set up the connection
    //cast it into the Generic type we wish to use
    using(var redis = new RedisClient())
    using(var client = redis.As<string>)
    {
        //Let's first get a reference to a set that we want to manipulate
     
        //As most ServiceStack.Redis actions this requires some kind of key
        var set = client.Sets["Set"];
     
        set.Add("bill");
        set.Add("bob");
        set.Add("jeremey");
     
        var otherSet = client.Sets["OtherSet"];
     
        otherSet.Add("bill");
        otherSet.Add("jeb");
        otherSet.Add("kermin"); 
     
        //HERE ARE SOME THINGS WE CAN DO
     
        //Returns the intersect of the sets
        //bill
        var intersect = set.GetIntersectFromSets(otherSet);
     
        //Returns a HashSet<T> containing unique items from both sets
        //bill
        //bob
        //jeremy
        //jeb
        //kermin
        var union = set.GetUnionFromSets(otherSet);
     
        //Returns a HashSet<T> containing the differences between the sets
        //bob
        //jeremey
        //jeb
        //kermin
        var differences = set.GetDifferencesFromSets(otherSet);
     
    }
     

### Hashes

What's so great about Hashes? Well a lot of things but one of the best things to know is that a hash is basically a Dictionary (in the case of the non-generic hash it's just a Dictionary). This makes the hash very useful for storing objects if you need to manipulate individual properties on the object directly and you don't want to manipulate the entire object.

    ///boilerplateAction you've seen this before
    using(var redis = new RedisClient())
    using(var client = redis.As<string>())
    {
        //so with hashes you have to get a hash first before you can manipulate it
        var hash = client.GetHash<string>("hashID");
     
        hash.SetValue("key", "value");
     
        //get an individual value from a hash
        var individualValue = hash.GetVlauesFromHash("key");
     
        hash.SetValue("otherKey", "value");
     
        //get all the values from a hash
        var allValues = hash.GetAllEntriesFromHash();
     
     
    }
 
The main thing to remember with hashes is that you first have to get the hash before you can manipulate values in the hash. This is because all of the hash methods are extensions of `IRedisHash<TKey, T>` for the generic hash.

This should provide you with an idea of where to get started when using ServiceStack.Redis to manipulate Sets and Hashes. These are powerful tools that can really allow you to quickly manipulate your data and make it available across your application.
