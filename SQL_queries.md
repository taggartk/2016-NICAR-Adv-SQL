# Below are the SQL commands used for Advanced SQL for analysis at NICAR16
* These commands were written in SQLite

## Get to know your data 

How many records are there? 

    SELECT count(*) 
    FROM inspection

Check for any duplicates — we know from the data dictionary 
  that 'activity_nr' should be the unique identifier for an 
  inspection. Let's check to see if the same 'activity_nr' 
  shows up more than once.

    SELECT activity_nr, count(*)
    FROM inspection
    GROUP BY 1
    HAVING count(*) > 1

Alternatively, we could count the number of distinct 'activity_nr' records.s

    SELECT count(DISTINCT activity_nr)
    FROM inspection
 
What's the date range? 

    SELECT open_date
    FROM inspection
    ORDER BY 1
    LIMIT 10

And the other end of the spectrum?

    SELECT open_date
    FROM inspection  
    ORDER BY 1 DESC
    LIMIT 10

Check out another field — are there are bunch of crazy entries? Can be a good indication of how clean or dirty your data is. 

    SELECT site_state, count(*)
    FROM inspection
    GROUP BY 1


Overall looks pretty good — some blanks, some entries for the UK.
  We could look into the blanks to see if there's an obvious reason for them. 

    SELECT *
    FROM inspection   
    WHERE site_state IS NULL


## Clean it up and prep for analysis
* I usually import everything as text, especially when I'm just getting to know the data.
    But what if we want to take a closer look at inspections by year, for example.
    (SQLite can be a little funky for dealing with dates. Here's the [documention](https://www.sqlite.org/lang_datefunc.html),
    if you want to run down that rabbit hole. And a little funky for dealing with data types generally. Here's more  [reading](http://www.sqlitetutorial.net/sqlite-data-types/) on
    that rabbit hole).


Let's make a column with just the year the inspection was opened. 

    ALTER TABLE inspection       
    ADD COLUMN open_year TEXT

And then:

    SELECT open_date, strftime('%Y', open_date) as open_year    
    FROM inspection

And then:

    UPDATE inspection    
    SET open_year = strftime('%Y', open_date)

How's it look?

    `SELECT open_date, open_year`        
    `FROM inspection`       

How many inspections were there per year?

    SELECT open_year, count(*)
    FROM inspection     
    GROUP BY 1 
    ORDER BY 2 DESC     

Check your work

    SELECT substr(open_date,1,4), count(*)       
    FROM inspection  
    GROUP BY 1      
    ORDER BY 2 DESC

Want to know which month inspections are opened?

    SELECT strftime('%m', open_date) as open_month, count(*)  
    FROM inspection
    GROUP BY 1

Now, how long was the longest case open? 

    SELECT activity_nr, open_date, close_case_date, (julianday(close_case_date) - julianday(open_date)) AS datediff
    FROM inspection
    ORDER BY datediff DESC

Translate to years (approximately)

    SELECT activity_nr, open_date, close_case_date,  
    	(julianday(close_case_date) - julianday(open_date))/365 AS datediff 
    FROM inspection
    ORDER BY datediff DESC

Let's drill down the United States Postal Service, to flex our filtering muscles.
    Take a look at some of the different ways it's spelled.

    SELECT estab_name, count(*)
    FROM inspection
    GROUP BY 1
    ORDER BY 2 DESC
    
This should catch most of them.

    SELECT estab_name, count(*)
    FROM inspection
    WHERE
        estab_name LIKE '%u%s%POSTAL SERVICE%' OR
        estab_name LIKE '%USPS%'
    GROUP BY 1
    ORDER BY 1       

What if we want to get a count of the number of inspections for each establishment?

    SELECT estab_name, count(DISTINCT activity_nr)
    FROM inspection
    GROUP BY 1

