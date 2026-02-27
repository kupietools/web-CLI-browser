# web-CLI-browser
Currently a browser-based frontend for browsing my personal website via CLI, written as a single-page javascript app. Eventually may become a slightly-to-moderately-fledged browser-based CLI web browser. See it live at [michaelkupietz.com/cli.html](michaelkupietz.com/cli.html).

This README is much less frequently updated than the code and is probably slightly out-of-date. Please see the code or run "help" in the browser CLI for more up-to-date information.

If you host this on your server, code runs an update check when it starts and lets you know if this repo has been updated. 

Available cli-html commands. Use 'help commandname' to see more detailed info on a command:

Loading and viewing web content:
- display : Display the currently loaded page
- links : List all links in the loaded page
- images : List all image descriptions and URLs in the loaded page
- image imagenumber : Display an ascii-converted image by image number, in blocks or ascii
- where : Display the URL of the currently loaded page

Navigating between web pages:
- follow linknumber : Follow a link in the current page by its number
- back : Go back to the previously loaded page
- forward : Go forward to the next page
- reload : Reload the currently loaded page

Controlling & filtering output:
- commandname1 | commandname2 : Redirect the output of commandname1 through a second command, such as 'grep' or 'more', before displaying it
- more : Use after pipe (|) to paginate output of previous command. [space] to continue. Esc, ctrl-c, or q to quit
- grep filterexpression : Use after pipe (|) to filter output of previous command by regular expression. Case-insensitive, no delimiters
- commandname > filename.html : Output commandname's output to a file download, rather than onscreen

Controlling this terminal:
- help [commandname] : Show this help message, or detailed help on commandname
- exit : End session and return this browser window to GUI browing mode
- info [news|versions|bugs|todo] : Display the latest news, version history/changelog, known bugs, and the developer's to-do list.
- clear : Clear the terminal output
- reboot : Reboot the terminal with its original startup options
- hint : A mysterious command, I don't know what it does
- beep : Plays the system bell (ctrl-G)
- options [parameter] [value] : Manually change URL parameters (given below) from the command line, or display current value(s). Enter a parameter and value to set that parameter, enter just a parameter to display its current value, or leave both out to list all current parameter values

Optional startup URL paramaters (starred value is the default if unspecified):
- language=[CLI-HTML*|unix|basic|pig-latin] : Change commands to match various platforms. This way you unix geeks don't have to fight your natural urge to type 'ls'
- prompt=[unix|PET*|apple] : Change style of command prompt to match various advanced platforms
- crt=[BW*|green|amber] : Choose between popular CRT colors. There may be some others available
- allcaps=[FALSE*|true] : Force all uppercase text for that vintage 1980 personal computer experience
- nobeep=[FALSE*|true] : [sigh] Yes... you can disable the error beep if you absolutely have to. You're no fun. 
