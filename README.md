#  Chicago-taxi-trips :taxi:
## GCP project using BigQuery

In this proyect I will use Google Cloud Platform (GCP) to explore the dataset called 
“Chicago Taxi Trips”, do some transformations with Dataflow, explore data with Big Query and finally get some visualizations with Looker. 
I will be using a “Free” GCP account which allows me to use 300 U$$ in resources during 90 days.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/0f9d45ce-ec6a-4335-9ba7-5071e3f55129)

This datasets shows the taxi trips of Chicago since 2013 to present. It has a monthly update, which is good.

Key point working in a cloud environment: __Just query what you need__.
During our live in the University, when you start working with SQL, in most of cases you’ll be working with a really small amount of records. For example you can be working with tables with around 100
records or things like that.
After that when you get a job and you need query records, in smalls companies in Uruguay, you can be dealing with 10’000 , 50’000 thing like that. 
But when you face a massive dataset of information like this one in a cloud environment you must be so carefully on what you are going to query. Because if you don’t, maybe you can have a trouble later in terms of billings, or in terms of time to query.
GCP has a particular feature in the right side, that indicates you the amount of GB will cost you run the query. 

For example, if I choose just 2 values from data set, it will cost 6.52GB like the picture says here:
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/729578e8-4ee0-405c-84e8-eea60dd331da)

But if I do a classic “__SELECT *__ ”  what probably mostly of us has been doing during all of our life (despite we know is a bad practice), you will see how the GB increases a lot:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/b2bbaff6-f257-4589-9604-34bfb0a0f981)

So, first thing I learned of all the GCP ecosystem, query __exactly__ what you need to. Not more, never.

Let’s see how many records will I be dealing with:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/73e24bbc-0c31-40aa-8d2c-3f157861b4d9)
208 million records. That is a good number to work with.

I know I have a resume of information in the description of dataset:
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/3f32da28-8ad8-4ad4-afa5-98e50a0be21f)

It says the type of data the dataset has, but also the description of each attribute which is really helpful to understand better the dataset.
Despite of the good resume from GCP, I need to see at least 1000 rows of the dataset, to check how the information is, how blanks I have, what kind of information I have.

I got examples like these ones:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/0aaca390-c873-4848-abe0-71edf3e89819)

I think the dataset has some problems with some columns. It doesn’t make sense I have trips with no “trip_seconds” which is impossible. But also I have trips that started later than they end date... which is even impossible (at least they are “time travel taxis”).
I guess this situation is due to the data of the travels, maybe at this point (10 years ago) they had some problems saving this 2 attributes.
So I checked the most recent trips, to see if this situation keeps going on or not:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/3b8fb612-4638-49cc-a40f-985ab0f45472)

It doesn’t seems ok but, I checked the documentation of the documentation about the attribute, it says:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/458d0544-a6f3-43a2-914e-f3f4cec870e2)

So if the previous example says the travel starts “around” 12.45 and it ended close to 13.00 and the duration was 639 secs, the logic says:

639 secs /60 = __10.65__ minutes duration.

For example, if the travel starts at 12:45 + 10.65 = 12:55:65 (exact end date) it could be the end date. And if the dataset chooses the closest .15 of an hour to set the end date, it’s ok, because the end date is near to 13:00:00.

Now I now the start and end date are “estimate” times, they are not exact.
Anyways I need to decide what I will do with the records have no duration time (trip_seconds) and also the records have a newest trip_start than their trip_end. This cases have no sense, so I will filter them and I will not be using them in the proyect, because this records will afect the entire dataset and the conclutions may I reach.

The historic dataset has 208 million records approximately, but I concluded the dataset has also some problems with an important amount of records. Because it has records with no time duration but also records with more recent date starting  trip than the date arriviving date. This situation has no sense. 
So I will filter this ones with a simple query just to figure out with how many records I will work with.

```  
#standardSQL
SELECT
count(*) as qty
FROM
  `bigquery-public-data.chicago_taxi_trips.taxi_trips`
WHERE
trip_seconds >=1
and trip_end_timestamp > trip_start_timestamp
```  


