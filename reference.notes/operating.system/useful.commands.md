
#### Bash command 반복 실행
$ watch -n0 _command_

#### JSON to CSV
$ find . -type f -name "*.log" -exec cat {} \; | grep ANDROID_UNIVERSAL_6_6_0 | jq -r '.di.cookies as $cookies | ([$cookies.Start_Time, $cookies.au_id, $cookies.IP_info, .di.ids.UNIVERSAL_6_6_0.ANDROID_UNIVERSAL_6_6_0, .di.headers["user-agent"]] | @csv)' > adtruth_log1.csv

