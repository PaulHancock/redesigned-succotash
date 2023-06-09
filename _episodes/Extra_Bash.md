---
title: "Advanced Bash"
teaching: 90
exercises: 0
questions:
- "What tools are available in bash?"
- "How can I write bash scripts?"
objectives:
- "Understand that bash is a programming language"
- "Learn some of the builtin unix tools"
keypoints:
- "Use the tools best suited for a job"
---

## Bash (and other unix shells)
Your favorite language may be python, but that doesn't mean that every problem should be solved using python.
More generally, it is good practice to know the basics of many different languages/tools, so that when you want to solve a problem or complete some task, you have a range of tools at your disposal, and can choose the one that is best suited for the job.
A tool that everyone has access to is the terminal (usually a Bash shell), and there are a number of powerful tools that come along with it that we will explore here.

### grep/sed/awk and regular expressions
These tools are extremely useful for manipulating streams (or files) of text.
`grep` (or egrep) is a filtering program that will return lines of text that match a given pattern.
`sed` is the stream editor, which is designed to match *and replace* text according to a pattern that you pass.
Finally there is `awk` which is a programming language geared toward text editing.
In order to ge the most out of these tools there is a meta-skill that is needed and that is regular expressions (or "regex"), which is a language for describing patterns.

Unknowingly you have already been using regular expressions when you execute a command such as:
```
ls *.fits
mv output?.dat output/.
```
{: .language-bash}

These "wildcards" of `*` and `?` in the above example are shortcuts for "anything" (`*`) or "just one character (`?`).
We can extend this further by using `[0-9]` which means "one of the digits between 0-9", or `[a-z]` to indicate "any lower case letter".
For example `ls 202[012]` would list all files/directories with a name that is either 2020, 2021, or 2022.
Similarly we could do `ls 201?` to show files/directories such as 2010, 2011, ..., 2019, but also 201a, 201X, and even 201#.

### Regular expression pattern matching

| Pattern            | Matches                                                              |
| ------------------ | -------------------------------------------------------------------- |
| `^`                | start of a line                                                      |
| `$`                | end of a line                                                        |
| a single character | that one character                                                   |
| `*`                | match zero or more of the preceding symbol                           |
| `+`                | match one or more of the preceding symbol                            |
| `?`                | match zero or one of the preceding symbol                            |
| `[]`               | one of the characters within the braces                              |
| `[^]`              | any character that is **not** with the braces                        |
| `()`               | nothing. Creates a group from whatever is matched within the braces. |
| `\n`               | the group number n (1-9)                                             |
| `\w`               | any word character (alphanumeric char), `\W` inverts                 |
| `\s`               | any white space char (eg, tab/space), `\S` inverts                   |
| `\t`               | the tab character                                                    |
| `[[:lower:]]`      | any lower case char, same as `[a-z]`                                 |
| `[[:upper:]]`      | any upper case char, same as `[A-Z]`                                 |
| `[[:alpha:]]`      | any alphabet character same as `[a-zA-z]`                            |
| `[[:digit:]]`      | any numerical digit (same as `[0-9]`)                                |
| `[[:alnum:]]`      | any alphanumeric character (same as `\w`)                            |
| `\meta`            | the meta-character itself                                            |

