# SAS_Code_Writing_Samples
This repository contains some SAS code writing samples that I constructed and used throughout my career.


### To import Excel files and rename variables
proc import datafile="excel_file_path"
  dbms=xlsx
  out=work.file(rename=(B=clinical_episode_category
                        C=year
                        D=HCPCS_CD))
  replace;
  sheet="name of the tab";
  getnames=no;
  range="B8:F276";
run;

### To read in a list of values of a variable and use it as a macro var using SQL procedure within ce-level looping;
proc sql no print;
select _NAME_,
       substr(_NAME_,4)
into: ce&ce._parameters separated by " ",
    : ce&ce._x separated by " "
from reference.ce&ce._type
where substr(_NAME_,1,3)="B1_" and substr(_NAME_,1,4)="B1_0"
;
quit;

### Calculations within SQL procedures;
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

### hash merge steps;   
data work.report;
  if 0 then set work.file (keep=A B C);
  if _N_=1 then do; 
    declare hash claim (dataset: "work.file"); 
    claim.definekey ("A"); 
    claim.definedata ("B", "C"); 
    claim.definedone(); 
  end;
  if claim.find()=0 then output;
run;




