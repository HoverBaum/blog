title: Log to Elasticsearch using curl
tags:
  - elasticsearch
  - ci
  - logging
categories:
  - Technical
---

Once you get a centralized logging solution like Elasticsearch setup you open up an increadible amount of possibilities. That is, if you actually send logs to your central place. On a recent project we wanted to "just log our deploys from the Jenkins Server". Thus we set out to find the "easiest" way to log to our Elasticsearch. We finally reached curl as our solution and today I will walk you through how to do that.

![Good old tools can do amazing jobs.](https://storage.googleapis.com/hoverbaum-blog-assets/teaser-images/curl-to-elastic.jpg)

<!-- more -->

This post uses Elasticsearch version 7.

## Console basics

If like me, you find the command line highly fascinating for it's meriad of tools and how simple tools can be composed into bigger workflows, much like good software is, BUT you have no idea what you are actually doing (also like me), follow along as we dive into some tools we will use today. Should you be a firm terminal hero, feel free to jump this section.

### curl

[curl](https://curl.haxx.se/) is a free and open source commandline tool to transfer data over the internet. As such you can use it to make GET requests or send POST requests to servers. Let's look at a simple example you can try out:

```bash
curl https://jsonplaceholder.typicode.com/posts/1
```

This will fire a simple get request towards the [JSON placeholder API](https://jsonplaceholder.typicode.com/) and return the post with index 1. curl will log the result to the console for you. Through this logging to the console you can also chain curl with other commands, like grep the title of the post. But a deeper dive into chaining is a topic for another day.

```bash
curl https://jsonplaceholder.typicode.com/posts/1 | grep title
```

Let us instead focus on understanding curl and relevant options better. First we will need to specify the method (POST) we want to use. For that we can use the `-X` (pay attention to the capital X) option and pass in the method we would like to use. Our placeholder API from before also allows us to post to it, so let us try:

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts
```

Now we will want to send some data along. For this curl has `-d` to pass data along as well as `-H` to set a header. We want to set a header for the [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) so that we can tell receivers that we are sending JSON. Try the code below and also try it without the `-H` option to get a feeling for what we get by using it. You will notice that the placeholder API interprest out entire data as a key with no value specified. Setting the right Content-Type helps us to pass the data along in a way that the server can handle.

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts -H "Content-Type: application/json" -d '{"title": "My awesome title"}'
```

Read more abou the used options in the curl manual [-X](https://curl.haxx.se/docs/manpage.html#-X), [-H](https://curl.haxx.se/docs/manpage.html#-H) and [-d](https://curl.haxx.se/docs/manpage.html#-d)

### Timestamps and Environment variables

One thing that we wanted to add to all our logs are timestamps. Luckily our terminal comes with `date` which can print the current date in many formats. You can refer to the [Ubuntu docs on date](https://manpages.ubuntu.com/manpages/bionic/en/man1/date.1.html) for a documentation on date and this [article on formatting dates](https://www.cyberciti.biz/faq/linux-unix-formatting-dates-for-display/) for some inspiration. Genreally you call `date` with a single parameter that passes in what you want. `date +%s` for example gives you the seconds since Epoch. 

For Elasticsearch however we need [date fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html) in milliseconds since Epoch. And our timestamp will be a field of type date so that we can filter by it. To achieve that we will add nanoseconds and then devide by 1000000. (This will probably not work on Macs)

```bash
echo $(($(date +%s%N)/1000000))
```

Now that we have our timestamp we want to use it in our POST request to the server. For that we will save it in an environment varialbe and reference that in our request.

```bash
NOW=$(($(date +%s%N)/1000000)) && curl -X POST https://jsonplaceholder.typicode.com/posts -H "Content-Type: application/json" -d '{"timestamp": "'"${NOW}"'"}'
```

Let me unpack that for you: first we assign the timestamp to an environment varialbe called *NOW* the we tell our shell to execute another command without loosing context through `&&` here we make our POST request. You would usually expect to now see `${NOW}` wherever you want to insert the environment variables value. But to get our shell to properly escape here we need to wrap our value into wrapped single quotes.

A word on Operation System compatibility and escaping JSON values. Windows does not support single quotes `'` in the command so you will have to instead use something like: `"{\"title\": \"My awesome title\"}"`. you can also use that Syntax on Macs and Linux machines but I found the single quote approach nicer to read, even though it imposes the escaping for environment variables. Up to you.

TODO this needs some more explaining on what is happening, also with `$()` Syntax.

## Elasticsearch APIs

Elasticsearch provides a great documentation on how to [get started](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html) with understanding, creating and searching data.

### Indexes and Documents

First let's recap how Elasticsearch organizes data. The basic element in Elasticsearch is called a *Document*. You can think of it as a single event being logged, proccessed and organized through Elasticsearch. They are still organized in Types whic are grouped in Indexes. [Types however have been deprecated](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-concepts.html#_type) in Elasticsearch 6 (at the time of writing version 7 is current) which currently results in each Index only being allowed a single Type. Indexes on the other hand is a collection of similar documents where you can have as many Indexes as you want. You will need those to insert and query documents.

### Local Elasticsearch

If you want to follow along examples from here run an Elasticsearch locally using Docker or replace the localhost domain with the one running your Elasticsearch. Run the command below for a Docker container with Elasticsearch and check out the [Elasticsearch Docker documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) if you run into trouble.

```bash
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.0.0
```

### Inserting Documents

You can post to any `/index/type` to add a new Document to the specific Type of an Index. It you want to get a pretty response, add `?pretty` to the URL.

Say for example you wanted to have an Index that groups Documents related to a deployments of you application. The object you initially want to log might contain the hash of your code being deployed and thus look like:

```json
{
  "gitHash": "1234567"
}
```

To now add this Document to the *deploys* Index using *_doc* as the standard Type (remember deprecated but still needed) you can simple call the `/deploys/_doc` endpoint with your data.

```bash
curl -X POST "localhost:9200/deploys/_doc/?pretty" -H 'Content-Type: application/json' -d'
{
  "gitHash": "1234567"
}
'
```

If you are not running Elasticsearch locally but on a remote server replace `localhost:9200` above with your domain.

### Reading Documents

Now we can also get the document we just added by querrying the `/index/_search` endpoint.

```bash
curl -X GET "localhost:9200/deploys/_search/?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} }
}
'
```

This will return us all documents within the Index. Check out the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-search-API.html) for more options.

### Mappings

We breifly touched on Mappings before when we noted that Elasticsearch expects timestamps (or date in general) to be in milliseconds since Epoch. Through Mappings we can add fields to an Index. With this we can controle the *type* that a field will get. While all fields default to "text" it is especially important for our timestamp that we set the field to be of type "date" before adding the first document to our Index. By doing so we ensure that we can later properly filter on our timestamp in Kibana.

Sadly this means we now need to delete our Index again (and any Kibana indexes we might have created to explore our documents before the Elasticsearch Indexes).

```bash
curl -X DELETE "localhost:9200/deploys"
```

Now we can add the Index back with the right mapping for our timestamp using:

```bash
curl -X PUT "localhost:9200/deploys/_mapping" -H 'Content-Type: application/json' -d'
{
  "properties": {
    "timestamp": {
      "timestamp": "date"
    }
  }
}
'
```


## Logging using curl

Now that we have the fundamentals of curl and date down, as well as an understanding of Elasticsearches API, it is finally time to get logging.

We already locked at sending a message containing a timestamp via a POST request. With our knowledge of the Elasticsearch API we can now log an event with a timestamp.

```bash
NOW=$(($(date +%s%N)/1000000)) && curl -H "Content-Type: application/json" -X POST "http://your.domain:9200/deploys/_doc_" -d'
{ 
  "timestamp": "'"${NOW}"'"
}
'
```

With the above you should have a solid understanding of what is happening here. Below we will dive into a more complex example with once more an explanation of what is going on.

### A practical example

Above I used an example of logging something containing a git hash related to a deployment. That in fact was my use case to dive into this. Let me present to you my solution to log the currently deployed git hash from our CI Server to Elasticsearch.

```bash
HASH=$(git rev-parse --short HEAD) && NOW=$(($(date +%s%N)/1000000)) && curl -H "Content-Type: application/json" -X POST "http://your.domain:9200/deploys/_doc_" -d '{ "gitHash" : "'"${HASH}"'", "timestamp": "'"${NOW}"'", "info": "some infos", "environment": "test"}'
```

TODO: Multiline bash for readability

You can probably figure out whats going on here. Let me just go through it to make sure we are on the same page. First we have three commands in total, let's examine them one at a time.

`HASH=$(git rev-parse --short HEAD)` asigns the short hash (first 7 symbols) of the git repository we are currently in to an enviornment variables named *HASH*.

`NOW=$(($(date +%s%N)/1000000)` should look familiar, it gets the current timestamp in milliseconds and assigns it to a variable named *NOW*.

Finally we bring it all toger in one big curl command. Let me just go over the options for this one.

`-H "Content-Type: application/json"` sets a "Content-Type" header with "application/json" so that our Elasticsearch knows that it gets valid json.

`-X POST "http://your.domain:9200/deploys/deploy" ` tells curl to fire a POST request and which endpoint to use.

` -d '{ "gitHash" : "'"${HASH}"'", "timestamp": "'"${NOW}"'", "info": "some infos", "environment": "test"}'` finally defines the json data we are going to send along with our request. Here we use the variables we created to fill in the git hash as well as timestamp. Note that once again we use `"'"` for proper escaping.

## Conclusion

Today we learned a lot about curl, date and Elasticsearch. We not only took a look at how these tools can be use din isolation but also how to use them together to "just quickly log something".

I hope this sets you up to do great things and achieve quick wins empowered by your existing systems.

![](https://storage.googleapis.com/hoverbaum-blog-assets/emojies/emoji-memo.png)