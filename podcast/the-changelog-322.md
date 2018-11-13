**Jerod Santo:** We have a two-pronged episode, two for the price of one today, and the price of one is zero, or free, so we're really luckin' out. We're here to talk, first of all, about Dgraph, which is the world's most advanced graph database, according to Dgraph.io. Then we're also going to talk about some licensing and some re-licensing woes, some of the stuff that open source developers and popular projects have to go through, but are kind of the difficult, weedy, "how do we do this, how do we re-license if we change our mind?" and Manish has done all that with Dgraph. It's gone through a few different iterations of licensing, and he's here to tell us that story. Manish, thanks for coming on the Changelog.

**Manish R Jain:** Thanks for having me, guys.

**Jerod Santo:** And we should probably give a shout-out to Ping, because this is an episode that started in Ping; if you've never heard of our Ping repo, it's on Github at thechangelog/ping. Hop in there, give us your thoughts on what shows we should do. This one was actually opened up by an epic transcriber, Horst Rutter... Adam, you know Horst.

**Adam Stacoviak:** Yes.

**Jerod Santo:** He has been faithfully fixing and improving our transcripts.

**Adam Stacoviak:** The "unintelligibles" are missing. When it's "unintelligible", Horst goes in there and corrects it. He has done a ton, and we appreciate that.

**Jerod Santo:** And he was interested in hearing about some of the decisions and some of the process of how do you change your license from one to another, and then a follow-up to that was vespertilian, a.k.a. Cameron, which is probably the real name, who pointed us at Dgraph as a user of Dgraph, and one who had watched the Common Clause License and the Apache 2.0, and the AGPL, and all of this over the last 6-8 months happening over at Dgraph. He said that this would be a good project to focus on that conversation, so thanks to those two for being a part of our community, and thanks for suggesting this and getting us hooked up with Manish.

With that out of the way, Manish, let's talk about Dgraph. Tell us about this project, where it came from, how long it's been around and what you're up to with it.

**Manish R Jain:** \[00:04:09.17\] Sure. Maybe I can start with my own journey a bit before I get into Dgraph. I used to work at Google in Mountain View, California for six and a half years, working in the web search infrastructure team. There we were dealing with real-time distributed systems. In fact, we built an incremental indexing system and launched that in 2010, got an \[unintelligible 00:04:32.02\] award for it... Basically, what that did was to reduce the latency that it takes for a web page to go from the first time we crawl it to the first time a user sees it on Google or Chrome from four days to a few hours.

**Jerod Santo:** Oh, wow.

**Manish R Jain:** That was the biggest Bigtable database installation at Google at the time, and it gave me a lot of freedom to work on real-time distributed systems. Now, back in 2010, after we launched this thing, I started looking around and seeing "Hey, what else could I dig my teeth in?" Google had acquired Metaweb, which is the company which brought Knowledge Graph to Google... So the Knowledge Graph that is here these days came from Metaweb... And I started a couple of projects there.

One of the projects was to unite all structured data at Google, that was all of what we call OneBoxes - that would be weather, and events, and movie showtimes, and flights etc. - and the Knowledge Graph into a single graph indexing and serving system. That was a big challenge, obviously.

We didn't have a graph serving system at Google; we had a web search index serving system, but not a graph one. So along with a few other tech leads - one was in India, one was in San Francisco, and I was in Mountain View - we started this project to build something which would be able to do \[unintelligible 00:06:06.03\] and would do traversals, and do them in a sub-second latency. In fact, we had a limit on how much latency it can have, because if the system does not respond to a web search request internally, that search would just move on and would not surface anything interesting from the Knowledge Graph.

So I was involved in that, and while building that, we obviously put together all the research that Google had done at that time, and I got to learn a lot. I left Google in 2013, I moved from the U.S. to Australia - I had some family reasons to move - and around 2015 I remember being involved in a freelancing gig where this person is like "Hey, can we use a graph database?" I was like, "Well, the existing graph databases - they are not that good. They don't scale very well, they have issues with consistency, and in general are just never considered primary databases." And that's what triggered me to say "Hey, maybe we should build something like that."

I looked around, and the biggest one was Neo4j, which is a single-server database... In fact, the most popular one on the market, but limited by data corruption issues, and performance issues. Then there were some others which were not databases, but more like graph layers. You would think of TitanDB, DataStax (DSE Graph), JanusGraph, which are built on top of other distributed databases. So you'd put HBase below it, or you'd put Cassandra... And when you put a layer above, it (again) suffers from performance; you need to run multiple systems.

\[00:08:02.03\] Dgraph really started as a way by which we could have a native graph database which could also scale horizontally, and perform with a pretty tight latency. I used a lot of concepts that I learned about at Google. On top of that, while we were building it, we realized if we were to build a database which has to be a primary database for big companies, it must support transactions, it must support synchronous replication, it must provide \[unintelligible 00:08:32.28\] because when you build these things into the database, applications have it a lot easier; they don't need to worry about whether they're hitting the master or the replica... They don't have to worry about any of that; they just hit any of the servers in the cluster and they are guaranteed to get the freshest response back. So those were the ideas that we built Dgraph for.

I launched 0.1 in December 2015. We went on to raise three million dollars over the course of two years, launched 1.0 in December 2017, and now we are in a place where Dgraph is close to being used in production at a few big companies. And obviously, we have a huge open source community.

**Jerod Santo:** Very cool. Well, you mentioned Neo4j - just in the news yesterday; I believe they raised a series E -- the company behind Neo4j. 80 million dollars series E, so definitely investment interest in this space, and Neo4j has been around for quite some time. You said Dgraph's advantage is that it's built for distributed from the ground up, and also potentially some of the technology, or just the timing of Dgraph in terms of it starting in 2015... Can you give some of the underlying technology languages or tools that you're using in the open source software, and speak to that for us?

**Manish R Jain:** I'm a big fan of Go language, and this was not when I was at Google; I was pretty much writing C++. But after I left, Python just could never stick with me, and the moment I got to know about Go, I started trying it out. Back in 2015, CockroachDB, another database company in New York - they had raised a series A; I saw their stack was Go, and that immediately excited me.

Dgraph is done purely in Go. We use gRPC for communication, both internal, between the cluster, and for external communication from the client to the cluster. We were initially using RocksDB as an embedded key-value database to put our data in, but then we realized that when you go from Go space to cGo to C++, which is where RocksDB is done in, it just causes a lot of headache. Go tools don't get to see the memory profilers; for example, you don't get to see what's happening in C land... The Go performance profilers do not get to see what's happening in C land either... So at some point, after much thought, we decided that we should just build a good RocksDB alternative in Go.

We looked at the alternatives at that time... One was BoltDB, which was a B+ tree-based key-value database. Then there was obviously LevelDB. RocksDB was already an improvement over LevelDB, so for us that seemed like another great choice.