A more complete summary of the various matching options is given on this [cheat sheet](https://cheatography.com/davechild/cheat-sheets/regular-expressions/).
Regular expressions can get extremely complicated and suffer from the curse of being an essentially a read only language.
A deep dive on regular expressions isn't worth doing, however a basic understanding is extremely helpful as it will allow you to do some fancy things with `grep` and `sed`.

Examples:
- `grep '^#SBATCH' jobfile.sh` will match lines that **start with** `#SBATCH`
- `grep 'amen\.$' bible.txt` will match lines that **end with** "amen." (including the trailing period)
- `grep '[Hh]ello' myfile.txt` will match instances of **either** "Hello" or "hello" within the file
- `grep '0[0-9]' ids.txt` will match pairs of digits from 00 -> 09
- `grep 'a(b|c)\1' text.dat` will match abb or acc but not abc. The \1 represents a copy of the group captured within the braces `()`.
- `grep '\*' -` will match the character `*`

Repeatedly testing regex patterns can be a pain, but I find [regex101.com](https://regex101.com) to be a useful place to test my regex on some example text.
The big strength of this site is that it shows you *how* the matches are being made and what each element of your pattern is matching to.
The site uses standard regex so you can't simply copy/paste into your sed/grep commands since you'll have to escape some characters (like `(` or `{`)

This site [GNU sed REPL](https://sed.js.org/) allows you to test out your `sed` commands either on example input, or input that you post yourself.
There is no highlighting being done, but you can directly copy the regex you use here onto your command line and it will work.

### back to grep/sed/awk
`grep` (or `egrep`) is a very handy tool that will allow you to quickly scan through files and find lines that match a pattern.

For example, if we have a very long log file, we can search for lines that contain the word "WARN" using: `grep 'WARN' logfile.txt`.
We could also search through a bunch of python scripts to find lines that make calls to the print function by using: `grep 'print' *.py`.

With some regex help we can look for lines that have been commented out in our scripts by searching for a # at the start of the line: `grep '^#' *.py *.sh *.c`.
If we want to exclude the "#SBATCH" and "#include" directives we can require that the line starts with a # followed by a space: `grep '^# ' *.py *.sh *.c`.
Furthermore we can allow the line to start with some indentation of space/tab combo: `grep '^\s*# ' *.py *.sh *.c`.
Note that this last example will not include lines that start with code, and have a trailing comment.

By default `grep` will just print the lines that match, and if you are searching multiple files, it will prepend the line with `<filename>:`.
Here are some commonly used parameters that you can use to modify the behavior of `grep`:

| option | effect                                                            |
| ------ | ----------------------------------------------------------------- |
| -i     | ignore case when making matches                                   |
| -v     | print **non**-matching lines                                      |
| -c     | just count the number of matching lines                           |
| -l     | just print the names of files which have at least 1 matching line |
| -n     | prepend the line number to the output                             |
| -A n   | also print n lines after a matching line, default 0               |
| -B n   | also print n lines before a matching line, default 0              |
| -C n   | do -A n and -B n at the same time, default 0                      |
| -r     | recurse into directories                                          |


`sed` is designed to let you edit a text file or stream, and requires that you provide a pattern for matching, and then a replacement for the text that is matched.
`sed` is particularly useful for editing files in bulk.
When developing this course I had some example job scripts that had lines that looked like the following:
```
#! /usr/bin/env bash
#
#SBATCH --job-name=start
#SBATCH --output=/fred/oz983/phancock/start_%A_out.txt
#SBATCH --error=/fred/oz983/phancock/start_%A_err.txt
#
#SBATCH --ntasks=1
#SBATCH --time=00:05
#SBATCH --mem-per-cpu=1G
```
{: .language-bash}

These scripts worked for me, however they will not work as intended for others because I have hardcoded the output/error directories to **my** work directory.
What I want to do is replace instances of `/phancock/` with `/%u/` so that SLURM will replace the `%u` with the username of whoever is running the script.
To achieve this I firstly need to find all instances of the pattern `/phancock/`, and for this I use grep:
```
grep '^#SBATCH.*/phancock/' *.sh
```
{: .language-bash}

I have included the `^#SBATCH` so that I only match lines that are in the SLURM directives header section, and then use `.*` to match any set of characters between the `#SBATCH` section and the `/phancock/` section.
If I were less pedantic and simply matched to `phancock` then I might end up matching lines elsewhere in the file.

```
branch.sh:#SBATCH --output=/fred/oz983/phancock/ngon_%A-%a_out.txt
branch.sh:#SBATCH --error=/fred/oz983/phancock/ngon_%A-%a_err.txt
collect.sh:#SBATCH --output=/fred/oz983/phancock/collect_%A_out.txt
collect.sh:#SBATCH --error=/fred/oz983/phancock/collect_%A_err.txt
mpi.sh:#SBATCH --output=/fred/oz983/phancock/MPI_Hello_%A_out.txt
mpi.sh:#SBATCH --error=/fred/oz983/phancock/MPI_Hello_%A_err.txt
openmp.sh:#SBATCH --output=/fred/oz983/phancock/OpenMP_Hello_%A_out.txt
openmp.sh:#SBATCH --error=/fred/oz983/phancock/OpenMP_Hello_%A_err.txt
start.sh:#SBATCH --output=/fred/oz983/phancock/start_%A_out.txt
start.sh:#SBATCH --error=/fred/oz983/phancock/start_%A_err.txt
```
{: .output}

We can see that there are 5 files, each with 2 instances of the mistake that I want to correct.

To replace all instances of `/phancock/` with `/%u/` I can use `sed` with the following syntax:
```
sed -e 's[separator][match pattern][separator][replacement text][separator]g' <filename>
```
{: .language-bash}

Usually people will use `/` or `:` as the separator, depending on what characters are being used in the match/replacement string. 
Any character can be used.
The `s` at the start of the string stands for *substitute* and the trailing `g` means *globally* (eg, all lines, possibly multiple times per line).
We can use the match pattern from our `grep` command above, however we don't want to replace the entire line, just the `/phancock/` section.
Thus we need to create some *capture groups* using `\( \)`.
Our `sed` command would look like this:

```
sed -e 's:\(^#SBATCH.*\)/phancock/:\1/%u/:g' branch.sh
```
{: .language-bash}

We have captured the first part of the line using `\(^#SBATCH.*\)` into group 1.
We then substitute the matched part of the line with this group using `\1` and then append our desired `/%u/`.
Effectively we have a long match pattern and are replacing only a subset of what was matched.
Each new set of `\(\)` creates a new group with an increasing number between 1-9.
We can nest these groups if desired, but we cannot have more than 9 automatically labeled groups.

By default `sed` will direct the output to STDOUT (eg your screen).
However we can use the `-i` flag to cause `sed` to do the replacement *inplace*, effectively editing the file.
Alternatively we can pipe the output to a new file using `> newfile`, or append to an existing file with `>> existingfile`.

Once I am confident that my replacements are going to wreck my files I run the following (in-place) substitution on all the `.sh` files in my directory:

```
sed -i -e 's:\(^#SBATCH.*\)/phancock/:\1/%u/:g' *.sh
```
{: .language-bash}

I find [this page](https://sed.js.org/) quite useful for testing my sed expressions on example text.
It has saved me from nerfing my files more than once.


## Bash as a programming language
The Bash shell acts as both a Unix shell (command line / command prompt / interactive terminal) and as a command language.
Bash is the default shell in most Linux distributions making it widely available.
You have no-doubt been using the Bash as a shell, especially since this is the preferred (and often only) way of interacting with HPC resources.
Learning more about the programming language side of Bash can help you to write better job scripts, scripts for your own quality of life improvements, and also make you more of a command line wizard.

Features of Bash as a programming language:
- control flow with `if/then/else/elif/fi` or `case/in/.../esac/`
- loops with `for/do/done` and `while/do/done`
- variable assignment with `var=value`
- variable lookup with `${var}` ( or just `$var` if you are lazy/brave)
- array assignment with `arr=(val1, val2, ...)`
- array access with `${arr[0]}`

Bash has a lot of built in commands which you can think of as functions.
A common design principle for Unix can be summarized as:
- Write programs that do one thing and do it well.
- Write programs to work together.
- Write programs to handle text streams, because that is a universal interface.

You should therefore think of the various Bash command line tools as functions that will modify streams of information.

Processes can emit output to one or both of STDERR (standard error) or STDOUT(standard out).
Standard practice is to put output into STDOUT and warning/error messages into STDERR.
When using `echo` the output will be directed to STDOUT.
By default both of these streams will be displayed in your terminal, however they can be redirected.
- redirection of STDOUT with `|`, `>`, and `>>`
  - `|` will *pipe* the output of one command to the input of another
    - example: `ls | wc -l` to count the number of lines generated by `ls`
    - example: `ls | more ` to show the results of `ls` one page at a time.
  - `>` will redirect the STDOUT of a command into a file, *overwriting* an existing file
    - example: `ls *.fits > image_files.txt`
  - `>>` will redirect the STDOUT of a command into a file, *appending* to an existing file
- redirection of STDERR with `2>`
  - example: `find . 2> /dev/null` will send all error messages to `/dev/null` (an information black hole)
  - example: `echo "Warning: Things are bad" >&2` will send the message to STDERR instead of STDOUT
  - example: `my_script.sh > output.txt 2> log.txt` will save STDOUT into `output.txt` and STDERR into `log.txt`
  - example: `my_script.sh 2>&1 > all.txt` will redirect STDERR to STDOUT (`2>&1`) and then send both to the file `all.txt`


- process substitution with `<( )`
  - `cmd1 <(cmd2)` is equivalent to `cmd2 | cmd1`
  - If you need to do multiple pipes then process substitution can be handy
    - `
- process substitution with `>( )`
  - example: `my_script.sh > output.txt 2> >(grep -e "^Warn" > warnings.txt)` will save STDERR to `warnings.txt` but only after passing it through `grep`.

When a bash command completes it will return an exit code.
By convention 0 means "success" and anything else means "error".
The exit code of the last completed command is stored in the special variable `$?`.
Note that this exit code is independent of what the program may or may not write to STDOUT or STDERR.
The `grep/sed` programs we looked at earlier will return 0 if a match is found and something else otherwise.

```
$ grep 'SBATCH' collect.sh 
#SBATCH --job-name=collect
#SBATCH --output=/fred/oz983/%u/collect_%A_out.txt
#SBATCH --error=/fred/oz983/%u/collect_%A_err.txt
#SBATCH --ntasks=1
#SBATCH --time=00:05
#SBATCH --mem-per-cpu=200

$ echo $?
0

$ grep 'other things' collect.sh 
$ echo $?
1
```
{: .output}

Confusingly, but also usefully, Bash interprets 0 as "true" and 1 and "false" in boolean comparisons.
Therefore, we can combine this with some short-cut binary logic functions to great effect:
- `ls myfile.sh &&  rm myfile.sh` will firstly test the truth of the left expression (try to locate a file using ls), and **only** if successful (exit code 0) it will then test the right expression (removing the file)
- `ls myfile.sh || echo "file not found"` will test (execute) the right expression only if the left expression is evaluated to be false

If you are writing your own bash scripts then you can set the exit code simply by using `exit 0` or `exit 1` (or whatever return code you want to use).


### Example bash script
The following pair of scripts `sfind.sh` and `aegean.sh` act as a wrapper around the `aegean` and `BANE` programs that are provided within the `aegean_main.sif` singularity container.

> ## sfind.sh
> This is the (template) job script that will be run by SLURM.
> It contains a basic set of SBATCH directives and two variables `base`/`image` that will be modified by another script.
> The user will not interact with this script directly.
> ~~~
> #! /bin/bash -l
> #SBATCH --export=NONE
> #SBATCH --time=10:00:00
> #SBATCH --nodes=1
> 
> base=BASEDIR
> image=IMAGE
> 
> # load singularity
> module load apptainer/latest
> # list my loaded modules (for debugging)
> module list
> 
> # define my container setup
> cont="singularity run -B $PWD:$PWD aegean_main.sif"
> # define some macros that run aegean/BANE from within the container
> aegean="${cont} aegean"
> BANE="${cont} BANE"
> 
> # Print commands and their arguments as they are executed (for debugging)
> set -x
> 
> # start a code block
> {
> 
> # move into the working directory
> cd ${base}
> 
> # look for the background/noise files
> if [ ! -e "${image%.fits}_bkg.fits" ] # Use the bash string substitution syntax
> then
>     ${BANE} ${image}
> fi
> 
> # process the image
> ${aegean} --autoload ${image} --table out.fits,out.csv
> 
> # for this code block:
> #  redirect STDERR/STDOUT to a subprocess that will prepend all lines
> #  with a date and time, and then redirect back to STDERR/STDOUT
> } 2> >(awk '{print strftime("%F %T")";",$0; fflush()}' >&2) \
>   1> >(awk '{print strftime("%F %T")";",$0; fflush()}')
> ~~~
> {: .language-bash}
>
{: .solution}

> ## aegean.sh
> This is the script that the user will interact with.
> It has a basic command line interface.
> ~~~
> #! /bin/bash
> function usage()
> {
> echo "obs_sfind.sh [-g group] [-d dep] [-q queue] [-M cluster] [-t] image
>   -g group   : group (account) to run as, default=oz983
>   -d dep     : job number for dependency (afterok)
>   -q queue   : job queue, default=<blank>
>   -t         : test. Don't submit job, just make the batch file
>                and then return the submission command
>   image      : the image to process" 1>&2;
> # ^ we send the help documentation to STDERR with 1>&2
> exit 1;
> # exit with status 1 (not ok)
> }
> 
> #default values for my arguments
> account="#SBATCH --account oz983"
> depend=''
> queue=''
> tst=
> extras=''
> 
> 
> # Parse option arguments.
> #    
> # Getopts is used by shell procedures to parse positional parameters
> # as options.
> # 
> # OPTSTRING contains the option letters to be recognised; if a letter
> # is followed by a colon, the option is expected to have an argument,
> # which should be separated from it by white space.
> while getopts 'g:d:q:M:t' OPTION
> do
>     case "$OPTION" in
> 	g)
> 	    account="#SBATCH --account ${OPTARG}"
> 	    ;;
> 	d)
> 	    depend="#SBATCH --dependency=afterok:${OPTARG}"
> 	    ;;
> 	q)
> 	    queue="#SBATCH -p ${OPTARG}"
> 	    ;;
> 	t)
> 	    tst=1
> 	    ;;
> 	? | : | h)
> 	    usage
> 	    ;;
>   esac
> done
> # set the obsid to be the last argument provided
> # by renaming the positional parameters
> shift  "$(($OPTIND -1))"
> image=$1
> 
> # Treat unset variables as an error when substituting.
> # Catches silly errors earlier on
> set -u
> 
> # if obsid is empty then just print help
> if [[ -z ${image} ]]
> then
>     usage
> fi
> 
> 
> # The working directory for our script
> base='/fred/oz983/phancock/'
> # The name of the script that we'll create and the location of the log files
> script="${base}queue/sfind_${image%.fits}.sh"
> output="${base}queue/logs/sfind_${image%.fits}.o%A"
> error="${base}queue/logs/sfind_${image%.fits}.e%A"
> 
> # build the sbatch header directives
> sbatch="#SBATCH --output=${output}\n#SBATCH --error=${error}\n${queue}\n${account}\n$> {depend}"
> 
>  # replace IMAGE and BASEDIR
> cat sfind.sh | sed -e "s:IMAGE:${image}:g" \
>                    -e "s:BASEDIR:${base}:g"  \
>                    -e "0,/#! .*/a ${sbatch}" > ${script}
> # ^appeend ${sbatch} after the first line matching the given pattern
> 
> # job invocation command, with a pause of 15s before the job starts
> sub="sbatch --begin=now+15 ${script}"
> 
> # if tst is not zero then
> # return the location of the script and the submission command
> if [[ ! -z ${tst} ]]
> then
>     echo "script is ${script}"
>     echo "submit via:"
>     echo "${sub}"
>     exit 0
> fi
> 
> # submit job and capture the output as a list
> jobid=($(${sub}))
> # output looks like "Submitted batch job 0123456"
> jobid=${jobid[3]}
> 
> # rename the err/output files as we now know the jobid
> error=$( echo ${error} | sed "s/%A/${jobid}/" )
> output=$( echo ${output} | sed "s/%A/${jobid}/" )
> 
> echo "Submitted ${script} as ${jobid}"
> echo "STDOUT is looged to ${output}"
> echo "STDERR is logged to ${error}"
> ~~~
> {: .language-bash}
>
{: .solution}

## WCSTools
According to the [README](http://tdc-www.harvard.edu/software/wcstools/wcstools.readme.html)
```
WCSTools is a set of software utilities, written in C, which create,
display and manipulate the world coordinate system of a FITS or IRAF
image, using specific keywords in the image header which relate pixel
position within the image to position on the sky.  Auxillary programs
search star catalogs and manipulate images.
```
{: .output}

WCSTools is not installed by default but you can download a copy from [http://tdc-www.harvard.edu/wcstools/](http://tdc-www.harvard.edu/wcstools/).
On ubuntu you can also install via `sudo apt install wcstools`.

Once again we have a powerful and many featured tool, of which a few functions are extremely useful, a few are available elsewhere, and some probably you'll never use.
I note this package here because it provides this functionality via the command line, making it easy (er) to write bash scripts that do smart things with fits files.

The functions which I find to be the most useful are:

| command | description                                                         |
| ------- | ------------------------------------------------------------------- |
| getfits | Extract portion of a FITS file into a new FITS file, preserving WCS |
| gethead | Return values for keyword(s) specified after filename               |
| sethead | Set header keyword values in FITS or IRAF images                    |
| getpix  | Return value(s) of specified pixel(s)                               |
| setpix  | Set specified pixel(s) to specified value(s)                        |
| gettab  | Extract values from tab table data base files                       |
| imhead  | Print FITS or IRAF header                                           |
| sky2xy  | Print image pixel coordinates for given sky coordinates             |
| xy2sky  | Print sky coordinates for given image pixel coordinates             |
| skycoor | Convert between J2000, B1950, galactic, and ecliptic coordinates    |

So for example I can write a script that will look at the header of a fits file and figure out how large the image is using:
```
imfile=1904-66_SIN.fits
xy=($(gethead ${imfile} NAXIS1 NAXIS2))
x=${xy[0]}
y=${xy[1]}
echo "File ${imfile} is ${x} by ${y} pixels in size"
```
{: .language-bash}
