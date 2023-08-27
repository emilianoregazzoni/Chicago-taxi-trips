#  Chicago-taxi-trips :taxi:
## GCP project using BigQuery

In this proyect I will use Google Cloud Platform (GCP) to explore the dataset called 
“Chicago Taxi Trips”, do some transformations with Dataflow, explore data with Big Query and finally get some visualizations with Looker. 
I will be using a “Free” GCP account which allows me to use 300 U$$ in resources during 90 days.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/4733a816-cdbd-4608-bdc6-25e0fb398d47)


## __Exploring__  

This datasets shows the taxi trips of Chicago since 2013 to present. It has a monthly update, which is good.

Key point working in a cloud environment: __Just query what you need__.
During our live in the University, when you start working with SQL, in most of cases you’ll be working with a really small amount of records. For example you can be working with tables with around 100
records or things like that.
After that when you get a job and you need query records, in smalls companies in Uruguay, you can be dealing with 10’000 , 50’000 thing like that. 
But when you face a massive dataset of information like this one in a cloud environment you must be so carefully on what you are going to query. Because if you don’t, maybe you can have a trouble later in terms of billings, or in terms of time to query.
GCP has a particular feature in the right side, that indicates you the amount of GB will cost you run the query. 

For example, if I choose just 2 values from data set, it will cost 6.52GB like the picture says here:
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/f704b242-04e2-4728-b9b8-6dae2bdbb194)


But if I do a classic “__SELECT *__ ”  what probably mostly of us has been doing during all of our life (despite we know is a bad practice), you will see how the GB increases a lot:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/f0c3a11e-b88b-47c6-9070-d38cea29c950)


So, first thing I learned of all the GCP ecosystem, query __exactly__ what you need to. Not more, never.

Let’s see how many records will I be dealing with:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/0a28c079-38ad-4bb7-ab8d-ee7047d535c8)

208 million records. That is a good number to work with.

I know I have a resume of information in the description of dataset:
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/d7e591fe-546d-495a-827c-f9990f5381d5)

It says the type of data the dataset has, but also the description of each attribute which is really helpful to understand better the dataset.
Despite of the good resume from GCP, I need to see at least 1000 rows of the dataset, to check how the information is, how blanks I have, what kind of information I have.

I got examples like these ones:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/714822e5-6326-4bf3-8879-a69b7f6cc90e)

I think the dataset has some problems with some columns. It doesn’t make sense I have trips with no “trip_seconds” which is impossible. But also I have trips that started later than they end date... which is even impossible (at least they are “time travel taxis”).
I guess this situation is due to the data of the travels, maybe at this point (10 years ago) they had some problems saving this 2 attributes.
So I checked the most recent trips, to see if this situation keeps going on or not:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/c9aa2e81-b9f9-4c4f-be22-11d022c77c13)

It doesn’t seems ok but, I checked the documentation of the documentation about the attribute, it says:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/9fa96814-6d88-4b53-b4cb-8739efbbf7a3)

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
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/4310d3ba-4192-47c3-98a2-ca995923da9b)

It seems I will use approximately 129 million records instead of 208 million. 62% of the original dataset. All of this records have at least 1 second duration and their starting time is previous than their arriving time.

## __Creating a dataset__  

Creating a dataset is almost like creating a schema in SQL. First I need to create the dataset, and after that I will create tables al fill them with data.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/84ceec2a-3afe-4df6-9f59-b3a65c5e7f19)


![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/58472431-9df7-4f30-bfd6-87f880b71ead)


Then I need to fillthe dataset with data.
So I query the entire dataset with the filters previously mentioned to get the valid trips and all the columns.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/e7948983-b67f-4bba-b53b-a48bfc9559d7)


After the query is made I can save the result as a new table in the dataset.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/e8ea7346-edb9-4dcb-bae8-a4f0cfe0010c)

Here I choose the dataset previously created, and I create a table name for the new dataset.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/dedcd281-ba0e-49cb-afff-01d4f64b51d7)

Then I query my new dataset and I check I have the same amount of records I had when I queried the original dataset from bigquery.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/b166e900-af9d-48de-a1b5-f7b3db23c132)

## __Query time__  


Now lets try to explore the clean dataset :blush:

How much money has been made on chicago taxi drives?

I already have the amount of the total trip, which is reflected on “trip_total”, I checked with some examples of the dataset.
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/72801e33-29cc-4069-95fa-4f3ae133d66c)

``` 
SELECT SUM(trip_total) FROM `intense-reason-393613.taxisTrips.taxisTravels`
```  
I got the result of: 2.636.262.703,37 US$
was the amount of money that has been spent since 2013 in taxi drives in chicago.

Now I will explore:  how many companies are involved in this information?

Well technically the dataset has 172 different companies without considering the records that have no company name.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/66ecc819-7a60-4795-8a03-7141a340af51)


But, unfortunately the dataset data quality is poor, again I have another problem: repeated records.
In this case for example the dataset has 4 different companies, but I know the records are referring to the same company.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/f86401f3-d1dc-4148-89a2-c4bff67c5683)

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/81154fa3-9c69-4e2d-904f-d12d54729a80)

I can say the dataset has less than 172 taxi companies.

Now I want to know in which hour the taxi demand is the highest:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/b98f3cab-0e38-4ee9-be25-3997dcd08391)


The time when more taxis are requested is 18.
I want to see which of the previous records has the highest average in Tips:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/fbb77313-c1e2-4228-9b44-2e49baddddd1)

I can see the travelers of the 6am hour are very generous .

I know that the hours 16,17,18,19 and 20 are the hours in which more taxis are requested, but I want to see the behaviour during the year.
I will try to get the months which less trips in the entire dataset:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/abf74dab-bb03-4455-92bb-a01a551a85c7)

I can see that during the winter, the demand of taxis is lower than the rest of the year. The month that have less demand of taxis are January, December, February and November. So Chicago people prefer to stay at home in the winter.
Let’s see how many travels had the taxi with most quantity of travels:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/584d75c1-2105-4344-ae50-eb263aa20316)

50039 travels in a period of ten years if I check the max y the min year the dataset has:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/497958e3-1fda-44f4-b56a-c6a79d786189)

So I can say the taxi with more work in ten years had approximately 5000 travels by year.

Now I want to check the average cost of the trips without tips and extra cost, considering all the travels that has a fare > to zero.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/5538cb7c-be15-43a9-8727-0d938feb5f06)

It is 17.26 USD.

I want to see which is the company with the highest profit in Chicago:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/0a662442-53bd-45ba-8b21-e6ce67db685e)


Taxi Affiliation Services is the most dominant taxi company in Chicago in terms of profit.

Now I want to know how it is distributed the payment method considering all the trips:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/0cd053ff-77de-4c59-bdb9-928826b8a126)


The majority of them were paid via cash, and almost the other half was paid using credit card

I would love to do some transformations with dataflow and apache beam, but the dataset doesn't seems to be a good one, too many repeated records for taxi companies, almost a million of travels with higher starting date than end date, and it doesn't have many more information. 

I guess I will choose another dataset to do dataflow transformations in the future, but It was a good experience to explore into Big query on GCP :relaxed:


