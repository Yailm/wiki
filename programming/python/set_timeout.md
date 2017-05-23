
set timeout
------------------------------------

    def func(signum, frame):
        pass

    signal.signal(signal.SIGALRM, func)
    signal.alarm(second_to_timeout)
    # some code here
    signal.alarm(0) # cancel
