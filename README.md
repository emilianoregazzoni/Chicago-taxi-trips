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