\[00:11:53.03\] BoltDB's write performance -- and not just BoltDB, but in general, any B+ trees' write performance is definitely always a bottleneck... So we wrote something which was based upon a new paper by the University of Wisconsin-Madison. What it did was it took some of the negatives of LSM trees and spread it by separating the values from the keys. So the values go into a log, and the keys go into the LSM tree. We based our main design upon that... And it took us a while to really get it right, because the paper didn't talk about all the nuances involved with having a separate value log, so that's something that we have been sort of perfecting over time.

The end result was that the performance of this thing called Badger basically outperforms RocksDB on a lot of use cases. It works out pretty well for us, so we use Badger as the underlying embedded key-value database.

**Jerod Santo:** Very cool. One thing you mentioned earlier - you said that many people were using graph databases not for their primary data store, but as perhaps a secondary data store. Maybe they put their social network-style data in the graph database, but maybe they have a more traditional relational database management system for their primary tables... Can you give a high-level decision -- of course, once you decide "I need a graph database", you may say, okay, Dgraph, or Neo4j, or perhaps a proprietary option, but what about even like "Do I need a graph database versus a Postgres or a MySQL?" Help people with that decision. Is there a pretty simple flow you can go through in your mind to decide "Is this the data store for me?" especially if you're gonna pick it as a primary?

**Manish R Jain:** That is a tough question for a lot of people. MySQL and Postgres have been around for such a long time. Literally, SQL is being taught in schools and colleges all over the world. It's hard to convince somebody who was, let's say, a Postgres fan or a SQL fan to switch to something else... I try not to engage directly or try to convince anybody to use graphs. What happens for us as the companies -- so Postgres and MySQL are very popular with the young startups, but as they progress and they start to realize the limits of these systems, the limitations of the \[unintelligible 00:14:42.22\] the limitations of not being able to do recursive queries across tables, and stuff... All that code that goes into the application because the database is so simple, as the company size grows, they start to hit those limitations... And at some point, a new project would be like "Hey, it would be great if we had a graph database for this. It would really save a lot of work." Or "Hey, we tried this with SQL. It's just too slow for our users. Maybe we should switch over to a graph database." So that's what happens. Then they start looking into a graph database. Obviously, they come across some of the popular choices, they try them out, and then accidentally almost, they get to hear about Dgraph, and that sticks.

**Jerod Santo:** So it's one of these things where you'll know it if you'll need it, because you'll have grown past certain needs potentially in your traditional relational database. That makes it actually a pretty nice space for an enterprise offering, because your community is enterprise. It's companies that have grown, at least data-wise, to a size where they feel the need already, so they're probably a certain level of successful, at least hopefully.

**Adam Stacoviak:** \[00:16:08.05\] Or they're even doing special things with their data more so than simply like "Hey, we have a web app with basic CRUD." MySQL, Postgres - those databases are perfect and great and fine for those types of apps, but once you're past a certain point, you won't actually make more sense, or get insights, or analytics that really draw relations or different things from a database. You may want to experiment, and even use in addition to, versus simply replacing.

**Manish R Jain:** Yeah, absolutely. We do see some of these medium to big-size companies - I think they're the most active users of graph technologies. Even if you were to look at yesterday's news article about Neo4j getting the 80 million dollars, they said that 20 out of 27 (or 24) top banks in the U.S. are using Neo4j, so it gives you some idea for how popular graphs are with enterprises. But you know, I do wanna say one thing though... I feel -- and we've actually done some work on that as well... Even for some basic stuff which you typically think is squarely in the SQL space, for example building a question-answering website, right? You have Quora, you have Stack Overflow, and you have a bunch of these things (even Facebook) where you have a post, you have comments on the post, you have likes on those comments, you have comments on comments, likes on those, and so on and so forth - it's a very recursive sort of... You know, if you need to show a post, it's a recursive traversal, and that's exactly what graphs are great at.

What we did, for example, (I think it was) last year was we -- Stack Overflow does these data dumps that you can just pick up as an XML file; you can pick it up and you can do whatever with it. So we picked that up and we loaded that into Dgraph, and we thought "Hey, let's build the three most popular pages on Stack Overflow." One of them is the questions page, one of them is the homepage, and there was one more page, I forget which one it was... And we just built those three pages. The amount of back-end code that we needed was not that much, because the query language, in this case of Dgraph, was sufficiently complex that it could just retrieve all the data for you and give it to you in a nice JSON, so all the work that needs to be done is just in the front-end in rendering it, as opposed to in the back-end where you pick up the question from the questions table, then pick up the answer from the answers table, pick up the likes and upvotes from another table, and then try to join them together... You don't have to do any of that code; it just happens automatically at a database level.

So I feel graphs can be used in a lot more broad way, and they are a lot nicer and faster for developers, but that level of developer awareness - that takes some time to build.

**Jerod Santo:** Yeah. That's a great idea for getting people to see how easy it is to build these recursive data fetches, is to use something we're all very well aware of, which -- I don't know, do you think developers know what Stack Overflow looks like, perhaps...? \[laughter\] Also, you another cool one on your homepage - play with 21M facts from the Freebase Film Data, load it up on a demo Dgraph instance, so you can just hop in there and see what different queries will look like.

Speaking a little bit to the timing of Dgraph in terms of its competitive advantage over potential other graph databases is its query language is inspired by GraphQL, which just couldn't have been inspired by GraphQL if it was ten years ago... So this is something that's very familiar, at least to front-end web developers. Can you talk about that?

**Manish R Jain:** \[00:20:09.29\] Yeah, I think GraphQL was a great choice for us. It was very early on, in fact... I think Facebook had just released GraphQL, and I remember looking at it and I'm like "This just fits", because when you go to a graph database, you want to get a subgraph back; you don't wanna get a list back... Because if you get a list back, it's hard to know what was connected to what. You cannot create a subgraph from a list, but you can take a subgraph and convert it to a list. And most of the other graph queries - Cypher and Gremlin - they are all returning lists of things back, just like SQL does, so they lose some of that relationship data between things.

I looked at GraphQL and I was like, "Hm, this s very interesting." In fact, I went back and checked with the CTO at Metaweb who was at Google; I showed it to him, and I was like "What do you think about this?" He said that he was very close to Metaweb's own query language, called MQL, which was popular at the time. So we decided, "Hey, let's use this as a query language."

Now, the thing with GraphQL that we did not realize at the time was that it was really a replacement for REST APIs, and it was still designed keeping SQL in mind \[unintelligible 00:21:36.25\] think of them as SQL tables, and the connections are similar. So we decided to quickly hit some of the seams of GraphQL, where we felt like we could not really work with it if we wanna build a graph database... So we had to then start to modify the spec, and basically go outside of the spec and modify... We simplified some, we added some features \[unintelligible 00:22:08.12\] we added filters in a simple way, and so on and so forth... And we still don't have a good name for this language; we just call it GraphQL+-, because we added some and we removed some. \[laughter\]

