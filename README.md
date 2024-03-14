This is the third homework. The deadline is March 17, 11:59 PM. (you get an extra week because of spring break)

# Goals of the exercise. 

* Learn to configure as AWS EC2 instance and install a NoSQl(Unstructured) DB used in Bigdata systems taking MongoDB as an example.
* Learn to configure and load MongoDB with the same MLB Data you loaded in last assignment and compare its features. 
* Write queries.

# Checklist

+ [Step-1 Create the EC2 Instance](#step-1-create-the-ec2-instance)
+ [Step-2 Install the MongoDb packages](#step-2-install-the-mongodb-packages)
+ [Step-3 Enable remote access to the mongoDB server running on EC2](#step-3-enable-remote-access-to-the-mongodb-server-running-on-ec2)
+ [Step-4 Loading the MongoDB with Lahman database](#step-4-loading-the-mongodb-with-lahman-database)
+ [Step-5 Check initial Colab Connection](#step-5-check-initial-colab-connection)
+ [Step-6 Queries - 90 points-](#step-6-queries---90-points-)
+ [10-points for clarity and comment](#10-points-for-clarity-and-comment)


# Background Material

## Prior Reading

* Read all the reading materials given by Prof. Dubey
* Complete the tutorials in week 5 content.
* Additional links: https://www.tutorialspoint.com/mongodb/index.htm

## Useful Instructions -- Please watch. Most of your problems will be answered here.

* In the assignment you will create an EC2 instance and install your own mongodb server. For this you will need to see the following videos
  * Creating an EC2 Instance:  Refer to the first assignment you completed.
  * [Installing MongoDB on the EC2 Instance PDF](Create%20MongoDB%20in%20AWS%20EC2.pdf) and a [video for the same showing the steps](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2271401/View)
  * It is very important to take authentication seriousl. Change the password that provide you admin access and set it to your own.
* Once you setup mongodb and ensure that you have correctly set up an ip address and configured security groups - step 6 in pdf document referred above) you can try connecting to the database using pymongo from colab. [See the video](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2271402/View)
* Look at the examples and tutorial references provided to you in [week 5 content on brightspace](https://brightspace.vanderbilt.edu/d2l/le/content/342996/Home?itemIdentifier=D2L.LE.Content.ContentObject.ModuleCO-2258653). Note you dont have access to run these queries. But you can use them as examples.
* Remember you can also use [mongo compass](https://www.mongodb.com/products/compass) to work with the database. You can design the queries in compass and transfer to colab notebook later.

## We have also created some youtube videos that may help you in addition to the links above if you get stuck

* https://youtu.be/Ev9pLdmV6J4
* https://youtu.be/DDxIxd2HPzE
* https://youtu.be/_wpIJBoSYpQ

## Important

* It is likely that some of your tests will fail due to precision errors or other mistakes. I have created one cell per test. So you should be able to isolate them.

## Remember to update the links of all notebooks

* You did this in previous assignments - see [AcceptingaGithubassignment.pdf](AcceptingaGithubassignment.pdf)


## AWS

To access AWS go to your academy account. Follow steps like in previous assignments.

# Assignment

## Dataset Description

The updated version of the database contains complete batting and pitching statistics from 1871 to 2019, plus fielding statistics, standings, team stats, managerial records, post-season data, and more. For more details on the latest release, please read the documentation in the dataset folder. The dataset is of year 2012.
 
## Step-1 Create the EC2 Instance

Create an EC2 machine like in first assignment. You can also follow https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/?trk=gs_card&e=gs&p=gsrc

Caution: After doing your assignment make sure to shut down the EC2 instance and logout. This is necessary to avoid unnecessary charging to your AWS account.
Follow the instructions carefully to remain within **free tier**. That last part is very important.


**Note** open the security group to allow incoming connections from anywhere on port 27017. You did this in previous assignment for MYSQL. It will work similarly here.Also see the figure below.


![showing the security group configuration](images/incomingsecurityconfiguration.png)

**Note** - you dont need to create a public elastic ip. You can use the hostname given by the connection string -- when you were connecting to the instance for accessing the instance. This will be the host string that you can give for connection -- the test connection notebook.

## Step-2 Install the MongoDb packages

Login to your EC2 then type the commands:

Import the public key used for accessing package management system
	
	$wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -

Create a list file for mongoDB
	
	$echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list

	$sudo apt-get update


	$sudo apt-get install -y mongodb-org

Start the mongodb:

	$sudo service mongod start

Verify the mongod service
	
	$sudo service mongod status
	
## Step-3 Enable remote access to the mongoDB server running on EC2

Follow the instruction in the below link:

Create the remote users, but first create admin -

Enter the mongo shell on EC2

	$sudo mongo

Select admin DB

	>use admin

**Change the admin password to something else**

Create the “admin” user (you can call it whatever you want). the exit command is used to close the shell

	> db.createUser({ user: "admin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
	> db.auth("admin", "adminpassword")
	> exit



We are now going to enable authentication on the MongoDB instance, by modifying the mongod.conf file. If you’re on Linux:

	$sudo vim  /etc/mongod.conf

**Note:** to enter edit/insert mode in vim, press 'i'. To save/exit, press escape and type ':x':

Add these lines at the bottom of the YAML config file:

```
security:
    authorization: enabled
````

This will enable authentication on your database instance. 

**Important -- external access** 

By default MongoDB instance is listening on the local loopback interface only. This means that the DBMS will be accepting connections to the databases only when they come from the host itself.

So, open mongod.conf in edit mode again, as we’re going to check out the net.bindIp option. That option tells the mongod process on which interfaces it should listen.

```
net:
    bindIp: 0.0.0.0
```

With this configuration, MongoDB will be listening on 0.0.0.0 (“all the networks”). It means that mongod will listen on all the interfaces configured on your system. Pay attention that in this way you are likely going to allow everyone on the Internet to access your database (as far as they have the credentials, of course, so pay particular attention to poor passwords).

**Restart**

Now restart the mongod service (Ubuntu syntax) for the changes to take effect.
	$sudo service mongod restart
You can check if the service is up with:
	$sudo service mongod status

To create a external user login to mongo db account such as 'ubuntu'- 
Now login to mongo shell and select admin db and authenticate

	$sudo mongo

	>use admin
	>db.auth("admin", "adminpassword")

now create lahman database in mongo

	>use  lahman;

create remote user name - 'ubuntu' and a passowrd who can use lahman db (this is generally a good idea. You restrict access for people)	 

	>db.createUser({ user: "ubuntu", pwd: "yourpassword", roles: [{ role: "dbOwner", db: "lahman" }] })

Check that everything went fine by trying to authenticate, with the db.auth(user, pwd) function.

	>db.auth("ubuntu", "yourpassword")
  
 **Note** - keep your username and password private. Very important. This is what you will use to connect to the database. 

Refer to the link if you get stuck: https://medium.com/@matteocontrini/how-to-setup-auth-in-mongodb-3-0-properly-86b60aeef7e8


## Step-4 Loading the MongoDB with Lahman database

Download the lahman database to your windows or Mac Host  from http://www.seanlahman.com/files/database/
Use the lahman_sql_2012 comma delimited version (CSV) files. 

	$mkdir rawfiles
	$cd rawfiles
	$wget http://www.seanlahman.com/files/database/lahman2012-csv.zip
	$sudo apt install unzip
	$unzip lahman2012-csv.zip

if everything went well - it will look like following

```
~/rawfiles$ ls 
 AllstarFull.csv      AwardsPlayers.csv         Batting.csv       FieldingOF.csv     Managers.csv       Pitching.csv       Salaries.csv         SeriesPost.csv        TeamsHalf.csv
 Appearances.csv      AwardsShareManagers.csv   BattingPost.csv   FieldingPost.csv   ManagersHalf.csv   PitchingPost.csv   Schools.csv          Teams.csv
 AwardsManagers.csv   AwardsSharePlayers.csv    Fielding.csv      HallOfFame.csv     Master.csv        'readme 2012.txt'   SchoolsPlayers.csv   TeamsFranchises.csv
```


Then import the csv files into mongoDB using the below command.

**You may consider using an online "find-and-replace" tool to change the "username" and "yourpassword" fields for all below queries. This will make this process faster.**

	mongoimport -d lahman -c AllstarFull --type csv --file AllstarFull.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsSharePlayers --type csv --file AwardsSharePlayers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Appearances --type csv --file Appearances.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsManagers --type csv --file AwardsManagers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsShareManagers --type csv --file AwardsShareManagers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsPlayers --type csv --file AwardsPlayers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Batting --type csv --file Batting.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Fielding --type csv --file Fielding.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c FieldingOF --type csv --file FieldingOF.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c FieldingPost --type csv --file FieldingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c HallOfFame --type csv --file HallOfFame.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c Managers --type csv --file Managers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c ManagersHalf --type csv --file ManagersHalf.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Master --type csv --file Master.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Pitching --type csv --file Pitching.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c PitchingPost --type csv --file PitchingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Salaries --type csv --file Salaries.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Schools --type csv --file Schools.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c SchoolsPlayers --type csv --file SchoolsPlayers.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c SeriesPost --type csv --file SeriesPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Teams --type csv --file Teams.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c TeamsFranchises --type csv --file TeamsFranchises.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c TeamsHalf --type csv --file TeamsHalf.csv --headerline --username "ubuntu" --password "yourpassword"


**Alternatively, you can use a cool shell command to import all**

	$for file in `ls *.csv`; do mongoimport -d lahman -c `basename $file .csv` --type csv $file --headerline --username "ubuntu" --password "yourpassword";done



## Step-5 Check initial Colab Connection

Run the Colab connection script [test-setup.ipynb](test-setup.ipynb) and ensure that you get the connection and the number of tables correctly. Make sure that you update the database name, the username and the password. 

**Note** the port should be 27017 and the host should be the hostname of your AWS instance.

Remember to shutoff the EC2 instance when you are not using it.

At this point check initial connection from compass as well. During connection choose lahman as the authentication database. And provide the username and password you created for lahman database.

 If you opened the ports correctly the connection will work and you can get something like following


![](images/compass.png)

![](images/compass2.png)

![](images/compass3.png)

## Step-6 Queries - 90 points

Implement a function per query in a file called [hw3.ipynb](hw3.ipynb). Record the answers there and save it back to your repository. 

The queries are

1. The number of all stars in allstarfull.
2. The most home runs in a season by a single player (using the batting table). (Already solved as an example for you.)
3. The playerid of the player with the most home runs in a season.
4. The number of leagues in the batting table.
5. Barry Bond's average batting average (playerid = 'bondsba01') where batting average is hits / at-bats. Note you will nead to cast hits to get a decimal: cast(h as real)
6. The teamid with the fewest hits in the year 2000 (ie., yearid = '2000'). Return both the teamid, and the number of hits. Note you can use ORDER BY column and LIMIT 1.
7. The teamid in the year 2000 (i.e., yearid = '2000')  with the highest average batting average. Return the teamid and the average. To prevent divsion by 0, limit at-bats > 0.
8. The number of all stars the giants (teamid = 'SFN') had in 2000.
9. The yearid which the giants had the most all stars.
10. The average salary in year 2000.
11. The number of positions (e.g., catchers, pitchers) that have average salaries greather than 2000000 in yearid 2000. You will need to join fielding with salaries. Also consider using a HAVING clause.
12. The number of errors Barry Bonds had in 2000. 
13. The average salary of all stars in 2000.
14. The average salary of non-all stars in 2000.

## 10-points for clarity and comment

I expect you to write a few points describing the logic of each query. Why you did what you did.

This is the third homework. The deadline is March 17, 11:59 PM. (you get an extra week because of spring break)

# Goals of the exercise. 

* Learn to configure as AWS EC2 instance and install a NoSQl(Unstructured) DB used in Bigdata systems taking MongoDB as an example.
* Learn to configure and load MongoDB with the same MLB Data you loaded in last assignment and compare its features. 
* Write queries.

# Checklist

+ [Step-1 Create the EC2 Instance](#step-1-create-the-ec2-instance)
+ [Step-2 Install the MongoDb packages](#step-2-install-the-mongodb-packages)
+ [Step-3 Enable remote access to the mongoDB server running on EC2](#step-3-enable-remote-access-to-the-mongodb-server-running-on-ec2)
+ [Step-4 Loading the MongoDB with Lahman database](#step-4-loading-the-mongodb-with-lahman-database)
+ [Step-5 Check initial Colab Connection](#step-5-check-initial-colab-connection)
+ [Step-6 Queries - 90 points-](#step-6-queries---90-points-)
+ [10-points for clarity and comment](#10-points-for-clarity-and-comment)


# Background Material

## Prior Reading

* Read all the reading materials given by Prof. Dubey
* Complete the tutorials in week 5 content.
* Additional links: https://www.tutorialspoint.com/mongodb/index.htm

## Useful Instructions -- Please watch. Most of your problems will be answered here.

* In the assignment you will create an EC2 instance and install your own mongodb server. For this you will need to see the following videos
  * Creating an EC2 Instance:  Refer to the first assignment you completed.
  * [Installing MongoDB on the EC2 Instance PDF](Create%20MongoDB%20in%20AWS%20EC2.pdf) and a [video for the same showing the steps](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2271401/View)
  * It is very important to take authentication seriousl. Change the password that provide you admin access and set it to your own.
* Once you setup mongodb and ensure that you have correctly set up an ip address and configured security groups - step 6 in pdf document referred above) you can try connecting to the database using pymongo from colab. [See the video](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2271402/View)
* Look at the examples and tutorial references provided to you in [week 5 content on brightspace](https://brightspace.vanderbilt.edu/d2l/le/content/342996/Home?itemIdentifier=D2L.LE.Content.ContentObject.ModuleCO-2258653). Note you dont have access to run these queries. But you can use them as examples.
* Remember you can also use [mongo compass](https://www.mongodb.com/products/compass) to work with the database. You can design the queries in compass and transfer to colab notebook later.

## We have also created some youtube videos that may help you in addition to the links above if you get stuck

* https://youtu.be/Ev9pLdmV6J4
* https://youtu.be/DDxIxd2HPzE
* https://youtu.be/_wpIJBoSYpQ

## Important

* It is likely that some of your tests will fail due to precision errors or other mistakes. I have created one cell per test. So you should be able to isolate them.

## Remember to update the links of all notebooks

* You did this in previous assignments - see [AcceptingaGithubassignment.pdf](AcceptingaGithubassignment.pdf)


## AWS

To access AWS go to your academy account. Follow steps like in previous assignments.

# Assignment

## Dataset Description

The updated version of the database contains complete batting and pitching statistics from 1871 to 2019, plus fielding statistics, standings, team stats, managerial records, post-season data, and more. For more details on the latest release, please read the documentation in the dataset folder. The dataset is of year 2012.
 
## Step-1 Create the EC2 Instance

Create an EC2 machine like in first assignment. You can also follow https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/?trk=gs_card&e=gs&p=gsrc

Caution: After doing your assignment make sure to shut down the EC2 instance and logout. This is necessary to avoid unnecessary charging to your AWS account.
Follow the instructions carefully to remain within **free tier**. That last part is very important.


**Note** open the security group to allow incoming connections from anywhere on port 27017. You did this in previous assignment for MYSQL. It will work similarly here.Also see the figure below.


![showing the security group configuration](images/incomingsecurityconfiguration.png)

**Note** - you dont need to create a public elastic ip. You can use the hostname given by the connection string -- when you were connecting to the instance for accessing the instance. This will be the host string that you can give for connection -- the test connection notebook.

## Step-2 Install the MongoDb packages

Login to your EC2 then type the commands:

Import the public key used for accessing package management system
	
	$wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -

Create a list file for mongoDB
	
	$echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list

	$sudo apt-get update


	$sudo apt-get install -y mongodb-org

Start the mongodb:

	$sudo service mongod start

Verify the mongod service
	
	$sudo service mongod status
	
## Step-3 Enable remote access to the mongoDB server running on EC2

Follow the instruction in the below link:

Create the remote users, but first create admin -

Enter the mongo shell on EC2

	$sudo mongo

Select admin DB

	>use admin

**Change the admin password to something else**

Create the “admin” user (you can call it whatever you want). the exit command is used to close the shell

	> db.createUser({ user: "admin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
	> db.auth("admin", "adminpassword")
	> exit



We are now going to enable authentication on the MongoDB instance, by modifying the mongod.conf file. If you’re on Linux:

	$sudo vim  /etc/mongod.conf

**Note:** to enter edit/insert mode in vim, press 'i'. To save/exit, press escape and type ':x':

Add these lines at the bottom of the YAML config file:

```
security:
    authorization: enabled
````

This will enable authentication on your database instance. 

**Important -- external access** 

By default MongoDB instance is listening on the local loopback interface only. This means that the DBMS will be accepting connections to the databases only when they come from the host itself.

So, open mongod.conf in edit mode again, as we’re going to check out the net.bindIp option. That option tells the mongod process on which interfaces it should listen.

```
net:
    bindIp: 0.0.0.0
```

With this configuration, MongoDB will be listening on 0.0.0.0 (“all the networks”). It means that mongod will listen on all the interfaces configured on your system. Pay attention that in this way you are likely going to allow everyone on the Internet to access your database (as far as they have the credentials, of course, so pay particular attention to poor passwords).

**Restart**

Now restart the mongod service (Ubuntu syntax) for the changes to take effect.
	$sudo service mongod restart
You can check if the service is up with:
	$sudo service mongod status

To create a external user login to mongo db account such as 'ubuntu'- 
Now login to mongo shell and select admin db and authenticate

	$sudo mongo

	>use admin
	>db.auth("admin", "adminpassword")

now create lahman database in mongo

	>use  lahman;

create remote user name - 'ubuntu' and a passowrd who can use lahman db (this is generally a good idea. You restrict access for people)	 

	>db.createUser({ user: "ubuntu", pwd: "yourpassword", roles: [{ role: "dbOwner", db: "lahman" }] })

Check that everything went fine by trying to authenticate, with the db.auth(user, pwd) function.

	>db.auth("ubuntu", "yourpassword")
  
 **Note** - keep your username and password private. Very important. This is what you will use to connect to the database. 

Refer to the link if you get stuck: https://medium.com/@matteocontrini/how-to-setup-auth-in-mongodb-3-0-properly-86b60aeef7e8


## Step-4 Loading the MongoDB with Lahman database

Download the lahman database to your windows or Mac Host  from http://www.seanlahman.com/files/database/
Use the lahman_sql_2012 comma delimited version (CSV) files. 

	$mkdir rawfiles
	$cd rawfiles
	$wget http://www.seanlahman.com/files/database/lahman2012-csv.zip
	$sudo apt install unzip
	$unzip lahman2012-csv.zip

if everything went well - it will look like following

```
~/rawfiles$ ls 
 AllstarFull.csv      AwardsPlayers.csv         Batting.csv       FieldingOF.csv     Managers.csv       Pitching.csv       Salaries.csv         SeriesPost.csv        TeamsHalf.csv
 Appearances.csv      AwardsShareManagers.csv   BattingPost.csv   FieldingPost.csv   ManagersHalf.csv   PitchingPost.csv   Schools.csv          Teams.csv
 AwardsManagers.csv   AwardsSharePlayers.csv    Fielding.csv      HallOfFame.csv     Master.csv        'readme 2012.txt'   SchoolsPlayers.csv   TeamsFranchises.csv
```


Then import the csv files into mongoDB using the below command.

**You may consider using an online "find-and-replace" tool to change the "username" and "yourpassword" fields for all below queries. This will make this process faster.**

	mongoimport -d lahman -c AllstarFull --type csv --file AllstarFull.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsSharePlayers --type csv --file AwardsSharePlayers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Appearances --type csv --file Appearances.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsManagers --type csv --file AwardsManagers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsShareManagers --type csv --file AwardsShareManagers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c AwardsPlayers --type csv --file AwardsPlayers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Batting --type csv --file Batting.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Fielding --type csv --file Fielding.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c FieldingOF --type csv --file FieldingOF.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c FieldingPost --type csv --file FieldingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c HallOfFame --type csv --file HallOfFame.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c Managers --type csv --file Managers.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c ManagersHalf --type csv --file ManagersHalf.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Master --type csv --file Master.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Pitching --type csv --file Pitching.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c PitchingPost --type csv --file PitchingPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Salaries --type csv --file Salaries.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Schools --type csv --file Schools.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c SchoolsPlayers --type csv --file SchoolsPlayers.csv --headerline --username "ubuntu" --password "yourpassword"
  	mongoimport -d lahman -c SeriesPost --type csv --file SeriesPost.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c Teams --type csv --file Teams.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c TeamsFranchises --type csv --file TeamsFranchises.csv --headerline --username "ubuntu" --password "yourpassword"
 	mongoimport -d lahman -c TeamsHalf --type csv --file TeamsHalf.csv --headerline --username "ubuntu" --password "yourpassword"


**Alternatively, you can use a cool shell command to import all**

	$for file in `ls *.csv`; do mongoimport -d lahman -c `basename $file .csv` --type csv $file --headerline --username "ubuntu" --password "yourpassword";done



## Step-5 Check initial Colab Connection

Run the Colab connection script [test-setup.ipynb](test-setup.ipynb) and ensure that you get the connection and the number of tables correctly. Make sure that you update the database name, the username and the password. 

**Note** the port should be 27017 and the host should be the hostname of your AWS instance.

Remember to shutoff the EC2 instance when you are not using it.

At this point check initial connection from compass as well. During connection choose lahman as the authentication database. And provide the username and password you created for lahman database.

 If you opened the ports correctly the connection will work and you can get something like following


![](images/compass.png)

![](images/compass2.png)

![](images/compass3.png)

## Step-6 Queries - 90 points

Implement a function per query in a file called [hw3.ipynb](hw3.ipynb). Record the answers there and save it back to your repository. 

The queries are

1. The number of all stars in allstarfull.
2. The most home runs in a season by a single player (using the batting table). (Already solved as an example for you.)
3. The playerid of the player with the most home runs in a season.
4. The number of leagues in the batting table.
5. Barry Bond's average batting average (playerid = 'bondsba01') where batting average is hits / at-bats. Note you will nead to cast hits to get a decimal: cast(h as real)
6. The teamid with the fewest hits in the year 2000 (ie., yearid = '2000'). Return both the teamid, and the number of hits. Note you can use ORDER BY column and LIMIT 1.
7. The teamid in the year 2000 (i.e., yearid = '2000')  with the highest average batting average. Return the teamid and the average. To prevent divsion by 0, limit at-bats > 0.
8. The number of all stars the giants (teamid = 'SFN') had in 2000.
9. The yearid which the giants had the most all stars.
10. The average salary in year 2000.
11. The number of positions (e.g., catchers, pitchers) that have average salaries greather than 2000000 in yearid 2000. You will need to join fielding with salaries. Also consider using a HAVING clause.
12. The number of errors Barry Bonds had in 2000. 
13. The average salary of all stars in 2000.
14. The average salary of non-all stars in 2000.

## 10-points for clarity and comment

I expect you to write a few points describing the logic of each query. Why you did what you did.

