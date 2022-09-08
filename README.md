# SQL-Murder-Mystery

A fun game to test your querying skills.

This contains every query I made in order, so you can see my thought process utilising SQL.

Link:											
https://mystery.knightlab.com/											

| Database: |
| ---------------- |
| name			|										
| crime_scene_report	|												
| drivers_license				|									
| person | 					
| facebook_event_checkin |													
| interview |											
| get_fit_now_member |		
| get_fit_now_check_in |							
| income |								
| solution | 


First we look at the initial crime scene report


| date      | type     |	description | city | 
------------ | -------- | ------------ | ---- | 
| 20180115	| murder	| Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".| SQL City | 

now we want to find the person who lives at the last house on northwestern drive using the person database

|  id	 | name | 	license_id | address_number | 	address_street_name	 | ssn | 				
 | ---- | --- | ---------- | -------------- | ---------------------- | ------ | 
| 14887 | Morty Schapiro | 118009 | 4919 | Northwestern  Dr	| 111564949	 | 

the last house could be at either end of the road:

| id| name	| license_id | 	address_number | 	address_street_name	| ssn		| 			
| --- | ----- | ------------- | ------------- | ------------------ | -------| 
| 89906 | 	Kinsey Erickson| 	510019 | 309	| Northwestern Dr | 	63528766	|

now to find the full name of the other witness

| id | 	name	| license_id	| address_number	| address_street_name	| ssn | 			
| --- | ------  | ---------- | ------------- | --------------------- | --- | 
| 16371	| Annabel Miller	| 490173	| 103	| Franklin Ave	| 318771143	| 

I have noticed that license_id appears in both the person database and the license database.	We can create a table with all of our persons of interest in one table with all their personal information 

| id |	age	| height |	eye_color |	hair_color |	gender |	plate_number |	car_make |	car_model	 |	
|	-- |	--- |	------ |	--------- |	--------- |	-------- |	------------ |	------ |	--------- |
| 490173 |	35 |	65 |	green |	brown |	female |	23AM98 |	Toyota |	Yaris	

|name           |license_id|address_number|address_street_name|ssn      |age |height|eye_color|hair_color|gender|plate_number|car_make     |car_model|
|---------------|----------|--------------|-------------------|---------|----|------|---------|----------|------|------------|-------------|---------|
|Morty Schapiro |118009    |4919          |Northwestern Dr    |111564949|64  |84    |blue     |white     |male  |00NU00      |Mercedes-Benz|E-Class  |     
|Annabel Miller |490173    |103           |Franklin Ave       |318771143|35  |65    |green    |brown     |female|23AM98      |Toyota       |Yaris    |      
|Kinsey Erickson|510019    |309           |Northwestern Dr    |635287661|null|null  |null     |null      |null  |null        |null         |null     |

We can see that Kinsey has a license id but there is no information.						
We can check who the interview records using person id, I didn’t include this column in the previous query so I will update it.	

|name           |person_id|address_number|address_street_name|ssn      |age |height|eye_color|hair_color|gender|plate_number|car_make     |car_model|FIELD14|
|---------------|---------|--------------|-------------------|---------|----|------|---------|----------|------|------------|-------------|---------|-------|
|Morty Schapiro |14887    |4919          |Northwestern Dr    |111564949|64  |84    |blue     |white     |male  |00NU00      |Mercedes-Benz|E-Class  |       |
|Annabel Miller |16371    |103           |Franklin Ave       |318771143|35  |65    |green    |brown     |female|23AM98      |Toyota       |Yaris    |       |
|Kinsey Erickson|89906    |309           |Northwestern Dr    |635287661|null|null  |null     |null      |null  |null        |null         |null     |       |

I renamed the column to match the column name in the interview table

|person_id      |transcript|
|---------------|----------|
|14887          |I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags.

It turns out Kinsey was not a witness so we will exclude her information.

|Name           |person_id|address_number|address_street_name|ssn      |age |height|eye_color|hair_color|gender|plate_number|car_make     |car_model|
|---------------|---------|--------------|-------------------|---------|----|------|---------|----------|------|------------|-------------|---------|
|Morty Schapiro |14887    |4919          |Northwestern Dr    |111564949|64  |84    |blue     |white     |male  |00NU00      |Mercedes-Benz|E-Class  |
|Annabel Miller |16371    |103           |Franklin Ave       |318771143|35  |65    |green    |brown     |female|23AM98      |Toyota       |Yaris    |

We will add the statement to this information table

|person_id      |name  |statement|address_number |address_street_name|ssn |age |height|eye_color|hair_color|gender|plate_number |car_make|car_model|
|---------------|------|--------------------------------------------------------------|---------------|-------------------|----|----|------|---------|----------|------|-------------|--------|---------|
|14887          |Morty Schapiro|<sub>I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".</sub>|4919           |Northwestern Dr    |111564949|64  |84    |blue     |white     |male  |00NU00       |Mercedes-Benz|E-Class  |
|16371          |Annabel Miller|<sub>I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.</sub>|103            |Franklin Ave       |318771143|35  |65    |green    |brown     |female|23AM98       |Toyota  |Yaris    |


Starting with Morty's statement we can narrow down the list of potential suspects from the gym's member data

