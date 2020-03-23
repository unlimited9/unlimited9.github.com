# JVM Memory Analyze

$ ps -ef | grep java  
> $ sudo netstat -ntlp

## jstat
$ jstat -h
```
Usage: jstat --help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as 
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
  -? -h --help  Prints this help message.
  -help         Prints this help message.
```
$ jstat -gcutil [pid] 1000

항목 | 설명
-- | --
S0 | Survivor 0 영역의 사용량
S1 | Survivor 1 영역의 사용량
E | Eden 영역의 사용량
O | Old 영역의 사용량
M | Metaspace 영역의 사용량
CCS | Compressed Class 영역의 사용량
YGC | Young 영역의 GC 횟수
YGCT | Young 영역의 GC 누적시간
FGC | Full GC 횟수
FGCT | Full GC 누적시간
GCT | 전체 GC 누적시간




## jmap
$ jmap -h
```
Usage:
    jmap -clstats <pid>
        to connect to running process and print class loader statistics
    jmap -finalizerinfo <pid>
        to connect to running process and print information on objects awaiting finalization
    jmap -histo[:live] <pid>
        to connect to running process and print histogram of java object heap
        if the "live" suboption is specified, only count live objects
    jmap -dump:<dump-options> <pid>
        to connect to running process and dump java heap
    jmap -? -h --help
        to print this help message

    dump-options:
      live         dump only live objects; if not specified,
                   all objects in the heap are dumped.
      format=b     binary format
      file=<file>  dump heap to <file>

    Example: jmap -dump:live,format=b,file=heap.bin <pid>
```

## jcmd
$ jcmd -h
```
Usage: jcmd <pid | main class> <command ...|PerfCounter.print|-f file>
   or: jcmd -l                                                    
   or: jcmd -h                                                    
                                                                  
  command must be a valid jcmd command for the selected jvm.      
  Use the command "help" to see which commands are available.   
  If the pid is 0, commands will be sent to all Java processes.   
  The main class argument will be used to match (either partially 
  or fully) the class used to start Java.                         
  If no options are given, lists Java processes (same as -l).     
                                                                  
  PerfCounter.print display the counters exposed by this process  
  -f  read and execute commands from the file                     
  -l  list JVM processes on the local machine                     
  -? -h --help print this help message                            
```
> add jvm option : -XX:NativeMemoryTracking=summary  
> jcmd <pid> VM.native_memory baseline  
> jcmd <pid> VM.native_memory summary.diff


## 9. Appendix

#### reference site

+ JAVA: Debugging a permgen leak    
https://gist.github.com/JDatta/7f82db25ccef7772a0e73fe9ceb329d7
