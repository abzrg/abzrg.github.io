+++
title = 'How I Look for CFD Jobs from the Command Line'
+++

## Introduction

On a daily or weekly basis, I look for Ph.D. studentships on various websites, chief among which being the good old [cfd-online](https://www.cfd-online.com), which mostly posts CFD-related jobs at [this URL](https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship).
The issue with this website is the absence of an email notification feature or an RSS feed, which would otherwise enable me to check for new job postings directly from the comfort of the command line.
This post is my attempt to automate the process of checking for new jobs.

I want to write a script to: 1. List all the recent (let's say 10) jobs, and 2. Check whether any new jobs have been posted.

### Dependencies

For the following commands and scripts to work, we need two programs that may not be pre-installed by default on all Unix operating systems, namely `curl` and `perl`.

```sh
# Debian-based Linux Distros
sudo apt install curl perl

# MacOS
brew install curl perl
```

### Final One-Liner

For those who just want to use the command without delving into the forthcoming details, here’s the one-liner:

```sh
curl --silent --location 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' | perl -0777 -nle 'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=[0-9]{5}">(.*?)<\/a>.*?\s*<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' | head -10
```

## 1. Getting the List of Jobs

I am looking for the following information:

- The title of the position
- The name of the institute or university
- The location of the job

To extract these data, we first retrieve the data:


```sh
curl --silent --location 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' --output jobs.html  # or curl -sL
```

- `-s/--silent`: suppresses the progress information.
- `-L/--location`: tells curl to follow redirects (If the requested URL has moved to a new location, `curl` will follow the HTTP Location headers to fetch the new location).

We can examine the structure of the page. The following section appears somewhere in the middle of the page:

```html
...

<h2>CFD Jobs Database - List of Jobs</h2><p>
<table border=2 cellpadding=5 width=100%>
<H3>Category: PhD Studentship</H3>    <tr>
      <td>
         <b><a href="showjob.php?record_id=19178">PhD in Thermal Management for Future Aerospace Propulsion </a></b><br>
         <I>Heriot Watt University</I><br>
                  United Kingdom, Edinburgh<br>
         <font size=-2 face=helvetica>Record Last Modified 12:43:29 May 17 2024, Closure Date Jul 22 2024</font>
      </td>
      <td align=CENTER>
         <form action="showjob.php" method=GET><input type="hidden" name="record_id" value="19178"><input type="submit" value=" View Full Record ">
         </form>
         <font size=-2 face=helvetica>Read 166 times</font>
    </td>
    </tr>
      <tr>
      <td>
         <b><a href="showjob.php?record_id=19173">Modeling and numerical simulation of parietal heat transfer</a></b><br>
         <I>PROMES-CNRS</I><br>
                  France, Perpignan<br>
         <font size=-2 face=helvetica>Record Last Modified 22:42:21 May 16 2024, Closure Date Not Specified</font>
      </td>
      <td align=CENTER>
         <form action="showjob.php" method=GET><input type="hidden" name="record_id" value="19173"><input type="submit" value=" View Full Record ">
         </form>
         <font size=-2 face=helvetica>Read 184 times</font>
    </td>
    </tr>

...
```

As you can see, jobs are listed in a `<table>`, with each title surrounded by a `b` tag, the name of the institute inside an `I` tag, and the location shortly follows the name of the institution before reaching a `<br>`.

Before we go over how to extract the necessary information, I would like to point you to an extremely useful website, [regex101.com](https://regex101.com), which allows you to experiment with (various dialects of) regex. Here's my regex:

{{ image(src="regex101.png", alt="regex101 experimentation", cap="Regex101 Experimentation") }}

Moving on, the following regex matches the information I need:

```
<a href="showjob\.php\?record_id=[0-9]{5}">(.*?)<\/a>.*?\s*<I>(.*?)<\/I><br>\s*(.*?)<br>
```

Let's break it down step by step:

1. `<a href="showjob\.php\?record_id=[0-9]{5}">`:
   - `<a href="showjob\.php\?record_id=`: Matches the literal string `<a href="showjob.php?record_id=`. The `\?` is used to escape the question mark (`?`), and `\.` is used to escape the dot (`.`), making sure they are treated as literal characters.
   - `[0-9]{5}`: Matches exactly five digits, representing the job record ID.
   - `">`: Matches the closing quote and greater-than sign (`">`) following the record ID.

2. `(.*?)`:
   - `(.*?)`: This is the first capture group. The `.*?` is a non-greedy match that captures any character (except for newline characters) as few times as possible. This capture group extracts __the job title__.

3. `<\/a>`:
   - `<\/a>`: Matches the closing anchor tag (`</a>`). The backslash (`\`) escapes the forward slash (`/`), ensuring it is treated as a literal character.

4. `.*?\s*`:
   - `.*?`: Matches any character (except for newline characters) as few times as possible (non-greedy match). This ensures we match everything between the job title and the institution name.
   - `\s*`: Matches any whitespace characters (including spaces, tabs, and newlines) zero or more times.

5. `<I>(.*?)<\/I>`:
   - `<I>`: Matches the opening italic tag (`<I>`).
   - `(.*?)`: This is the second capture group, capturing __the institution name__ non-greedily.
   - `<\/I>`: Matches the closing italic tag (`</I>`).

6. `<br>\s*(.*?)<br>`:
   - `<br>`: Matches the HTML line break tag (`<br>`).
   - `\s*`: Matches any whitespace characters zero or more times, allowing for any extra spaces between the HTML tags.
   - `(.*?)`: This is the third capture group, capturing __the location__ non-greedily.
   - `<br>`: Matches the closing line break tag (`<br>`).


As shown in the image above, this is a PCRE-2 regex. Perl allows us to extract the data using the pattern above.

```sh
curl --silent --location 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' | perl -0777 -nle 'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=[0-9]{5}">(.*?)<\/a>.*?\s*<I>(.*?)<\/I><br>\s*(.*?)<br>/gs'
```

Let's also dissect it step by step:

1. `perl -0777 -nle '...'`:
   - `-0777`: Enables slurping mode, making Perl read the entire input file as a single string. This is crucial for matching patterns that span multiple lines.
   - `-n`: Wraps the code in an implicit `while (<>) { ... }` loop, iterating over the input file or standard input.
   - `-l`: Handles newline characters by automatically chomping the input line separator and adding it to the `print` statement.
   - `-e`: Allows execution of the provided Perl code directly from the command line.

2. `'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /PATTERN/gs'`:
   - `my $i = 1;`: Initializes a counter variable `$i` to 1. This variable keeps track of the line number for each match.
   - `print "$i. $1 ($2; $3)\n"`:
     - `"$i. $1 ($2; $3)\n"`: Constructs the output string. It prints the current value of the counter followed by the captured groups.
       - `$i`: The current line number.
       - `$1`: The first capture group, which contains the job title.
       - `$2`: The second capture group, which contains the institution name.
       - `$3`: The third capture group, which contains the location.
     - `"\n"`: Adds a newline character at the end of the output string.
   - `and $i++`: Increments the counter variable `$i` after printing. The `and` operator ensures that `$i++` is executed only if the `print` statement is successful.
   - `while /PATTERN/gs`:
     - `while ...`: A loop that continues as long as the regex pattern matches.
     - `/PATTERN/gs`: This is the regex pattern applied globally (`g`) and treating the input as a single string (`s`) (i.e., dot/period matches a newline as well).

Finally, we can pipe it to `head -10` to list the 10 most recent jobs:

```sh
curl --silent --location 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' | perl -0777 -nle 'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=[0-9]{5}">(.*?)<\/a>.*?\s*<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' | head -10
```

Output of the command (date: May 18, 2024):

```
1. PhD in Thermal Management for Future Aerospace Propulsion  (Heriot Watt University; United Kingdom, Edinburgh)
2. Modeling and numerical simulation of parietal heat transfer (PROMES-CNRS; France, Perpignan)
3. PhD studentship - Modelling bubble-particle interactions  (University of Birmingham; United Kingdom, Birmingham)
4. Numerical investigation on casting in ESF (Faculty of Eng and Info Sciences, University of Wollongong; Australia, NSW, Wollongong)
5. PhD within the Centre of Computational Engineering Sciences (Cranfield University; United Kingdom, Bedfordshire, Bedford)
6. Stochastic particle methods for two-phase flows (University of Stuttgart; Germany, Baden-Württemberg, Stuttgart)
7. PhD Candidate in fire safety and fire development in buildings  (Norwegian University of Science and Technology; Norway, Trondheim)
8. Graduate Research assistant for CFD simulations of RDE (University of Texas San Antonio; United States, TX, SAN ANTONIO)
9. Atmospheric Thermal Transport  (University of South Florida; United States, FL, Tampa)
10. Simulations of offshore wind farm turbulence (University of Twente; Netherlands, Overijssel, Enschede)
```

Now, at this stage, we have the option to encapsulate this process within a function or a shell script

```sh
getcfdjobs() {
    # number of recent jobs (by default: 10 jobs)
    cnt=${1:-10}

    curl --silent --location 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' \
    | perl -0777 -nle \
        'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=[0-9]{5}">(.*?)<\/a>.*?\s*<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' \
    | head -$cnt
}
```

With this, one can retrieve, for example, the last 5 jobs:

```
$ getcfdjobs 5
1. PhD in Thermal Management for Future Aerospace Propulsion  (Heriot Watt University; United Kingdom, Edinburgh)
2. Modeling and numerical simulation of parietal heat transfer (PROMES-CNRS; France, Perpignan)
3. PhD studentship - Modelling bubble-particle interactions  (University of Birmingham; United Kingdom, Birmingham)
4. Numerical investigation on casting in ESF (Faculty of Eng and Info Sciences, University of Wollongong; Australia, NSW, Wollongong)
5. PhD within the Centre of Computational Engineering Sciences (Cranfield University; United Kingdom, Bedfordshire, Bedford)
```


## 2. Checking for New Jobs

The second step involves checking if the website has been updated. There are several ways to accomplish this:

1. Checking the hash of the content of the page.
1. Looking for the last update time.
1. Checking if the list of jobs has been changed (or simply if the most recent job has been updated).

The first option isn't suitable here, as the number of visits to jobs is updated constantly regardless of whether jobs themselves have been updated. Between the second and third options, the latter seems more efficient, as it only requires reading the page once. The second option involves first reading the last update time and then reading the jobs.

To implement this, we need to store the jobs in a sort of database file:


```sh
getcfdjobs 100 > $HOME/.cfdjobs
```

The following command informs us whether the page has been updated with new jobs:

```sh
diff <(getcfdjobs 1) <(head -1 $HOME/.cfdjobs) && echo "New job not detected" || echo "No new job"
```

However, the `<()` syntax is not available in simpler and more restricted POSIX-compliant shells like dash (commonly used in many Linux distributions where `/bin/sh` is a symlink to `/usr/bin/dash`). In such shells, we can achieve the same result by doing the following:

```sh
# Make temporary files
trap 'rm -f cfd_jobs_old cfd_jobs_new' EXIT INT TERM
cfd_jobs_old=$(mktemp)
cfd_jobs_new=$(mktemp)

# First get the latest job
head -1 "$HOME/.cfdjobs" > cfd_jobs_old

# Update the list of jobs and get the newest job
getcfdjobs 100 | tee $HOME/.cfdjobs | head -1 > cfd_jobs_new

# Perform diff on the named pipes
if ! diff cfd_jobs_old cfd_jobs_new; then
  echo "New job detected"
else
  echo "No new job"
fi
```


## Conclusion

With all of these components in place, we can now write a script to check and update the jobs and notify when it happens.
The script is named `checkcfdjobs`.[^1]

```sh
#!/bin/sh -e

# file:        checkcfdjobs
# description: Search for jobs on cfd-online.com and notify if new jobs added
# usage:       checkcfdjobs

notify() {
    if [ `uname -s` = "Linux" ]; then
       notify-send "[CFD-Online] New job!"
    elif [ `uname -s` = "Darwin" ]; then
       osascript -e 'display notification "New job!" with title "CFD-Online" sound name "Blow"'
   else
       printf "OS not supported!\n" && exit 1
    fi
}

getcfdjobs() {
    cnt=${1:-10}
    curl --silent --location 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' \
    | perl -0777 -nle \
        'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=[0-9]{5}">(.*?)<\/a>.*?\s*<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' \
    | head -$cnt
}

trap 'rm -f cfd_jobs_old cfd_jobs_new' EXIT INT TERM
cfd_jobs_old=$(mktemp)
cfd_jobs_new=$(mktemp)

head -1 "$HOME/.cfdjobs" > cfd_jobs_old
getcfdjobs 100 | tee $HOME/.cfdjobs | head -1 > cfd_jobs_new

if ! diff cfd_jobs_old cfd_jobs_new; then
  notify
else
  echo "No new job"
fi
```


### Shell startup

Add this line to your shell startup configuration file (`.zshrc` or `.bashrc`):

```sh
checkcfdjobs
```

The script will now execute every time a new shell is spawned.

### Cron-job

Add the following job to your crontab (`crontab -e`):

```cron
# Every 3 hours
0 */3 * * * /absolute/path/to/checkcfdjobs
```

Additionally there are ways to send email from command-line, but I skip discussing that here.

---

[^1]: Put this script somewhere in your `PATH` (usually in `$HOME/bin` or `$HOME/.local/bin`).
