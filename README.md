# Clever Little CTE
----
## Project Overview

Nothing overly fancy but a quick and clever way to determine a running YTD total of operational business days by business unit needed for calculations on back end use.
Our origin tables were simple breaksouts of Month, Unit and Days operational - 
----

## Data Acquisition

- Data is broken out by Month and Business unit, in this case, across five seperate entities (I,M,O,S,U in this instance)
- The requirements were to have a TY_DAYS, LY_DAYS comparitively as well as running totals across each BU and Year for aggregate calculations downstream.
- This is just an example of thinking of ways to aprroach an issue with simple yet effective code.

 ~~~~
With D1 as 
(	Select
	MO_Nam		= case			when [Time Period Num (YYYYMM)]%100=1 then 'JAN'
								when [Time Period Num (YYYYMM)]%100=2 then 'FEB'
								when [Time Period Num (YYYYMM)]%100=3 then 'MAR'
								when [Time Period Num (YYYYMM)]%100=4 then 'APR'
								when [Time Period Num (YYYYMM)]%100=5 then 'MAY'
								when [Time Period Num (YYYYMM)]%100=6 then 'JUN'
								when [Time Period Num (YYYYMM)]%100=7 then 'JUL'
								when [Time Period Num (YYYYMM)]%100=8 then 'AUG'
								when [Time Period Num (YYYYMM)]%100=9 then 'SEP'
								when [Time Period Num (YYYYMM)]%100=10 then 'OCT'
								when [Time Period Num (YYYYMM)]%100=11 then 'NOV'
								when [Time Period Num (YYYYMM)]%100=12 then 'DEC'
								end

			, YY		= [Time Period Num (YYYYMM)]/100
			, MM		= [Time Period Num (YYYYMM)]%100
			, YYMM		= [Time Period Num (YYYYMM)]
			, BU		= [BusinessUnit]
			, TY_DAYS	= sum(case when [Marketing Plan Cd]=4 then [Plan Period Workdays Quantity] else 0 end)
			, LY_DAYS	= sum(case when [Marketing Plan Cd]=2 then [Plan Period Workdays Quantity] else 0 end)
from		csinternalstaging.dbo.[USBusinessDaysbyBusinessUnit]
where		[Time Period Num (YYYYMM)]%100 between 1 and 12
group by	[Time Period Num (YYYYMM)]
			, [BusinessUnit]

), 

D2 as 

(select 
	MO_Nam		=case			when [Time Period Num (YYYYMM)]%100=1 then 'JAN'
								when [Time Period Num (YYYYMM)]%100=2 then 'FEB'
								when [Time Period Num (YYYYMM)]%100=3 then 'MAR'
								when [Time Period Num (YYYYMM)]%100=4 then 'APR'
								when [Time Period Num (YYYYMM)]%100=5 then 'MAY'
								when [Time Period Num (YYYYMM)]%100=6 then 'JUN'
								when [Time Period Num (YYYYMM)]%100=7 then 'JUL'
								when [Time Period Num (YYYYMM)]%100=8 then 'AUG'
								when [Time Period Num (YYYYMM)]%100=9 then 'SEP'
								when [Time Period Num (YYYYMM)]%100=10 then 'OCT'
								when [Time Period Num (YYYYMM)]%100=11 then 'NOV'
								when [Time Period Num (YYYYMM)]%100=12 then 'DEC'
								end
			, YY		= [Time Period Num (YYYYMM)]/100
			, MM		= [Time Period Num (YYYYMM)]%100
			, YYMM		= [Time Period Num (YYYYMM)]
			, BU		= [BusinessUnit]
			, TY_DAYS	= sum(case when [Marketing Plan Cd]=4 then [Plan Period Workdays Quantity] else 0 end)
			, LY_DAYS	= sum(case when [Marketing Plan Cd]=2 then [Plan Period Workdays Quantity] else 0 end)
from		csinternalstaging.dbo.[USBusinessDaysbyBusinessUnit]
where		[Time Period Num (YYYYMM)]%100 between 1 and 12
group by	[Time Period Num (YYYYMM)]
			, [BusinessUnit]

)

	Select 
			D1.MO_Nam, 
			D1.YY, 
			D1.MM, 
			D1.YYMM, 
			D1.BU, 
			D1.TY_DAYS, 
			D1.LY_DAYS, 
			
		(   	SELECT sum(D2.TY_DAYS) 
			    FROM D2 
				WHERE D1.MM >= D2.MM  and D1.BU=D2.BU and D1.YY=D2.YY
		)		as 'YTD_TY_DAYS',

		(   	SELECT sum(D2.LY_DAYS) 
			    FROM D2 
				WHERE D1.MM >= D2.MM  and D1.BU=D2.BU and D1.YY=D2.YY
		)		as 'YTD_LY_DAYS'

			FROM D1
			Where YY in('2015','2016','2017','2018')
~~~~


## Results Set

![Days](https://github.com/DonChart/Clever_Little_CTE/assets/168656623/7d3a4316-d57b-4a5d-9ae3-3924b519f33e)

