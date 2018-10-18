# Culture-Works-Spark-Bot---API-and-Misc-Updates


## Project Objectives
Update the spark bot to add REST APIs for the admin user interface
Update the spark bot accordingly to work with the added REST APIs
The prototype is also provided to you, and you need to ensure the REST APIs cover all the functionalities in the prototype
 
## Project Background
We previously launched two challenges to build the spark bot:
http://www.topcoder.com/challenge-details/30066653/?type=develop&noncache=true
http://www.topcoder.com/challenge-details/30066882/?type=develop&noncache=true
 
But a user interface for admin is missing for admin users to manage the data in the spark bot.
 
Technology Stack
Node 10+
MS Team Bot
Postgres
 
## Individual requirements
Please take a look at the UI prototype spec first: http://www.topcoder.com/challenge-details/30072468/?type=develop&noncache=true

1. Postgres DB Persistence - all data should be stored to Postgres, this includes admin accounts data, point records data, hashtag (or challenge) configuration data as well as rank (badge) configuration data. 

Note that in the point data record, the point value should be stored together with it. So it doesn't matter if the user changes the point-value associated with the hashtag (or challenge). 
'''
adminUser table:
         adminUserId: Primary Key
adminUsername: String, unique, case-insensitive
password: String
role: String - super-admin or non-super-admin
active: Boolean
createdOn: Timestamp
updatedOn: Timestamp

msTeam // MS Team
teamId: Primary Key
msTeamId: the team id from the bot
msTeamName: the team name from the bot
active: Boolean
createdOn: Timestamp
updatedOn: Timestamp

msTeamUser  // MS Team user
teamUserId: Primary Key
msTeamUserId: the team user id from the bot
msTeamUserName: the team user name from the bot
active: Boolean
createdOn: Timestamp
updatedOn: Timestamp

adminUserTeam table: // stores the association between adminUser and msTeam
adminUserId - references adminUser table
teamId - references msTeam table

teamChallenge table:   (teamId + challengeName are unique), note that we only allow 7 active challenges for each team. Make the 7 number configurable. 
         challengeId: Primary Key
teamId: references msTeam table
challengeName: String
pointValue: Number
active: Boolean
createdOn: Timestamp
updatedOn: Timestamp

teamRank table:
teamId: references the msTeam table, it's used as the primary key
ranks: [{
   name: String,
   pointValue: Number
}],
createdOn: Timestamp
updatedOn: Timestamp

pointData table
teamId: references msTeam table
teamUserId: reference msTeamUser table
challengeId: references teamChallenge table
pointValue: Number
proof: String
createdOn: Timestamp
updatedOn: Timestamp  - this field is returned as timestamp in response
'''

