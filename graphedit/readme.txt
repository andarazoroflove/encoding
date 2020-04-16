To install simply double click on register bat. If it comes up with some errors search for regsvr32.exe
and put the full path of that file before every call to regsvr32 in the batch file and run it again.


Sysenum is a tool that helps you unregister troublesome directshow codecs.

To do this, run GraphEdit, then render a media file that uses the
troublesome codec.  This tells you the codec's human-readable name. 
(E.g. "Mediamatics MPEG-2 Splitter")

Then, run SysEnum, go to the "DirectShow Filters" section, scroll in the
list at right till you find the codec's name, click on it, and down at
the bottom of the window the filename shows up.  Now you can "regsvr32
/u" it.
