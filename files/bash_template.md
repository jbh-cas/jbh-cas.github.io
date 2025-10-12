**BASH Script File Template Structure**

This the structure for bash script files that structures them to read from top to bottom.
Only one line, at the very end, is not in a bash function, so the entire script is parsed before running.

``` bash
#!/bin/bash

: replace contents of start_processing, get_args and usage functions and add others as needed.

main() {
   get_args $@
   start_processing  # usually renamed to describe porcessing here and in function name below
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
