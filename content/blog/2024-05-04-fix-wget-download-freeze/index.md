+++
title = 'Fix wget Download Freezing'
+++

One annoying problem of downloading files using `wget` is that when some network
problem happens, it does not (by default) retry, and the download gets stuck,

```
60%[========================>                ]  71.62M --.-KB/s  eta 7m 43s
```

and I have to `^C` and retry myself manually.

To fix this, `--read-timeout=seconds` comes to the rescue. According to the `wget`'s man page:

>     --read-timeout=seconds
>         Set the read (and write) timeout to seconds seconds.  The
>         "time" of this timeout refers to idle time: if, at any
>         point in the download, no data is received for more than the
>         specified number of seconds, reading fails and the
>         download is restarted.  This option does not directly affect
>         the duration of the entire download.

We can adjust the timeout to a shorter interval, like every `20` seconds. So after 20 seconds of not receiving data, download is failed, and `wget` will automatically restart the download.

```
wget --read-timeout=20 <url>
```

However, we want `wget` to not ignore the previous download progress and start from where it left off. For that, we use the `-c/--continue` option.

```
wget --read-timeout=20 --continue <url>
```

And finally, we want `wget` to try infinitely after each download failure. For this, we specify the `--tries=number` option, whose default `number` is set to `20` tries. We specify `0` (or `inf`) for infinite trying.[^1] So our final command is the following:

```
wget --read-timeout=20 --continue --tries=0 <url>
```

<br>

---

[^1]: Note that it does not retry for fatal errors like "connection refused" or "not found," which is nice.
