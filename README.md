# MariaDB-Batch-File
Usage of batch file-Batch file is created to execute the multiple usecases at the same time. 

Prerequisite for running  mySuperMon MariaDB batch file
First add your MariaDB agent, application and data source of your application.

Do the following changes in the MariaDB batch files folder

Step 1: In Application_Configuration.json file configure ServiceURL,Username, Password, ApplicationIdentifier, recordingStopperTime and UsecaseID like this
```ruby
{"serviceUrl":"https://app.mysupermon.com",
 "username":"Enter Your Username", 
"password":"Enter Your Password", 
"applicationIdentifier":"Enter Your applicationidentifier", 
"recordingStopperTime":15,
"useCaseId":"Enter Your usecaseId"}
```

In above json configure your details. 
 
Step 2: Create Usecase_Customer_Queries.txt file add the queries of the configured database.

Step 3: Result_Usecase1.txt file is created to store the Usecase_Customer execution result.

Step 4: The response.json file is created in order to store the report of the usecase after executed

Step 5: In start and Stop Recording.bat file the start/stop functionality using start/stop API of mySupermMon is implemented.

5.1 Configured all the details specified in Application_Configuration.json in start/stop batch file using this below code
```ruby
more Application_Configuration.json | jq-win64.exe ".serviceUrl" >> temp.txt
set /p serviceUrl=<temp.txt
del -f temp.txt
```
You just need to add your Application_Configuration.json file here

5.2 Generated the access token using configured application details using this code
```ruby
curl -X POST %serviceUrl%oauth/token -u "performanceDashboardClientId:ljknsqy9tp6123" -d "grant_type=password" -d "username=%username%" -d "password=%password%" >> temp.txt
more temp.txt | jq-win64.exe ".access_token" >> accessToken.txt
set /p accessToken=<accessToken.txt
del -f temp.txt
del -f accessToken.txt
if %accessToken%==null (goto :showMessage)
```
In above code the curl is is a free command line tool. It uses URL syntax to transfer data to and from servers.

5.3 Start recording of mySuperMon using start reccording API 
```ruby
curl -v -H "Authorization: Bearer %accessToken%", -H "applicationIdentifier:%applicationIdentifier%" -X GET %serviceUrl%devaten/data/startRecording?usecaseIdentifier="Signup"
```
In above the usecaseIdentifier parameter is passed to the start recording API. 


5.4 Now pass your Usecase_Customer_Queries.txt as input and Result_Usecase1.txt as ouput file for storing the result, using this code
```ruby
mysql -h localhost -P 3307 -uABC -pabc -D test < Usecase_Customer_Queries.txt > Result_Usecase1.txt
```
In above code -h is Host name, -P is Port of host, -u is username, -p is password and -D for giving database name

5.5 Stop recording of mySuperMon using stop reccording API 
```ruby
curl -v -H "Authorization: Bearer %accessToken%", -H "applicationIdentifier:%applicationIdentifier%" -X GET %serviceUrl%devaten/data/stopRecording?usecaseIdentifier="Signup&inputSource=batFile"
```

This API aceepts usecaseIdentifier and inputSource as parameters.

5.6 Generate the report start/stop recording using mySuperMon generate report API 
```ruby
curl -v -H "Authorization: Bearer %accessToken%" -H "applicationIdentifier:%applicationIdentifier%" -X GET %serviceUrl%devaten/data/generateReport >> response.json
 ```
In this API at the end passed the response.json file to store the response.

Step 6: You can see all executed usecase and usecase details on mySuperMon dashboard.





 
 

 
