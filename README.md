# utl-creating-variables-that-flag-changes-over-time-for-a-medication-sql-sas-r-python-excel
Creating variables that flag changes over time for a medication sql sas r-python excel
    %let pgm=utl-creating-variables-that-flag-changes-over-time-for-a-medication-sql-sas-r-python-excel;

    %stop_submission;

    Creating variables that flag changes over time for a medication sql sas r-python excel

    EXCEL OUTPUT
    https://tinyurl.com/3cmsbznd
    https://github.com/rogerjdeangelis/utl-creating-variables-that-flag-changes-over-time-for-a-medication-sql-sas-r-python-excel/blob/main/wantxl.xlsx

      FOUR  SOLUTIONS

              1 sas proc sql
              2 r sql
              3 python sql
              4 excel sql

    The best way to document the process is this state table sql-sas-r-python

    Rather easy to create predose and next dose

    STATE DIAGRAMS CAN BE VERY USEFUL FOR DOCUMENTATION

                           MY OUTPUT      OPS OUTOUT. AGREES WITH MY SOLUTION.
                           =============  ===================================
                           ?MY BETTER     DRUG_A_  DRUG_A_   DRUG_A_   DRUG_A_
    ID MONTH PREDOSE DOSE  BETTER RESULT   REDUCE INCREASE DISCONTINUE RESTART

     1     1      NA  500  1-NOPREDATA       .        .         .         .
     1     2     500  500  2-NOCHANGE        .        .         .         .
     1     3     500   NA  3-DISCON          .        .         1         .
     1     4      NA   NA  5-OFFDRUG         .        .         .         .

     2     1      NA  500  1-NOPREDATA       .        .         .         .
     2     2     500  500  2-NOCHANGE        .        .         .         .
     2     3     500  500  2-NOCHANGE        .        .         .         .
     2     4     500  250  4-REDUCE          1        .         .         .
     2     5     250  250  2-NOCHANGE        .        .         .         .

     3     1      NA  500  1-NOPREDATA       .        .         .         .
     3     2     500  250  4-REDUCE          1        .         .         .
     3     3     250  250  2-NOCHANGE        .        .         .         .
     3     4     250  500  7-INCREASE        .        1         .         .
     3     5     500  100  4-REDUCE          1        .         .         .

     4     1      NA  500  1-NOPREDATA       .        .         .         .
     4     2     500   NA  3-DISCON          .        .         1         .
     4     3      NA   NA  5-OFFDRUG         .        .         .         .
     4     4      NA  500  6-RESTART         .        .         .         1


    OUTPUT EXCEL WORKBOOK


    SOAPBOX ON
       Given the data is a timeseries it seems counter-intuitive to transpose
        to a denormalize fat data structure.
       Also creating codes can aid in further analysis
    SOAPBOX OFF


    RELATED
    https://tinyurl.com/2xnmk2xt
    https://github.com/rogerjdeangelis/utl-create-a-state-diagram-table-hash-corresp-and-transpose

    sas communities
    https://tinyurl.com/4xhvhxyz
    https://communities.sas.com/t5/SAS-Programming/Creating-variables-that-flag-changes-over-time-for-a-medication/m-p/961161#M374721

    /**************************************************************************************************************************/
    /*                       |                                                         |                                      */
    /*     INPUT             |         PROCESS                                         |        OUTPUT                        */
    /*                       | DOSE SCRIPT CHANGE FROM PREVIOUS TO CURRENT DOSING      |                                      */
    /*                       |                                                         |                                      */
    /*                       |                                                         |                                      */
    /*                       | 1 SAS PROC SQL                                          |                                      */
    /*                       | =============                                           |                                      */
    /*                       |                                                         |                                      */
    /* ID MONTH DOSE         | proc datasets lib=sd1 nolist nodetails;                 |  ID MONTH PREDOSE DOSE   SCRIPTS     */
    /*                       |  delete want;                                           |                                      */
    /*  1   1    500         | run;quit;                                               |  1    1       .    500  1-NOPREDATA  */
    /*  1   2    500         |                                                         |  1    2     500    500  2-NOCHANGE   */
    /*  1   3      .         | proc sql;                                               |  1    3     500      .  3-DISCON     */
    /*  1   4      .         |   create                                                |  1    4       .      .  5-OFFDRUG    */
    /*  2   1    500         |     table sd1.want as                                   |                                      */
    /*  2   2    500         |   select                                                |  2    1       .    500  1-NOPREDATA  */
    /*  2   3    500         |    cur.id                                               |  2    2     500    500  2-NOCHANGE   */
    /*  2   4    250         |   ,cur.month                                            |  2    3     500    500  2-NOCHANGE   */
    /*  2   5    250         |   ,cur.dose as dose                                     |  2    4     500    250  7-REDUCE     */
    /*  3   1    500         |   ,pre.dose as predose                                  |  2    5     250    250  2-NOCHANGE   */
    /*  3   2    250         |   ,case                                                 |                                      */
    /*  3   3    250         |      when cur.month=1               then '1-NOPREDATA'  |  3    1       .    500  1-NOPREDATA  */
    /*  3   4    500         |      when pre.dose=. and cur.dose=. then '5-OFFDRUG'    |  3    2     500    250  7-REDUCE     */
    /*  3   5    100         |      when pre.dose>0 and cur.dose=. then '3-DISCON'     |  3    3     250    250  2-NOCHANGE   */
    /*  4   1    500         |      when pre.dose=. and cur.dose>0 then '6-RESTART'    |  3    4     250    500  4-INCREASE   */
    /*  4   2      .         |      when cur.dose=pre.dose and                         |  3    5     500    100  7-REDUCE     */
    /*  4   3      .         |        (cur.dose>0 and pre.dose>0)  then '2-NOCHANGE'   |                                      */
    /*  4   4    500         |      when cur.dose<pre.dose and                         |  4    1       .    500  1-NOPREDATA  */
    /*                       |        (cur.dose>0 and pre.dose>0)  then '7-REDUCE'     |  4    2     500      .  3-DISCON     */
    /*                       |      when cur.dose>pre.dose and                         |  4    3       .      .  5-OFFDRUG    */
    /*                       |        (cur.dose>0 and pre.dose>0)  then '4-INCREASE'   |  4    4       .    500  6-RESTART    */
    /* options               |      else ""                                            |                                      */
    /*  validvarname=upcase; |    end as scripts                                       |                                      */
    /* libname sd1 "d:/sd1"; |   from                                                  |                                      */
    /* data sd1.have;        |     sd1.have as cur left join sd1.have as pre           |                                      */
    /*  input id Month dose; |   on                                                    |                                      */
    /* datalines;            |          cur.id = pre.id                                |                                      */
    /* 1 1 500               |    and   cur.month = pre.month + 1                      |                                      */
    /* 1 2 500               | order                                                   |                                      */
    /* 1 3 .                 |    by cur.id, cur.month                                 |                                      */
    /* 1 4 .                 | ;quit;                                                  |                                      */
    /* 2 1 500               |                                                         |                                      */
    /* 2 2 500               |                                                         |                                      */
    /* 2 3 500               |  2 R SQL (SAME CODE SEE BELOW)                          |                                      */
    /* 2 4 250               |  ==============================                         |                                      */
    /* 2 5 250               |                                                         |                                      */
    /* 3 1 500               |  3 PYTHON SQL (SAME CODE SEE BELOW)                     |                                      */
    /* 3 2 250               |  ===================================                    |                                      */
    /* 3 3 250               |                                                         |                                      */
    /* 3 4 500               |  4 EXCEL & R SAME CODE                                  |                                      */
    /* 3 5 100               |  =====================                                  |                                      */
    /* 4 1 500               |                                                         |                                      */
    /* 4 2 .                 |                                                         |                                      */
    /* 4 3 .                 |                                                         |                                      */
    /* 4 4 500               |                                                         |                                      */
    /* run;                  |                                                         |                                      */
    /*                       |                                                         |                                      */
    /**************************************************************************************************************************/


    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options
     validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input id Month dose;
    datalines;
    1 1 500
    1 2 500
    1 3 .
    1 4 .
    2 1 500
    2 2 500
    2 3 500
    2 4 250
    2 5 250
    3 1 500
    3 2 250
    3 3 250
    3 4 500
    3 5 100
    4 1 500
    4 2 .
    4 3 .
    4 4 500
    run;

    /**************************************************************************************************************************/
    /*ID MONTH DOSE                                                                                                           */
    /*                                                                                                                        */
    /* 1   1    500                                                                                                           */
    /* 1   2    500                                                                                                           */
    /* 1   3      .                                                                                                           */
    /* 1   4      .                                                                                                           */
    /* 2   1    500                                                                                                           */
    /* 2   2    500                                                                                                           */
    /* 2   3    500                                                                                                           */
    /* 2   4    250                                                                                                           */
    /* 2   5    250                                                                                                           */
    /* 3   1    500                                                                                                           */
    /* 3   2    250                                                                                                           */
    /* 3   3    250                                                                                                           */
    /* 3   4    500                                                                                                           */
    /* 3   5    100                                                                                                           */
    /* 4   1    500                                                                                                           */
    /* 4   2      .                                                                                                           */
    /* 4   3      .                                                                                                           */
    /* 4   4    500                                                                                                           */
    /**************************************************************************************************************************/

    /*                             _
    / |  ___  __ _ ___   ___  __ _| |
    | | / __|/ _` / __| / __|/ _` | |
    | | \__ \ (_| \__ \ \__ \ (_| | |
    |_| |___/\__,_|___/ |___/\__, |_|
                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete want;
    run;quit;

    proc sql;
      create
        table sd1.want as
      select
       cur.id
      ,cur.month
      ,cur.dose as dose
      ,pre.dose as predose
      ,case
         when cur.month=1               then '1-NOPREDATA'
         when pre.dose=. and cur.dose=. then '5-OFFDRUG'
         when pre.dose>0 and cur.dose=. then '3-DISCON'
         when pre.dose=. and cur.dose>0 then '6-RESTART'
         when cur.dose=pre.dose and
           (cur.dose>0 and pre.dose>0)  then '2-NOCHANGE'
         when cur.dose<pre.dose and
           (cur.dose>0 and pre.dose>0)  then '7-REDUCE'
         when cur.dose>pre.dose and
           (cur.dose>0 and pre.dose>0)  then '4-INCREASE'
         else ""
       end as scripts
      from
        sd1.have as cur left join sd1.have as pre
      on
             cur.id = pre.id
       and   cur.month = pre.month + 1
    order
       by cur.id, cur.month
    ;quit;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /*   ID MONTH PREDOSE DOSE   SCRIPTS                                                                                      */
    /*                                                                                                                        */
    /*   1    1       .    500  1-NOPREDATA                                                                                   */
    /*   1    2     500    500  2-NOCHANGE                                                                                    */
    /*   1    3     500      .  3-DISCON                                                                                      */
    /*   1    4       .      .  5-OFFDRUG                                                                                     */
    /*                                                                                                                        */
    /*   2    1       .    500  1-NOPREDATA                                                                                   */
    /*   2    2     500    500  2-NOCHANGE                                                                                    */
    /*   2    3     500    500  2-NOCHANGE                                                                                    */
    /*   2    4     500    250  7-REDUCE                                                                                      */
    /*   2    5     250    250  2-NOCHANGE                                                                                    */
    /*                                                                                                                        */
    /*   3    1       .    500  1-NOPREDATA                                                                                   */
    /*   3    2     500    250  7-REDUCE                                                                                      */
    /*   3    3     250    250  2-NOCHANGE                                                                                    */
    /*   3    4     250    500  4-INCREASE                                                                                    */
    /*   3    5     500    100  7-REDUCE                                                                                      */
    /*                                                                                                                        */
    /*   4    1       .    500  1-NOPREDATA                                                                                   */
    /*   4    2     500      .  3-DISCON                                                                                      */
    /*   4    3       .      .  5-OFFDRUG                                                                                     */
    /*   4    4       .    500  6-RESTART                                                                                     */
    /**************************************************************************************************************************/

    /*___                     _
    |___ \   _ __   ___  __ _| |
      __) | | `__| / __|/ _` | |
     / __/  | |    \__ \ (_| | |
    |_____| |_|    |___/\__, |_|
                           |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete want;
    run;quit;

    %utl_rbeginx;
    parmcards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    have<-read_sas("d:/sd1/have.sas7bdat")
    print(have)
    str(have)
    want<-sqldf('
     select
       cur.id
      ,cur.month
      ,cur.dose as dose
      ,pre.dose as predose
      ,case
         when cur.month=1                           then "1-NOPREDATA"
         when pre.dose is null and cur.dose is null then "5-OFFDRUG"
         when pre.dose>0 and cur.dose is null       then "3-DISCON"
         when pre.dose is null and cur.dose>0       then "6-RESTART"
         when cur.dose=pre.dose and
           (cur.dose>0 and pre.dose>0)              then "2-NOCHANGE"
         when cur.dose<pre.dose and
           (cur.dose>0 and pre.dose>0)              then "7-REDUCE"
         when cur.dose>pre.dose and
           (cur.dose>0 and pre.dose>0)              then "4-INCREASE"
         else ""
       end as scripts
     from
       have as cur left join have as pre
     on
             cur.id = pre.id
       and   cur.month = pre.month + 1
     order
       by cur.id, cur.month
     ')
    want
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /* R                                      SAS                                                                             */
    /*  ID MONTH DOSE PREDOSE     SCRIPTS     ROWNAMES    ID    MONTH    DOSE    PREDOSE      SCRIPTS                         */
    /*                                                                                                                        */
    /*   1     1  500      NA 1-NOPREDATA         1        1      1       500        .      1-NOPREDATA                       */
    /*   1     2  500     500  2-NOCHANGE         2        1      2       500      500      2-NOCHANGE                        */
    /*   1     3   NA     500    3-DISCON         3        1      3         .      500      3-DISCON                          */
    /*   1     4   NA      NA   5-OFFDRUG         4        1      4         .        .      5-OFFDRUG                         */
    /*   2     1  500      NA 1-NOPREDATA         5        2      1       500        .      1-NOPREDATA                       */
    /*   2     2  500     500  2-NOCHANGE         6        2      2       500      500      2-NOCHANGE                        */
    /*   2     3  500     500  2-NOCHANGE         7        2      3       500      500      2-NOCHANGE                        */
    /*   2     4  250     500    7-REDUCE         8        2      4       250      500      7-REDUCE                          */
    /*   2     5  250     250  2-NOCHANGE         9        2      5       250      250      2-NOCHANGE                        */
    /*   3     1  500      NA 1-NOPREDATA        10        3      1       500        .      1-NOPREDATA                       */
    /*   3     2  250     500    7-REDUCE        11        3      2       250      500      7-REDUCE                          */
    /*   3     3  250     250  2-NOCHANGE        12        3      3       250      250      2-NOCHANGE                        */
    /*   3     4  500     250  4-INCREASE        13        3      4       500      250      4-INCREASE                        */
    /*   3     5  100     500    7-REDUCE        14        3      5       100      500      7-REDUCE                          */
    /*   4     1  500      NA 1-NOPREDATA        15        4      1       500        .      1-NOPREDATA                       */
    /*   4     2   NA     500    3-DISCON        16        4      2         .      500      3-DISCON                          */
    /*   4     3   NA      NA   5-OFFDRUG        17        4      3         .        .      5-OFFDRUG                         */
    /*   4     4  500      NA   6-RESTART        18        4      4       500        .      6-RESTART                         */
    /**************************************************************************************************************************/

    /*____               _   _                             _
    |___ /   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) | | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
            |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete pywant;
    run;quit;

    %utl_pybeginx;
    parmcards4;
    exec(open('c:/oto/fn_python.py').read());
    have,meta = ps.read_sas7bdat('d:/sd1/have.sas7bdat');
    want=pdsql('''
     select
       cur.id
      ,cur.month
      ,cur.dose as dose
      ,pre.dose as predose
      ,case
         when cur.month=1                           then "1-NOPREDATA"
         when pre.dose is null and cur.dose is null then "5-OFFDRUG"
         when pre.dose>0 and cur.dose is null       then "3-DISCON"
         when pre.dose is null and cur.dose>0       then "6-RESTART"
         when cur.dose=pre.dose and
           (cur.dose>0 and pre.dose>0)              then "2-NOCHANGE"
         when cur.dose<pre.dose and
           (cur.dose>0 and pre.dose>0)              then "7-REDUCE"
         when cur.dose>pre.dose and
           (cur.dose>0 and pre.dose>0)              then "4-INCREASE"
         else ""
       end as scripts
     from
       have as cur left join have as pre
     on
             cur.id = pre.id
       and   cur.month = pre.month + 1
     order
       by cur.id, cur.month
       ''');
    print(want);
    fn_tosas9x(want,outlib='d:/sd1/',outdsn='pywant',timeest=3);
    ;;;;
    %utl_pyendx;

    proc print data=sd1.pywant;
    run;quit;

    /*************************************************************************************************************************/
    /* PYTHON                                       |   SAS                                                                  */
    /*      ID  MONTH   DOSE  PREDOSE      SCRIPTS  |   ID    MONTH    DOSE    PREDOSE      SCRIPTS                          */
    /*                                              |                                                                        */
    /* 0   1.0    1.0  500.0      NaN  1-NOPREDATA  |    1      1       500        .      1-NOPREDATA                        */
    /* 1   1.0    2.0  500.0    500.0   2-NOCHANGE  |    1      2       500      500      2-NOCHANGE                         */
    /* 2   1.0    3.0    NaN    500.0     3-DISCON  |    1      3         .      500      3-DISCON                           */
    /* 3   1.0    4.0    NaN      NaN    5-OFFDRUG  |    1      4         .        .      5-OFFDRUG                          */
    /* 4   2.0    1.0  500.0      NaN  1-NOPREDATA  |    2      1       500        .      1-NOPREDATA                        */
    /* 5   2.0    2.0  500.0    500.0   2-NOCHANGE  |    2      2       500      500      2-NOCHANGE                         */
    /* 6   2.0    3.0  500.0    500.0   2-NOCHANGE  |    2      3       500      500      2-NOCHANGE                         */
    /* 7   2.0    4.0  250.0    500.0     7-REDUCE  |    2      4       250      500      7-REDUCE                           */
    /* 8   2.0    5.0  250.0    250.0   2-NOCHANGE  |    2      5       250      250      2-NOCHANGE                         */
    /* 9   3.0    1.0  500.0      NaN  1-NOPREDATA  |    3      1       500        .      1-NOPREDATA                        */
    /* 10  3.0    2.0  250.0    500.0     7-REDUCE  |    3      2       250      500      7-REDUCE                           */
    /* 11  3.0    3.0  250.0    250.0   2-NOCHANGE  |    3      3       250      250      2-NOCHANGE                         */
    /* 12  3.0    4.0  500.0    250.0   4-INCREASE  |    3      4       500      250      4-INCREASE                         */
    /* 13  3.0    5.0  100.0    500.0     7-REDUCE  |    3      5       100      500      7-REDUCE                           */
    /* 14  4.0    1.0  500.0      NaN  1-NOPREDATA  |    4      1       500        .      1-NOPREDATA                        */
    /* 15  4.0    2.0    NaN    500.0     3-DISCON  |    4      2         .      500      3-DISCON                           */
    /* 16  4.0    3.0    NaN      NaN    5-OFFDRUG  |    4      3         .        .      5-OFFDRUG                          */
    /* 17  4.0    4.0  500.0      NaN    6-RESTART  |    4      4       500        .      6-RESTART                          */
    /*************************************************************************************************************************/

    /*  _                       _                    _
    | || |     _____  _____ ___| |  _ __   ___  __ _| |
    | || |_   / _ \ \/ / __/ _ \ | | `__| / __|/ _` | |
    |__   _| |  __/>  < (_|  __/ | | |    \__ \ (_| | |
       |_|    \___/_/\_\___\___|_| |_|    |___/\__, |_|
                                                  |_|
    */

    proc datasets lib=sd1 nolist nodetails;
     delete want;
    run;quit;

    %utlfkil(d:/xls/wantxl.xlsx);

    %utl_rbeginx;
    parmcards4;
    library(openxlsx)
    library(sqldf)
    library(haven)
    have<-read_sas("d:/sd1/have.sas7bdat")
    wb <- createWorkbook()
    addWorksheet(wb, "have")
    writeData(wb, sheet = "have", x = have)
    saveWorkbook(
        wb
       ,"d:/xls/wantxl.xlsx"
       ,overwrite=TRUE)
    ;;;;
    %utl_rendx;

    %utl_rbeginx;
    parmcards4;
    library(openxlsx)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
     wb<-loadWorkbook("d:/xls/wantxl.xlsx")
     have<-read.xlsx(wb,"have")
     addWorksheet(wb, "want")
     want<-sqldf('
     select
       cur.id
      ,cur.month
      ,cur.dose as dose
      ,pre.dose as predose
      ,case
         when cur.month=1                           then "1-NOPREDATA"
         when pre.dose is null and cur.dose is null then "5-OFFDRUG"
         when pre.dose>0 and cur.dose is null       then "3-DISCON"
         when pre.dose is null and cur.dose>0       then "6-RESTART"
         when cur.dose=pre.dose and
           (cur.dose>0 and pre.dose>0)              then "2-NOCHANGE"
         when cur.dose<pre.dose and
           (cur.dose>0 and pre.dose>0)              then "7-REDUCE"
         when cur.dose>pre.dose and
           (cur.dose>0 and pre.dose>0)              then "4-INCREASE"
         else ""
       end as scripts
     from
       have as cur left join have as pre
     on
             cur.id = pre.id
       and   cur.month = pre.month + 1
     order
       by cur.id, cur.month
      ')
     print(want)
     writeData(wb,sheet="want",x=want)
     saveWorkbook(
         wb
        ,"d:/xls/wantxl.xlsx"
        ,overwrite=TRUE)
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/sd1/"
         ,outdsn ="want"
         )
    ;;;;
    %utl_rendx;

    proc print data=sd1.want;
    run;quit;

    /**************************************************************************************************************************/
    /*  --------------+                                                                                                       */
    /*  | A1| fx ID   |                                                                                                       */
    /*  ------------------------------------------                                                                            */
    /*  [_] | A |  B  |  C  |  D   |    E        |                                                                            */
    /*  -----------------------------------------|                                                                            */
    /*   1  |ID |MONTH|DOSE |PREDOSE|SCRIPTS     |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   2  |1  | 1   | 500 | .     | 1-NOPREDATA|                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   3  |1  | 2   | 500 | 500   | 2-NOCHANGE |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   4  |1  | 3   | .   | 500   | 3-DISCON   |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   5  |1  | 4   | .   | .     | 5-OFFDRUG  |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   6  |2  | 1   | 500 | .     | 1-NOPREDATA|                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   7  |2  | 2   | 500 | 500   | 2-NOCHANGE |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   8  |2  | 3   | 500 | 500   | 2-NOCHANGE |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*   9  |2  | 4   | 250 | 500   | 7-REDUCE   |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  10  |2  | 5   | 250 | 250   | 2-NOCHANGE |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  11  |3  | 1   | 500 | .     | 1-NOPREDATA|                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  12  |3  | 2   | 250 | 500   | 7-REDUCE   |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  13  |3  | 3   | 250 | 250   | 2-NOCHANGE |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  14  |3  | 4   | 500 | 250   | 4-INCREASE |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  15  |3  | 5   | 100 | 500   | 7-REDUCE   |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  16  |4  | 1   | 500 | .     | 1-NOPREDATA|                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  17  |4  | 2   | .   | 500   | 3-DISCON   |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  18  |4  | 3   | .   | .     | 5-OFFDRUG  |                                                                            */
    /*   -- |---+-----+-----+-------+------------|                                                                            */
    /*  19  |4  | 4   | 500 | .     | 6-RESTART  |                                                                            */
    /*   -- |---+-----+-----+-------+------------                                                                             */
    /*  [WANT]                                                                                                                */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