|id             |person_id|name|membership_start_date|membership_status
|---------------|---------|----|---------------------|-----------------|
|48Z7A          |28819    |Joe Germuska|20160305             |gold             | 
|48Z55          |67318    |Jeremy Bowers|20160101             |gold             |

 We can use this and Annabel's statement to see who was at the gym at the same time as her on January 9th.
 
 |membership_id|check_in_date|check_in_time|check_out_time|
|-------------|-------------|-------------|--------------|
|X0643        |20180109     |957          |1164          |
|UK1F2        |20180109     |344          |518           | 
|XTE42        |20180109     |486          |1124          |
|6LSTG        |20180109     |399          |515           |  
|7MWHJ        |20180109     |273          |885           |    
|GE5Q8        |20180109     |367          |959           |   
|48Z7A        |20180109     |1600         |1730          |      
|48Z55        |20180109     |1530         |1700          |     
|90081        |20180109     |1600         |1700          |    

We need to join the check in data with the member data to get names and add Annabel's data to see who had a crossover

|person_id|name    |check_in_date|check_in_time|
|---------|--------|-------------|-------------|
|28819    |Joe Germuska|20180109     |1600         |
|67318    |Jeremy Bowers|20180109     |1530         |
|16371    |Annabel Miller|20180109     |1600         |

 We have 2 suspects that fit the time given by Annabel and account details at the gym given by Morty. We can now find out who owns a car with a license plate that contains H42W.
 
|id   |age     |height|eye_color|hair_color|gender|plate_number|car_make |car_model|
|-----|--------|------|---------|----------|------|------------|---------|---------|
|183779|21      |65    |blue     |blonde    |female|H42W0X      |Toyota   |Prius    |   
|423327|30      |70    |brown    |brown     |male  |0H42W2      |Chevrolet|Spark LS |  
|664760|21      |71    |black    |black     |male  |4H42WR      |Nissan   |Altima   |  

We will now join this with the person database to get their personal details.

|name |license_id|plate_number|car_make|car_model|
|-----|----------|------------|--------|---------|
|Tushar Chandra|664760    |4H42WR      |Nissan  |Altima   |  
|Jeremy Bowers|423327    |0H42W2      |Chevrolet|Spark LS |    
|Maxine Whitely|183779    |H42W0X      |Toyota  |Prius    |  

So now we have 2 potential suspects and 3 potential co-conspiritors.

Lets add on the personal information of the conspiritors.

|id   |name    |license_id|address_number|address_street_name  |ssn      |id    |age|height|eye_color|hair_color|gender|plate_number|car_make |
|-----|--------|----------|--------------|---------------------|---------|------|---|------|---------|----------|------|------------|---------|
|51739|Tushar Chandra|664760    |312           |Phi St               |137882671|664760|21 |71    |black    |black     |male  |4H42WR      |Nissan   |
|67318|Jeremy Bowers|423327    |530           |Washington Pl, Apt 3A|871539279|423327|30 |70    |brown    |brown     |male  |0H42W2      |Chevrolet|
|78193|Maxine Whitely|183779    |110           |Fisk Rd              |137882671|183779|21 |65    |blue     |blonde    |female|H42W0X      |Toyota   |

Jeremy Bowers fits both descriptions, we found the murderer.

## Part 2: Finding out more about the murder	

Lets look at the transcript given by Jeremy

|person_id|name    |transcript|
|---------|--------|----------|
|67318    |Jeremy Bowers|I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.|  

 We can use this information to query the driver license table
 
|id   |age     |height|eye_color|
|-----|--------|------|---------|
|202298|68      |66    |green    | 
|291182|65      |66    |blue     |  
|918773|48      |65    |black    |  

We can join the person table to get names and SSN, using this to also join data from the income table as Jeremy descibed her as wealthy.

|id   |license_id|name|ssn |
|-----|----------|----|----|
|99716|202298    |Miranda Priestly|987756388| 
|90700|291182    |Regina George|337169072| 
|78881|918773    |Red Korb|961388910|

The annual income did not help narrow it down, as 2 have a large income and the other could be wealthy without a job.

We can look at facebook check in data to see who attended the SQL Symphony Concert 3 times in December 2017.

|person_id|event_id|event_name          |date    |
|---------|--------|--------------------|--------|
|99716    |1143    |SQL Symphony Concert|20171206|   
|99716    |1143    |SQL Symphony Concert|20171212|
|99716    |1143    |SQL Symphony Concert|20171229|  

We can see one person attended 3 times and matching to the previous table confirm that Jeremy was hired by Miranda Priestly.

|id   |name|license_id          |address_number|
|-----|----|--------------------|--------------|
|99716|Miranda Priestly|202298              |1883          | 

## Challenge: Part 2 in just 2 queries		

|person_id|name|transcript          |
|---------|----|--------------------|
|67318    |Jeremy Bowers|I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.| 

|id   |license_id|name                |ssn     |annual_income|event_name          |event_date|
|-----|----------|--------------------|--------|-------------|--------------------|----------|
|99716|202298    |Miranda Priestly    |987756388|310000       |SQL Symphony Concert|20171206  |  
|99716|202298    |Miranda Priestly    |987756388|310000       |SQL Symphony Concert|20171212  |
|99716|202298    |Miranda Priestly    |987756388|310000       |SQL Symphony Concert|20171229  |  
|90700|291182    |Regina George       |337169072|null         |null                |null      |  
|78881|918773    |Red Korb            |961388910|278000       |null                |null      |  

 Miranda is the only person to match all the entire description.
