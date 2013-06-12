---
layout: post
title: "Using the ServiceStack.Redis Client"
date: 2013-06-12 11:53
comments: true
categories: 
---

#Using the ServiceStack Redis Client to Work with Redis Lists (ServiceStack.Redis)

I'm not going to focus on using Redis as a Cache, this is pretty straight foward and a lot of what I am going to talk about can be
applied to using the Redis as a Cache.

First: Familarize yourself with Redis.

I would suggest checking out redis.io (the main redis site) and giving The Little Redis Book (http://openmymind.net/2012/1/23/The-Little-Redis-Book) a read.

Redis is 'just' a key-value store, but it allows the values to be actual data structures instead of just strings. ServiceStack has taken this farther for us and provided serialization interfaces to allow us to use a Typed client to interact with redis without worrying about this ourselves.

Second: Get Redis running locally

The fastest way to do this is to download the latest version from https://github.com/dmajkic/redis/downloads and start the redis-server.exe on your machine.

Third: All hail NuGet.

`Install-Package ServiceStack.Redis`

Fourth: Connecting to Redis

ServiceStack has made this simple. The easiest way to do this is like this

    using(var client = new RedisClient())
    {
        //code goes here
    }

This works great but it creates a new client everytime we connect to redis. You can however do the following into your IoC container

    Container.Register<IRedisClientsManager>(new BasicClientManager("http://localhost:6378")) 
or
    Container.Register<IRedisClientsManager>(new PooledClientManager("http://localhost:6378")) 

then instead of calling `new RedisClient()` you can call `Container.Resolve<IManagedRedisClients>().GetClient()`!

Which one should I use?
Both of these implementations will pool your clients. `BasicClientManager`, is best when your redis server is on localhost `PooledClientManger`, is best when your reids service is on a remote machine.
So now that you've got your application hooked up to redis what can you do?

The simplest thing to do with Redis is just store and read some values.

    using(var redis = new RedisClient())
    {
        redis.SetEntry("name", "Michael Sarchet");
        console.log(redis.GetEntry("name"));
    }
    What if we want to store a value that is a type

    public class DataType
    {
        public string Name { get; set; }
    }
     
    using(var redis = new RedisClient())
    {
        var client = redis.As<DataType>();
        client.SetEntry("Data1", new DataType() { Name = "Michael" });
        console.log(client.GetEntry("Data1").Name);
    }
    //Remember in a production environment you are going to want to use the IRedisClientsManager.GetClient() method, since this is the thread safe way to get a client. For now we can just use new RedisClient().

Now we've just stored a value with a type into Redis. This is done via the Asmethod on the client. This tells the client to serialize and deserialize the values as that type.

If you open up the redis-cli and run

    get Data1

you should see something that looks like

    {\"Name\" : \"Michael\"}
    
This is becuase ServiceStack.Redis is serializing these values, and handling the deserialization on the read. This covers the basic use case for Redis, storing some data and reading it back from Redis. The client exposes most (if not all) of the redis command line actions directly. Play around with it and get a feel for how this will work in your application.

Now that you are more familiar with how the RedisClient works directly we will look at using the List structure that redis provides. This is one of the more directly useful things that you can do via redis without getting into a lot of the specific redis functionality.

So let's look at persisting a larger more complex object and what we would need to do to access a list of these objects:

    public class Customer
    {
        public string Name { get; set; }
        public string Phone { get; set; }
        public string Address { get; set; }
    }
     
    public class CustomerAccess
    {
        public static IList<Customer> Customers 
        { 
            get
            {
                using(var client = new RedisClient())
                {
     
                    TClient = client.As<Cusomter>(); //woah I'll get to this don't worry
                    return TClient.Lists["cusomters"].ToList();
                }
            }
        }
     
        public static void AddCustomer(Customer customer)
        {
            using(var client = new RedisClient())
            {
                TClient = client.As<Cusomter>();
                TClient.Lists["customers"].Add(customer);
            }
     
        }
     
        public static bool RemoveCustomer(Customer customer)
        {
            using(var client = new RedisClient())
            {
                TClient = client.As<Customer>();
                return TClient.Lists["cusomters"].Remove(customer);
            }
        }
    }
What just happened?

`client.As<T>` returns a IRedisTypedClient which just handles the serialization for us.

`TClient.Lists[key]` gives us access to an `IRedisList` (which is the wrapper for the internal Redis List data structure). `IRedisList<T>` implements `IList<T>` which lets us access it like a normal list.

Internally ServiceStack is handling all actions to call these methods (proTip! most of these are native to redis  so it's super fast)

If you notice in the Customers Property I called `Tclient.Lists["customer"].ToList()`, this is because if you just return the `IRedisList` any time you act on it it will reopen a connection to redis, which can be kind of bad. Instead we are just getting a snapshot of the data at that time. However if you wanted and you ignored the `ToList()` you could add data and then get it and it will all come directly from redis, which can be useful in the right circumstances.

My next post will cover the Set and Hash Implementations and some of the more powerful things you can do with those.
