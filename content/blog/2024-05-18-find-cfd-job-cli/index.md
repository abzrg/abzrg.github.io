+++
title = 'How I Look for CFD Jobs from the Command Line'
+++

I regularly check [CFD-Online PhD Jobs](https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship), but the site lacks RSS or email alerts. To automate job checking, I use a shell script that fetches and parses the latest listings.

### Quick One-Liner

```sh
curl -sL 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' | perl -0777 -nle 'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=\d+">(.*?)<\/a>.*?<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' | head -10
```

### Shell Function

```sh
getcfdjobs() {
  cnt=${1:-10}
  curl -sL 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' \
  | perl -0777 -nle 'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=\d+">(.*?)<\/a>.*?<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' \
  | head -$cnt
}
```

### Job Change Checker

```sh
getcfdjobs 100 > $HOME/.cfdjobs
diff <(getcfdjobs 1) <(head -1 $HOME/.cfdjobs) && echo "No new job" || echo "New job detected"
```

For POSIX shells without `<()`, use `mktemp`.

### Notifier Script (`checkcfdjobs`)

```sh
#!/bin/sh -e

notify() {
  case $(uname) in
    Linux) notify-send "[CFD-Online] New job!" ;;
    Darwin) osascript -e 'display notification "New job!" with title "CFD-Online"' ;;
    *) echo "Unsupported OS" && exit 1 ;;
  esac
}

getcfdjobs() {
  cnt=${1:-10}
  curl -sL 'https://www.cfd-online.com/Jobs/listjobs.php?category=PhD%20Studentship' \
  | perl -0777 -nle 'my $i = 1; print "$i. $1 ($2; $3)\n" and $i++ while /<a href="showjob\.php\?record_id=\d+">(.*?)<\/a>.*?<I>(.*?)<\/I><br>\s*(.*?)<br>/gs' \
  | head -$cnt
}

tmp_old=$(mktemp)
tmp_new=$(mktemp)
trap 'rm -f $tmp_old $tmp_new' EXIT

head -1 "$HOME/.cfdjobs" > $tmp_old
getcfdjobs 100 | tee "$HOME/.cfdjobs" | head -1 > $tmp_new

diff $tmp_old $tmp_new || notify
```

### Automation

Schedule with `cron`:

```cron
0 */3 * * * /absolute/path/to/checkcfdjobs
```
