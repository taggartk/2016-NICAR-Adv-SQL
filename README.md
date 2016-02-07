# 2016-NICAR-Adv-SQL

*** This is still a work in progress. Feel free to contribute! [Conference](http://ire.org/conferences/nicar2016/) begins March 10, 2016 ***

The queries and data for an Advanced SQL class at the [NICAR 2016 conference](ire.org/conferences/nicar2016/).

This is a slightly modified version of the Advanced SQL class Liz Lucas [taught](https://github.com/eklucas/NICAR-Adv-SQL) at NICAR 2015. We're going to be using SQLite instead of MySQL. We're also going to use a free Firefox plugin that makes it easier to use SQLite on your computer. It should already be on the classroom computers, but you can also download it [here](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/).

All the SQL for the class is in this file: [SQL_queries.md](SQL_queries.md)

inspection, accident, accident_injury were downloaded from the [DOL's OSHA download site](http://ogesdw.dol.gov/views/data_catalogs.php) on TKTK MM/DD/YYYY.

Record layouts [Links TKTK]
* Data dict from OSHA
* inspection
* accident
* accident_injury


What we'll cover:

* Truthing data tables and joins
* DISTINCT
* CREATE TABLE
* ALTER TABLE to add columns
* UPDATE to populate new columns
* Dealing with dates in SQLite (strftime)
* How to calculate difference in days between two dates
* Wildcards for more complex filtering
* Aliases in table names
* Outer joins
* Subqueries