**Adam Stacoviak:** It's the first I've heard that.

**Jerod Santo:** That's hilarious. I was looking at that +- and I thought maybe it was like a typo there, because it looks like it was accidentally in the link, but... Yeah, that's a good name - just plus some and minus some.

**Adam Stacoviak:** Is ++ still being used often? I feel like it's had its heyday... I remember it from maybe ten years ago, maybe even eight; I don't know, it doesn't seem like it was a couple. Is it still kind of a current, known naming pattern? ...like a hacker pattern, something++?

**Jerod Santo:** I think so. I think it's still out there. Hackers are still typing daily, for sure...

**Adam Stacoviak:** Okay...

**Jerod Santo:** +- is brand new.

**Manish R Jain:** We still use C++, and we were like "Hey, is it GraphQL++?"

**Adam Stacoviak:** Sure.

**Manish R Jain:** But then I was like, "Well, it doesn't do everything that GraphQL does, so it would be wrong to call it ++. It has to be +-."

**Jerod Santo:** I dig it. So is that something that potentially those pluses (or maybe the minuses) could work their back into GraphQL, or is it just because working with graph databases there are things that just don't make sense for the broader web API GraphQL?

**Manish R Jain:** \[00:23:36.29\] Honestly, that's the question on my mind almost every other day. We do see how popular GraphQL has become; in fact, this has become way more popular than I anticipated... And there's an open ticket on Dgraph to support the official GraphQL spec, so it will play well with all the tooling out there. Apollo raised a bunch of money and Apollo is being used quite a lot in the GraphQL community... And we would like GraphQL to play well with all of those tools.

I think that is definitely something that we wanna do, is to support the official one. It probably takes a deeper discussion with the authors of GraphQL to see if they would like to integrate some of the modifications that we have done back into the spec. That's probably a harder discussion thought.

**Break:** \[00:24:37.06\]

**Jerod Santo:** Manish, help us understand some of the killer use cases, the sweet spot for graph databases. Similar to the idea of -- I think Mongo came out really talking about document-based data stores and saying "If you're running an e-commerce site such as Magento, look at all these crazy joints on these different tables just to pull together a shopping cart. Really, that's a document, so let's have a document database", and that was a compelling use case, or at least selling point for that style data store. When I think of graph databases, I think of social networks, but that's just me. From your perspective, what's the sweet spot for these types of data stores?

**Manish R Jain:** Yeah, so there are certain uses cases where people immediately think about using a graph database, and I think there's a sweet spot there. The top one which comes to my mind is real-time recommendations; these days companies have a lot of data around the users, for example... You have \[unintelligible 00:26:49.04\] and you have rewards cards from even the airlines, or hotels, or e-commerce companies around what users have purchased in the past, and what other people have purchased... Amazon comes to mind; Amazon runs an amazing recommendation system. That's one of probably the most demanded features or the most demanded use cases from a graph database.

Then we have seen particularly medium to big companies go really hard after real-time fraud detection. It's very easy in a graph to find circles where they can identify if there's the same person or entity trying to create multiple cards, or multiple money sources, and figure out if it's \[unintelligible 00:27:41.03\]

\[00:27:45.06\] We have also seen identity reconciliation. People trying to figure out the same person on Instagram, Facebook, Twitter, and so on and so forth. So those kinds of reconciliations - now you can apply them to other data sources. That's actually a good usage for graphs.

And the last one - this is the most relevant to particularly big companies... They have a lot of data silos. They have a lot of different databases, or even just different database instances where they actually grab data, and one silo never talks to the other. What they then do is they unify all of the data from these different silos into a graph database, because remember, graph databases do not have any boundaries; the idea of graphs is that you just put all the data into one place, and it can traverse from any node in the graph to any other node, however far away it might be. There's no tables, there's no different databases, it's just one graph. That concept really helps when you want to query across multiple data sources.

The fifth one, which is really jumping up these days, is around artificial intelligence. There was a paper by Google that I was reading last week around how they realized that they have reached the limits and they need to use a graph database to be able to do better AI. They even launched a small graph library that you can use to integrate with TensorFlow. In fact, just reading it from yesterday's post around Neo4j funding, AI was the top thing that they're gonna go after with the new money that they're getting... So I think for AI, graphs are a no-brainer.

**Adam Stacoviak:** If you had to give somebody a graph database 101, would you just say it's like a string that threads different data points, and that string (as you said there) can infinitely scale? If you had to give a 101 of what a graph database is, how long might that be and could you do it here?

**Manish R Jain:** Absolutely. I think graphs are probably the simplest things to think about, really. People think about SQL tables - you have a row and you have some columns. Think of a graph as three columns where you have a subject, a predicate, and an object. If you put together a whole bunch of these things, you get a graph. A subject is essentially -- think of it as an entity; a predicate is the relationship, and the object is either another entity, or a value.

The subject could be, let's say, me. The relationship might be "lives in" and object might be "San Francisco." Or it could be me - the name is Manish, and that's sort of like a property. So you just put together a whole bunch of these facts, or triples, and you get a graph.

Then other people who live in San Francisco would have similar facts, and then you could run a graph query around "Hey, tell me all the people who live in San Francisco and who eat sushi." So you pick up all the people who live in San Francisco, you intersect with people in the world who eat sushi - which are completely different facts. You didn't create them as "This person lives in San Francisco AND eats sushi." This is something that we're doing on the fly. So you pick up all the people in San Francisco, you pick up all the people in the world who eat sushi, you intersect the two lists, and now you get people in San Francisco who eat sushi. Now you can take that result and say "Intersect it with all the people who have been to Japan." You pick up another list of people who have been to Japan, intersect it with this, and now you get people who live in San Francisco, who eat sushi, and who have been to Japan. So the power of graphs is really in these joins that you can do, coming from just very simple facts.

**Adam Stacoviak:** \[00:32:12.29\] It makes sense too why in part one you mentioned not having to rewrite a bunch of code. When you explained it in the 101 that these things naturally appear based on the way you query the data, versus traditional ways you might have done it with MySQL, or Postgres, or relational databases... In this case, the graph or these points become more and more clear as you intersect across over the data, because it's just naturally how it works... And you're saving time, but also insights that were just so much harder to get to in traditional ways, or other database ways.

**Manish R Jain:** That's absolutely true. I was playing with the Freebase movie set that we have also on our website, and one of the interesting things is that you can look at the data all you want and never really find these tidbits; but I put it into Dgraph and run some queries, and it turns out that the directors of Indiana Jones movies were also in the movie. Steven Spielberg was in one of the Indiana Jones movies as one of the characters in the movie.

**Adam Stacoviak:** Yeah.

