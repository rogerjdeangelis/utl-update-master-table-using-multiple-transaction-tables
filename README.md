# utl-update-master-table-using-multiple-transaction-tables
Update master table using multiple single tables
    Update master table using multiple tranaction tables

    github
    https://tinyurl.com/yafgdv26
    https://github.com/rogerjdeangelis/utl-update-master-table-using-multiple-transaction-tables

    SAS Forum
    https://tinyurl.com/y7qnr84u
    https://communities.sas.com/t5/SAS-Programming/Update-base-table-using m/m-p/641804


    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data details;
    infile datalines truncover;
    input Run_ID :5. Company_code :$3. Datasource :$10. Table_name :$10. table_count : 2.;
    cards4;
    12345 ABC Database
    12345 ABD Database
    12345 ABE Database
    12345 ABF Database
    12346 DEF Excel
    ;;;;
    run;quit;

    data insurance insurancf insurancg insuranch  all;
       input Company_code :$5. Table_name :$10. Table_count :2. Run_ID :5.;
       select (_n_);
         when (1) output insurance  ;
         when (2) output insurancf ;
         when (3) output insurancg ;
         when (4) output insuranch ;
       end; /* leave off otherwise to force error in not exclusive */
     output all;
    cards4;
    ABC Insurance 87 12345
    ABD Insurancf 88 12345
    ABE Insurancg 89 12345
    DEF Insuranch 90 12346
    ;;;;
    run;quit;

    WORK.DETAILS total obs=5

                COMPANY_                  TABLE_    TABLE_
      RUN_ID      CODE      DATASOURCE     NAME      COUNT

       12345      ABC        Database                  .
       12345      ABD        Database                  .
       12345      ABE        Database                  .
       12345      ABF        Database                  .
       12346      DEF        Excel                     .


    WORK.INSURANCE total obs=1

      COMPANY_     TABLE_      TABLE_
        CODE        NAME        COUNT    RUN_ID

        ABC       Insurance      87       12345

    ...

    WORK,INSURANCH total obs=1

      COMPANY_     TABLE_      TABLE_
        CODE        NAME        COUNT    RUN_ID

        DEF       Insuranch      90       12346

     *           _
     _ __ _   _| | ___  ___
    | '__| | | | |/ _ \/ __|
    | |  | |_| | |  __/\__ \
    |_|   \__,_|_|\___||___/

    ;

    WORK.DETAILS total obs=5     FILL IN THESE VALUES     |  Values to fill   |
                                 USING TRANSACTION TABLES |                   |     INSURANCE TRANSACTION TABLE
                                                          |                   |
              COMPANY_             TABLE_    TABLE_       |  TABLE_    TABLE_ | COMPANY_   TABLE_    TABLE_
      RUN_ID    CODE    DATASOURCE  NAME      COUNT       |   NAME      COUNT |   CODE      NAME      COUNT   RUN_ID
                                                          |                   |
       12345    ABC      Database               .         |  Insurance   87   |   ABC     Insurance    87      12345

       12345    ABD      Database               .         |               .   |
       12345    ABE      Database               .         |               .   |
       12345    ABF      Database               .         |               .   |
       12346    DEF      Excel                  .         |               .   |
                                                          |                   |
       _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    WORK.DETAILS total obs=5

                     COMPANY_                   TABLE_      TABLE_
    Obs    RUN_ID      CODE      DATASOURCE      NAME        COUNT

     1      12345      ABC        Database     Insurance      87
     2      12345      ABD        Database     Insurancf      88
     3      12345      ABE        Database     Insurancg      89
     4      12345      ABF        Database                     .
     5      12346      DEF        Excel        Insuranch      90

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %array(tbls,values=insurance insurancf insurancg insuranch);

    proc sql;
       %do_over(tbls,phrase=%str(
          update details a
            set
              table_name   = (select table_name  from ? b where a.run_id=b.run_id and a.company_code=b.company_code),
              table_count  = (select table_count from ? b where a.run_id=b.run_id and a.company_code=b.company_code)
              where cats(a.run_id,a.company_code) in (select cats(b.run_id,b.company_code) from ? as b)
          ;))
    ;quit;

    *              _
      ___ ___   __| | ___  __ _  ___ _ __
     / __/ _ \ / _` |/ _ \/ _` |/ _ \ '_ \
    | (_| (_) | (_| |  __/ (_| |  __/ | | |
     \___\___/ \__,_|\___|\__, |\___|_| |_|
                          |___/
    ;


    data _null_;

        %do_over(tbls,phrase=%str(put
          "update details a" /
              "set" /
              "table_name   = (select table_name  from ? b where a.run_id=b.run_id and a.company_code=b.company_code)," /
              "table_count  = (select table_count from ? b where a.run_id=b.run_id and a.company_code=b.company_code)" /
              "where cats(a.run_id,a.company_code) in (select cats(b.run_id,b.company_code) from ? as b)"
          ;));
    run;quit;


    update details a
    set
    table_name   = (select table_name  from insurance b where a.run_id=b.run_id and a.company_code=b.company_code),
    table_count  = (select table_count from insurance b where a.run_id=b.run_id and a.company_code=b.company_code)
    where cats(a.run_id,a.company_code) in (select cats(b.run_id,b.company_code) from insurance as b)
    update details a
    set
    table_name   = (select table_name  from insurancf b where a.run_id=b.run_id and a.company_code=b.company_code),
    table_count  = (select table_count from insurancf b where a.run_id=b.run_id and a.company_code=b.company_code)
    where cats(a.run_id,a.company_code) in (select cats(b.run_id,b.company_code) from insurancf as b)
    update details a
    set
    table_name   = (select table_name  from insurancg b where a.run_id=b.run_id and a.company_code=b.company_code),
    table_count  = (select table_count from insurancg b where a.run_id=b.run_id and a.company_code=b.company_code)
    where cats(a.run_id,a.company_code) in (select cats(b.run_id,b.company_code) from insurancg as b)
    update details a
    set
    table_name   = (select table_name  from insuranch b where a.run_id=b.run_id and a.company_code=b.company_code),
    table_count  = (select table_count from insuranch b where a.run_id=b.run_id and a.company_code=b.company_code)
    where cats(a.run_id,a.company_code) in (select cats(b.run_id,b.company_code) from insuranch as b)






