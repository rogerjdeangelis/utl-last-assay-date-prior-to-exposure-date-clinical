# utl-last-assay-date-prior-to-exposure-date
Last assay date prior to exposure date.
    Last assay date prior to exposure date                                                                                   
                                                                                                                             
    I included time                                                                                                          
                                                                                                                             
    SAS Forum                                                                                                                
    https://tinyurl.com/ycth2bm4                                                                                             
    https://communities.sas.com/t5/SAS-Programming/Flag-Last-Assay-Date-Prior-to-Exposure-Date/m-p/530358                    
                                                                                                                             
    Novinosrin Profile                                                                                                       
    https://communities.sas.com/t5/user/viewprofilepage/user-id/138205                                                       
                                                                                                                             
    INPUT                                                                                                                    
    =====                                                                                                                    
                                                                                                                             
    data have ;                                                                                                              
       input                                                                                                                 
             @1 seq 1.                                                                                                       
             @3 subject 3.                                                                                                   
             @7 assay_name $24.                                                                                              
            @32 assay_date  ymddttm16.                                                                                       
            @49 exposure_date ymddttm16.                                                                                     
       ;                                                                                                                     
    format                                                                                                                   
          assay_date exposure_date datetime18.;                                                                              
    cards4;                                                                                                                  
    1 001 Alanine Aminotransferase 2017-07-05T12:01 2017-07-19T09:55                                                         
    2 001 Alanine Aminotransferase 2017-07-19T09:22 2017-07-19T09:55                                                         
    3 001 Albumin                  2017-08-15T09:38 2017-07-19T09:55                                                         
    4 001 Albumin                  2017-08-30T09:09 2017-07-19T09:55                                                         
    5 001 Albumin                  2017-07-15T08:10 2017-07-19T09:55                                                         
    ;;;;                                                                                                                     
    run;quit;                                                                                                                
                                                                                                                             
                                                              |                                                              
     SUBJECT  ASSAY_NAME  ASSAY_DATE      EXPOSURE_DATE   | DELTA_SECS    WANT                                               
                                                          |                                                                  
        1     Alanine  05JUL17:12:01:00  19JUL17:09:55:00 |    1202040     N                                                 
        1     Alanine  19JUL17:09:22:00  19JUL17:09:55:00 |       1980     Y  asy closest before expos                       
                                                                                                                             
        1     Albumin  15AUG17:09:38:00  19JUL17:09:55:00 | 1.7977E308     N                                                 
        1     Albumin  30AUG17:09:09:00  19JUL17:09:55:00 | 1.7977E308     N                                                 
        1     Albumin  15JUL17:08:10:00  19JUL17:09:55:00 |     351900     Y  asy closest before expos                       
                                                                                                                             
                                                                                                                             
    EXAMPLE OUTPUT                                                                                                           
    --------------                                                                                                           
                                                                                                                             
     WORK.WANT total obs=5                                                                                                   
                                                                                                                             
                                                                   EXPOSURE_                                                 
      SEQ    SUBJECT    ASSAY_NAME                  ASSAY_DATE       DATE       DELTA_SECS    WANT                           
                                                                                                                             
       1        1       Alanine Aminotransferase    1814875260    1816077300       1202040     N                             
       2        1       Alanine Aminotransferase    1816075320    1816077300          1980     Y                             
       3        1       Albumin                     1818409080    1816077300    1.7977E308     N                             
       4        1       Albumin                     1819703340    1816077300    1.7977E308     N                             
       5        1       Albumin                     1815725400    1816077300        351900     Y                             
                                                                                                                             
    PROCESS                                                                                                                  
    =======                                                                                                                  
                                                                                                                             
    proc sql;                                                                                                                
      create                                                                                                                 
         table want as                                                                                                       
      select                                                                                                                 
         *                                                                                                                   
        ,case                                                                                                                
           when (exposure_date-assay_date<0) then constant('bigint')                                                         
           else exposure_date-assay_date                                                                                     
         end as delta_secs                                                                                                   
        ,case                                                                                                                
           when (min(calculated delta_secs)=calculated delta_secs) then 'Y'                                                  
           else 'N'                                                                                                          
         end as want                                                                                                         
      from                                                                                                                   
         have                                                                                                                
      group                                                                                                                  
         by  subject                                                                                                         
            ,assay_name                                                                                                      
      order                                                                                                                  
         by seq                                                                                                              
    ;quit;                                                                                                                   
                                                                                                                             
