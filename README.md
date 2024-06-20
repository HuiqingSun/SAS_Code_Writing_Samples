# SAS_Code_Writing_Samples
This repository contains some SAS code writing samples that I constructed and used throughout my career.

### Following codes can use hash merge to calculate the total grades by student_id level
```
data _NULL_;
   if _N_=0 then do;
      length student_id total_grades 8.;
   end;

   if _N_=1 then do;
      declare hash grade_hash();
      grade_hash.definekey('student_id');
      grade_hash.definedata('student_id','total_grades');
      grade_hash.definedone();
   end;
   call missing (of _ALL_);

   set work.grades end=last;

   found=grade_hash.find()=0;
   total_grades=sum(grade,total_grades);

   if not found then grade_hash.add();
   grade_hash.replace();

   if last then grade_hash.output(dataset:'out.total_grades_by_student_id');
run;

```
### Use Proc SQL to create a series of macro variables that contain the course code, location, and starting date of all courses that are scheduled in 2020. 
```
proc sql noprint;
   select course_code, location, begin_date format=mmddyy10.
   into :course1- ,
        :place1- ,
        :date1-
from course.schedule
where year(begin_date)=2020
order by begin_date;
quit;

%put There are &sqlobs courses in 2020;
%put _user_;
```

### Use CALL SYMPUTX to create a macro variable to use

This example creates three macro variables. The macro variable Csrname records the
value of the DATA step variable Course_title. The macro variable date records the value
of the DATA step variable Begin_date in MMDDYY10. format. Finally, the macro
variable Due uses the values of the DATA step variables Paidup, Total, and Fee to record
the current amount of unpaid fees in DOLLAR8. format. These macro variables are
referenced later in the program in the TITLE and FOOTNOTE statements.

```
%let crsnum=3;
data work.test;

set course.all end=last;

   where course_number=&crsnum;
   total+1;
   if paid='Y' then paidup+1;
   if last then do;
      call symputx('crsname',course_title);
      call symputx('date',put(begin_date,mmddyy10.));
      call symputx('due',put(fee*(total-paidup),dollar8.));
   end;
run;

proc print data=wor.test;
   var student_name student_company paid;
   title "Fee Status for &crsname (#&crsnum) Held &date";
   footnote "Note: &due in Unpaid Fees";
run;

```

### To import Excel files and rename variables
```
proc import datafile="excel_file_path"
  dbms=xlsx
  out=work.file(rename=(B=category
                        C=year
                        D=HCPCS_CD))
  replace;
  sheet="name of the tab";
  getnames=no;
  range="B8:F276";
run;
```
### To read in a list of values of a variable and use it as a macro var using SQL procedure within c-level looping;
```
proc sql no print;
select _NAME_,
       substr(_NAME_,4)
into: ce&c._parameters separated by " ",
    : ce&c._x separated by " "
from reference.ce&c._type
where substr(_NAME_,1,3)="B1_" and substr(_NAME_,1,4)="B1_0"
;
quit;
```
### Calculations within SQL procedures;
```
proc sql;
  create table out.summary as
  select count(*) as total
    %do r=1 %to &nreasons.;
    %let reason=&&reasons&r..;
      , sum(drop_&reason.) as number_of_&reason.,
    %end;
    case flag when  'drop'  then 'Yes'
              when  'keep'  then 'No'
              else  'N/A'
    end as flags
from work.file;
quit;


proc sql;
    select Department,
        avg(Employee_annual_salary) as Avg_salary format=DOLLAR12.2
    from work.salary
    group by Department
    having Avg_salary < (select avg(Employee_annual_salary) from work.salary)
    order by Avg_salary;
quit;
```
### hash merge steps;   
```
data work.report;
  if 0 then set work.file (keep=A B C);
  if _N_=1 then do; 
    declare hash claim (dataset: "work.file(where=(flag=1))"); 
    claim.definekey ("A"); 
    claim.definedata ("B", "C"); 
    claim.definedone(); 
  end;
  if claim.find()=0 then output;
run;
```




