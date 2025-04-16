# web-CLI-browser
Currently a browser-based frontend for browsing my personal website via CLI, written as a single-page HTML and javascript. Eventually may become a slightly-to-moderately-fledged browser-based CLI web browser.

This README is much less frequently updated than the code. Please see the code or run "help" in the browser CLI for more up-to-date information.

If you host this on your server, code runs an update check when it starts and lets you know if this repo has been updated. 

Available commands in cli-html:
- display                    : Display the currently loaded page
- links                      : Show all links in the currently loaded page
- follow [num]               : Follow a link by its number
- back                       : Go back to the previous page
- forward                    : Go forward to the next page
- reload                     : Reload last loaded page
- hint                      : A mysterious command, I don't know what it does.
- versions                   : Version history/changelog of this browser
- clear                      : Clear the terminal
- exit                       : Return to GUI browser
- [command1] | [command2]    : Redirect output into another command such as 'grep' or 'more'
- [command] > filename.html  : Redirect output to a file download in accordance with your browser's settings, rather than onscreen
- grep [pattern]             : Use after pipe (|) to filter output by regular expression. Case-insensitive
- more                       : Use after pipe (|) to paginate output. [space] to continue. Esc, ctrl-c, or q to quit
- help                       : Show this help message
- options [parameter [value]] : Manually set URL parameters (given below) from the command line. Omit value to display current value, omit both to see all current values

Optional startup URL paramaters (starred value is default used when none specified):
- language [cli-html*|unix|basic|piglatin]    : Change commands to match various platforms. This way you unix geeks don't have to fight your natural urge to type 'ls'
- prompt  [unix*|pet|apple]                   : The style of command prompt
- crt [bw|green|amber]                        : Choose between popular CRT colors. There may be some others available
- forceuppercase [false*|true]                : Force all uppercase text for that vintage 1980 personal computer experience. (You can tell I'm a nice guy because I defaulted this to "false".)

