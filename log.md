# Cosumnes River Groundwater Observatory Research Log

***

February 21, 2018  

* performed load tests for the webapp on:  
    + googlesheets : failed  
    + github : passed

pros of using github: no need to set up remote DB  
cons of using github: the webapp is as real-time as the user who downloads, cleans, and uploads data from SQLite  


* successfully queried the SQlite database on Amy's computer using `RSQlite`.  
* the body of each email needs to cleaned for the appropriate data  
* determined that [AWS RDS](https://aws.amazon.com/rds/) is an option for remote storage  
* communicated with Mauricio and Solonist people to investigate remote DB options  

***

February 22, 2018  

* confirmed with Mauricio and Solonist that a remote DB is out of the question  
    + Solonist only offers SQLite, email and text notifications with the hardware we're using (level sender)  
* investiagted the potential of `gmailr` (access to gmail API) to query email data  
    + load test: failed  
        + 200 emails ~= 30 seconds  
        + 2000 emails ~= 300 seconds (5 minutes)  
* new plan:  
    + use either UCD servers or AAWS RDS to store clean data  
    + write an automated script that queries SQLite database daily, cleans data, and pushes it to the remote DB  
        + pros: very fast for webapp to access data, because the query is smaller, and we remove the cleaning step  
        + cons: technical to set up, but I love a new challenge  
* sent email to Chris to investigate if UCD can give us remote server space, and if not, if he recommends AWS RDS  
    + he recommends campus resources. mySQL is free  
    + the R interface to mySQL is well documented  
        + [slides](https://www.slideshare.net/RsquaredIn/rmysql-tutorial-for-beginners)  
        + [CRAN](https://cran.r-project.org/web/packages/RMySQL/RMySQL.pdf)  
* found these resources in setting up an automated task in Windows  
    + [Stack Overflow question](https://stackoverflow.com/questions/2793389/scheduling-r-script)  
    + [blog post](https://www.techradar.com/news/software/applications/how-to-automate-tasks-in-windows-1107254)  
* Talked with Omen (IT) and named project mySQL database "**gw_observatory**"  
    + cost is $0 to set up the database, and $39/hr to pay a developer to build the application.  
    + Rich will build the application, so the cost will be free.  
* Database details:  
    + type: mySQL  
    + host = sage.metro.ucdavis.edu  
    + user = gw_observatory  
    + schema = gw_observatory  
    + password = not displayed for security purposes  


TO DO:  

* Amy:  
    + change the reporting interval from 6 hour intervals to 24 hour intervals to save battery life  
    + read up on strings to prepare for cleaning SQLite data  
* Rich:  
    + set up the remote DB  
    + perform load tests on Shiny App -  aim to account for 10 years of hourly data.  

***

February 23, 2018  

* Asked System Admin to add shinyapps.io IP addresses to mySQL databse whitelist.  
    + [RStudio Support](https://support.rstudio.com/hc/en-us/articles/217592507-How-do-I-give-my-application-on-shinyapps-io-access-to-my-remote-database-#)  
    + [Pool documentation](https://cran.r-project.org/web/packages/pool/pool.pdf)  
    + [Databases in R](http://db.rstudio.com/pool/)  

    

***

February 26, 2018  

* was blocked from accessing mySQL server, so asked Omen to whitelist my IP and Amy's IP  
* wrote some dummy data to the mySQL server large enough for a load test (about 2 years of data for 13 wells)  
* created a minimal ShinyApp to query and load test the mySQL database  
    + query: works well  
    + load test: works well--size of data is not a problem  
* uploaded to shinyapps.io, and get the following error:  
    + `An error has occurred. Unable to connect to worker after 60.00 seconds; startup took too long.`  


***

February 27, 2018  

* Troubleshooting--it works locally, but not online because:  
    + data is too big, or memory needed > 1GB, and I need to upgrade to a Shiny Apps plan  
        + test with small data (USArrests) -- same error as before.  
        + not size or memory limited  
    + [try using ROCBC package](http://docs.rstudio.com/shinyapps.io/applications.html#config-package)  
        + not relevant  
    + check out the `config` package  
        + not relevant  
    + try `pool` package  
        + same error  
    + shinyapps.io IP addresses are not whitelited  
        + confirm they are whitelisted with Omen  
    + Stack Overflow  
        + posted  
    + shiny apps google group  
        + posted  
    
    
***

February 28, 2018  

* Successfully deployed app on shinyapps.io and resolved connection issue with campus MySQL server  
    + issue was that we needed to create a port to the database--metro has a burly firewall  

***

March 6, 2018  

* code to clean SQLite data and organize it into a table  
* the data is pretty messy at this point, because the sampling and reporting interval have changed so much during setup, and this changes the output of each reporting email
* In order to clean this data most effectively, set the reporting rate to 24 hours, the sample rate to 1 hour, and collect new data, so that the solution built doesn't have to account for emails of different size. 


***

March 7, 2018  

* features to build:  
    + shiny dashboard structure  
    + download data  
    + plotly hydrograph of daily averages  
    + map to click on  
* spent time looking at CCLite4 apps and reading code  
* brainstorming of app features  
* try to find workaround without rCharts  
* implemented well locations into cclite4 app  

***
    
March 8, 2018  

* performed load test on highcharts and plotly for group plot.  
    + Plotly much faster, easily scalable to daily measurements at 15 wells over 5 years in ~20 seconds  
    + plotly doesn't requre xts data types, as highchart does  
* built lots of features  
    + leaflet  
    + individual hydrograph for a selected well  
        + connection between selected well, drop down menu, and hydrograph  
    + network plot with geom_smooth average line  
    + logo and theme  
* still needs:  
    + about_the_site .md with a nice AI figure    
    + about floodplain recharge .md with figures  
    + about page:


***

March 9, 2018  

* implemented a download button  
* tested putting in a data table, and removed it  
* blocked by  
    + example of some 24 hour data to clean  
    + actual data stream coming in that I can push to UCD MySQL server  



***

March 14, 2018  

* still blocked by 24 hour data  
* learned that data won't come in regualrly, and I'll have to deal with missing data, as well as backing the data up  

**NEW PLAN**  

* write a script that:  
    + first queries the cloud db for missing values and updates them if new values are in the system  
    + checks the date on the computer, looks for the date closest to that in the sqlite database, cleans that data, and appends that data to the overall dataframe in the cloud server  
    + Missing data are stored as NA  
    + appends new data to the table  
    + [updates the table](https://www.w3schools.com/sql/sql_update.asp) with previously missing values  


***

March 15, 2018  

* moved sqlite files and levelsender from Amy's computer to mine  
* tail of data that came in was at:  

           ReceivedDate  
161 2018-03-14 17:12:16  
162 2018-03-14 14:37:42  
163 2018-03-14 13:45:14  
164 2018-03-14 13:01:03  
165 2018-03-15 13:01:03  
166 2018-03-15 14:38:56  

* check tomorrow that this new data is coming in  
* start with data that comes at 24 hour intervals: begins 3/15? Ask Amy.  

***

March 21-22, 2018  

* removed MW 14 (no data)  
* rewrote "About" page, as well well as "site info" and "floodplain recharge" buttons with filler images/text  
* filtered missing values from 2014-2017 data and published to on richpauloo.shinyapps.io/gw_observatory as an example    + aggregated data with `tibbletime` into daily means. This noticably boots load rates in plotly  
* communicated with Amy, Nathan and Mauricio about the 24hour reporting rate -- still blocked =(   


*** 

March 23, 2018  

* emailed Maurico at Solinst about changing reporting rate to 24 hours - still blocked.  



*** 

March 24- April 3, 2018  

To summarize:  

* LS # 284221 needs a hard restart which must be done in-field  
* Nathan sent reconfiguration emails (to change report interval to 24 hours) to ALL other LS units ( 283687, 284195, 284215, 284216, 284217  )  
* report interval was successfully changed to 24 hours for ONE LS unit ( 283687 )  

I did:  

* send config email to remaining level senders ( 284195, 284215, 284216, 284217 )  

Next steps:  

* reconfigure LS # 284221 in-field and set reporting interval to 24 hours  

Email must be sent as plain text with scheduled report time change to AFTER the next reporting interval (restart time must be a time after the next report)  

To: idLS@gmail.com  
subject: <leave blank>  

Stop report  
Location:  
Sample Rate: 60 minutes  
Report Rate: 24 hours  
Mail Rate: 24 hours  
Start Report: 05/04/2018 13:00:00  


***

April 10, 2018  

* charged $9/month to personal CC to keep shinyapp on shinyapps.io  
* long term solution: migrate shiny app to AWS. Began that process.  


***

Ocotber 8, 2018  

* after a big hiatus, work is underway once more!  
* $9/month charge has been removed  
* reorganized github folder  
* worked on extracting data from email bodies  in sqlite file  

* Andrew working with Mauricio to get sqlite file on my computer to automatically refresh  
* Andrew working to get me clean data through August  
* I will write rules for extracting data through October  


***  

October 9, 2018  

* Met with Graham, Laura, Andrew to get on the same page  
* talked about QAQC:  
  + discussed using Gmail R client to send email notifications during cleaning if something looks funny in the data (i.e. -  a hydrograph signal with vartiance above some threshold, say 2SD of the distribution of hydrograph values)  
* include the following information in well popups:  
  + screened interval depths  
  + well depth  
  + whether the well is pumped or not  
  + some geenral notes per well  


* `scratch/popup_CSS.R` shows how to override default leaflet CSS and use mapview CSS in `popupTable`s.  
* added two .exe files in `clean/rsqlite.Rmd` that refresh the emails and append them to the sqlite database. These need to be run at the top of the cleaning script.  
* waiting on Chris from IT to set up another home station email so I can refresh the emails coming in from the levelsenders on THIS computer.  


***  

Ocotber 15-16 2018  

* Worked with Andrew to set up a new email for my (Rich's) computer.  
* Andrew transfered a copy of the clean historical data to my computer, and I cleaned this a bit more and pushed it to the sloud SQL database under the table name `clean_historical_data_through_ocotber`.  
* can retreive emails via R now.  
* 4 wells reporting. Proceeding with these while Andrew debugs the others.  
* working on logic to query incoming emails and clean for appending to (now) clean database.  

* reformuated code into two files:  
    + `00_integrate_andrews_clean_historical_data.Rmd` and  
    + `01_query_sqlite_clean_and_push_to_cloud_SQL.Rmd`  
    
* these will eventually be turned into `.R` files which are sourced every 24 hours.  

* in `01_query_sqlite_clean_and_push_to_cloud_SQL.Rmd`, I:  
    + define the emails to check as a variable `current`, which includes all emails within 30 days of `Sys.Date()`  
    + wrote a beautiful function, `get_data()` that takes a vector of lines and returns a clean, formatted `data.frame` with: `datetime`, `temp`, `level`, and `id`.  
    + this function handles a level sender with two level loggers: one in the water, and one recording barometric pressure  
    

*** 

October 17, 2018  

* integrate elevation data into cloud database  
* finalize function to clean emails  
* automate separation of baro and monitoring well timeseries    
* convert barometric pressure from PSI to meters  
* adjust monitoring well data by barometric pressure  

***  

Ocotber 22-23, 2018

* completed cleaning script!  
* methods for adjusting final water levels by the elevation and cable length will be documented by Andrew, but they are implemented in the code in `01_query_sqlite_clean_and_push_to_cloud_SQL`.  
* Batteries for Level Senders fail around 60%. This is noted for the email triggers to be set later in the batch file. Some data from 10/18 onwards in MW5 needs to be updated...
    + the appending should be flexible enough to not break if the batteries for a well go out, we fix the records in the cloud db, and then new emails come in with new data. Think about solving this.  


***  

October 31, 2018  

* blocked by 3-day UC Water conference last week and hardware failing (low battery in MW5)  
* develop protocol to deal with battery outages, and a high-level CSV `battery_outages.csv` that and the resulting mess they require:  

## Outage Hot Fixes. This will unfortunately be a messy part of the data cleaning pipeline.    

Battery outages in the LevelSender throw impossible values at the temp and level, e.g. - level = 0.00. Never let battery outages in the LS happen again because this hot fix actually takes some time. It requires downloading the Level Logger data, and manually inserting it into the database. When this does unfortunately happen you should:  

### IN THE FIELD:  

1. Immediately change the batteries in the dead LS. Reported values will be noticably low and impossible (e.g. - level = -0.1 or 0.0).  

***  

### IN THE GITHUB REPO (only possible after the field is done):  

2. Identify the outage window and omit these data and record in `clean/dependencies/ls_battery_outages.csv`  
3. Separately, extract the unobstructed data from the levelloggers during the outage window, and upload the raw (unadjusted) levels into `clean/dependencies/ls_recovered_data.csv`  
4. Merge the recovered data.  
5. Repeat for every new battery outage.  


***  

November 1-2, 2018  

* moved baro adjusted water level and cable length calculation to the `elev.csv`  
* code to extract battery life  
* convert all .csv to .txt for unambiguous date formats  


***  

November 5, 2018  

* `02_daily_report.Rmd`: automated daily report that prints battery life table, map of battery life table, and last 30 days of hydrograph into a PDF and emails it.  


***  

November 6, 2018  

* finalized daily report script  
* created new gmail account cosumnes_gw_observatory@gmail.com, and a Gmail API client to send emails  



***  

November 7, 2018  

* converted `01_query_sqlite...Rmd` into a .R file  
* wrote `03_run.R` that sources the newly created R file  
* fixed some latex bugs in the automated report by following error messages. Basically need to use a few latex packages.  
   - `\usepackage[table]{xcolor}`  
   - `\usepackage{tabu}`  
   - `\usepackage{makecell,interfaces-makecell}`  


***  

November 8, 2018  

* added coordinates for wells from Andrew's Slack message  
* wrote list of emails and code to send report to that list  
* wrote Windows task scheduler to run `03_run.R` every 24 hours  
    + had to tell R where to find pandoc so the report could be generated by `RScript.exe`  
    + had to reauthenticate through Google  
    + had to add user "AD3\rpauloo" in `gpedit.msc`: [Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment] [link](https://social.technet.microsoft.com/Forums/lync/en-US/68019b24-78a5-4db0-a150-ada921930924/task-scheduler-failed-to-start-additional-data-error-value-2147943785?forum=winservergen)
* integrate datastream with online app  





***  


November 13, 2018  

* Completed app.  
* Running at `richpauloop.shinyapps.io/gw_observatory`  


***  


Additional Feautures Wishlist (in no order)  

* move site to shiny server with vanity domain name  
* battery changes in level senders also require a hotfix. This can probably be scripted in and controlled by an external CSV, something like `clean/dependencies/ll_battery_outages.csv`  










* Put somewhere on dashboard: Hydrograph values may reflect anomolies or errors. If you believe you see an error, please contact the app author for calification.  


    
    
    
    
    
    
    
    
    
    
    
    