# utl-Deleting-some-obs-conditionally-based-on-the-previous-obs-and-flip-file-upside-down
    Deleting some obs conditionally based on the previous obs and flip file upside down

       Two Solutions

            1, Turn the file upside down and delete '/' and two subsequent records.
               rogerjdeangelis@gmail.com

            2, Uses two input streams. When a '/' record is found in the primary stream the
               secondary stream  is stopped two records before the '/' record and
               records are removed.

               by  Keintz, Mark
               mkeintz@wharton.upenn.edu



    Flipping a file upside down can very useful, especially useful when the data you need is
    close to the end of the file.

    SAS cannot easily flip a file.

    SAS needs to improve the integration with what I consider the big three, Python, R and Perl.
    Varchar would also help.

    github
    https://tinyurl.com/yxwf64d3
    https://github.com/rogerjdeangelis/utl-Deleting-some-obs-conditionally-based-on-the-previous-obs-and-flip-file-upside-down

    SAS Forum
    https://tinyurl.com/y4sva2xu
    https://communities.sas.com/t5/SAS-Programming/Deleting-some-obs-conditionally-of-the-previous-obs/m-p/560238/highlight/true#M156611

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data _null_;
      file "d:/txt/have.txt";
      do rec=75000 to 1 by -1;
         if mod(rec,7)=0 then do;
            put rec z8.;
            put rec z8.;
            txt=cats('/',put(rec,z8.));
            put txt;
         end;
         else put rec z8.;;
      end;
    run;quit;


    d:/txt/have.txt


    00075000
    00074999
    00074998
    00074998
    /00074998
    00074997
    00074996
    00074995
    00074994
    00074993
    00074992
    00074991
    00074991
    /00074991
    00074990
    00074989
    00074988
    00074987
    00074986
    00074985
    00074984
    00074984
    /00074984
    ...
    0000003
    0000002
    0000001

    *           _
     _ __ _   _| | ___  ___
    | '__| | | | |/ _ \/ __|
    | |  | |_| | |  __/\__ \
    |_|   \__,_|_|\___||___/

    ;

    d:/txt/have.txt

    00075000
    00074999

    00074998
    00074998
    /00074998  Remove this record and the previous two

    00074997
    00074996
    00074995
    00074994
    00074993
    00074992

    00074991
    00074991
    /00074991  Remove this record and the previous two

    00074990
    00074989
    00074988
    00074987
    00074986
    00074985

    00074984
    00074984
    /00074984  Remove this record and the previous two
    ...
    0000003
    0000002
    0000001

    And flip the file upside down

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    d:/txt/flip.txt

    Flipped file triplet records removed

    00000001
    00000002
    00000003
    00000004
    00000005
    00000006
             00000007 is out because it was part of a triplet
    00000008
    00000009
    00000010
    00000011
    00000012
    00000013
    00000014
    ...
    00074997
             00074998 was deleted because it was part of a triplet
    00074999
    00075000

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    *************************************************************************
    1, Turn the file upside down and delete '/' and two subsequent records. *
    *************************************************************************

    * no SAS table needed;

    %utlfkil(d:/txt/flip.txt);

    data _null_;

     if _n_=0 then do; %let rc=%sysfunc(dosubl(%nrstr(

         %utl_submit_py64('

           infile=open("d:/txt/have.txt","r").readlines();
           ofile=open("d:/txt/flip.txt", "w");
           ofile.writelines(infile[::-1]);

         ');

        )));
     end;
     length txt $9.;

     infile "d:/txt/flip.txt";

     file "d:/txt/flipfix.txt";

     input txt;

     if txt =: "/" then do;
        input;
        input;
     end;

     else put txt;

    run;quit;

    ****************************************************************************
    2, Two input streams and when a '/' is found in the primary stream input   *
    ****************************************************************************

    data _null_;
      file "d:/txt/have.txt";
      do rec=75000 to 1 by -1;
         if mod(rec,7)=0 then do;
            put rec z8.;
            put rec z8.;
            txt=cats('/',put(rec,z8.));
            put txt;
         end;
         else put rec z8.;;
      end;
    run;quit;

    filename in1 'd:/txt/have.txt';
    filename in2 'd:/txt/have.txt';

    data _null_;

       infile in1 end=end1;
       do i=1 by 1 until (_infile_=:"/" or end1);
         input;
       end;
       if _infile_=:'/' then testrec=input(substr(_infile_,2,8),8.);

       infile in2;
       do j=1 to i;
         if j<i then input rec 8. ;
         else input;
         file "d:/txt/preflip.txt" ;
         if (rec^=testrec and j<i) or testrec=. then put _infile_;
       end;

       * flip the file;
       if dne then do; %let rc=%sysfunc(dosubl(%nrstr(
          %utl_submit_py64('

            infile=open("d:/txt/preflip.txt","r").readlines();
            ofile=open("d:/txt/flip.txt", "w");
            ofile.writelines(infile[::-1]);

          ');
         )));
       end;

    run;quit;



    NOTE: The infile IN1 is:
          Filename=d:\txt\have.txt,
          RECFM=V,LRECL=384,File Size (bytes)=974994,
          Last Modified=22May2019:19:31:46,
          Create Time=03Dec2018:19:41:43

    NOTE: The infile IN2 is:
          Filename=d:\txt\have.txt,
          RECFM=V,LRECL=384,File Size (bytes)=974994,
          Last Modified=22May2019:19:31:46,
          Create Time=03Dec2018:19:41:43

    NOTE: The file "d:/txt/preflip.txt" is:
          Filename=d:\txt\preflip.txt,
          RECFM=V,LRECL=384,File Size (bytes)=0,
          Last Modified=22May2019:19:32:05,
          Create Time=22May2019:19:14:14

    NOTE: 96428 records were read from the infile IN1.
          The minimum record length was 8.
          The maximum record length was 9.
    NOTE: 96428 records were read from the infile IN2.
          The minimum record length was 8.
          The maximum record length was 9.

    NOTE: 64285 records were written to the file "d:/txt/preflip.txt".
          The minimum record length was 8.
          The maximum record length was 8.
    NOTE: DATA statement used (Total process time):
          real time           0.24 seconds

    3604!     quit;



