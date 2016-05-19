# alp

alp is Access Log Profiler for Labeled Tab-separated Values(LTSV).  
(See: [Labeled Tab-separated Values](http://ltsv.org))

# Installation

```
curl -sLO https://github.com/tkuchiki/alp/releases/download/VERSION/alp_linux_amd64.zip
unzip alp_linux_amd64.zip
mv alp_linux_amd64 /usr/local/bin/alp
```

# Usage

Read from stdin or an input file(`-f`).  

```
$ ./alp --help
usage: alp [<flags>]

Access Log Profiler for LTSV (read from file or stdin).

Flags:
      --help                     Show context-sensitive help (also try --help-long and --help-man).
  -c, --config=CONFIG            config file
  -f, --file=FILE                access log file
  -d, --dump=DUMP                dump profile data
  -l, --load=LOAD                load profile data
      --max                      sort by max response time
      --min                      sort by min response time
      --avg                      sort by avg response time
      --sum                      sort by sum response time
      --cnt                      sort by count
      --uri                      sort by uri
      --method                   sort by method
      --max-body                 sort by max body size
      --min-body                 sort by min body size
      --avg-body                 sort by avg body size
      --sum-body                 sort by sum body size
      --p1                       sort by 1 percentail response time
      --p50                      sort by 50 percentail response time
      --p99                      sort by 99 percentail response time
      --stddev                   sort by standard deviation response time
  -r, --reverse                  reverse the result of comparisons
  -q, --query-string             include query string
      --tsv                      tsv format (default: table)
      --apptime-label="apptime"  apptime label
      --size-label="size"        size label
      --method-label="method"    method label
      --uri-label="uri"          uri label
      --time-label="time"        time label
      --location=LOCATION        location name
      --limit=5000               set an upper limit of the target uri
      --includes=PATTERN,...     don't exclude uri matching PATTERN (comma separated)
      --excludes=PATTERN,...     exclude uri matching PATTERN (comma separated)
      --noheaders                print no header line at all (only --tsv)
      --aggregates=PATTERN,...   aggregate uri matching PATTERN (comma separated)
      --start-time=TIME          since the start time
      --end-time=TIME            end time earlier
      --start-time-duration=TIME_DURATION
                                 since the start time (now - time.Duration)
      --end-time-duration=TIME_DURATION
                                 end time earlier (now - time.Duration)
      --version                  Show application version.
```

## Log format

See "Labels for Web server's Log" of http://ltsv.org .

### Apache

```
LogFormat "time:%t\tforwardedfor:%{X-Forwarded-For}i\thost:%h\treq:%r\tstatus:%>s\tmethod:%m\turi:%U%q\tsize:%B\treferer:%{Referer}i\tua:%{User-Agent}i\treqtime_microsec:%D\tapptime:%D\tcache:%{X-Cache}o\truntime:%{X-Runtime}o\tvhost:%{Host}i" ltsv
```

### Nginx

```
log_format ltsv "time:$time_local"
                "\thost:$remote_addr"
                "\tforwardedfor:$http_x_forwarded_for"
                "\treq:$request"
                "\tstatus:$status"
                "\tmethod:$request_method"
                "\turi:$request_uri"
                "\tsize:$body_bytes_sent"
                "\treferer:$http_referer"
                "\tua:$http_user_agent"
                "\treqtime:$request_time"
                "\tcache:$upstream_http_x_cache"
                "\truntime:$upstream_http_x_runtime"
                "\tapptime:$upstream_response_time"
                "\tvhost:$host";
```

## Config

[sample config file](./example/config.yml)

## Dump/Load profile data

```
$ cat access.log | ./alp --dump profile.dat
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 | 0.057 | 0.057 | 0.100 |  0.065 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp --load profile.dat
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 | 0.057 | 0.057 | 0.100 |  0.065 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp --load profile.dat -r
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 | 0.057 | 0.057 | 0.100 |  0.065 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
```

## Sample

[sample log file](./access.log)

### Basic

```
$ cat access.log | ./alp
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 | 0.057 | 0.057 | 0.100 |  0.065 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access.log
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 | 0.057 | 0.057 | 0.100 |  0.065 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access.log -r
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 | 0.057 | 0.057 | 0.100 |  0.065 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access.log -q
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-----------------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |             URI             |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-----------------------------+
| 2     | 0.057 | 0.057 | 0.114 | 0.057 | 0.057 | 0.057 | 0.057 |  0.000 |    12.000 |    12.000 |    24.000 |    12.000 | POST   | /foo/bar?token=xxx&uuid=xxx |
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar?token=xxx          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234?          |
| 3     | 0.100 | 0.234 | 0.434 | 0.145 | 0.100 | 0.100 | 0.100 |  0.063 |    34.000 |    34.000 |   102.000 |    34.000 | POST   | /foo/bar?token=xxx          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo?id=xxx           |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678?          |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-----------------------------+

$ ./alp -f access.log -q -r
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-----------------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  |  P1   |  P50  |  P99  | STDDEV | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |             URI             |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-----------------------------+
| 1     | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 | 0.432 |  0.000 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678?          |
| 3     | 0.100 | 0.234 | 0.434 | 0.145 | 0.100 | 0.100 | 0.100 |  0.063 |    34.000 |    34.000 |   102.000 |    34.000 | POST   | /foo/bar?token=xxx          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 | 0.234 |  0.000 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo?id=xxx           |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 | 0.135 |  0.000 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234?          |
| 2     | 0.123 | 0.123 | 0.246 | 0.123 | 0.123 | 0.123 | 0.123 |  0.000 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar?token=xxx          |
| 2     | 0.057 | 0.057 | 0.114 | 0.057 | 0.057 | 0.057 | 0.057 |  0.000 |    12.000 |    12.000 |    24.000 |    12.000 | POST   | /foo/bar?token=xxx&uuid=xxx |
+-------+-------+-------+-------+-------+-------+-------+-------+--------+-----------+-----------+-----------+-----------+--------+-----------------------------+
```

### TSV

```
$ ./alp -f access.log --tsv
Count	Min	Max	Sum	Avg	Max(Body)	Min(Body)	Sum(Body)	Avg(Body)	Method	Uri
1	0.123	0.123	0.123	0.123	56	56	56	56	GET	/foo/bar
3	0.057	0.234	0.391	0.130	12	34	80	26.667	POST	/foo/bar

$ ./alp -f access.log --tsv --noheaders
1	0.123	0.123	0.123	0.123	56	56	56	56	GET	/foo/bar
3	0.057	0.234	0.391	0.130	12	34	80	26.667	POST	/foo/bar
```

### Include

```
$ ./alp -f access.log
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access.log --includes "foo,\d+"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
```

### Exclude

```
$ ./alp -f access.log
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access.log --excludes "foo,\d+"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |    URI     |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
```

### Aggregate

```
$ ./alp -f access.log
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar          |
| 1     | 0.135 | 0.135 | 0.135 | 0.135 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar          |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo        |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access.log --aggregates "/diary/entry/\d+"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |       URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------------+
| 2     | 0.123 | 0.123 | 0.246 | 0.123 |    56.000 |    56.000 |   112.000 |    56.000 | GET    | /foo/bar         |
| 5     | 0.057 | 0.234 | 0.548 | 0.110 |    12.000 |    34.000 |   126.000 |    25.200 | POST   | /foo/bar         |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo       |
| 2     | 0.135 | 0.432 | 0.567 | 0.283 |    15.000 |    30.000 |    45.000 |    22.500 | GET    | /diary/entry/\d+ |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------------+
```

### Time

[sample log file](./access2.log)

```
$ ./alp -f access2.log --start-time "2015-10-28T11:54:39+09:00"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| 1     | 0.135 | 0.135 | 0.135 | 0.135 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /foo/bar          |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access2.log --end-time "2015-10-28 11:45:39+09:00"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+----------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |   URI    |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+----------+
| 1     | 0.123 | 0.123 | 0.123 | 0.123 |    56.000 |    56.000 |    56.000 |    56.000 | GET    | /foo/bar |
| 3     | 0.057 | 0.234 | 0.391 | 0.130 |    12.000 |    34.000 |    80.000 |    26.667 | POST   | /foo/bar |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+----------+

$ ./alp -f access2.log --start-time "2015-10-28T11:45:39+09:00" --end-time "2015-10-28 11:55:39+09:00"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |    URI     |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| 2     | 0.057 | 0.100 | 0.157 | 0.079 |    12.000 |    34.000 |    46.000 |    23.000 | POST   | /foo/bar   |
| 1     | 0.123 | 0.123 | 0.123 | 0.123 |    56.000 |    56.000 |    56.000 |    56.000 | GET    | /foo/bar   |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+

# current date = 2015-10-28, local timezone: +09:00
$ ./alp -f access2.log --start-time "11:45:39" --end-time "11:55:39"
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |    URI     |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| 2     | 0.057 | 0.100 | 0.157 | 0.079 |    12.000 |    34.000 |    46.000 |    23.000 | POST   | /foo/bar   |
| 1     | 0.123 | 0.123 | 0.123 | 0.123 |    56.000 |    56.000 |    56.000 |    56.000 | GET    | /foo/bar   |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+

# current date = 2015-10-28, local timezone: +00:00
$ ./alp -f access2.log --start-time "11:45:39" --end-time "11:55:39" --location JST
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |    URI     |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+
| 2     | 0.057 | 0.100 | 0.157 | 0.079 |    12.000 |    34.000 |    46.000 |    23.000 | POST   | /foo/bar   |
| 1     | 0.123 | 0.123 | 0.123 | 0.123 |    56.000 |    56.000 |    56.000 |    56.000 | GET    | /foo/bar   |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    34.000 |    34.000 |    34.000 |    34.000 | POST   | /hoge/piyo |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+------------+

```

#### Duration

Use time.Duration.  
Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

```
$ LANG=C date
Wed Oct 28 11:55:39 JST 2015

$ ./alp -f access2.log --start-time-duration 1m # from 2015-10-28T11:54:39+09:00
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |        URI        |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+
| 1     | 0.135 | 0.135 | 0.135 | 0.135 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /diary/entry/1234 |
| 1     | 0.234 | 0.234 | 0.234 | 0.234 |    15.000 |    15.000 |    15.000 |    15.000 | GET    | /foo/bar          |
| 1     | 0.432 | 0.432 | 0.432 | 0.432 |    30.000 |    30.000 |    30.000 |    30.000 | GET    | /diary/entry/5678 |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+-------------------+

$ ./alp -f access2.log --end-time-duration 10m # to 2015-10-28T11:45:39+09:00
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+----------+
| COUNT |  MIN  |  MAX  |  SUM  |  AVG  | MAX(BODY) | MIN(BODY) | SUM(BODY) | AVG(BODY) | METHOD |   URI    |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+----------+
| 1     | 0.123 | 0.123 | 0.123 | 0.123 |    56.000 |    56.000 |    56.000 |    56.000 | GET    | /foo/bar |
| 3     | 0.057 | 0.234 | 0.391 | 0.130 |    12.000 |    34.000 |    80.000 |    26.667 | POST   | /foo/bar |
+-------+-------+-------+-------+-------+-----------+-----------+-----------+-----------+--------+----------+
```