Let's take a look at how to use subqueries. What if I want to know how many of the total inspections were initiated because of a complaint or an accident? Take a look at the insp_type field. This query will get us part of the way.

    SELECT estab_name, COUNT(insp_type) AS acc_or_complaint_insp
    FROM inspection
    WHERE insp_type = 'A' OR insp_type = 'B'
    GROUP BY 1

Then let's combine it with another query to be able to compare it to the total inspections:

    SELECT a.estab_name, count(DISTINCT activity_nr) AS totalinspections, b.acc_or_complaint_insp
    FROM inspection AS a
    JOIN
        (SELECT estab_name, COUNT(insp_type) AS acc_or_complaint_insp
        FROM inspection
        WHERE insp_type = 'A' OR insp_type = 'B'
        GROUP BY 1) as b
    ON a.estab_name = b.estab_name
    GROUP BY 1

How often was advanced notice given?

    SELECT adv_notice, count(*)
    FROM inspection
    GROUP BY 1

It's fair to say that advanced notice doesn't seem to be the norm, but what if it is for one company?
Let's do another subquery to compare this to the total number of inspections.

    SELECT a.estab_name, count(DISTINCT activity_nr) AS totalinspections, b.advanced_notice_insp
    FROM inspection AS a
    JOIN
        (SELECT estab_name, COUNT(insp_type) AS advanced_notice_insp
        FROM inspection
        WHERE adv_notice = 'Y'
        GROUP BY 1) as b
    ON a.estab_name = b.estab_name
    GROUP BY 1

We can make that into a separate table to query: 

    CREATE TABLE advanced_notice_inspections AS
    SELECT a.estab_name AS establishment_name, count(DISTINCT activity_nr) AS totalinspections, 
        b.advanced_notice_insp AS advanced_notice_count
    FROM inspection AS a
    JOIN
        (SELECT estab_name, COUNT(insp_type) AS advanced_notice_insp
        FROM inspection
        WHERE adv_notice = 'Y'
        GROUP BY 1) as b
    ON a.estab_name = b.estab_name
    GROUP BY 1

And then take a look at the results (what percent of inspections were announced in advanced? I filtered for places that had
	had more than 5 inspections)

    SELECT establishment_name, (advanced_notice_count*1.0)/(totalinspections*1.0) as pct_adv_notice
    FROM advanced_notice_inspections
    WHERE totalinspections > 5
    GROUP BY 1

    
Now we'll take a look at the inspections that are connected with accidents. 

The three tables we're messing with are connected through these fields:  
    Inspection.activity_nr --> accident_injury.rel_insp_nr, accident_injury.summary_nr --> accident.summary_nr

How many inspections included an accident that involved an injury?

    SELECT count(*)
    FROM inspection
    INNER JOIN accident_injury 
    ON inspection.activity_nr =  accident_injury.rel_insp_nr
    
Let's check to make sure that every record in the accident_injury table match a record in the inspection table.

    SELECT *
    FROM accident_injury
    LEFT JOIN inspection
    ON accident_injury.rel_insp_nr = inspection.activity_nr
    WHERE inspection.activity_nr IS NULL
    
Let's make a table with the result so we can query it later

    CREATE TABLE inspection_plus_injury AS     
    SELECT *    
    FROM inspection
    INNER JOIN accident_injury
    ON inspection.activity_nr =  accident_injury.rel_insp_nr

Now let's join this with data from the accident table, which includes fields like:
	1) 'event_desc': Short description of event and 2) 'event_date'

    CREATE TABLE osha_analysis_master AS
    SELECT * 
    FROM inspection_plus_injury
    INNER JOIN accident     
    ON inspection_plus_injury.summary_nr = accident.summary_nr

And now let's take a peak.

    SELECT estab_name, event_desc
    FROM osha_analysis_master
    WHERE event_desc LIKE '%horse%'

Or...

    SELECT estab_name, event_keyword
    FROM osha_analysis_master
    WHERE event_keyword LIKE '%burn%'
