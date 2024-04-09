---
title: "/dev/null | stdout | stderr | redirections"
categories:
  - blog
tags:
  - linux
  - command-line
---

This is a post regarding `/dev/null` known as the blackhole of linux where everything we sent to is discarded. So, we will be looking into how we can discard the unnecessary messages from `stdout` and `stderr` during command execution that clutter the terminal. Also, to separate the streams into independent files.

## Prerequisites:

- Basic linux knowledge
- Different streams in linux i.e `stdin`, `stdout`, `stderr`
- Redirection

## Explanation:

`stdout` is the place where all the normal output goes. That is the same case for the `stderr`. If it's not caught or redirected, by default, they
go to the terminal screen.

# Streams

```
0 -> Standard Input (stdin)
1 -> Standard Output (stdout)
2 -> Standard Error (stderr)
```

Let us take an example of a script:

```
#!/bin/bash
function get_latest_version() {
	echo "Resolving Host" 1>&2
	echo "Starting connection" 1>&2
	echo "Downloading JSON" 1>&2
	echo "Extracting version"
	echo "3.2.28"
	echo "Linux Redirection Test"
}

# 1 print everything both stdout as well as sterr
echo "the current version of foo is $(get_latest_version)"

# 2 redirect stdout stream to stderr stream
echo "the current version of foo is $(get_latest_version 1>&2 )"

# 3 redirect stdout stream to /dev/null | only print stderr
echo "the current version of foo is $(get_latest_version 1>/dev/null)"

# 4 redirect stderr stream to /dev/null | only print stdout
echo "the current version of foo is $(get_latest_version 2>/dev/null)"

# 5 redirect stdout stream to output_log.txt file and stderr stream to error_log.txt file
echo "the current version of foo is $(get_latest_version 2>error_log.txt 1>output_log.txt)"


## 6 watch the order of streams being redirected
# 1 redirect stderr to /dev/null and stdout to stderr resulting in both streams being redirected to /dev/null
echo "the current version of foo is $(get_latest_version 2>/dev/null 1>&2)"

# 2 redirect stdout to stderr | at this point stderr is pointing to its default destination so stdout is printed then finally stderr is then redirected to /dev/null
echo "the current version of foo is $(get_latest_version 1>&2 2>/dev/null)"

```

You can try this example by cloning from [github repo][github-linux-redirection-link]

# Script Breakdown

So, we have got a function `get_latest_version` which contains a lot of echo commands. By default, the `echo` command prints to the stdout stream and we see the result in the terminal. On contrast, when we use `1>&2`, it redirects the stdout stream to the error stream. So, the function `get_latest_version` has both `stdout` as well as `stderr` streams to be precise 3 each.

As mentioned previously, by default both `stdout` and `stderr` streams are outputed to the terminal. So,

1. After the `#1` command execution, everything is outputed to the screen both the `stdout` and `stderr` stream.

```
Resolving Host                                         ->2
Starting connection                                    ->2
Downloading JSON                                       ->2
the current version of foo is Extracting version       ->1
3.2.28                                                 ->1
Linux Redirection Test                                 ->1
```

2. After the `#2`, `stdout` stream is redirected to `stderr` and finally `stderr` is outputed to the terminal

```
Resolving Host                  ->2
Starting connection             ->2
Downloading JSON                ->2
Extracting version              ->1
3.2.28                          ->1
Linux Redirection Test          ->1
the current version of foo is
```

We can see in both cases, everything is outputed in the terminal. Having said that I still have confusion regarding why there is difference in order of echos `the current version of foo is`being printed in the terminal.

3. After the `#3`, `stdout` is redirected to `/dev/null` so, only the `stderr` is outputed.

```
Resolving Host          ->2
Starting connection     ->2
Downloading JSON        ->2
the current version of foo is
```

4. After the `#4`, `stderr` is redirected to `/dev/null` so, only the `stdout` is outputed.

```
the current version of foo is Extracting version    ->1
3.2.28                                              ->1
Linux Redirection Test                              ->1
```

5. After the `#5`, `stdout` is redirected to file `output_log.txt` and `stderr` is redirected to file `error_log.txt`.
   This enables us to view output and error in their respective files making debugging easier.

```
the current version of foo is
```

6. 1. After the `#6.1`, first `stderr` is redirected to `/dev/null` and then, `stdout` is redirected to `stderr`. This enables redirecting both the `stdout` and `stderr` streams to `/dev/null`.

```
the current version of foo is
```

This can also be used to redirect both `stdout` and `stderr` in the same file.

```
echo "the current version of foo is $(get_latest_version 2>log.txt 1>&2)"
```

6. 2. After the `#6.2`, here notice the execution order of `2>/dev/null` and `1>&2` is reversed. You may be thinking they are almost similar and this should also give same result as `#6.1`. But that is not the case. Here, `1>&2` redirects `stdout` to where `stderr` is set to at that point and at that point you haven't redirected `stderr` yet. So, `stdout` outputs to stderr's default direction which is `terminal`. Then, later you redirect stderr to `/dev/null`. So,`stdout` is printed in the terminal and only the `stderr` is discarded to `/dev/null`.

```
Extracting version              ->1
3.2.28                          ->1
Linux Redirection Test          ->1
the current version of foo is
```

## Key TakeWays:

- `stdout` and `stderr` streams can be separated into independent files enabling easier debugging of issues.
- `stdout` and `stderr` streams can be discarded to the `/dev/null` to declutter the terminal output during script execution`

## To Be Explored

We can see the output of `the current version of foo is` being at different places. Sometimes, at the top, at the middle and at the bottom, find out why does this happen.

[github-linux-redirection-link]: https://github.com/xerox007/linux-playground/blob/master/linux_redirection.sh
