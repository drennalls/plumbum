Plumbum: Shell Combinators
==========================

Ever wished the compactness of shell scripts be put into a **real** programming language? 
Say hello to *Plumbum Shell Combinators*. Plumbum (Latin for *lead*, which was used to create 
pipes back in the day) is a small yet feature-rich library for shell script-like programs in Python. 
The motto of the library is **"Never write shell scripts again"**, and thus it attempts to mimic 
the **shell syntax** ("shell combinators") where it makes sense, while keeping it all **Pythonic 
and cross-platform**.

Apart from shell-like syntax and handy shortcuts, the library provides local and remote command 
execution (over SSH), local and remote file-system paths, easy working-directory and environment 
manipulation, and a programmatic Command-Line Interface (CLI) application toolkit. 
Now let's see some code!

*This is only a teaser; the full documentation can be found at*
`Read the Docs <http://plumbum.readthedocs.org>`_

Cheat Sheet
-----------
**Basics** ::

    >>> from plumbum import local
    >>> ls = local["ls"]
    >>> ls
    LocalCommand(<LocalPath /bin/ls>)
    >>> ls()
    u'build.py\ndist\ndocs\nLICENSE\nplumbum\nREADME.rst\nsetup.py\ntests\ntodo.txt\n'
    >>> notepad = local["c:\\windows\\notepad.exe"]
    >>> notepad()                                   # Notepad window pops up
    u''                                             # Notepad window is closed by user, command returns

Instead of writing ``xxx = local["xxx"]`` for every program you wish to use, you can 
also ``import`` commands::
    
    >>> from plumbum.cmd import grep, wc, cat, head
    >>> grep
    LocalCommand(<LocalPath /bin/grep>)

**Piping** ::
    
    >>> chain = ls["-a"] | grep["-v", "\\.py"] | wc["-l"]
    >>> print chain
    /bin/ls -a | /bin/grep -v '\.py' | /usr/bin/wc -l
    >>> chain()
    u'13\n'

**Redirection** ::

    >>> ((cat < "setup.py") | head["-n", 4])()
    u'#!/usr/bin/env python\nimport os\n\ntry:\n'
    >>> (ls["-a"] > "file.list")()
    u''
    >>> (cat["file.list"] | wc["-l"])()
    u'17\n'

**Working-directory manipulation** ::
    
    >>> local.cwd
    <Workdir /home/tomer/workspace/plumbum>
    >>> with local.cwd(local.cwd / "docs"):
    ...     chain()
    ... 
    u'15\n'
    
**Foreground and background execution** ::

    >>> from plumbum import FG, BG
    >>> (ls["-a"] | grep["\\.py"]) & FG         # The output is printed to stdout directly
    build.py
    .pydevproject
    setup.py
    >>> (ls["-a"] | grep["\\.py"]) & BG         # The process runs "in the background"
    <Future ['/bin/grep', '\\.py'] (running)>
    
**Command nesting** ::
    
    >>> from plumbum.cmd import sudo
    >>> print sudo[ifconfig["-a"]]
    /usr/bin/sudo /sbin/ifconfig -a
    >>> (sudo[ifconfig["-a"]] | grep["-i", "loop"]) & FG
    lo        Link encap:Local Loopback  
              UP LOOPBACK RUNNING  MTU:16436  Metric:1

**Remote commands (over SSH)**

Supports `openSSH <http://www.openssh.org/>`_-compatible clients, 
`PuTTY <http://www.chiark.greenend.org.uk/~sgtatham/putty/>`_ (on Windows)
and `Paramiko <https://github.com/paramiko/paramiko/>`_ (a pure-Python implementation of SSH2) ::

    >>> from plumbum import SshMachine
    >>> remote = SshMachine("somehost", user = "john", keyfile = "/path/to/idrsa")
    >>> r_ls = remote["ls"]
    >>> with remote.cwd("/lib"):
    ...     (r_ls | grep["0.so.0"])()
    ... 
    u'libusb-1.0.so.0\nlibusb-1.0.so.0.0.0\n'

**CLI applications** ::

    import logging
    from plumbum import cli
    
    class MyCompiler(cli.Application):
        verbose = cli.Flag(["-v", "--verbose"], help = "Enable verbose mode")
        include_dirs = cli.SwitchAttr("-I", list = True, help = "Specify include directories")
        
        @cli.switch("--loglevel", int)
        def set_log_level(self, level):
            """Sets the log-level of the logger"""
            logging.root.setLevel(level)
        
        def main(self, *srcfiles):
            print "Verbose:", self.verbose
            print "Include dirs:", self.include_dirs 
            print "Compiling:", srcfiles
    
    
    if __name__ == "__main__":
        MyCompiler.run()

Sample output::

    $ python simple_cli.py -v -I foo/bar -Ispam/eggs x.cpp y.cpp z.cpp
    Verbose: True
    Include dirs: ['foo/bar', 'spam/eggs']
    Compiling: ('x.cpp', 'y.cpp', 'z.cpp')



.. image:: https://d2weczhvl823v0.cloudfront.net/tomerfiliba/plumbum/trend.png
   :alt: Bitdeli badge
   :target: https://bitdeli.com/free

