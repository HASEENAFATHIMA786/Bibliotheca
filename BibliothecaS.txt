menu_choice=""
student_file="student"
book_file="book"
trans_file="transaction"
hist_file="hist"
stud_hist_file="stu_hist";
book_hist_file="book_hist";
correct="true"
ch=1;
ch1=1;

 insert_record() 
  {
    echo $* >>$student_file
    return
  }

 insert_book_record() 
  {
   echo $* >>$book_file
   return
  }

 insert_trans_record()
  {
   echo $* >>$trans_file
   return
  }

 insert_his_record()
  {
   echo $* >>$hist_file
   return
  }

 student_report()
  {
   echo "      ID	Name	dob" 
   echo "    ---- ---------  -----" 
   awk -F":" '{ print "\t"$1 $2 $3 }' student
  }


 book_report()
  {
  # cat book;
   echo "      ID       Name    doI" 
   echo "    ---- ---------  -----" 
   awk -F":" '{ print "\t"$1 $2 $3 }' book
  }
 
 trans_report()
  {
   echo "           Students_ID    Book_ID   Date of Issue     Date of return"
   echo "          -------------  --------- ---------------   -----------------"
awk -F":" '{ print "\t"$1  "\t"      $2 "\t"            $3    "\t"        $4}' transaction

  }

 hist_report()
  {
   echo "           Students_ID    Book_ID   Date of Issue     Date of return"
   echo "          -------------  --------- ---------------   -----------------"
 awk -F":" '{ print "\t"$1  "\t"      $2 "\t"            $3    "\t"        $4 }' hist

  }


 hist_stud_report()
  {   
   printf 'Enter the student id                     : '
   read tmp
   hs_sid=${tmp%%,*}
   cat hist | awk  'BEGIN{FS=" ";OFS=" "}{if($1==var3){ print $1,$2,$3,$4 >"temp3"} }' var3=$hs_sid
   cp temp3 stu_hist;
   cat stu_hist;  

  }

 hist_book_report()
  {
   printf 'Enter the book id                        : '
   read tmp
   hs_bid=${tmp%%,*}
   cat hist | awk  'BEGIN{FS=" ";OFS=" "}{if($1==var4){ print $1,$2,$3,$4 >"temp4"} }' var4=$hs_bid
   cp temp4 book_hist;
   cat book_hist;
  }

 hist_report_choice()
 {
 #printf '\nWhat you you like to check history for\n'
 #echo "[1] Student id "
 #echo "[2] Book id    "
 #printf 'Enter your choice :'
 #read hist_menu_choice
 #case $hist_menu_choice in
 #1) echo "hai1";  
 #hist_stud_report;; 
 #2) echo "hai2";  
 #hist_book_report;;
 #esac
 #printf '\nHai\n'
 cat hist;


 }



reports() 
  {
    printf '\n'
    echo "[1] Student Reports"
    echo "[2] Book Reports"
    echo "[3] Transaction Reports"
    echo "[4] History Reports"
    printf '              Enter your choice      :'
    read report_menu_choice

     case $report_menu_choice in
            1)student_report;;
            2)book_report;;
            3)trans_report;;
            4)hist_report_choice;;
              esac
  }  

