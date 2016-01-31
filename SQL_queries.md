# Below are the SQL commands used for Advanced SQL for analysis at NICAR16
* These commands were written in SQLite

## Get to know your data 

* How many records are there? 

    `SELECT count(*)`  
    `FROM inspection`

    There should be 473,955

* Check for any duplicates — we know from the data dictionary 
  that 'activity_nr' should be the unique identifier for an 
  inspection. Let's check to see if the same 'activity_nr' 
  shows up more than once.

    `SELECT activity_nr, count(*)`  
    `FROM inspection`  
    `GROUP BY 1`  
    `HAVING count(*) > 1`

* What's the date range? 

    `SELECT open_date`  
    `FROM inspection`  
    `ORDER BY 1`  
    `LIMIT 10`

    `SELECT open_date`  
    `FROM inspection`  
    `ORDER BY 1 DESC`  
    `LIMIT 10`

* Check out another field — are there are bunch of crazy entries? Can be a good indication of how clean or dirty your data is. 

    `SELECT site_state, count(*)`   
    `FROM inspection`   
    `GROUP BY 1`


* Overall looks pretty good — some blanks, some entries for the UK.
  We could look into the blanks to see if there's an obvious reason for them. 

    `SELECT *`  
    `FROM inspection`   
    `WHERE site_state IS NULL`


## Clean it up and prep for analysis
* I usually import everything as text, especially when I'm just getting to know the data.
    But what if we want to take a closer look at inspections by year, for example.
    (SQLite can be a little funky for dealing with dates. Here's the [documention](https://www.sqlite.org/lang_datefunc.html),
    if you want to run down that rabbit hole).


* Let's make a column with just the year the inspection was opened. 

    `ALTER TABLE inspection`        
    `ADD COLUMN year TEXT`      

    `SELECT open_date, strftime('%Y', open_date) as year `     
    `FROM inspection`
    
    `UPDATE inspection`     
    `SET year = strftime('%Y', open_date)`

* How's it look?

    `SELECT open_date, year`        
    `FROM inspection`       

* How many inspections were there per year?

    `SELECT year, count(*)`     
    `FROM inspection`       
    `GROUP BY 1`        
    `ORDER BY 2 DESC`       

* Check your work

    `SELECT substr(open_date,1,4), count(*)`        
    `FROM inspection`      
    `GROUP BY 1`        
    `ORDER BY 2 DESC`       