**Manish R Jain:** There's all these interesting things - they just become really obvious when you put them in a graph.

**Adam Stacoviak:** That's interesting.

**Jerod Santo:** So you add that, the built-in asset transactions, which gives you a lot of safety... And what are you missing then? Is everything better in graph database land, or are there things that relational databases still do better today? What are the drawbacks?

**Manish R Jain:** I used to say the drawback was that Dgraph was not great for financial transactions... But then we added transactions, so now it's great for financial transactions. The other drawback that we still have is that it's not really great for flat data... And by flat data I mean like time series data. You just have tons of things which are not really connections, but just more and more record points for the same thing. That kind of flat data is just not done very well with graph databases. You could use a graph for that, but it's better if you aggregate it somewhere else and bring the results into a graph database than to try to do the aggregation or storage in the graph database.

**Adam Stacoviak:** So basically in a world full of subjects that have many verbs, with many like-minded objects, graph databases apply.

**Manish R Jain:** Absolutely. Any SQL table, which is essentially row and column and data, can be easily converted into graphs. And I think every time we have tried to switch from a SQL use case to a graph use case, just the \[unintelligible 00:35:06.20\] reduces by at least half, because the query language is so much more powerful.

**Adam Stacoviak:** To go further into Jerod's question of where you reach for a graph database over, say, a Postgres or MySQL or a relational, you said you use to not recommend it for transactional, but then you built it... Is there a checklist of things that is like you'd reach for Postgres over Dgraph or other graph databases, that is consistently being chiseled away where graph databases just went out?

**Manish R Jain:** Sorry, could you repeat that question?

**Adam Stacoviak:** Is there a list of things where you recommend "Okay, if you're in these scenarios, don't use a graph database"? You said before you don't recommend it for transactional database, and then you built transactions, so now you take that back.

**Jerod Santo:** Maybe that was the list.

**Adam Stacoviak:** \[00:36:05.22\] That was the list? Okay... I wasn't sure if it was comprehensive or not...

**Jerod Santo:** "We had one thing in our list..." Well, it's flat data, so if you don't have a lot of relationships, then it's -- I mean, you can, according to what you've just said, Manish... You can use them, but they're not necessarily optimized for that.

**Adam Stacoviak:** Right.

**Jerod Santo:** You're not gonna get any advantages necessarily.

**Manish R Jain:** Right. I think the time series data is the one which I mentioned...

**Jerod Santo:** Time series...

**Manish R Jain:** Yeah. It's just not great for graphs.

**Jerod Santo:** What about management and maintenance? I am a Postgres user, and I have been for years, and so I always look at the shiny different data stores, and I think "This sounds great when I'm in development", and then I have to actually put the thing into the world and run it, and back it up, and make sure it's always up, and so on and so forth... And then it's like "Now I have to learn a brand new set of maintenance or management skills that I already own on the Postgres side." I think that's probably a barrier for a lot of people. What's the story with deploying this thing? I know it's built-in distributed, so it's gonna shard horizontally for you, which sounds amazing, but also potentially scary... I don't know. Tell us about deployment.

**Adam Stacoviak:** Deployment is where you lose customers, I think. Not for Dgraph, but I'm talking about in general this is where you can easily lose customers, because dev ops guys are always hard to impress... And we have spent a lot of time making sure that dev ops guys are happy with Dgraph. We already built in -- as I said, it's distributed, so it can shard the data for you... But it is also replicated, and all of that is part of the open core. A bunch of deployments that we're doing right now, they use what we call a \[unintelligible 00:37:54.07\] where we have three replicas for Dgraph 0 and three replicas for Dgraph Alpha. Don't worry about the terminology here, but just understand that it's three replicas each.

Dgraph uses a consensus algorithm called Raft to make sure that every data that you put into Dgraph, it reaches a quorum and gets replicated across a majority of these replicas before the acknowledgment is sent back to the user. So in case one of the servers crash, nothing happens. Your queries will keep on running, your data will keep on mutating, everything will just be fine. The dev ops guy would get a notification, they can either swap the machine, or if you're using Kubernetes, the machine just comes back up automatically, and your users don't even see it. So it becomes really easy as a dev ops person to just run Dgraph and keep everything happy.