add_student() 
  {
   set $(wc -l $student_file)
   tmp=$1
   tmp=$((tmp+1))
   num=${tmp%%,*}  
    printf 'Enter the student name                            :'
    read tmp
    name=${tmp%%,*}
    printf 'Enter the date of joining  in yyyy/mm/dd format   :' 
    read tmp
    doj=${tmp%%,*}
    printf "$num $name $doj"
    printf "\n"
    echo "                            The data is recorded sucessfully in the student file";
    insert_record $num $name $doj
    return
  }

 add_book()
  {
   set $(wc -l book)
   tmp=$1
   tmp=$((tmp+1))
   bnum=${tmp%%,*}
   printf 'Enter the book name                                : '
   read tmp
   bname=${tmp%%,*}
   tmp=$(date +%y/%m/%d)
   bdoj=${tmp%%,*}
   printf 'Enter the number of copies of book issued          : '
   read tmp
   copy=${tmp%%,*}
   printf "$bnum $bname $bdoj $copy"
   echo "                                  The data is recorded successfully in the book file"; 
   insert_book_record $bnum $bname $bdoj $copy
   return
  }


 trans_add_book()
  {
   printf 'Enter the student id                               : '
   read tmp
   ta_sid=${tmp%%,*}
   printf 'Enter the book id                                  : '
   read tmp
   ta_bid=${tmp%%,*}
   tmp=$(date +%y/%m/%d)
   ta_doj=${tmp%%,*}
   tmp=$(date +%y/%m/%d -ud "UTC + 15 day ") 
   ta_dor=${tmp%%,*}
   printf "$ta_sid $ta_bid $ta_doj $ta_dor"
   cat student | awk  'BEGIN{FS=" ";}{if($1==var5){print $2 > "temp5"} }' var5=$ta_sid
   echo "Student Name: "
   cat temp5;
   cat book | awk  'BEGIN{FS=" ";}{if($1==var5){print $2 > "temp6"} }' var5=$ta_bid
   echo "Book Name: "
   cat temp6;

   cat book | awk 'BEGIN{FS=" ";count=1}{if($1==var && $4==0){count =0;}}END{ print count >"count"}' var=$ta_bid
   ch=`cat count`;
   if [ "$ch" -eq 0 ]; then
	{	echo "                              Sorry. No book is available.";} fi

 if [ "$ch" -eq 1 ]; then
  {
   cat book | awk  'BEGIN{FS=" ";OFS=" "}{if($1==var){ $4=$4-1; print $1,$2,$3,$4 >"temp"}else{ print $0 >"temp"} }' var=$ta_bid
   cp temp book;
   insert_trans_record $ta_sid $ta_bid $ta_doj $ta_dor;
   insert_his_record $ta_sid $ta_bid $ta_doj $ta_dor;
  }  fi

   return;
  }

 trans_remove_book()
  {
   printf 'Enter the student id                               : '
   read tr_sid
   printf 'Enter the book id                                  : '
   read tr_bid
   searchstr=$tr_sid$tr_bid
  cat transaction | awk 'BEGIN{FS=" ";OFS=" "}{if($1==var1 && $2==var2){print $4 > "temp9"} }' var1=$tr_sid var2=$tr_bid

 echo "The Due Date: " 
  cat temp9;
  ch2=`cat temp9`;
  ch3=`date +%y/%m/%d `;
  echo "---------------------------------------------------------------------------------------------------------"
  echo "                                        The Fine Amount is Rs."
  echo $(($(($(date -d "2014/03/03" "+%s")-$(date -d "2014/03/03" "+%s"))) / 86400))
  echo "---------------------------------------------------------------------------------------------------------"

  
    cat student | awk  'BEGIN{FS=" ";}{if($1==var5){print $2 > "temp7"} }' var5=$tr_sid
   echo "Student Name: "
   cat temp7;
   cat book | awk  'BEGIN{FS=" ";}{if($1==var5){print $2 > "temp8"} }' var5=$tr_bid
   echo "Book Name: "
   cat temp8;

   cat book | awk 'BEGIN{FS=" ";count1=0}{if($1==var){count1=1;}}END{print count1 >"count1"} ' var=$tr_bid
   ch1=`cat count1`;
   if [ "$ch1" -eq 0 ]; then
        {       echo "Sorry. No book is available.";} fi
 if [ "$ch1" -eq 1 ]; then
  {
   cat book | awk  'BEGIN{FS=" ";OFS=" "}{if($1==var){ $4=$4+1; print $1,$2,$3,$4 >"temp1"}else{ print $0 >"temp1"} }' var=$tr_bid
   cp temp1 book;

   insert_his_record $tr_sid $tr_bid $ta_doj $ta_dor;
    cat transaction | awk  'BEGIN{FS=" ";OFS=" "}{if($1!=var1 && $2!=var2){ print $1,$2,$3,$4 >"temp2"} }' var1=$tr_sid var2=$tr_bid
    cp temp2 transaction;

    printf "                                    Record has been recorded in the history file\n"
    printf "                                         The entry is closed successfully\n"
  }  fi



  #printf "record has been removed\n"
  return
  }


 transaction()
  {
    printf '\n'
    echo "[1] Book issue : "
    echo "[2] Book return: "
    printf '              Enter your choice      :'
    read trans_menu_choice
    #echo "the choice is: $trans_menu_choice"

     case $trans_menu_choice in
            1)trans_add_book;;
            2)trans_remove_book;;
              esac
  }

hist_stud_report() 
{
  cat hist;
}
 hist() 
 {
 printf '\n'
 echo "[1] Student id                   "
 echo "[2] Book id                      "
 printf 'Enter your choice             : '
 read hist_menu_choice
 case $hist_menu_choice in
  1)hist_stud_report;;
  2)hist_book_report;;
 esac
 }



 choice()  
  {
   while [ "$correct" != "false" ] 
    do
      printf '\n'
      echo "[1] Students Database"
      echo "[2] Book Database"
      echo "[3] Transaction"
      echo "[4] Report"
      echo "[5] History"
      echo "[6] Exit"
      printf 'Enter your choice        : '
      read menu_choice
      echo "the choice is,$menu_choice"
       
        case $menu_choice in
          1) add_student;;
          2) add_book;;
          3) transaction;;
          4) reports;;
          5) hist_report;;
          6) echo "                            Thank you for viewing the site"
             correct="false";;
             esac
    done
  }

echo "                                           WELCOME TO GRD MEMORIAL"
 case $ch in
    1)choice;;
    esac