2. The following APIs should be defined (and please remove all existing APIs). 
For all APIs, the input should be validated, and the 400 (for bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 500 (Internal Error) should be properly used.
 
Login page:
<code> POST /login </code> - the request payload should contain username and password, the response should contain a jwt token and user role (super-admin or non-super-admin). 
Common Dropdown data - these APIs are only used on point-management page
<code> GET /teams?active=xx </code> (authentication needed) - it should return teams from msTeam table. The active parameter can be true or false to return active or inactive teams. If not provided return all teams. The response should contain the team id and team name. (No need to return the msTeamId field)
GET /teams/{teamId}/users?active=xx (authentication needed) - it should return users in given team from msTeamUser table. The active parameter can be true or false to return active or inactive team users. If not provided return all team users. The response should contain the user id and user name. (No need to return msTeamUserId field)
Page Header for non-super admin user:
GET /user/teams (for non-super admin only) - it should return all the teams associated with non-super admin user. The response should contain the team id and team name. 
Team Settings Page - Challenges Section - for non-super admin
GET /teams/{teamId}/challenges?active=xx - it should return all the challenges for the given team. The active parameter can be true or false to return active or inactive challenges. If not provided return all challenges. The response contains an array of challenge data, each challenge will have id, name, active and pointValue field. 
POST /teams/{teamId}/challenges - the request payload contains the challenge data, and it will add a new challenge to the team.
PUT /teams/{teamId}/challenges/{challengeId} - the request payload contains the challenge data (no id), and it will update an existing challenge in team
DELETE /teams/{teamId}/challenges/{challengeId} - delete the challenge, note that if the challenge is referenced, the delete should fail. 
Team Settings Page - Rank Section - for non-super admin
GET /teams/{teamId}/ranks - it should try to load the ranks from the db first, if not present, load the default value from config file. The response is an array like: [ { "name": "Scout", "point": 100 }, .... ]. 
PUT /teams/{teamId}/ranks - it should update the ranks for the team, the request payload is the same as the GET response above. 
Point Management Page for non-super admin:
GET /teams/{teamId}/points?username=xx&challengeId=xx&startDate=xx&endDate=xx&pageNo=xx&pageSize=xx&sortBy=xx&sortOrder=xx  - it should return the points data in the given team. The response should contain the totalPages as well as an array of point records. Each point record will contain: user id, username, challenge id, challenge name, point value, timestamp, and proof. And it should support the following filters: 
username - match with start with, case-insensitive
challengeId - match this exactly
startDate & endDate - range to match the timestamp
pageNo & pageSize - used for paging, show 1st page with 20 records if not provided.
sortBy & sortOrder - used for sorting, sort by timestamp in descending if not provided.
all filters are optional. 
POST /teams/{teamId}/points to add a point record in the team. The request payload will contain: user id, username, challenge id, point value, timestamp, proof. 
PUT /teams/{teamId}/points/{pointId} to update a point record in the team. The request payload will contain: user id, username, challenge id, point value, timestamp, proof. 
DELETE /teams/{teamId}/points/{pointId} to remove a point record from the team. 
GET /teams/{teamId}/points/export - download all point records in the given team as CSV file. The CSV file should have a header, and each record should contain: point record id, user id, username, challenge id, challenge name, point value, timestamp, and proof. 
POST /teams/{teamId}/points/import - the request should be a multi-part form including the CSV input file. The CSV file should have a header, and each record should contain:  user id, username, challenge id, challenge name, point value, timestamp, and proof. The data will be *added* into the team. 
Point Management Page for super admin:
GET /points?teamId=xx&username=xx&challengeId=xx&startDate=xx&endDate=xx&pageNo=xx&pageSize=xx&sortBy=xx&sortOrder=xx  - it should return the points data. The response should contain the totalPages as well as an array of point records. Each point record will contain: team id, team name, user id, username, challenge id, challenge name, point value, timestamp, and proof. And it should support the following filters: 
teamId - match this exactly
username - match with start with, case insensitive
challengeId - match this exactly
startDate & endDate - range to match the timestamp
pageNo & pageSize - used for paging, show 1st page with 20 records if not provided.
sortBy & sortOrder - used for sorting, sort by timestamp in descending if not provided.
all filters are optional. 
GET /points/export - download all point records for all teams as CSV file. The CSV file should have a header, and each record should contain: point record id, team id, team name, user id, username, challenge id, challenge name, point value, timestamp, and proof. 

Reporting page - for non-super admin
GET /teams/{teamId}/leaderboard - return the top 10 users with highest points in the team. For each user, the user id, username, and total points should be returned. 
GET /teams/{teamId}/challengeStat - it should return the statistics for all challenges in the team (all challegens, even some has 0 values). For each challenge, the challenge id, challenge name, count of point record for this challenge, and sum of the points for this challenge. 
GET /teams/{teamId}/challengeStatOvertime?type=xxx - it should return statistics for all challenges over the last-12-month (including current month) or last-24-week (including current week) in the team. The type can be last-12-month or last-24-week. The response should be an array with each element containing the: period (e.g. 2018/11 for month, 2018/10/8 for start date of the week, challenge id, challenge name, count of point record for this challenge in current period, and sum of the points for this challenge in current period . 
Reporting page - for super admin
GET /teams/{teamId}/leaderboard - same as non-super admin, except super admin is able to view all teams' leaderboards
GET /challengeStat - it should return the statistics for all challenges in all teams (all challegens, even some has 0 values). For each challenge, the challenge id, challenge name, count of point record for this challenge, and sum of the points for this challenge. 
GET /challengeStatOvertime?type=xxx - it should return statistics for all challenges over the last-12-month (including current month) or last-24-week (including current week) in all teams. The type can be last-12-month or last-24-week. The response should be an array with each element containing the: period (e.g. 2018/11 for month, 2018/10/8 for start date of the week, challenge id, challenge name, count of point record for this challenge in current period, and sum of the points for this challenge in current period . 
Admin settings page - for super admin
Users section:
GET /admins?teamId=xx&username=xx&active=xxx&pageNo=x&pageSize=x&sortBy=x&sortOrder=x - it should return all non-super users. The repsonse should contain a totalPages as well an array with element containing: adminUserId, adminUsername, active, and an teams array with each element containing: teamId, teamName. It supports following filters:
teamId - match this exactly
adminUsername - match with start with, case insensitive
pageNo & pageSize - used for paging, show 1st page with 20 records if not provided.
sortBy & sortOrder - used for sorting, sort by team name in descending if not provided.
all filters are optional. 
POST /admins - it's used to create a new non-super admin. The request payload contains field: adminUsername, password, active, teams array (each element is a teamId). Note that the adminUsername is unique.
PUT /admins/{adminUserId} - it's used to update a non-super admin. The request payload contains field: adminUsername, active, teams array, resetPassword (flag indicating the password should be changed or not), newPassword (only needed if resetPassword is true). 

3. Update the bot implementation
The hashtags (challenges) are now changeable. So the bot implementation should be updated to load it from db, and show them in the welcome message and dropdown list. 
Combine the command with @Spark with a single command. e.g. "@Spark #Blog  link to blog" or "@Spark #Blog @Person’s name link to blog". The current approach requires two separate steps to get all required data to earn points. 
The bot should listen to messages and insert/update/delete (mark as inactive) records in the msTeam & msTeamUser table. 
4. Please also create a script to create a super admin user as well. Note that this needs to work on Azure, so there should be step to create one in Azure Postgres db as well. 
Final Submission Guidelines
Submission Deliverable
- Updated Source code (with updated postman and swagger file)
- Updated deployment guide and verification guide
- Working deployment on Azure
