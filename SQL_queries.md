# Below are the SQL commands used for Advanced SQL for analysis at NICAR16
* These commands were written in SQLite

## Get to know your data 

* How many records are there? 

    `SELECT count(*)`  
    `FROM inspection`


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
    `ADD COLUMN open_year TEXT`      

    `SELECT open_date, strftime('%Y', open_date) as open_year`     
    `FROM inspection`
    
    `UPDATE inspection`     
    `SET open_year = strftime('%Y', open_date)`

* How's it look?

    `SELECT open_date, open_year`        
    `FROM inspection`       

* How many inspections were there per year?

    `SELECT open_year, count(*)`     
    `FROM inspection`       
    `GROUP BY 1`        
    `ORDER BY 2 DESC`       

* Check your work

    `SELECT substr(open_date,1,4), count(*)`        
    `FROM inspection`      
    `GROUP BY 1`        
    `ORDER BY 2 DESC` 

* Want to know which month inspections are opened?

    `SELECT strftime('%m', open_date) as open_month, count(*)`  
    `FROM inspection`  
    `GROUP BY 1`  

* Now, how long was the longest case open? 

    `SELECT activity_nr, open_date, close_case_date, (julianday(close_case_date) - julianday(open_date)) AS datediff`  
    `FROM inspection`  
    `ORDER BY datediff DESC`

* Translate to years (approximately)

    `SELECT activity_nr, open_date, close_case_date,`   
    	`(julianday(close_case_date) - julianday(open_date))/365 AS datediff`  
    `FROM inspection`  
    `ORDER BY datediff DESC`

* Let's drill down the United States Postal Service, to flex our filtering muscles.
    Take a look at some of the different ways it's spelled.

    `SELECT estab_name, count(*)`  
    `FROM inspection`  
    `GROUP BY 1`  
    `ORDER BY 2 DESC`  
    
    This should catch most of them
    
    `SELECT estab_name, count(*)`       
    `FROM inspection`      
    `WHERE`     
        `estab_name LIKE '%u%s%POSTAL SERVICE%' OR`     
        `estab_name LIKE '%USPS%'`      
    `GROUP BY 1`        
    `ORDER BY 1`        

* Let's take a look at how to use subqueries. For every record in the inspection table, let's get a total record count for the establishment associated with that inspection. (This took about 50 seconds when I timed it).

    `SELECT b.estab_count, a.*`     
    `FROM inspection as a`      
    `JOIN`   
        `(SELECT estab_name, count(*) as estab_count`      
	    `FROM inspection`  
		`GROUP BY 1) as b`  
    `ON a.estab_name = b.estab_name`
    
* Now we'll take a look at the inspections that are connected with accidents. 
	* The three tables we're messing with are connected through these fields:  
	Inspection.activity_nr --> accident_injury.rel_insp_nr, accident_injury.summary_nr --> accident.summary_nr

* How many inspections included an accident that involved an injury?

    `SELECT count(*)`  
    `FROM inspection`  
    `INNER JOIN accident_injury`  
    `ON inspection.activity_nr =  accident_injury.rel_insp_nr`
    
* Let's check to make sure that every record in the accident_injury table match a record in the inspection table.

    `SELECT *`
    `FROM accident_injury`
    `LEFT JOIN inspection`
    `ON accident_injury.rel_insp_nr = inspection.activity_nr`
    `WHERE inspection.activity_nr IS NULL`
    
* Let's make a table with the result so we can query it later

    `CREATE TABLE inspection_plus_injury AS`        
    `SELECT *`        
    `FROM inspection`  
    `INNER JOIN accident_injury`  
    `ON inspection.activity_nr =  accident_injury.rel_insp_nr`  

* Now let's join this with data from the accident table, which includes fields like:
	1) 'event_desc': Short description of event and 2) 'event_date'

    `CREATE TABLE osha_analysis_master AS`
    `SELECT *`      
    `FROM inspection_plus_injury`       
    `INNER JOIN accident`       
    `ON inspection_plus_injury.summary_nr = accident.summary_nr`      




 

