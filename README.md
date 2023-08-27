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
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/1e5b727a-5e5a-4fed-b6c9-c935d166fd9f)

It seems I will use approximately 129 million records instead of 208 million. 62% of the original dataset. All of this records have at least 1 second duration and their starting time is previous than their arriving time.

## __Creating a dataset__  

Creating a dataset is almost like creating a schema in SQL. First I need to create the dataset, and after that I will create tables al fill them with data.
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/619c3217-f4df-4eae-8366-c8e7f4b8db46)

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/5c0759cf-0f35-4106-a483-738cb64f4060)

Then I need to fillthe dataset with data.
So I query the entire dataset with the filters previously mentioned to get the valid trips and all the columns.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/6ef49388-f195-4f49-a712-34f25ce9c3f8)

After the query is made I can save the result as a new table in the dataset.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/aa3fac69-fb49-41f5-82d5-0e99ece19430)

Here I choose the dataset previously created, and I create a table name for the new dataset.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/27e348a8-207d-43bd-8776-584d8cef77da)

Then I query my new dataset and I check I have the same amount of records I had when I queried the original dataset from bigquery.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/1058493e-054a-416f-9732-2e91a8f455ac)

## __Query time__  


Now lets try to explore the clean dataset :blush:

How much money has been made on chicago taxi drives?

I already have the amount of the total trip, which is reflected on “trip_total”, I checked with some examples of the dataset.
![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/b204ccb2-34b1-48c4-9166-40cddb463a8d)

``` 
SELECT SUM(trip_total) FROM `intense-reason-393613.taxisTrips.taxisTravels`
```  
I got the result of: 2.636.262.703,37 US$
was the amount of money that has been spent since 2013 in taxi drives in chicago.

Now I will explore:  how many companies are involved in this information?

Well technically the dataset has 172 different companies without considering the records that have no company name.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/07b7430b-70ee-4adf-92a8-6a1da75c8b1d)

But, unfortunately the dataset data quality is poor, again I have another problem: repeated records.
In this case for example the dataset has 4 different companies, but I know the records are referring to the same company.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/244beee0-856f-41c4-af7c-a0983a2b7634)

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/06e7d57f-bcce-417e-b288-7f4314d98542)

I can say the dataset has less than 172 taxi companies.

Now I want to know in which hour the taxi demand is the highest:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/334244f5-7fa5-4cd4-a157-fc8a7d3796d3)

The time when more taxis are requested is 18.
I want to see which of the previous records has the highest average in Tips:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/2543f6d4-628d-482c-a476-0d36003f21ee)

I can see the travelers of the 6am hour are very generous .

I know that the hours 16,17,18,19 and 20 are the hours in which more taxis are requested, but I want to see the behaviour during the year.
I will try to get the months which less trips in the entire dataset:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/451e2439-f9ae-4227-be1c-6f537ed0e62e)

I can see that during the winter, the demand of taxis is lower than the rest of the year. The month that have less demand of taxis are January, December, February and November. So Chicago people prefer to stay at home in the winter.
Let’s see how many travels had the taxi with most quantity of travels:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/0590546a-e679-4fe6-ac92-26934c503fe4)

50039 travels in a period of ten years if I check the max y the min year the dataset has:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/6370408f-9d52-47aa-985f-f98fb95d224f)

So I can say the taxi with more work in ten years had approximately 5000 travels by year.

Now I want to check the average cost of the trips without tips and extra cost, considering all the travels that has a fare > to zero.

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/a4e0e1a2-3d3b-4c89-8112-643600b81500)

It is 17.26 USD.

I want to see which is the company with the highest profit in Chicago:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/ccaa5d0d-da60-4750-8695-73b4a1683bc9)


Taxi Affiliation Services is the most dominant taxi company in Chicago in terms of profit.

Now I want to know how it is distributed the payment method considering all the trips:

![image](https://github.com/emilianoregazzoni/Chicago-taxi-trips/assets/20979227/a0d75fdf-aadd-4e41-b393-6d7a166e2193)

The majority of them were paid via cash, and almost the other half was paid using credit card

I would love to do some transformations with dataflow and apache beam, but the dataset doesn't seems to be a good one, too many repeated records for taxi companies, almost a million of travels with higher starting date than end date, and it doesn't have many more information. 

I guess I will choose another dataset to do dataflow transformations in the future, but It was a good experience to explore into Big query on GCP :relaxed:


