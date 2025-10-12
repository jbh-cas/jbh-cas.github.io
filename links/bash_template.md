**BASH Script File Template Structure**

This the structure for bash script files that structures them to read from top to bottom.
Only one line, at the very end, is not in a bash function, so the entire script is parsed before running.

``` bash
#!/bin/bash

: replace contents of start_processing, get_args and usage functions and add others as needed.

main() {
   get_args $@
   start_processing  # usually renamed to describe processing here and in function name below
}

function start_processing {
   : replace with your own processing code
}


# support functions here

# arguments and usage

function get_args {
   BoldRed="\033[1;31m"; NC="\033[0m"; Blue="\033[0;34m"; Green="\033[0;32m"

   [ -z $1 ] && usage

   : args into vars here, eg tsv_file=$1
}

function usage {
   local script_name=$(basename $0)

   [ ! -z "$1" ] && err_msg "\n    $@"  # error message passed in to show

   # change this to match the arguments that your script expects
   msg "
    usage: $script_name <args> <here>
"
   exit 1
}
function msg { echo -e "$@" > /dev/stderr; }
function err_msg { echo -e $BoldRed"$@"$NC > /dev/stderr; }  # bold red


# call main to start things off, passing all the arguments

main $@
```

<br>

**Template Version 1**

This is an example version of the template that shows simple positional based argument parsing.  
It uses bawk, i.e., bioawk_cas, at https://github.com/calacademy-research/bioawk.CAS and  
get_int_val.sh at https://github.com/calacademy-research/assembly_etc/scripts/get_int_val.sh  

``` bash
#!/bin/bash

: replace contents of start_processing, get_args and usage functions and add others as needed.

main() {
   get_args $@
   start_processing
}

function start_processing {
   # replace with your own processing code

   local start_time=$(date +%s)
   local nrecs=$(num_fasta_recs)

   msg "tsv:\t$tsv\nfasta:\t$fasta, $nrecs recs\nthreads:$threads"
   sleep $((RANDOM % 6))

   show_time_spent $start_time
}


############################################################
#              utility and support functions               #
############################################################

function num_fasta_recs {
   bawk '
      { recs++ }
      END { print recs }
   ' $fasta  # uses var already determined to be a fasta file
}

function show_time_spent {
   [ -z $1 ] && return

   local start_time=$1
   local seconds_spent=$(seconds_to_now $start_time)
   msg "time spent:" $(seconds_to_duration $seconds_spent)
}


############################################################
#                   arguments and usage                    #
############################################################

: get_args when just positional args, use template_v2.sh when dash args and/or positional args

function get_args {  # just positional arguments, in this example a regular file, fasta file and optional number for thread count
   [ -z $1 ] && usage

   tsv=$1  # first arg must be name of a non-empty file
   [ ! -s "$tsv" ] && usage "first arg must be a non-empty file"

   fasta=$2  # looking for a fasta file as second arg
   ! is_fasta "$fasta" && usage "second arg requires a fasta file"

   threads=$3  # optional 3rd arg for thread count
   [ -z $threads ] && threads=16  # default to 16 threads
   [ -z $(get_int_val.sh $threads) ] && usage "thread value is not a number"
}

function usage {
   local pgm_name=$(basename $0)

   [ ! -z "$1" ] && err_msg "\n    $@"  # error message passed in to show

   # change this to match the arguments that your script expects
   msg "
    usage: $pgm_name <file> <fasta file> [<num threads>]
"
   exit 1
}

function msg { echo -e "$@" > /dev/stderr; }
function err_msg { echo -e "\033[1;31m$@\033[0m" > /dev/stderr; }  # bold red
function inf_msg { echo -e "\033[1;34m$@\033[0m" > /dev/stderr; }  # bold blue
function succ_msg { echo -e "\033[1;32m$@\033[0m" > /dev/stderr; } # bold green
function is_fasta { get_file_first_char $1; [[ $first_char == ">" ]]; }
function is_fastq { get_file_first_char $1; [[ $first_char == "@" ]]; }
function is_fastx { get_file_first_char $1; [[ $first_char == "@" || $first_char == ">" ]]; }
function get_file_first_char { local file=$1; [ ! -s "$file" ] && first_char="X" && return;  first_char=$( zgrep -m 1 -o ^. $file); }
function seconds_to_now { [ -z $1 ] && return; echo $(( $(date +%s) - $1 )); }
function seconds_to_duration {
   t=$1; d=$((t/60/60/24)); h=$((t/60/60%24)); m=$((t/60%60)); s=$((t%60))
   printf '%dd%02dh%02dm%02ds\n' $d $h $m $s | sed -e s/^0d// -e s/00[mh]//g  # do not show 0 days, hours, min
}


#############################################################
: calls main to start things off, passing all the arguments :
#############################################################

main $@
```

<br>

**Template Version 2**

This version handles more complex argument parsing with dash args and positional args, but is otherwise the same as Version 1.

``` bash
#!/bin/bash

: replace contents of start_processing, get_args and usage functions and add others as needed.

main() {
   version="0.1"
   get_args $@
   start_processing
}

function start_processing {
   # replace with your own processing code

   local start_time=$(date +%s)
   local nrecs=$(num_fasta_recs)

   msg "tsv:\t$tsv\nfasta:\t$fasta, $nrecs recs\nthreads:$threads"
   sleep $((RANDOM % 6))  # sleep from 0 to 5 seconds depending on RANDOW value

   show_time_spent $start_time
}


############################################################
#              utility and support functions               #
############################################################

function num_fasta_recs {
   bawk '
      { recs++ }
      END { print recs }
   ' $fasta  # uses var already determined to be a fasta file
}

function show_time_spent {
   [ -z $1 ] && return

   local start_time=$1
   local seconds_spent=$(seconds_to_now $start_time)
   msg "time spent:" $(seconds_to_duration $seconds_spent)
}


############################################################
#                   arguments and usage                    #
############################################################

: get_args to handle dash arguments as well as positional ones. use template_v1.sh when just positional args

function get_args {  # for handling dash arguments as well as positional ones
   pgm_name=$(basename $0)  # for usage and version info

   [ -z $1 ] && usage  # comment or delete if no args is OK
   last_arg=${!#}  # example of how to get last argument. set -- "${@:1:$#-1}"  # removes current last arg and resets the argument list and count

   while (( $# > 0 ));  do
      arg=$1; shift

      if [[ $arg != "-"* ]]; then  # positional arguments handled here
         [ -z $tsv ] && tsv=$arg && continue  # first positional is tsv filename
         [ -z $fasta ] && fasta=$arg && continue  # second positional is fasta filename

         int_val=$(get_int_val.sh $arg)  # 3rd positional is int. empty str if not integer style val
         [ ! -z $int_val ] && threads=$int_val && continue
      else  # dash arguments, both alone and with a value handled here
         [[ $arg == "-h" || $arg == "--help" ]] && usage
         [[ $arg == "-v" || $arg == "--version" ]] && [ ! -z $version ] && msg $pgm_name $version && exit

         # handle dash args with value here, e.g. for threads, -t 32
         dash_arg_val=$1;  : important to shift with those that use this value as below
         [[ $arg == "-t" || $arg == "--threads" ]] && threads=$dash_arg_val && shift && continue

         usage unrecognized argument '"'$arg'"'
      fi
   done

   # validate args here and set defaults
   [ ! -s "$tsv" ] && usage "first arg must be a non-empty file"
   ! is_fasta "$fasta" && usage "second arg requires a fasta file"

   set_threads $threads
}

function usage {
   [ ! -z "$1" ] && err_msg "\n    $@"  # error message passed in to show

   # change this to match the arguments that your script expects
   msg "
    usage: $pgm_name <file> <fasta file> [ -t <num_threads> ]
"
   exit 1
}

function set_threads {  # potential thread arg passed in as arg 1
   [ -z $1 ] && threads=16  # set thread default if not set on command line
   threads=$(get_int_val.sh $threads)
   [ -z $threads ] && usage "thread value is not a number"
}

function msg { echo -e "$@" > /dev/stderr; }
function err_msg { echo -e "\033[1;31m$@\033[0m" > /dev/stderr; }  # bold red
function inf_msg { echo -e "\033[1;34m$@\033[0m" > /dev/stderr; }  # bold blue
function succ_msg { echo -e "\033[1;32m$@\033[0m" > /dev/stderr; } # bold green
function is_fasta { get_file_first_char $1; [[ $first_char == ">" ]]; }
function is_fastq { get_file_first_char $1; [[ $first_char == "@" ]]; }
function is_fastx { get_file_first_char $1; [[ $first_char == "@" || $first_char == ">" ]]; }
function get_file_first_char { local file=$1; [ ! -s "$file" ] && first_char="X" && return;  first_char=$( zgrep -m 1 -o ^. $file); }
function seconds_to_now { [ -z $1 ] && return; echo $(( $(date +%s) - $1 )); }
function seconds_to_duration {
   t=$1; d=$((t/60/60/24)); h=$((t/60/60%24)); m=$((t/60%60)); s=$((t%60))
   printf '%dd%02dh%02dm%02ds\n' $d $h $m $s | sed -e s/^0d// -e s/00[mh]//g  # do not show 0 days, hours, min
}


#############################################################
# call main to start things off, passing all the arguments. #
#############################################################

main $@
```
