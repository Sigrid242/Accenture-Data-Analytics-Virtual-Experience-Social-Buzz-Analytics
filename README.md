# Social-Buzz-Analytics

This is Accenture Data Analytics virtual experience project with Forage. The goal was to help a company named "Social Buzz" leverage the use of their massive amount of data. Social Buzz has reached huge scale in recent years to become recognized as a global unicorn company. At Accenture, we have embarked on a 3 month pilot to help tackle their biggest challenges on 3 main tasks:

1- Audit of Social Buzz big data practice

2- Implimentation of an IPO

3- Analysis of top 5 pupular categories content




Google Presentation: https://bit.ly/3Cke9la

Viz Link: https://tabsoft.co/3xVaojt

![social buzz dashboard](https://user-images.githubusercontent.com/59377247/192843759-79d1e9e7-6d10-4d9d-9329-d66bcb41e76a.PNG)


******CODE*****

/* Import CONTENT file */
options validvarname=v7;
PROC IMPORT DATAFILE="/home/sigrid0/Content1.xlsx"
		    OUT=WORK.CONTENT1
		    DBMS=XLSX
		    REPLACE;
run;

/*Check data type*/
proc contents data=content1;
run;

/*Create content dataset*/
data content_ (rename=(Type= Content_Type));
set content1;
Category_= compress(category,'"'); /*remove quotation marks in category*/
drop url category;
run;

/*Create new content dataset for processing*/
data content; 
set content_;
Category= lowcase(Category_);
if Category = 'public speakin' then category_ = 'public speaking';
drop category_;
run;

/*check missing values*/
proc freq data=content order=freq;
table content_id--category / missing;
run;



/* Import REACTIONS file */
options validvarname=v7;
PROC IMPORT DATAFILE="/home/sigrid0/Reactions1.xlsx"
		    OUT=WORK.REACTIONS1
		    DBMS=XLSX
		    REPLACE;
RUN;

/*check data type*/
proc contents data=reactions1;
run;

/*create reactions dataset*/
data reactions_;
set reactions1;
if user_id= '' then user_id= 'missing'; /*Flagging missing*/
if type= '' then type= 'missing';       /*values*/
run;

/*frequency*/
proc freq data= reactions_;
table content_id--type;
run;

/*Split datetime variable into date & time*/
data reactions; 
set reactions_; 
Date= datepart(datetime); 
format date date7.; 
time= timepart(datetime);
format time time8.;
drop datetime;
run;

/*Check for missing values*/
proc freq data=reactions;
tables _char_/ missing; /*character variable*/
run;

proc means data= reactions nmiss;
var _numeric_; /*numeric variables*/
run;



/* Import REACTION_Types file */
options validvarname=v7;
PROC IMPORT DATAFILE="/home/sigrid0/Reaction_Types1.xlsx"
		    OUT=WORK.REACTION_Types1
		    DBMS=XLSX
		    REPLACE;
RUN;

/*check data type*/
proc contents data=reaction_types1;
run;

/*Reaction_types dataset*/
data reaction_types;
set reaction_types1;
run;



/*Join Content, Reactions, Reaction_types tables*/  
proc sql;
create table social_B as
select a.content_id
	   ,a.user_id
	   ,a.type as reaction_type 
	   ,a.date
	   ,b.content_type
	   ,b.category
	   ,c.sentiment
	   ,c.score
from reactions as a
left join content as b  
on a.content_id= b.content_id
left join reaction_types as c 
on a.type=c.type;
quit;
