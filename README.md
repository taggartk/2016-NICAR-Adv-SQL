# 2016-NICAR-Adv-SQL

The queries and data for an Advanced SQL class at the [NICAR 2016 conference](https://ire.org/conferences/nicar2016/).

This is a slightly modified version of the Advanced SQL class Liz Lucas [taught](https://github.com/eklucas/NICAR-Adv-SQL) at NICAR 2015. We're going to be using SQLite instead of MySQL. We're also going to use a free Firefox plugin that makes it easier to use SQLite on your computer. It should already be on the classroom computers, but you can also download it [here](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/).

All the SQL for the class is in this file: [SQL_queries.md](SQL_queries.md)

We're going to use inspection data from the	Department of Labor/Occupational Safety and Health Administration.

Here's the description from [DOL's website](http://ogesdw.dol.gov/views/data_summary.php): "The dataset consists of inspection case detail for approximately 100,000 OSHA inspections conducted annually. The dataset includes information regarding the impetus for conducting the inspection, and details on citations and penalty assessments resulting from violations of OSHA standards. Additionally, accident investigation information is provided, including textual descriptions of the accident, and details regarding the injuries and fatalities which occurred."

inspection, accident, accident_injury were downloaded from the [DOL's site](http://ogesdw.dol.gov/views/data_summary.php) on 02/11/2016.

For this class, I limited the data to Jan 1, 2011 through December 31, 2015.

Record layouts:
* [Data dict from OSHA](http://enforcedata.dol.gov/views/data_dictionary.php)
* [Layouts for the tables we'll be using](record_layouts.csv)


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
* Subqueries
