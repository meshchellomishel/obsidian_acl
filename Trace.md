
***

**Maybe useful**

1. ***[add trace into code](https://stackoverflow.com/questions/10374005/how-to-trace-function-call-in-c)***
2. ***[kernel tracing](https://opensource.com/article/21/7/linux-kernel-trace-cmd)***
	list of traceble funtions:
	`sudo trace-cmd list -f | grep nfnetlink`
	trace with filter:
	`sudo trace-cmd record -l nfnetlink* -p function_graph`
	open report:
	`sudo trace-cmd report`
3. ***[trace-cmd-github](https://github.com/rostedt/trace-cmd)***