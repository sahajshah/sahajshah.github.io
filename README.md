# sahajshah.github.io
Can you optimize and predict emergency calls?

import pandas as pd
import datetime as dt
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

data = pd.read_csv("/users/sahajshah/Desktop/data.csv") #reads the data csv file


datap = data[['unit_type', 'call_type']]

#Data Visuals: Display or graph 3 metrics or trends from the data set that are interesting to you.

datap['call_type'].value_counts().plot(kind='bar') #graph 1
plt.title('Distribution of incidences per type of call')
datap['unit_type'].value_counts().plot(kind='bar')
plt.title('Distribution of incidences per type of unit for dispatch') #graph 2

call_type_list = data.station_area.tolist() #converting relevant columns of data to lists
unit_type_list = data.unit_type.tolist()
address_list = data.address.tolist()
city_list = data.city.tolist()
latitude_list = data.latitude.tolist()
longitude_list = data.longitude.tolist()
dataset = data[['address', 'city', 'zipcode_of_incident', 'available_timestamp','unit_type']]

#print data.received_timestamp.dt.hour
times = pd.DatetimeIndex(data['received_timestamp']).hour
times_new = times.tolist()

plt.hist(times, bins=48) #graph3 of incidences with respect to time in terms of hour
plt.title('Distribution of incidences with respect to the time in terms of hour')
plt.show()

zipcode_list = data.zipcode_of_incident.tolist()
z=[]

dataset.apply(pd.value_counts)

#algorithm that outputs number of incidents with truck dispatch for every hour. I used this algorithm to find the predominant dispatch vehicle for every hour, plugging in different unit types for line 45.

count=0
for i in range(0,24):
    for index,j in enumerate(times_new):
        #indexes = times_new.index(j)
        if i==j and unit_type_list[index]=='TRUCK':
            count+=1
    print count
    count=0

    
#2: Given an address and time, what is the most likely dispatch to be required?

count=0
number_ask = raw_input('Enter the zipcode\n') #input information
address_ask = raw_input('Enter your address\n')
city_ask = raw_input('Enter your city\n')
hour_ask = raw_input('Enter what hour of the day in 24 hr format\n')
latitude_ask = float(raw_input('Enter your latitude\n'))
longitude_ask = float(raw_input('Enter your longitude\n'))

min_lat = min(latitude_list, key=lambda x:abs(x - latitude_ask)) #finds the closest user entered latitude and longitude from the list 
min_long = min(longitude_list, key=lambda x:abs(x - longitude_ask))
m=latitude_list.index(min_lat) #finds the index of the closest latitude and longitude from the list
n=longitude_list.index(min_long)
if (m==n):
    print unit_type_list[m]
elif address_ask in address_list: #if the address matches that from the data, then it will output the unit type from that address
    store = address_list.index(address_ask)
    print unit_type_list[store]
elif hour_ask == 3: #found using the algorithm above that outputs the predominant unit type for every hour. Engine was found to be the most predominat except the 3rd hour. 
    print 'MEDIC'
else:
    print 'ENGINE'
    
#Future Work: right now, the algorithm only takes in time with respect to hour. I hope to be able to estimate nearest time in the list and accurate upto the nearest second. 

#Question 3: Which areas take the longest time to dispatch to on average? How can this be reduced?

#puts the hours and minutes into respective lists
hour_received = pd.DatetimeIndex(data['received_timestamp']).hour
hour_receivedlist = hour_received.tolist()

minute_received = pd.DatetimeIndex(data['received_timestamp']).minute
minute_receivedlist = hour_received.tolist()


hour_response = pd.DatetimeIndex(data['response_timestamp']).hour
hour_responselist = hour_response.tolist()

minute_response = pd.DatetimeIndex(data['response_timestamp']).minute
minute_responselist = minute_response.tolist()


hour_list=[]
minute_list=[]
for indexh,h in enumerate(hour_receivedlist):
    diff = hour_responselist[indexh]-hour_receivedlist[indexh]
    hour_list.append(diff)

for indexm,m in enumerate(minute_receivedlist):
    diffm = minute_responselist[indexm]-minute_receivedlist[indexm]
    minute_list.append(diffm)

#Question 3: Which areas take the longest time to dispatch to on average? How can this be reduced?

minute_list_final=map(abs, minute_list) #taking only positive values of minutes
hour_list_final = map(abs, hour_list) #taking only positive values of hours
divided=[]
for element in minute_list_final:
    divided.append(element/100.0)
    

final_time=[]
for e,elements in enumerate(divided):
    s=hour_list_final[e]+elements
    final_time.append(s)
    

plt.scatter(zipcode_list, final_time,5)
axes=plt.gca()
axes.set_ylim([0,2])
plt.xticks(zipcode_list)
plt.title('Time for dispatch vs. Zipcode')
plt.show()

#BONUS:

heatmaps = sns.load_dataset("tuple_lists")
heatmaps = heatmaps.pivot("zipcode", "Time to dispatch", "Incidences")
ax = sns.heatmap(heatmaps)
