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