One more thing that happens at the developer level is that, as I said before, sometimes with Postgres for example - or any database which has eventual consistency in the replication system - they will (let's say) create a new account into the master; then they want to read this new user's account, and they end up going to a replica... And the replica still doesn't have that new record, so it will show "Hey, account not found", which is just a bad experience for users.

\[00:39:36.26\] There's a lot of systems built on top, or you have to build it yourself to make sure that if you're doing a read after write, then the read goes back to the master, which basically means your replicas are not used as well, or you have to do a bunch of application-level tweaks and techniques to make it work. In Dgraph you don't have to worry about any of that, because it's all consistent... So even if a node crashes and is down for a long time, comes back up and you immediately run a query, the query would block until the node has caught up to the rest of the cluster, and only once the data is up to date would reply back. Obviously, there is also ways by which you can time out and query another server. All of these things are built in to make sure that you always get the freshest data, what we call \[unintelligible 00:40:33.14\] It tackles some of the common issues from both the dev ops side, and also from the developer side.

**Jerod Santo:** So does it give up availability then, in that case when the query blocks until it's consistent, so you're losing availability?

**Manish R Jain:** Yeah. In the CAP Theorem it goes for consistency in partitioning, instead of availability...

**Jerod Santo:** Okay.

**Manish R Jain:** But note that a lot of people mistake this - CAP Theorem is not the same as high availability. Dgraph is highly available, but it still goes for CP instead of CA.

**Break:** \[00:41:21.01\]

**Adam Stacoviak:** Manish, based on what you've shared with us so far, it sounds like the initial start for Dgraph as a company was 2013. Is that right?

**Manish R Jain:** 2015.

**Adam Stacoviak:** 2015. And in 2015 you did a round, you raised 3,5 million dollars if I remember correctly... Is that right?

**Manish R Jain:** We did a round in early 2016, and that other round in late 2017.

**Adam Stacoviak:** Okay.

**Manish R Jain:** A total of, I think, 2.9(ish) million.

**Adam Stacoviak:** So that means somebody trusts you with millions of dollars, basically, is what I'm trying to get at. You're establishing a company, you've built a technology that's obviously proven itself, and somebody said "Yeah, here's money. I trust you, I trust what you're trying to build, and I think it makes sense to do so." Sometimes that means that you've licensed things appropriately - the project has been open core/open source; you can tell us more about the inner details of that and what that means, but somehow, someway, at some point you chose the right license that allowed you to take on funding and build a company around it.

\[00:44:17.24\] Can you walk us through what that is? Because I'm imagining there are just so many developers out there going to choosealicense.com, and they're getting enough information, but still yet the wisdom is not there maybe so much. The definitions and details are, but I feel like you can bring some bloodied knuckles and some wisdom here... So preach.

**Manish R Jain:** Absolutely. I think when I was starting Dgraph - towards the end of 2015 - I naturally went for open source. It was not clear to me at that time how the business model would work. In fact, a lot of people I talk to around this idea of "Hey, I'm gonna build a graph database and make it open source", and they were like "If you're putting all the API out there, then what's left for you to make money off?" So the business models around open source only became clear to me later, and I think a lot of people who are in the Valley are probably more aware of them, but definitely people in Australia were not. You get open core, and so on and so forth.

Now, the choice of licensing was kind of important to me. The behemoth in the graph space Neo4j, was licensed as AGPL, which is considered to be a copyleft license. Now what AGPL does is that if you were to touch any code and use this AGPL code as, let's say, library, then you must open-source your code also as AGPL; it's sort of like \[unintelligible 00:46:05.10\] if you touch it, it affects you as well.

We decided to go with a more permissive Apache license. Now, a lot of people think the reason to open-source something is around getting contributions from developers all over the world, and I would say that is true, but it is not the main benefit of open-sourcing something. The biggest benefit of open-sourcing software, in my mind, is around adoption. It's basically free marketing. You put your code as open source, anybody can see it and they feel more comfortable using it. They don't have to pay you a dime to use it, particularly in permissive licenses like Apache, and BSD, and MIT etc.

These days, if you wanna build an infrastructure company, I've noticed most startups, most tech-based companies really want the underlying technology to be open source. And they have multiple benefits of doing so; when they have the code available to them, they already have the engineering talent and that talent can potentially go and modify the codebase to improve it, or modify it to their liking etc.

So the biggest thing I've seen around permissive licenses is adoption. You also get contributions, but more importantly over the journey of both Dgraph and Badger that I've noticed is just the fact that people give you feedback around issues that they run into, and that feedback, I feel, is more important sometimes than the actual code contributions that you get.

\[00:48:10.10\] So if you look at any open source repository, you'll see 90% of the contributions are being done by the core team of 3-4 people, and then there's a whole long tail of small contributions done by the bigger open source community. That's sort of like the ugly truth, or unknown truth about open source projects... Really, I think it's the feedback that improves the robustness of code.

**Jerod Santo:** That's definitely an interesting take. I think most people would say that the contributions are the main reason, but I think that's a compelling statement that you have there, with regard to the feedback versus actual code contributions.

So you mentioned picking Apache versus AGPL... Tell us about AGPL, maybe even contrast it with GPL, which it's a modification of, to a certain degree, and then why it wasn't attractive to you as a license.

**Manish R Jain:** Let me start with explaining a bit about AGPL itself... Again, this is to the best of my understanding. With GPL, the idea is that the code is on the same place, and the users are sort of linking to it as a library. Again, the virality of this whole GPL series comes into play. So if you link your code to GPL code, your code is supposed to become GPL as well, and you must make it open source in the GPL terms.

Now, AGPL was then devised as a way by which it can tackle GPL running as a server, and you interfacing with it over the network. So I think the idea is to try to make the same virality affect you if you are writing GPL code in the server and interfacing with it over a client.

**Jerod Santo:** That's my understanding, as well. The GPL had a "loophole", because it was designed before the proliferation of services - websites, web servers, web services where you're not delivering the end code, you're delivering a byproduct of the code... And so the AGPL was basically a fix for that loophole to also make the server-side -- even if you don't deliver the code to the end user, still covered under the (as you said) the virality portion of the GPL... So I think we're in agreement with that being the primary means, and then for the aim, and also I think it was effective in that regard.

**Manish R Jain:** Absolutely. And a lot of companies who still wanna hold on tightly to their codebase tend to use AGPL as sort of like a stopgap between going fully permissive open source, while still trying to make sure that they have a more solid business \[unintelligible 00:51:17.00\]

Now, Dgraph initially -- we did also try to convert from Apache to AGPL. When you do such a conversion, the first thing that you have to make sure of is that, even before the project started, you have a good ICLA in place. What's an ICLA? It's an Individual Contributor License Agreement, which means that any contribution that you take into your open source project, the rights to that contribution are given back to the company running the open source. We put that in place into Dgraph very early on, even when we were under Apache.

\[00:52:04.29\] That means that, in a way, the authors of that contribution hand the rights back to the company, which means the company can now change the licensing if need be. We do not accept any contributions without the author signing the ICLA, and it's just a standard practice, I've noticed, across not just Dgraph, but other open source companies as well.

That meant that we could change the licensing terms, and we did change it to AGPL. This was, I think, after MongoDB went IPO, and MongoDB was using AGPL... And we felt maybe that's a better way for us to make sure that we have a good business model. Once we had switched over to AGPL, we started hitting some of these things that we did not really understand before.

To give you a bit of history, Google explicitly bans AGPL code. Google's open source guy, Chris DiBona, in fact famously said that "No AGPL code is useful or good, and we don't need to use it." They banned it. Now, when Google goes and bans a license, other companies follow. Facebook doesn't publish it openly, and I don't really know, but I know that much that in Facebook and in Apple and some of these big companies it is very hard or almost impossible to bring in any AGPL code... And we actually had some of these things. So if somebody wants to play with Dgraph at one of these big companies, they're unable to because they can't even bring the code into the company, at all.

We started realizing that because of this, people were having a hard time adopting Dgraph, and again, going back to my point about why would you choose open source over proprietary license - it's largely for adoption. So we started seeing some of those issues, and we switched over from Apache to AGPL in March 2017, if I'm not wrong, and then towards the end of 2017 we decided "Hey, we need a better solution here. AGPL seems too toxic to be used for Dgraph."

Around that time we started a discussion - or probably somewhere after that - with the Redis Labs folks, and together we came up with this thing called the Commons Clause. The idea behind the Commons Clause is that you use a permissive license like Apache, or in this case of Redis, they use BSD, and you had a clause which basically prohibits some company or some person to sell the software as it is... And why would we go to AGPL or why would we go to the Commons Clause? The reason is that what's been happening lately and what none of the open source licenses have thought about is that big companies, and these platform as a service or infrastructure as a service etc. companies, most notably Amazon and the Chinese counterparts - they would pick up an open source project and they would run it as a service at a much cheaper price, and because they have the bandwidth and the engineering talent and the money for it, they would run it as a service without contributing back to the open source project. The main thing that we were going for is to avoid that. If you wanna sell this thing to developers, you should at least contribute back, or you should help the company financially who is actually doing most of the contributions.

So all of these licenses - AGPL, or Commons Clause, and now Mongo's SSBL - they are really around trying to dissuade big service providers from just ripping off an open source project.

**Adam Stacoviak:** \[00:56:30.25\] It seems like this stems, based on your earlier points, as like your motivations, your lens from which you're navigating this... And in your case, in particular Dgraph, you are optimizing as open source for adoption, not so much contributions, right? So you still want contributions, it's still important, it's part of the world, it's how open source works, but you're doing it based on adoption, so you've had to go through different licenses. And you wanna have a liberal license, with a clause that protects you so you can be a company and actually be viable and sustainable... And there's some that say that that added clause basically makes you not open source. What do you say to that?

**Manish R Jain:** I think it's a very delicate trade-off between trying to choose a permissive license, which allows most users to just use the software, while also dissuading a big company from coming in and stealing your financial longevity in some sense, right? And if you put Commons Clause in place, it's true, the project is no longer open source, because the Commons Clause is not OSI-approved.

Now, Redis did a smart thing there - they kept most of their codebase under the BSD license, which is still open source, but chose some of the modules that they had built and put them under Commons Clause. So you can think of, again, as this open core model in some sense, where most of your code is open source, but then some of your code is not. When we applied Commons Clause for Dgraph, we applied it fully, which means all of the codebase was under Commons Clause. And we were just not convinced that that was the right move, and this became very apparent when, again, Google went in and banned the Commons Clause as well. Now, I don't agree with their reasoning for Google to ban the Commons Clause, which was that they feel that the Commons Clause prohibits all commercial usages, which is completely wrong, really... The Commons Clause has this term called "if the code is substantially the same as the original code, then they can't sell it." "Substantially" is a term used very commonly in legal documents to basically indicate that if you tweak things a bit, it doesn't make it different. That is just a way of saying that if largely you're selling the same thing, which is selling, let's say Redis modules in this case, or selling Dgraph, then you would not be allowed to do that. But you can build something on top of it, for example you could build a question asking website, you could build some other proprietary service on top of Dgraph, and you can sell that; nobody stops you from doing that, because it is not substantially the same thing.

That was the idea behind the Commons Clause. I feel like the intentions were correct, but it was very hard to convey to people in the community, and even \[unintelligible 00:59:53.04\] in this case, what "substantially" meant. I think we went through many debates around explaining to people "substantially doesn't mean this, substantially doesn't mean that", but I don't think it's a fight that is easy to win.

\[01:00:12.23\] After we realized that the Commons Clause was banned by Google - it brings us back to the same place where AGPL is banned by Google, and again, it affects that option, so we decided that we will switch back to the Apache license. Now, there's an interesting backdrop here. This is back in 2017, I think - CockroachDB, a database company in New York, had come up with a license which was essentially Apache plus enterprise license (what they called the Cockroach license), and what they did was instead of trying to closed-source their enterprise modules, they made it source visible, and they collocated it right next to their open source codebase. Now what they have is they have the main source tree, which is Apache-licensed, and then certain modules which are under the enterprise license, are still with the code visible.

That was a very attractive system, and it was very well-received by their community, and it's something that I had in the back of my mind for a while, and I felt that Dgraph was still young enough, and we had started to build our enterprise features, but I felt that we can easily switch over to that license and make it work... So what we have done now is that we have brought Dgraph back to Apache without any clause, and we're gonna build the enterprise modules, which will be source visible. This system is also adopted, I'm not wrong, by ElasticSearch, and is just in general a very big win for liberal open source licenses, in some sense.

One more thing on top of this is that -- so this is our journey; that's where our journey sort of concludes... But after we switched over to Apache license and enterprise license, MongoDB, which was previously AGPL, has gone even stricter and created a license called SSPL (Server Side Public License). Now, as AGPL was sort of stricter that GPL, SSPL is even stricter than AGPL. It tries to do the same thing as the Commons Clause in some sense, but it does it a bit differently. It says that if you run MongoDB as a service, then you must open-source the codebase which helps you run MongoDB as a service. Again, it's a jab at the big service providers, like Amazon, but it's just done in a different way, where they probably have a better chance of getting it approved by OSI... But in my mind, it's trying to achieve the same thing as what Redis was doing with the Commons Clause.

**Jerod Santo:** There are plenty of people out there that are vehemently opposed to Commons Clause, with regards to open source software... Because as you said, the OSI has not approved it, and potentially will not approve it. So there's Commons Clause-licensed projects that claim to be open source, and even on CommonsClause.com it says "Is this open source?" and it says "No" because of that specific thing. That being said, do you believe the Commons Clause is in the spirit of open source? Because I'm on the fence there.

\[01:03:57.21\] It seems like the freedom to modify, the freedom to dispute it - it seems like a bit anti-freedom, but only for a small subset; it's like "Large corporations/service providers, we'll take your freedom away, but everybody else is still free." I don't know... This was something you've gone down the path, you implemented it, it's kind of there and back again; Apache 2.0, maybe AGPL, maybe Commons Clause, you've had some pushback from your community, you mentioned Google banning it was the show-stopper - it makes a lot of sense for adoption... But all along the way, Manish, it seems like your intentions are good, from what I can tell from this conversation... So what do you think about the Commons Clause with regards to -- maybe it's not open source approved, but do you believe it's in the spirit of open source, or not?

**Manish R Jain:** I absolutely believe it is. I feel it is more in the spirit of open source than AGPL is.

**Jerod Santo:** Why is that?

**Manish R Jain:** The problem with AGPL, with being used at any medium to big company, is that the moment you bring in AGPL, you have to be afraid about "Hey, do I need to open-source my own codebase?" And the problem with companies is that they have this \[unintelligible 01:05:10.04\] code, which is part proprietary, part \[unintelligible 01:05:15.24\] It's very hard to say "Okay, this piece I can break off and maybe open-source this, but this piece I can keep proprietary." It's very hard to say that, and therefore if you look at Google, for example, when they built Kubernetes or when they built gRPC, they didn't just open-source the existing systems Borg and Stubby; they had to rewrite them from scratch to make it open source. So AGPL puts this restriction upon these companies that if they use any AGPL code, they must open-source because of virality... It's very prohibitive.

Now, you bring in Commons Clause + Apache. Apache gives you anything - basically, you can do anything with the codebase. You don't have to open-source, it's not viral... And Commons Clause stops you from selling the database, in this case, or whatever the codebase is, from selling that particular code. It works for big companies.

**Adam Stacoviak:** Very cut and dry.

**Manish R Jain:** It should work for, let's say, Google, it should work for Facebook, because they're not trying to sell Redis, they're not trying to sell Dgraph; they're just trying to use it. So I feel it is more permissive than AGPL. The only companies it should really affect is if you're Amazon and trying to sell Redis and all the particular modules that they put under Commons Clause, then you're not able to sell that... Which I feel is fine, because if they did not contribute, then maybe they shouldn't sell it, and maybe they should let the contributors sell that. That's my take on it.

**Adam Stacoviak:** For AGPL I might have a somewhat analogous take on this, so to speak. It reminds me of CSS, in a way; there's a cascade, an unwanted effect of using it, which is not always clear when you make changes or use a class, or something like that. There's hidden things. So if I use AGPL, it may affect licenses or other future software I ever use in unwanted ways, and those unwanted ways provides ambiguity and it's not clear, so if that's accurate, then I can see why it's less likely... Whereas Commons Clause is more like a razor blade; it's clear-cut. It's like "I can license my code permissively at one level, and then clause in or add an addendum", which is the point of it. "Here's one clause, and it's only for this project, and it doesn't affect any other things it touches. If you're trying to resell my thing here, then that's just not possible."

And I'm with you too, Jerod, on Manish's take. He seems to be a great guy, I like him...

**Jerod Santo:** \[01:08:09.14\] He's still here... \[laughter\]

**Adam Stacoviak:** Yeah, he's still here; we haven't hung up on him yet...

**Manish R Jain:** I can hear you...

**Jerod Santo:** \[laughs\]

**Adam Stacoviak:** Right...? This is where I think this needs to be a dialogue, and blog posts are great for getting points across. I really feel like this needs some sort of at-large, literal discussion, because behind all software, as human beings with often great intentions -- Manish isn't trying to hurt people; he just wants to be able to create awesome tech and have people use it. He said that here. And he is trying to look for, and he and his team, and I'm sure his investors too are trying to make sure that remains possible. So I'm for that--

**Jerod Santo:** But couldn't he just do that -- now that we're talking about him and he's not here anymore, couldn't he just do that by having closed source software? I mean, if you wanna do that... I'm just playing devil's advocate.

**Adam Stacoviak:** Right.

**Jerod Santo:** If you wanna do that - and obviously, Manish, please feel free to respond; we're not actually talking about you like we're not here... Couldn't you just closed-source? I mean, keep it proprietary, and then you get to say "Hands off." You don't have these problems.

**Manish R Jain:** The thing about closed source - again, it goes back to the reason about why do you wanna open-source in the first place. I think it's not about the contributions; I mean, obviously, if you get contributions... I always thank people for contributions, I thank them for the feedback, but the reason you make anything open source is adoption. You wanna build something which a lot of companies, a lot of people are going to base their entire tech stack upon, in this case a database. They're going to trust you with their data. They wanna be able to look at the code and make sure that the codebase is good quality, it doesn't have any weird bugs, that they are able to modify the code... And what if the company dies tomorrow? They should still be able to adopt that codebase and maybe run with it.

**Jerod Santo:** You can do that with a proprietary license, as well. You could ship them binaries plus source code as part of their license. This isn't something they wouldn't be able to do. That's the thing about proprietary - you can do whatever you want with it.

**Manish R Jain:** True. The other part of this equation is that when you make something proprietary, the selling becomes a lot more work. You need to have an entire sales team to be able to go to individual companies and be like "Hey, have you hear about this thing called Dgraph? It's a proprietary thing, you can't see it online, but we can sell it to you for use." It is a lot harder pitch than "Hey, developers, it's just free. It's out there, you can try it, and if you don't like it, it's fine. If you like it, it's fine. You don't have to talk to us." And I think that's the beauty of open source - it avoids having to have salespeople running around, and you just become part of a developer conversation anywhere in the world. Nobody has to pay you to try it.

**Jerod Santo:** Okay.

**Adam Stacoviak:** I think the problem though is being seen as masquerading as open source, but not really being open source.

**Jerod Santo:** Exactly.

**Adam Stacoviak:** It goes back to original things; it's been said like the anti-Commons Clause, or whatever... Just in terms of the spirit of open source -- and sure, it is open, you can see it, I can contribute back if I want to, but I think what the community is really pushing back on is less like "Hey, that's a bad thing" and more like "Hey, this really isn't open source, so just don't call it open source and it will be okay."

**Jerod Santo:** \[01:11:45.01\] Yeah, it's potentially a namespace conflict, as all things are... Because the benefits of open source are immense, as you said, Manish. And in many cases, especially in infrastructure-style, missions-critical enterprise software in 2018, it's almost table stakes for a success, because people expect it; as you said, your sales processes are easier, the trust is immediately there... And yet, when you add Commons Clause to it, it's restricting in that regard; so now it's like "Well, they're on CommonsClause.com. This is not open source, it's something else", but then it's almost like -- and I'm not saying this personally against you or against Dgraph, but it's as if you want the benefits of open source without actually being open source... So maybe it needs to be like "available source" or "readable source." It's almost like we've just gotta come up with some more nomenclature, similar to how we have copyleft/copyright, or free and libre versus open source - we have all these different terms; maybe there's a need for another term for this style. I don't know, what do you think about that, Manish?

**Manish R Jain:** We were very careful when we switched to Commons Clause and Apache that we removed all the references to open source, and we swapped them with "liberal license." It goes back to my take on this - that it's more liberal than AGPL and some of the other open source licenses. So we had to switch it over to liberal license. It was a bit of a heartache for me, because I've been an open source guy for a long time. Back in 2005 I rolled this thing flickrfs to build a file system on top of Flickr, which was the most popular image sharing site at the time... So I'm a through and through open source guy, and it was a bit of a hard decision.

Just to clarify - we have moved away from Commons Clause, but still, I would sort of defend... The thought at the time was that it is probably not approved by the folks who are at OSI, but in terms of the spirit of open source, I feel it was there. I think open source has to evolve to a point where people who are building open source can sustain themselves from what they are building, as opposed to having to ask for donations or having to work for another company, or having to be acquired by another company who is writing proprietary code.

Every time I see some open source author having to go join a company and abandon their open source project which is very popular, it hurts me in some sense; it just feels bad. Why shouldn't a person who is writing an amazing code not able to sustain themselves, with the right intentions in their mind, which is that "Hey, open source obviously makes sense"?

There should be a deeper conversation about "Hey, open source makes sense. We all agree. Let's figure out how do we make money, how do we make sure that people who are in open source continue to make money, and not just by making open source their secondary project, but having open source as their primary project AND the source of income?"

**Adam Stacoviak:** Hearing it from that perspective and then also knowing what a history you have in open source back to flickrfs, it makes you really consider what you say is a necessary evolution of open source, because based on what you've just said there and how you said it was that the free and libre of open source is there, but at some point it does restrict potentially the sustainability by restricting its original creators and maintainers and community from being able to profit in certain ways from it, because of just sheer competition. You can't compete with Amazon; maybe you can -- I mean, you really can't, but like most, if Amazon launches a furniture line, Wayfair's stock goes down 6% in a day. I mean, that happens, right?

\[01:16:10.14\] So how can we expect little ol' you guys in your team to compete...? And the restriction comes back to the original core team and how you can sustain it financially without having to -- as you said, the examples were either ask for donations, work for a company... You're not liberated to operate a company around this source code in a way that is financially feasible if you have to face the sheer weight of competition that is just so massive. Does that summarize somewhat of what you're trying to say there?

**Manish R Jain:** Yeah. I think one thing we failed to mention is the three models of open source money-making. I think we should quickly mention that, so it all ties together. The first one is that you have this open core, which is under an open source license, and you build proprietary features on top of it which you sell. That's the first one. In Redis Labs' case, they basically try to make those modules under a Commons Clause, so that they can sell those.

The second is that obviously support and training comes in. Red Hat pioneered this a long time ago, and every open source company does support and training. That's how they make money.

The third one is that you run that software as a service, and I think this is where Amazon's story comes in the picture. For example, with Redis Labs - Amazon is probably running Redis behind the scenes for their ElastiCache, or I forget where it is... And they're literally just running that without paying anything back to Redis Labs, and Redis Labs in this case also has a competing Redis as a service availability. Both MongoDB and Redis and whoever is trying to use Commons Clause is trying to avoid a big company like Amazon, and also these days their Chinese counterparts... I forget the name of that, but they also are running Redis and Mongo behind their service providers, and charging customers for it. These companies are like, "Hey, we built this thing. You shouldn't be competing with us on this, and we should be getting that money."

**Jerod Santo:** Trying to stop the leeches... You know, stop leeching off people. Contribute back. It makes you kind of mad, even though I totally get it; I can see it from Amazon's side, but... Yeah. It's like the Leech Clause, we should call it...

**Adam Stacoviak:** There you go, the Leech Clause.

**Jerod Santo:** \[laughs\] It's just a naming problem.

**Adam Stacoviak:** Well, in a free world, people are free to do literally whatever they want, and so I think in the spirit of open source, the idea has been for it to be a free world, in most or all senses of the word. And I think when you restrict that freedom, it does begin to munge the original intentions... But I think we do need to recognize this leech scenario, and the viability of it. If we continue to allow that to happen and not have conversations that hear all sides, then we essentially allow the freedom of the software, as good as it may be, to stagnate and potentially -- like you said, Jerod, why not go into proprietary, and then we wouldn't even be talking to Manish...

**Jerod Santo:** \[laughs\]

**Adam Stacoviak:** I mean, what would be the point? "Open source has won." That's in quotes, it's been said not just by us, but others...

**Jerod Santo:** That's in quotes... It's official.

**Adam Stacoviak:** \[01:19:58.17\] Well, Nadia Eghbal said it on request for commits many times, and others agreed, so that's why I say it's in quotes, because it's been said not just here by us, but by others... Yeah, I just think it needs more attention. I'm not saying I agree, or it's wrong, or it's right... I definitely see the pain points, and we need some sort of evolution.

**Manish R Jain:** I would like to add one thing. I think this seems like a fresh thing, it seems like a new thing, this attack on open source in some sense by this Commons Clause etc. But this was done before; if you look at GPL, the idea behind GPL was that "Hey, open source is important. We must do open source." In fact, before \[unintelligible 01:20:36.20\] if you use other code, you must also open-source your code, right? And then AGPL was the evolution of that, to say "Hey, also on the network, same thing." And then the MongoDB SSPL is extending that to say "Hey, if you run it as a service, same thing", right? But think about what they are really doing practically, what are the practical consequences of this - in some sense, they are dissuading others who have not contributed from leeching off it, in some sense... And I think that's the direction that Commons Clause and SSPL are all going.

**Jerod Santo:** I recall we had Joseph Jacks on the show (OSS Capital) a couple of weeks back, and we asked him about Commons Clause, because it was drafted by Heather Meeker, she's part of OSS Capital, so I'm sure you know her as well... And one thing that he said about it, he sees it as a stepping stone or as an effort in a specific direction, and that there are things, like you said, there's necessary evolution that has to happen for the greater open source community to continue to -- not strive so much, but thrive, right? And so I'm happy to have this conversation; I've learned a lot here, Manish... Thanks so much for coming on...

**Manish R Jain:** Thanks for having me.

**Jerod Santo:** ...and continuing to talk about these things. I know it's kind of the nitty-gritty licensing, kind of a dry topic, but there are so many facets to these decisions, and the implications of changing a license, picking a license - they're just massive. And we're definitely living in a brave new world where we're trying to figure this out together.

**Adam Stacoviak:** Including a world with big numbers. We've seen the headlines on Changelog.com this week in the news feed - billion-dollar valuations, hundreds of millions of dollars invested into new companies or companies that are now unicorns, HashiCorp being an open core model type company that's just taken on a new round of gigantic funding... So there's clearly lots of money at play here, and it's a new world for open source every single day.

So where do we go from here? Clearly, we've had a great conversation, that's led from not only Dgraph as a tech and how it applies, graph databases 101, on through how they could be used... You're clearly super-smart; you've had to relicense, you've been through a journey... What do you suggest may be the next step? Maybe not here today, because we're getting out of time, but what are some suggestions for you to continue this conversation in ways that are meaningful, that can get to meaningful change? Do we have conferences about it, do we do a Sustain-like unconference, or just kind of a gathering? How can this best be approached by the right people in ways that are not vicious and attacking, but in ways that are meant to actually get to change? What do you suggest?

**Manish R Jain:** \[01:23:30.13\] I think it's a tough conversation. It's a conversation of ideals versus practicality. It will require flexibility from the maintainers or the people in charge at OSI to think through some of the practical considerations of running an open source company into this environment, and I think it would definitely need a bigger dialogue. If MongoDB's SSPL gets approved by OSI, that would be probably a great outcome of this, and I can easily see a bunch of other companies jumping onto that bandwagon. If it gets rejected, then other open source companies are gonna keep coming up with something new, which might work. There's definitely a need for a change here, I think that much is clear.

**Adam Stacoviak:** Well, let's close the show with anything for you - I know you've got lots of stuff happening; we've obviously covered quite a bit of ground, but if people are following along with you, where do they go, what do they do? Do you have anything to announce here at the close of the show?

**Manish R Jain:** Yeah, I do want to announce something. We are solving really complex problems at Dgraph, and we also have Badger - both are done purely in Go, both open source. If you wanna help us and if you wanna experience these challenging problems, come join us. You can go to https://dgraph.io and see some of the job openings; we are looking for back-end engineers, so apply.

**Adam Stacoviak:** Manish, thank you so much for sharing not only your story, but your wisdom here. I know it's a tough subject, and going on record -- because we do have an awesome transcript for this show. Thank you, Alexander, for being so awesome, and all the contributors out there who help us... Like our friends we mentioned at the top of the show - they make our shows less unintelligible, and more intelligible, so to speak.

So I know it's tough to be on record about very tough subjects, and we just appreciate your courage to share how you feel, and the willingness to continue to go on the road, even when it's bumpy... And thank you for sharing your time with us.

**Manish R Jain:** Thanks for having me, guys.