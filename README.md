# utl_joining_and_updating_excel_sheets_without_importing_data
Joining and updating excel sheets without importing data. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Joining and updating excel sheets without importing data;

    I do not modify sheet1, which is possible, but rather create sheet3 with the table join.
    Not really a table join but rather a format lookup, sort of like vlookup in excel.
    Program uses an array processing in SQL.

    github
    https://github.com/rogerjdeangelis/utl_joining_and_updating_excel_sheets_without_importing_data

    see
    https://communities.sas.com/t5/Base-SAS-Programming/how-to-get-link-new-variables-in-SAS-from-2-Excel-sheets/m-p/467779

    Other excel repositories
    https://github.com/rogerjdeangelis?utf8=%E2%9C%93&tab=repositories&q=excel&type=&language=


    PROC IMPORT JUST COMPLCATES THE PROCESS. (Proc import has some very limited uses, But R and Python more than make up
    for 'proc import')



    INPUT
    =====

    ICD10 Codes (SHEET1)

      d:/xls/icd10.xlsx sheet1


    +----------+----------+----------+----------+----------+----------+----------+-----------+
    |    A     |     B    |    C     |    D     |   E      |    F     |    G     |    H      |
    +----------+----------+----------+----------+----------+----------+----------+-----------+
    |ptNumber  |id        |CAUSE1    |CAUSE2    |CAUSE3    |CAUSE4    |CAUSE5    |CAUSE6     |
    |----------+----------+----------+----------+----------+----------+----------+-----------|
    |  1       |100506737 |   A41    |   A41    |          |          |  E43     |           |
    +----------+----------+----------+----------+----------+----------+----------+-----------+
    |  2       |100506739 |   C61    |          |          |          |  E43     |           |
    +----------+----------+----------+----------+----------+----------+----------+-----------+
    |  3       |100506740 |   K76    |   K72    |   B16    |   E11    |  I10     |           |
    +----------+----------+----------+----------+----------+----------+----------+-----------+
    |  4       |100506741 |   J96    |   K72    |   K72    |   K70    |          |     K65   |
    +-----------------------------------------------------------------------------------------
    |..        |          |          |          |          |          |          |           |
    +----------+----------+----------+----------+----------+----------+----------+-----------+
    | 20       |100502818 |   R99    |          |          |          |          |           |
    +-----------------------------------------------------------------------------------------


    LOOKUP TABLE FOR CANCER DESCRIPTIONS  (SHEET2)
    ----------------------------------------------

     d:/xls/icd10.xlsx sheet2

    +----------+----------------------------------------+
    |          |                                        |
    |----------+----------------------------------------+                                       |
    |ptNumber  |Diseasename                             |
    |----------|----------------------------------------|
    | A00      |Cólera                                  |
    +----------+----------------------------------------+
    | A01      |Fiebres tifoidea y paratifoidea         |
    -----------+----------------------------------------+
    | A02      |Otras infecciones debidas a Salmonella  |
    +----------+----------------------------------------+
    | A03      |Otras infecciones intestinales          |
    +----------+----------------------------------------+
    | Z99      |Shigelosis                              |
    +----------+----------------------------------------+

     d:/xls/icd10.xlsx sheet3

                                                                      +-----------------------------------------------------+
                                                                      |                           RULES                     |
                                                                      |                           =====                     |
    EXAMPLE OUTPUT                                                    |  Only the ICD10s that begin with C are looked up    |
    --------------                                                    |  Cancer names shortened so example fits             |
                                                                      |                                                     |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |    A   |     B   |    C |    D |   E  |    F  |    G |    H | I |    J     |    K     |   L   |    M  |    N  |    O  |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |ptNumber|id       |CAUSE1|CAUSE2|CAUSE3|CAUSE4 |CAUSE5|CAUSE6|   |CANCER1   |CANCER2   |CANCER3|CANCER4|CANDER5|CANCER6|
    |--------+---------+------+------+------+-------+------+------|---+----------+----------+-------+-------+-------+-------|
    |  1     |100506737|   A41|   A41|      |       |  E43 |      |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  2     |100506739|   C61|      |      |       |  E43 |      |   | próstata |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  3     |100506740|   K76|   K72|   B16|   E11 |  I10 |      |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  4     |100506741|   J96|   K72|   K72|   K70 |      |  K65 |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  5     |100506743|   C78|   C16|      |     0 |      |      |   |digestivos| estómago |       |       |       |       |
    +------------------------------------------------------------------------------------------------------------------------
    |..      |         |      |      |      |       |      |      |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    | 20     |100502818|   R99|      |      |       |      |      |   |          |          |       |       |       |       |
    +------------------------------------------------------------------------------------------------------------------------


    PROCESS
    =======

    libname xel "d:/xls/icd10.xlsx";

    * create format lookup;
    data mkeFmt;
     retain fmtname '$icd2des' hlo ' ';
     *set xel.icd(where=(icd10 =: 'C'))  end=dne;
     set xel.'sheet2$'n(where=(icd10 =: 'C'))  end=dne;
       start=icd10;
       end=start;
       label=diseasename;
       output;
       if dne then do;
          hlo='O';
          label=' ';
          output;
       end;
    run;quit;

    proc format cntlin=mkeFmt;
    run;quit;

    %array(caus,values=1-6);
    proc sql;
            create
              table xel.sheet3 as
            select
              ptnumber
             ,id
             ,%do_over(caus,phrase=cause?,between=comma)
             ,%do_over(caus,phrase=put(cause?,$icd2des.) as cancername?,between=comma)
            from
             xel.'sheet1$a2:h22'n
    ;quit;

    libname xel clear;


    OUTPUT
    ======

    https://github.com/rogerjdeangelis/utl_joining_and_updating_excel_sheets_without_importing_data/blob/master/icd10.xls

    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |    A   |     B   |    C |    D |   E  |    F  |    G |    H | I |    J     |    K     |   L   |    M  |    N  |    O  |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |ptNumber|id       |CAUSE1|CAUSE2|CAUSE3|CAUSE4 |CAUSE5|CAUSE6|   |CANCER1   |CANCER2   |CANCER3|CANCER4|CANDER5|CANCER6|
    |--------+---------+------+------+------+-------+------+------|---+----------+----------+-------+-------+-------+-------|
    |  1     |100506737|   A41|   A41|      |       |  E43 |      |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  2     |100506739|   C61|      |      |       |  E43 |      |   | próstata |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  3     |100506740|   K76|   K72|   B16|   E11 |  I10 |      |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  4     |100506741|   J96|   K72|   K72|   K70 |      |  K65 |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    |  5     |100506743|   C78|   C16|      |     0 |      |      |   |digestivos| estómago |       |       |       |       |
    +------------------------------------------------------------------------------------------------------------------------
    |..      |         |      |      |      |       |      |      |   |          |          |       |       |       |       |
    +--------+---------+------+------+------+-------+------+------+---+----------+----------+-------+-------+-------+-------+
    | 20     |100502818|   R99|      |      |       |      |      |   |          |          |       |       |       |       |
    +------------------------------------------------------------------------------------------------------------------------


    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    * delete sheet3 and named range sheet3;
    https://github.com/rogerjdeangelis/utl_joining_and_updating_excel_sheets_without_importing_data/blob/master/icd10.xls


    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    see process

