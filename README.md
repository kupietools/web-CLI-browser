<!DOCTYPE html>
<html>
<!-- Thanks for your curiosity, ya big ol' geek! CLI browsing interface by Michael Kupietz -->
<!-- Much thanks to the Indieweb Homebrew Website Club Europe/London for much inspiration and input -->

<!-- the master version of this is hosted at https://github.com/kupietools/web-CLI-browser -->

<!-- Copyright (C) 2025  Michael E Kupietz. 

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/gpl-3.0-standalone.html>.

-->

<!-- Contact: 
     CLI@michaelkupietz.com
     Creative arts & tech showcase: https://michaelkupietz.com 
     Github: https://github.com/kupietools 
     Consulting: https://www.kupietz.com

Seeking work as of this writing. Somebody please offer me a job. 

-->

<head>
<script id="global-settings">

/* some user-settable defaults */

let versionhistory=`
Version History:
2025apr16 - add optional version checking against Github master
2025apr15 - change css to use variables and add commandOut to add hilight color for commands, allow options to accept "default" as value, Make 'pet' default prompt. Fix regression error where PET prompt wasn't putting cursor on next line. Fix terminal padding so LPT crt looks nice. Put a little dashicons globe next to the "loaded' message but then commented it out.
2025apr14 - fix regression error introduced by using ';' instead of ',' in arrays due to terrible habits reinforced by FileMaker, refactor how commands are executed to begin allowing inclusion of manpages and other meta information, get manpages going, cause click on terminal to focus command line element, minor css color changes. 
2025apr13 - add scaling to image command so ascii art doesn't wrap, add -a image option, fix event handlers so ArrowUp/ArrowDown position block cursor at end of line, add image scaling readout, add clear command
2025apr12 - update page title to reflect loaded page, add undocumented feature to let Display load other URLs or follow links, add dashicon, add redone image placeholders with saved image array and "images" commands, add image display commands
2025apr11 - add code to move cursor with arrows and clicks in command line
2025apr10 - for consistency, have it stop showing page automatically upon load and only use Display to view, add some fun display options, better formatting for help, handle invalid language selections, added a few more languages for fun. 
2025apr9 - add '>' download redirect, versions command, URL parameter settings for prompts/languages/force all caps/other secret sauce, add 'option' command to manipulate settings, add message to confirm page has loaded, add case insensitivitinositude to commands.
2025apr8 - first working version

Known bugs:
1.) if you scroll back the history and enter a new comment, history index doesn't reset to the end.
2.) Doesn't clear the image array if you explicitly load a new page
3.) Some options don't throw errors if you give them bad values, they're just ignored.

To Dos:
1.) add -i flag to make image list show images. 
2.) Need to test if noscript is handled correctly now
3.) add scale command to scale output
4.) Maybe try to wrap and indent long lines at #terminal width in help etc. 
5.) Maybe move languages into an attribute of commands, I dunno
6.) Maybe need a better name. Clish? Wish? Wash? 
7.) An option to choose the cursor: block, underscore, pipe
8.) Make the update checking just check the latest update date in versionhistory
`;

let defaultOptions={prompt:"pet",language:"cli-html",flavor:"",starturl:"",forceuppercase:false,crt:"bw"};

 let languages={ /* here you can define new synonyms for default commands in the form new:default, to create other languages. */
                 /* Defining a synonym for the command will DISABLE the default form of the command. */
                 "unix":{"curl":"load","cat":"display","imagemagick":"images","ls":"links","which":"where","man":"help","cd":"follow","setopt":"options"},
                 "basic":{"load":"load","print":"where","cls":"clear","run":"display","gr":"images","list":"links","goto":"follow","poke":"options"},
                 "pig-latin":{"oadlay":"load","ackbay":"back","earclay":"clear","imagesway":"images","optionsway":"options","erewhay":"where","isplayday":"display","exitway" : "exit","ollowfay":"follow","orwardfay":"forward","epgray":"grep","elphay":"help","inthay":"hint","inkslay":"links","oremay":"more","eloadray":"reload","ersionsvay":"versions","udosay":"sudo","akesnay":"snake","options":"optionsway"}
 
 };
   
 
 let githubInfo={updateChecking: true,
           username:"kupietools",
           reponame:"web-CLI-browser",
           branch:"master",
           filename:"cli.html" };
 /* if updateChecking is true, this is the information for where the master of this file is hosted on Github, against which the version history will be checked for changes on startup. */
 

    

/* end user settings */

</script>
<link href="https://cdn.jsdelivr.net/npm/@icon/dashicons@0.9.0-alpha.4/dashicons.min.css" rel="stylesheet">
 <meta charset="UTF-8">
<!-- link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Chivo+Mono:ital,wght@0,100..900;1,100..900&family=VT323&display=swap" rel="stylesheet" -->

<style id="bodystyle">/* styles that shouldn't be in download files go here. */
 body {
     
  
    margin: 0;
    padding: 20px;
    height: 100vh;
    overflow: hidden;
      /* had these, but needs more work... text input color & sizew didn't match
        font-size:14px;
        font-family: "Chivo Mono", monospace,"VT323";
  font-optical-sizing: auto;
  font-weight: 400;
  font-style: normal;
        */
}
</style>
<style id="cli_doc_style">
       :root {--cursorRight: 1 ;
--bgcolor: #1a1a1a;
--textcolor: #c0c0c0;
--commandcolor: #fff;}
body {  background: var(--bgcolor);
    color: var(--textcolor); /* #3f3;*/ /* dimmer base green */
font-family: monospace;}
.cliImgOut {
/* font-size:.5em; nah, not sure I like this. */
	line-height: 1em;
}
.cliImgOutAscii {
	line-height: 1em;
}
.text-title {
    color: #9cf; /*#5af;*/ /* bright blue */
}
.text-header {
    color: #ffc /* #ff9; */ /* bright white for headers instead of green */
text-transform: uppercase;
}
.text-link {
    color: #8af; /*#fdf;*//* #faf;*/ /* #f5f; *//* bright magenta */
  font-size:10px;
}
.text-list {
  /*  color: #ff5;*/ /* bright yellow */ 
}
.text-quote {
    color: #aff; /* #5ff;*/ /* bright cyan */
}
.text-img {
color:#964; /*#960;*/
}
.text-bold {
    color: #fcc; /*#f99; */ /* bright white */
}
.text-anchor {
    color: #cef;/#8de;*/ /* #9ef; *//* #8ef; *//* bright white */
}
.text-summary {
    color: #cfc;/* #9f9; */ /* bright white */
}.text-summary::before {
    text-content: '&#9654'; /* bright white */
}
.text-detail {
    color: #cd9 /* #6c9 */; /* bright white */
}

.mmarker-child-a { color: #ffb; /*#ff5;*/}
.screen-reader-text {display: none;}

.h6, .titleprefix {
	/*text-transform: uppercase;*
	/*letter-spacing: 2px;*/
	/*font-size: .6875em;*/
font-variant: small-caps;
	/*display: block;*/
    /*color:#8bc;*//*8ac*//*#def;*/
	font-weight: 400;
	font-style: normal;

}
.h6:after {content:": ";}

.meta-genre-term {font-size: .6875em;   color:#ccf; /*#99F;*/


display: inline-block !important;

	padding: 2px 3px !important;
	line-height: 0.75em;
	border: 1px solid #33c6ff !important;
	/* background: #ccc /* #33c6ff */ !important; */
	width: auto;
	font-weight: 400 !important;
border-radius:3px;
margin-inline:1px;


}
.titleprefix {/* color:#FFC; */text-transform: capitalize;}
.titleprefix:after {content:': ';}

        #terminal {
            height: calc(100vh - 40px);
            overflow-y: auto;
            white-space: pre-wrap;
            word-wrap: break-word;
            padding: 0 12px 40px 12px;
        }
        #input-line {
            display: flex;
            align-items: initial ; /*center; caused problems if input wrapped*/
            margin-top: 1rem;
        }
        /* #prompt { NO makes input text shift 8px to the right, then move left when line is entered
            margin-right: 8px;
        } */
      #command-input {
      --cursor-offset: 0px;
            position: relative;
    background: transparent;
    border: none;
   /* color: #ccc; *//* #33ff33; */
    font-family: monospace;
    font-size: inherit;
    flex-grow: 1;
    outline: none;
    padding: 0;
    margin: 0;
    display: inline-block;
    caret-color: transparent;
      height:inherit;
      color:var(--commandcolor);
}

.prevcommand{color:var(--commandcolor);  }

#prompt {
  min-width: fit-content;
}

#command-input br {
	display: none; /* needed because firefox adds a <br> if you vackspace in an empty contenteditable */
}

 #command-input:focus::after {
            content: "█";
            animation: blink 1s steps(1) infinite;
            right: var(--cursor-offset);
            position: relative;
        }


.spacers {color:#333;}

@keyframes blink {
    0% { opacity: 1; }
    50% { opacity: 0.25; }
}


@keyframes rainbow {
    0% { color: red; }
    15% { color: orange; }
    30% { color: yellow; }
    45% { color: green; }
   60% { color: blue; }
    75% { color: indigo; }
    90% { color: violet; }

   
}

 .output {
            margin: 0; /* 10px 0; left blank lines in More */
        }
.error {
            color: #ff3333;
        }
.loading {
            /* color: #ffff33; */
            animation: blink 1s steps(1) infinite/*, rainbow 5s  infinite */;
        }
.more-prompt {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: #1a1a1a;
            padding: 10px 20px;
            border-top: 1px solid #33ff33;
        }
    
    #input-line.hidden {
    display: none;
}
    </style>
</head>
<body><div id="terminal"><div class="output">&nbsp;<br>*** KUPIETZ ARTS+CODE ***<p>&nbsp;&nbsp;&nbsp;&nbsp;7167 BYTES FREE<p>
Welcome to CLI browsing mode.<br>Type '<span id="helplang" class="prevcommand">help</span>' for available commands.</div><div id="input-line" class="">
    <span id="prompt">$ </span>
    <div id="command-input" contenteditable="true" autofocus=""></div>
</div>
    </div>

    <script>
    
    
 const origCRT= defaultOptions["crt"];
     let theseOptions = Object.assign({},defaultOptions); /* all acceptable url parameters must be listed here or they will be ignored */

    theseOptions["language"]  = theseOptions["language"] || "cli-html";
    languages[theseOptions["language"]] = languages[theseOptions["language"]] || {};
    const urlParams = new URLSearchParams(window.location.search);

    for (const key in theseOptions) {
      if (urlParams.has(key)) { /* this is where urlparams with no key above get disregarded */
        theseOptions[key] = urlParams.get(key);
      }
    }
    
    
    if (theseOptions["forceuppercase"]) {document.body.style.textTransform="uppercase"; /* I did NOT make all-caps the default to give you that vintage 1980 personal computer experience, because I'm a nice guy. You kids don't know how good you have it. */}

   console.log(theseOptions["prompt"]);
    
    let prompts={"unix":()=>{return currentUrl ? `[${currentUrl}]$ ` : '$ ';},
                 "apple":()=>'] ',"pet":()=>"READY."};
    let promptLineBreak="";
   
     if (origCRT!= theseOptions["crt"] ) {setCrt(theseOptions["crt"]);}
    const cmdDict = {};
    // Process each language

     languages=Object.assign(languages,addDicts());
 
    for (const language in languages) {
      cmdDict[language] = {};
      // Create reverse mapping for this language
      for (const key in languages[language]) {
        const value = languages[language][key];
        cmdDict[language][value] = key;
      }
    }
    document.getElementById("helplang").textContent=cmdInLang("help");

  

const commandInput = document.getElementById('command-input');
    
        document.getElementById("terminal").addEventListener('click', function() {
    commandInput.focus();
});
    
/* begin code to handle moving cursor with mouseclicks */
        commandInput.addEventListener('input', updateCursor);
        commandInput.addEventListener('click', updateCursor);
        commandInput.addEventListener('keyup', updateCursor);

        function updateCursor(e = {}) {
      /*  input = document.getElementById('command-input'); */
        if (!e || !e.key || (e.key != "ArrowUp" && e.key != "ArrowDown" /* up and down arrows handle positioning the cursor after loading the next or prev history line in the keydown listener */)) {
            const text = commandInput.textContent;
            const cursorPos = window.getSelection().anchorOffset;
            const offset = text.length - cursorPos;
            commandInput.style.setProperty('--cursor-offset', offset + 'ch'); }
        }

        updateCursor();
    /* end code to handle moving cursor with mouseclicks */
    
commandInput.addEventListener('keydown', async (e) => {
    if (e.key === 'Enter') {
        e.preventDefault(); // Prevent default newline
        const commandLine = commandInput.textContent.trim();
        commandInput.textContent = ''; // Clear the input
        
        if (commandLine) {
            commandHistory.push(commandLine);
            addOutput(`\n${promptSpan.textContent}${promptLineBreak}<span class="prevcommand">${commandLine}</span>`);
            let result;
        switch(commandLine.toLowerCase()){
            case 'make me a sandwich': 
            case 'make me a sandwich.': 
             result = "What? Make your own sandwich"; break;
            case 'sudo make me a sandwich': 
            case 'sudo make me a sandwich.': 
             result = "Okay."; break;
            default:
             result = await executeCommand(commandLine);
            }
            if (result) {
                addOutput(result);
            }
            
            if (!currentMoreOutput) {
                terminal.scrollTop = terminal.scrollHeight;
            }
        }
    }
});

// Optional: Prevent pasting formatted text
commandInput.addEventListener('paste', (e) => {
    e.preventDefault();
    const text = e.clipboardData.getData('text/plain');
    document.execCommand('insertText', false, text);
});
        const terminal = document.getElementById('terminal');
        const input = document.getElementById('command-input');
        const promptSpan = document.getElementById('prompt');
        const urlPattern = /^(https?:\/\/)?([\w-]+\.)+[\w-]+(\/[\w- .\/?%&=]*)?$/;
        
        const PROXY_SERVICES = [
        /* a word of explanation here. These are third party CORS proxies included as a kludge, in case something needs to be fetched from another domain [wink] for [wink] some [wink] reason */
            (url) => `https://api.allorigins.win/raw?url=${encodeURIComponent(url)}`,
            (url) => `https://cors-anywhere.herokuapp.com/${url}`,
            (url) => `https://api.codetabs.com/v1/proxy?quest=${encodeURIComponent(url)}`
        ];
        
        let currentLinks = [];
        let currentUrl = '';
        let pageCache = new Map();
        let currentMoreOutput = null;
        let currentCommand = null;
        let isCommandRunning = false;
        let currentProxyIndex = 0;
        let morePromptDiv = null;
        let originalURL = "";
    
    let chWidth=chToPixels();
    
    let imageNum = 0;
      let imageArray=[];
        // New variables for history tracking
        let pageHistory = [];
        let currentHistoryIndex = -1;
        
        const commandHistory = [];
        let historyIndex = -1;
        let currentInput = '';
        let firstSudo=true;

        const SIGNALS = {
            SIGINT: '^C',
            SIGTSTP: '^Z',
            SIGQUIT: '^\\'
        };
    
    
function chToPixels() {
  const temp = document.createElement('span');
  temp.style.font = window.getComputedStyle(document.getElementsByTagName("body")[0]).font;
  temp.style.fontSize = window.getComputedStyle(document.getElementsByTagName("body")[0]).fontSize;
  temp.style.position = 'absolute';
 // temp.style.visibility = 'hidden';
  temp.textContent = '0';
  document.body.appendChild(temp);
  const zeroWidth = temp.offsetWidth;
//  document.body.removeChild(temp);

  return zeroWidth;
}

        function decodeHtmlEntities(text) {
    
            const textarea = document.createElement('textarea');
            textarea.innerHTML = text;
            return textarea.value;
        }

        function getTerminalDimensions() {
            const lineHeight = 40;
            const promptHeight = 40;
            const height = Math.floor((terminal.clientHeight - promptHeight) / lineHeight);
            const width = Math.floor(terminal.clientWidth / 8);
            return { height, width };
        }
    

        function truncateLines(text, width) {
            return text.split('\n').map(line => 
                line.length > width ? line.slice(0, width - 3) + '...' : line
            ).join('\n');
        }

        
    
    
function updatePrompt() {
       
 if (theseOptions["prompt"]=="pet") {document.getElementById("input-line").style.flexDirection="column";promptLineBreak="\n";}else{document.getElementById("input-line").style.flexDirection="row";promptLineBreak="";}

    promptSpan.textContent = prompts[theseOptions.prompt]();/*was currentUrl ? `[${currentUrl}]$ ` : '$ '; */

}

    function toggleInputLine(show) {
    const inputLine = document.getElementById('input-line');
    if (show) {
        inputLine.classList.remove('hidden');
        input.focus();
    } else {
        inputLine.classList.add('hidden');
    }
}

      function addOutput(text, className = 'output') {
    const output = document.createElement('div');
    output.className = className;
    if (typeof text === 'string' && !text.includes('</')) {
        output.textContent = text;
    } else {
        output.innerHTML = text;
       
    }
    terminal.insertBefore(output, input.parentElement);
    terminal.scrollTop = terminal.scrollHeight;
    return output;
}

        function showMorePrompt(text) {
            if (morePromptDiv) {
                morePromptDiv.remove();
            }
            morePromptDiv = document.createElement('div');
            morePromptDiv.className = 'more-prompt';
            morePromptDiv.textContent = text;
            document.body.appendChild(morePromptDiv);
        }

        function removeMorePrompt() {
            if (morePromptDiv) {
                morePromptDiv.remove();
                morePromptDiv = null;
            }
        }

   function displaySignal(signal) {
    const currentLine = commandInput.textContent;
    commandInput.textContent = '';
    addOutput(`${promptSpan.textContent}${currentLine}${signal}`);
}


 function createMoreHandler(fullText) {
    const dims = getTerminalDimensions();
    
    // Split into lines while preserving HTML tags
    const lines = [];
    let currentLine = '';
    let inTag = false;
    let tagBuffer = '';
    let tagStack = [];
    
    // First split on newlines while preserving HTML tags
    for (let i = 0; i < fullText.length; i++) {
        const char = fullText[i];
        
        if (char === '<') {
            inTag = true;
            tagBuffer = char;
        } else if (char === '>') {
            inTag = false;
            tagBuffer += char;
            
            // Check if this is a closing tag
            if (tagBuffer.startsWith('</')) {
                tagStack.pop();
            } else if (!tagBuffer.endsWith('/>')) { // Not a self-closing tag
                const tagName = tagBuffer.match(/<([^\s>]+)/)?.[1];
                if (tagName) tagStack.push(tagName);
            }
            
            currentLine += tagBuffer;
            tagBuffer = '';
        } else if (inTag) {
            tagBuffer += char;
        } else if (char === '\n') {
            // Only split at newlines if we're not in the middle of a tag
            if (!inTag) {
                lines.push(currentLine);
                currentLine = '';
            } else {
                currentLine += char;
            }
        } else {
            currentLine += char;
        }
    }
    
    if (currentLine) {
        lines.push(currentLine);
    }
    
    let position = 0;
    
    function displayMore() {
        if (position >= lines.length) {
            currentMoreOutput = null;
            removeMorePrompt();
            toggleInputLine(true);
            return null;
        }
        
        const endPosition = Math.min(position + dims.height - 1, lines.length);
        const output = lines.slice(position, endPosition).join('\n');
       
        position = endPosition;
        
        const percentage = Math.floor((position / lines.length) * 100);
        const lineInfo = `lines ${position}/${lines.length}`;
        
        addOutput(output);
        
        if (position < lines.length) {
            showMorePrompt(`--Press Space For More--${lineInfo} (${percentage}%)`);
            return '';
        }
        
        removeMorePrompt();
        toggleInputLine(true);
        return null;
    }
    
    return {
        next: displayMore,
        cancel: () => {
            removeMorePrompt();
            toggleInputLine(true);
            const remainingLines = lines.length - position;
            return `...output cancelled (${remainingLines} lines remaining)`;
        }
    };
}
    
    
    /**
   * Triggers browser file download with specified content
   * @param {string} output - Content to be downloaded
   * @param {string} filename - Name of the file to be downloaded
   */
  function download_txt(output, myFile) {
    // Create a blob with the output content
  thisStyle=document.getElementById("cli_doc_style").innerHTML;
  output='<!doctype HTML><html><head><link href="https://cdn.jsdelivr.net/npm/@icon/dashicons@0.9.0-alpha.4/dashicons.min.css" rel="stylesheet"><style>'+thisStyle+'</style><body>'+output.replace(/(?:\r\n|\r|\n)/g,"<br>")+'</body></html>';
    const blob = new Blob([output], { type: 'text/html' });

    // Create a URL for the blob
    const url = URL.createObjectURL(blob);

    // Create a temporary anchor element
    const a = document.createElement('a');
    a.href = url;
    a.download = myFile;

    // Append to the document, click it, then remove it
    document.body.appendChild(a);
    a.click();

    // Clean up
    setTimeout(() => {
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    }, 100);
   return blob.size+" bytes written for download";
  }
async function fetchWithRetry(url) {
    let lastError;
    const loadingIndicator = addOutput(`Loading ${url}... waiting for data (0s)`, 'loading');
    let startTime = Date.now();
    let waitTimer = setInterval(() => {
        const seconds = Math.floor((Date.now() - startTime) / 1000);
        loadingIndicator.textContent = `Loading ${url}... waiting for data (${seconds}s)`;
    }, 1000);
    
    // Check if URL is same origin and try direct fetch first


if (new URL(url).origin === window.location.origin) {
    // Add mklynx parameter to URL
    const modifiedUrl = new URL(url);
 modifiedUrl.searchParams.append('mklynx', '1'); 
    try {
        const response = await fetch(modifiedUrl);


/* that was to add parameter. Orig was 
    if (new URL(url).origin === window.location.origin) {
        try {
            const response = await fetch(url); */
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }

            clearInterval(waitTimer);
            const reader = response.body.getReader();
            const contentLength = response.headers.get('Content-Length') || 0;
            let receivedLength = 0;
            let chunks = [];

            while(true) {
                const {done, value} = await reader.read();
                if (done) break;
                chunks.push(value);
                receivedLength += value.length;
                
                const percent = contentLength ? 
                    Math.round((receivedLength / contentLength) * 100) :
                    `${(receivedLength / 1024).toFixed(1)}KB`;
                loadingIndicator.textContent = contentLength ?
                    `Loading ${url}... ${percent}% (${(receivedLength / 1024).toFixed(1)}KB of ${(contentLength / 1024).toFixed(1)}KB)` :
                    `Loading ${url}... ${percent} received`;
            }

            loadingIndicator.remove();
            const chunksAll = new Uint8Array(receivedLength);
            let position = 0;
            for(let chunk of chunks) {
                chunksAll.set(chunk, position);
                position += chunk.length;
            }
            return new TextDecoder("utf-8").decode(chunksAll);
        } catch (err) {
            console.log('Direct fetch failed:', err);
            // Fall through to proxy system
        }
    }

    // proxy logic 
    for (let i = 0; i < PROXY_SERVICES.length; i++) {
        const proxyUrl = PROXY_SERVICES[(currentProxyIndex + i) % PROXY_SERVICES.length](url);
        try {
            const response = await fetch(proxyUrl, {
                headers: {
                    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'
                }
            });
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }

            clearInterval(waitTimer);
            const reader = response.body.getReader();
            const contentLength = response.headers.get('Content-Length') || 0;
            let receivedLength = 0;
            let chunks = [];

            while(true) {
                const {done, value} = await reader.read();
                if (done) break;
                chunks.push(value);
                receivedLength += value.length;
                
                const percent = contentLength ? 
                    Math.round((receivedLength / contentLength) * 100) :
                    `${(receivedLength / 1024).toFixed(1)}KB`;
                loadingIndicator.textContent = contentLength ?
                    `Loading ${url}... ${percent}% (${(receivedLength / 1024).toFixed(1)}KB of ${(contentLength / 1024).toFixed(1)}KB)` :
                    `Loading ${url}... ${percent} received`;
            }

            currentProxyIndex = (currentProxyIndex + i) % PROXY_SERVICES.length;
            loadingIndicator.remove();
            
            const chunksAll = new Uint8Array(receivedLength);
            let position = 0;
            for(let chunk of chunks) {
                chunksAll.set(chunk, position);
                position += chunk.length;
            }
            return new TextDecoder("utf-8").decode(chunksAll);
        } catch (err) {
            lastError = err;
            loadingIndicator.textContent = `Loading ${url}... Retrying with different proxy (${err.message})`;
            continue;
        }
    }
    
    clearInterval(waitTimer);
    loadingIndicator.remove();
  /* was  throw lastError; but wanted more detail */
throw new Error(`Sorry, we couldn't load ${url} right now after trying ${PROXY_SERVICES.length} different methods. This could be because the site is down, blocking access, or our connection methods aren't working. Last error was: ${lastError}. The site may be blocking proxy access. Please try again in a few minutes.`);


}
    

function processNode(node, linkMap, indent = 0) {
    let content = '';
    let spacer = '<span class="spacers">'+('|       '.repeat(indent))+'</span>';
    if (node.nodeType === Node.TEXT_NODE) {
        const text = node.textContent.trim();
        if (text) {
            const parentAnchor = node.parentElement.closest('a');
            if (parentAnchor && linkMap.has(parentAnchor)) {
                if (!parentAnchor._numbered) {
                    content += `<span class="text-link">[${linkMap.get(parentAnchor) + 1}]</span>`;
                    parentAnchor._numbered = true;
                }
            }
            content += text;
        }
        return content;
    }

    if (node.nodeType !== Node.ELEMENT_NODE) {
        return content;
    }

    const tagName = node.tagName.toLowerCase();
    
    if (['script', 'style', 'iframe' /*, 'noscript' */].includes(tagName)) {
        return '';
    }



    switch (tagName) {

        case 'h1':
        case 'h2':
        case 'h3':
        case 'h4':
        case 'h5':
        case 'h6':
            content += `\n<!-- 546 -->${spacer}<span class="text-header">${'='.repeat(indent + 2)} `;
            break;
            
        case 'blockquote':
            content += `\n<!-- 550 -->${spacer}<span class="text-quote">`;
            break;
         case 'summary':
            content += `\n<!-- 553 -->${spacer}<span class="text-summary">`;
            break;
         case 'details':
            content += `<span class="text-detail">`;
            break;
            
        case 'strong':
        case 'b':
            content += `<span class="text-bold">`;
            break;
                
        case 'a':
            content += `<span class="text-anchor">`;
            break;
  
        case 'p':
        case 'div':
        case 'section':
        case 'article':
            if (node.textContent.trim()) {
                content += '\n<!-- 574 -->'+(spacer);
            }
            break;
            
        case 'br':
            content += '\n<!-- 578 -->'+(spacer);
            break;
            
        case 'li':
           if (!(node.classList.contains('postscroll')||node.classList.contains('menusection'))) {   content += /*\n*/`\n<!-- 582 -->${spacer}<span class="text-list">• </span>`;}
            break; /* don't want the LIs with no real content on menus to display on their own lines with bullets */
            
        case 'ul':
        case 'ol':
          content += '\n<!-- 587 -->'+(spacer);
            indent += 1;
            break;
    case 'img':
   
    let thisFile=(node.getAttribute('data-src')||node.src);
    let thisAlt=node.getAttribute('alt') || "";
    let theDimensions = node.naturalWidth?`${node.naturalWidth}x${node.naturalHeight}`:( node.width?`${node.width}x${node.height}`:'');
    content += `<span class="text-img">[image ${++imageNum}${thisAlt?": "+thisAlt:""}${theDimensions?' '+theDimensions:''}]`; 
     imageArray[imageNum]={url:thisFile,alt:thisAlt,dim:theDimensions};
    break;
  
    }
    
/*    case 'img': 
 content += "[image "+node.dataset.src+"]"; */
    
    



 if (node.classList.contains('titleprefix') ) {
        content += '<span class="titleprefix">';
    }
    if (node.classList.contains('menutitleprefix') || node.classList.contains('h6')) {
        content += '<span class="h6 menutitleprefix">';
    }
 if (node.classList.contains('meta-genre-term')) {
        content += '<span class="meta-genre-term">';
    }

 if (node.classList.contains('screen-reader-text')) {
        content += '<span class="screen-reader-text">';
    }

 if (node.classList.contains('mmarker-child-a')) {
        content += '<span class="mmarker-child-a">';
    }

if(node.classList.contains('dashicons')) {
classOut = Array.from(node.classList).filter(item => item.includes("dashicons")).join(" "); /* node.classList is a DOMTokenList object, not a regular array, so it doesn't have the filter method. You need to convert it to an array first. [...node.classList] would work too. */
if (classOut) {content += `<span class="${classOut}"></span>`;}

}



    // Process all child nodes
    for (const child of node.childNodes) {
        const childContent = processNode(child, linkMap, indent);
        // If this is a text node and we're inside a menutitleprefix span,
        // make sure we close the span before adding the text
        if (child.nodeType === Node.TEXT_NODE && 
            node.classList.contains('menutitleprefix')) {
            content += childContent;
        } else {
            content += childContent;
        }
    }
    
    if (node.classList.contains('h6') || node.classList.contains('titleprefix') || node.classList.contains('menutitleprefix') || node.classList.contains('meta-genre-term') || node.classList.contains('screen-reader-text') || node.classList.contains('mmarker-child-a')) {
        content += '</span>';
    }
    if (node.classList.contains('h6')) {
        content += '\n'+(spacer);
    }
    
    switch (tagName) {
    
    case 'img': content += "</span>"; break;
       case 'ul':
        case 'ol':
          content += '\n<!-- 587 -->'+(spacer);
          
            break;
    
   
        case 'h1':
        case 'h2':
        case 'h3':
        case 'h4':
        case 'h5':
        case 'h6':
            content += `${'='.repeat(indent + 2)}</span>`;
            break;
            
        case 'blockquote':
            content += '</span><!-- 631 -->\n'+(spacer);
            break;
          case 'summary':
            content += '</span>\n<!-- 634 -->'+(spacer);
            break;
          case 'details':
            content += '</span>\n<!-- 637 -->'+(spacer);
            break;
            
        case 'strong':
        case 'b':
            content += '</span>';
            break;
            
        case 'a':
            content += '</span> ';
            break;
            
        case 'p':
        case 'div':
        case 'section':
        case 'article':
         /*   if (node.textContent.trim()) {
                content += '\n<!-- 653 -->'+(spacer);
            } ( */
            break;
    }
    
    return content;
}
    
    
     async function processHTML(html, url) {

    const parser = new DOMParser();
    const doc = parser.parseFromString(html  .replace(/&lt;/gi, '&amp;lt;'), 'text/html'); /* DOMparser automatically renders entities in text so code in the bodies made with fake entities renters is real code in the browser if you don't substitute. */
    
    const linkMap = new Map();
    const links = Array.from(doc.querySelectorAll('a')).map((a, index) => {
        if (a.href) {
            try {
                const resolvedUrl = new URL(a.href, url).href;
                linkMap.set(a, index);
                return {
                    text: a.textContent.trim(),
                    href: resolvedUrl
                };
            } catch (e) {
                return null;
            }
        }
        return null;
    }).filter(Boolean);
  

    let textContent = '';
    
    const title = doc.querySelector('title');
    if (title) {

        const titleText = decodeHtmlEntities(title.textContent.trim());
    document.title="CLI Browser ["+titleText+"]";
        textContent += '<span class="text-title">';
        textContent += '='.repeat(40) + '\n';
        textContent += titleText + '\n';
        textContent += '='.repeat(40) + '</span>\n\n';
    }

       
     
    textContent += processNode(doc.body, linkMap);
  
    textContent = textContent
        .replace(/\n{3,}/mg, '\n\n')
         .replace(/(\n[-<!0-9> ]*<span class="spacers">[ |]*<\/span>){3,}/igm, '$1$1')
        .replace(/[ \t]+/g, ' ')
        .trim();
    
    textContent += '\n\n' + '-'.repeat(40) + '\n';
    textContent += `${links.length} links found. Use 'links' command to show them.\n`;
    
    return { content: textContent, links };
}

    
    function locCmd(commandIn) {
    result = (cmdInLang(commandIn)!=commandIn)? "" : (languages[theseOptions["language"]][commandIn] || commandIn); /* make it so when using a language, the original commands fail if alternatives are defined for them */
    return result;
  
    };
    
    function cmdInLang(commandIn,description="",length=0) {
   
    return cmdDict[theseOptions["language"]][commandIn]||commandIn;
   
    }
    
    
    function cmdOut(commandIn,description="",length=0) {
    firstCommand=cmdInLang(commandIn);
    theCommand= '<span class="prevcommand">'+firstCommand+'</span>'; 
     theFirstOutput=firstCommand+((firstCommand && description)?" ":"")+description;
      theOutput=theCommand+((firstCommand && description)?" ":"")+description;
    padding=Math.max(length-theFirstOutput.length,0);
    return theOutput+" ".repeat(padding);
    
    }
    
  async function executeCommand(commandLine, canInterrupt = true) {
    isCommandRunning = true;
    currentCommand = { canInterrupt };
    toggleInputLine(false);
    
    try {
        const pipes = commandLine.split('|').map(cmd => cmd.trim());
    
        let output = '';
        
        for (let i = 0; i < pipes.length; i++) {
            if (!currentCommand) {
                return '';
            }

            const [commandRaw, redirect, ...redirectArgs ] = pipes[i].split('>').map(cmd => cmd.trim());
    
        
            const [commandInput, ...args] = commandRaw.split(' ');
        command = locCmd(commandInput.toLowerCase()) ;
            
            if (command === 'more') {
                if (!output) {
                    return 'No input to more';
                }
                currentMoreOutput = createMoreHandler(output);
             console.log(1,commandRaw,command);
                return currentMoreOutput.next();
           
            }
            console.log(2,commandRaw,command);
            const cmd = commands[command];
            if (!cmd) {
            return `\n?SYNTAX ERROR - Type '${cmdOut("help")}' for available commands.`;
               /* return `Command not found: ${command}`; */
            }
            
            // Pass the previous output as the last argument for piped commands
            if (i > 0) {
                args.push(output);
            }
            
            const result = await doCommand(command)(...args);
            output = (redirect==undefined)?(result || ''):download_txt(result,redirect);
        }
        
        return output;
    } finally {
        isCommandRunning = false;
        currentCommand = null;
        if (!currentMoreOutput) {
            toggleInputLine(true);
        }
    }
}  
    
    
    
        const commands = { /* hid - load     : Load a website in text mode from help */
        versions: {man: ()=>`USAGE: ${cmdOut("versions")}
        
        This displays the version history of this software, as well as other development-related info.`,
                 do:() => versionhistory},
       
           help: {man: ()=>`Unfortunately the ${cmdInLang("help")} page for '${cmdOut("help")}' has been recalled by the Department Of Redundancy Department.`,
                  do:(command="") => {
                  if (command=="") {theLength=30;return `
Available ${theseOptions["language"]} commands. Use '${cmdOut("help")} somecommand' to see more detailed info on a command:
- ${cmdOut("display","",theLength)} : Display the loaded page
- ${cmdOut("links","",theLength)} : List all links in the loaded page
- ${cmdOut("images","",theLength)} : List all image descriptions and URLs in the loaded page
- ${cmdOut("image","imagenumber",theLength)} : Display an ascii-converted image by image number, in blocks or ascii
- ${cmdOut("follow","linknumber",theLength)} : Follow a link by its number
- ${cmdOut("back","",theLength)} : Go back to the previous page
- ${cmdOut("forward","",theLength)} : Go forward to the next page
- ${cmdOut("reload","",theLength)} : Reload last loaded page
- ${cmdOut("where","",theLength)} : Display the URL of the currently loaded page
- ${cmdOut("clear","",theLength)} : Clear the terminal output
- ${cmdOut("options","[parameter [value]]",theLength)} : Manually read or change URL parameters (given below) from the command line. Omit value to display current value, omit both to see all current values
- ${cmdOut("exit","",theLength)} : End session and return to GUI browing mode
- ${cmdOut("help","[somecommand]",theLength)} : Show this help message, or detailed help on a single command
- ${cmdOut("versions","",theLength)} : Version history/changelog of this browser
- ${cmdOut("hint","",theLength)} : A mysterious command, I don't know what it does.
- somecommand1 ${cmdOut("|","somecommand2",theLength-13)} : Redirect output into another command such as 'grep' or 'more'
- ${cmdOut("more","",theLength)} : Use after pipe (|) to paginate output. [space] to continue. Esc, ctrl-c, or q to quit
- ${cmdOut("grep","filterexpression",theLength)} : Use after pipe (|) to filter output by regular expression. Case-insensitive, no delimiters
- somecommand ${cmdOut(">",'filename.html',theLength-12)} : Save the output to a file download, rather than ${theseOptions["language"]=="unix"?"STDOUT":"onscreen"}

Optional startup URL paramaters (starred value is the default if unspecified):
- language [cli-html*|unix|basic|pig-latin]   : Change commands to match various platforms. This way you unix geeks don't have to fight your natural urge to type 'ls'
- prompt [unix|pet*|apple]                    : Change style of command prompt to match various advanced platforms
- crt [bw*|green|amber]                       : Choose between popular CRT colors. There may be some others available
- forceuppercase [false*|true]                : Force all uppercase text for that vintage 1980 personal computer experience
`;}
                  else


                  console.log("1. locCmd(command)", locCmd(command));
                     console.log("2. commands[locCmd(command)].man", commands[locCmd(command)].man);
                  if (commands[locCmd(command)])
{console.log("3. inside if.");return `\n${commands[locCmd(command)].man()}`|| "\nNo man page for "+command;} 
else {return "\n?BAD VALUE - "+command;}
                  }}, /* extra verbiage: Execute a command with elevated privileges */
 options: {man: ()=>`USAGE: ${cmdOut("options")} [parameter [value|default]]
 
            This program allows you to initially set options by including URL parameters when you first call it. (see '${cmdOut("help")}' command.) You can view or change the same set of options from the command line by using this command. '${cmdOut("options")}' alone will list the currently set options. '${cmdOut("options")} parametername' will show you the current setting for parametername. '${cmdOut("options")} parametername value' will set parametername to value. You can pass any option a value of "default" to return to the original session start value.`, 
                       
            do:(option,value) =>   { if (option)
 {
 const queryString = window.location.search;
const urlParams = new URLSearchParams(queryString);
console.log("xyz",value, urlParams.get(option), defaultOptions[option]);
 value=(value=="default")?(urlParams.get(option)?urlParams.get(option):defaultOptions[option]):value;
  if ((option=="language" && languages[value]==undefined) || theseOptions[option]==undefined) {return "\n?ILLEGAL VALUE\n";}
 if (value) {
            
             theseOptions[option]=value;
             setCrt(theseOptions["crt"]);
  
   
             updatePrompt();
             addOutput("\n"+option+" is '"+theseOptions[option]+"'"+(option=="language"?` - Type '${cmdOut("help")}' for available commands.`:""));
        
 } }
                             else
                             {for (const opt in theseOptions) {theseOptions[opt]?addOutput(opt+" is '"+theseOptions[opt]+"'\n"):"";} }
                              
                              
                              
 }},
       
        where: {man: ()=>`USAGE: ${cmdOut("where")}
        
        Simply outputs the URL of the currently loaded page.`,
                do:()=>currentUrl},
        
hint: {man: ()=>`There is no man page for hint.`,do:() => {let hints=[`If you ${cmdOut("load")} a can of URLs, you do so at your own risk.`,
                    "You can do as the su say, not as the su do.",
"Don't be startled by hollow voices. It's not personal.", 
"There are things here Indiana Jones would tremble in fear of.",
"Before CRTs, there were lineprinters.",
                       "For every screen, there is an inversion. (Turn, turn, turn.)",
                        "Yes, Virginia, there is a Commodore 64 screen setting.",
                       "We can't play snakes & ladders, because we have no ladders."];
           
             index=Math.floor(Math.random() * hints.length);
  return hints[index];
            }},
              
       /*  crt: (color) => {setCrt(color);
            },*/
xyzzy: {man: ()=>`USAGE: ${cmdOut("xyzzy")}

        It is pitch black. You are likely to be eaten by a grue.`,
        do:() => {return "\nA hollow voice says 'fool.'";}},
            
        display: {man: ()=>`USAGE: ${cmdOut("display")}
        
        Display the currently loaded page.`,
                  do:async (url="") => {
            if (currentLinks[url] != undefined && currentLinks[url].href != undefined && currentLinks[url].href != "") {url = currentLinks[url].href}
                if ((!currentUrl) && (!url)) {
                    return 'No page currently loaded.';
                }
                return doCommand("load")(url || currentUrl, false);
            }}, 

          back: {man: ()=>`USAGE: ${cmdOut("back")}
        
        Reloads the previous page in the browsing history. Use '${cmdOut("forward")}' to move forward again.`,
do:async () => {
    if (currentHistoryIndex <= 0) {
        return 'No previous pages in history.';
    }
    toggleInputLine(false);
    currentHistoryIndex--;
    const result = await doCommand("load")(pageHistory[currentHistoryIndex], false);
    if (!currentMoreOutput) {
        toggleInputLine(true);
    }
    return result;
}},
        
        forward: {man: ()=>`USAGE: ${cmdOut("forward")}
        
        If you have previously used '${cmdOut("back")}' to move backwards in the browsing history, this reloads the following page in the history.`,
                  do:async () => {
    if (currentHistoryIndex >= pageHistory.length - 1) {
        return 'No forward pages in history.';
    }
    toggleInputLine(false);
    currentHistoryIndex++;
    const result = await doCommand("load")(pageHistory[currentHistoryIndex], false);
    if (!currentMoreOutput) {
        toggleInputLine(true);
    }
    return result;
}},
        
image: {man: ()=>`USAGE: ${cmdOut("image")} [imagenumber]

            Loads the image indicated by imagenumber, the number of an image in the currently loaded page. To see these numbers, use the '${cmdOut("images")}' command.`,
        
        do:async (param1,param2) => {

num=isNaN(param1)?param2:param1;
ascii=isNaN(param1)?(param1=="-a" || param1=="--ascii-art"):param2;
if (isNaN(param1) && !["-a","--ascii-art"].includes(param1)) {return "unrecognized option: "+param1;}
if (imageArray[num] && imageArray[num]["url"]) {
imageUrl = imageArray[num]["url"]; } else {return "Invalid image number: "+num;}
 
 return new Promise((resolve, reject) => {
        const img = new Image();
        img.crossOrigin = "Anonymous";
        
        img.onload = function() {
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
          const scale = Math.max(1.05*img.width /window.innerWidth, 1.05*img.height/window.innerHeight);
        /* scale if image > 98% of window in either dimension */

           const blockWidth = chWidth*Math.max(scale,1);
            const blockHeight =chWidth*2*Math.max(scale,1);        
            canvas.width = img.width;
            canvas.height = img.height;
            ctx.drawImage(img, 0, 0);
                  /* remember if the console is open, it will scale if the vertical direction is bigger than the remaining available window height */
      
            const blocksAcross = Math.floor(img.width / blockWidth);
            const blocksDown = Math.floor(img.height / blockHeight);
            
            // ASCII chars from darkest to brightest
            const asciiChars = ' .`\'",-:;~>+!?░@l#$&▒§¶MWB▓█';
            /* alternative character sets: 
             Original :".-░*%#@$&▒§¶▓█';
            " .'`^\",_-:;~>+!/?1}(|l[irxczb%khmwqpdXQ0OZ#MWB@";
            " .'`^\",_-:;~><+?][}1(|/Il!itfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@"; */
            let result = '';
            
            for(let y = 0; y < blocksDown; y++) {
                for(let x = 0; x < blocksAcross; x++) {
                    const data = ctx.getImageData(x * blockWidth, y * blockHeight, blockWidth, blockHeight).data;
                    
                    let r = 0, g = 0, b = 0;
                    for(let i = 0; i < data.length; i += 4) {
                        r += data[i];
                        g += data[i + 1];
                        b += data[i + 2];
                    }
                    
                    const pixelCount = (blockWidth * blockHeight);
                    r = Math.floor(r / pixelCount);
                    g = Math.floor(g / pixelCount);
                    b = Math.floor(b / pixelCount);
                    const asciiMax = 200/Math.max(r,g,b) //upscale displayed color brightness a bit for ascii
                    let ra= Math.floor(r*asciiMax);
                    let ga= Math.floor(g*asciiMax);
                    let ba= Math.floor(b*asciiMax);
                    
                    // Calculate brightness (0-255)
                    const brightness = (r + g + b) / 3; //upscale brightness:  Math.min((r + g + b) / 3,255); 
                    
                    // Map brightness to ASCII character
                    const charIndex = Math.floor(brightness / 255 * (asciiChars.length - 1));
                    const char = asciiChars[charIndex];
                    
                    // Add colored character
                    result += ascii?`<span class="termImgAscii" ${ascii?`style="color: rgb(${ra},${ga},${ba})"`:`style="color: rgb(${r},${g},${b})"`}>${char}</span>`:`<span class="termImg" style="color: rgb(${r},${g},${b})">█</span>`;
                }
                result += '<br>';
            }
                    result = `<div class="cliImgOut${ascii?'Ascii':''}">${result}${scale>1?"[Scaled "+Math.floor(100*(1/scale))+"%]":""}</div>`
            resolve(result);
        };
        
        img.onerror = reject;
        img.src = imageUrl;
    });

}}


,


 images: {man: ()=>`USAGE: ${cmdOut("images")}
 
        Returns a numbered list of the images in the currently loaded dcument, with alt text, URLs, and dimensions.`,
          do: () => { let outImages="";for (const imNum in imageArray)
 {
    
outImages+=`<span class="text-link">[${imNum}]</span> ${imageArray[imNum]["alt"]?imageArray[imNum]["alt"]+" ":""}<span class="text-img">${imageArray[imNum]["url"]}${imageArray[imNum]["dim"]?' '+imageArray[imNum]["dim"]:''}</span>\n`;
 }
 return outImages;
 ;} }
        ,
        
 sudo: {man: ()=> `USAGE: sudo somecommand [option1 option2 ...]
 
        This executes somecommand with elevated privileges, which removes the security restrictions placed on ordinary users. Useful for completely erasing the internet or launching nuclear warheads.`,
        do:async (user) => {
    const line = document.createElement('div');
    line.style.display = 'inline-block';
    line.style.whiteSpace = 'pre';
    line.contentEditable = 'true';
    line.style.outline = 'none';
   /* line.style.width = '200x';  // Fixed width
    line.style.height = '1.2m'; // Fixed height 
    NOT NEEDED */
    line.style.color = 'transparent';
    line.style.caretColor = '#3f3';
    line.style.verticalAlign = 'top'; // Prevent vertical movement
if (firstSudo == true) {
    addOutput(`
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

`);firstSudo=false;}

    let attempts = 0;
    while (attempts < 3) {
      //  const promptDiv = addOutput(`${user}'s password: `, 'output'); NAH, it should be  sudo command, not sudo user. */
    const promptDiv = addOutput(`Password: `, 'output');
        promptDiv.appendChild(line);
        line.focus();

        const password = await new Promise(resolve => {
            const handler = e => {
                if (e.key === 'Enter') {
                    e.preventDefault();
                    line.removeEventListener('keydown', handler);
                    resolve(line.textContent);
                }
            };
            line.addEventListener('keydown', handler);
        });

      /*  promptDiv.removeChild(line); THIS WASN'T EVEN NEEDED. */
        line.textContent = '';
        addOutput('');
        
        attempts++;
        if (attempts < 3) {
            addOutput('Sorry, try again.');
        } else {
            addOutput('sudo: 3 incorrect password attempts');
        }
    }
    return '';
}},
 snake: {man: ()=>"Sorry, Indi.",do:() => {calcAssistance()}},
        grep: {man: ()=>`USAGE: ${cmdOut("somecommand")} | grep [searchstring]
        
        Filters the output of somecommand so that only lines that contain searchstring will show. The match is case-insensitive: "THE" and "The" are considered to be the same thing. Searchstring can be a regular expression (if you don't know what that is, you don't need to.) Regular expressions should not be surrounded by //. `,
               do:(pattern, prevOutput) => {
    if (!pattern) {
        return 'Error: grep requires a pattern';
    }
    try {
        const regexp = new RegExp(pattern, 'i');
        const lines = (prevOutput || '').split('\n');
        const matches = lines.filter(line => regexp.test(line));
        return matches.join('\n');
    } catch (e) {
        return `Error: Invalid ${cmdOut("grep")} expression - ${e.message}`;
    }
}},

exit:{man: ()=> `USAGE: ${cmdOut("exit")}
        Ends the command line browsing session and opens the currently loaded page in your usual GUI browser.`,
do: () => {
const now = new Date();
window.location.href=(currentUrl?currentUrl:originalURL); currentMoreOutput=true;return `Goodbye!\n\nLogged out at ${now.toLocaleString()}`;} } ,      
        

            links: {man: ()=>`USAGE: ${cmdOut("links")}
            
        Returns a numbered list of the links in the currently loaded dcument.`,
                    do:() => {
                if (!currentLinks || currentLinks.length === 0) {
                    return 'No links available. Load a page first using the Load command.';
                }
                return currentLinks.map((link, i) => 
                    `<span class="text-link">[${i + 1}]</span> <span class="text-anchor">${link.text}</span> -> ${link.href}</a>`
                ).join('\n');
            }},

            follow: {man: ()=>`USAGE: ${cmdOut("follow")} [linknumber]
            
            Loads the page indicated by linknumber, the number of a link in the currently loaded page. To see these numbers, use the '${cmdOut("links")}' command.`,
                     do:async (num) => {
                if (!currentLinks || currentLinks.length === 0) {
                    return 'No links available. Load a page first that contains links.';
                }
                const index = parseInt(num) - 1;
                if (isNaN(index) || index < 0 || index >= currentLinks.length) {
                    return 'Invalid link number. Use the links command to see available links.';
                }
                return doCommand("load")(currentLinks[index].href);
            }},
        
    
        
reload: {man: ()=>`USAGE: ${cmdOut("reload")}

        This reloads whatever page was last loaded. Useful if the page has changed since you loaded it.`,
         do:async () => {return doCommand("load")(currentUrl, false, true);}},
           
        clear:{man: ()=>`USAGE: ${cmdOut("clear")}
        
        Clears the terminal contents and moves the command line prompt up to the top.`,
               do: () => {elements = document.getElementsByClassName("output");
    while(elements.length > 0){
        elements[0].parentNode.removeChild(elements[0]);
    }
return}
              },
        
        load: {man: ()=> `USAGE load [url]
        
        Load a web page. Once a page is loaded, you can DISPLAY, LINKS, RELOAD, or any of the other commands that act on loaded webpages.`,
               do: async (url, addToHistory = true, reload=false) => {
    if (!url) return 'Error: Please provide a URL';

    if (!url.startsWith('http')) {
        url = 'https://' + url;
    }

    try {
        new URL(url);
    } catch (e) {
        return 'Error: Invalid URL format';
    }

    try {
        let content, links;
        let gotFromCache=pageCache.has(url) && reload==false;
        if (gotFromCache) {
            ({ content, links } = pageCache.get(url));
        } else {
            try {
               
                const html = await fetchWithRetry(url);
                const result = await processHTML(html, url);
                content = result.content;
                links = result.links;
                if (content) {
                    pageCache.set(url, { content, links });
                }
            } catch (err) {
                throw new Error(`Failed to fetch: ${err.message}`);
            }
        }

        currentUrl = url;
        currentLinks = links;
        
        if (addToHistory) {
            if (currentHistoryIndex < pageHistory.length - 1) {
                pageHistory = pageHistory.slice(0, currentHistoryIndex + 1);
            }
            pageHistory.push(url);
            currentHistoryIndex = pageHistory.length - 1;
        }
        
        updatePrompt();
       if (content && !gotFromCache) {addOutput( '<!-- span class="dashicons dashicons-admin-site" style="font-size:.7rem"> </span -->'+url+" loaded, "+ content.length+" bytes received. Ready to "+cmdInLang("display"));}
    thisOutput= (addToHistory || reload)?"":("\n"+(content|| 'Error: No content processed')); /* don't show on load or reload; make them use Display. Should really have a separate parameter to pass explicitly telling it whether to display or not, but for now, it's ok to do it this way. */
        return thisOutput ;
    } catch (err) {
        currentUrl = '';
        currentLinks = [];
        updatePrompt();
        return `\nError loading ${url}: ${err.message}`;
    }
} 
              }
        };
    
    /**** end of commands definition ***/

        document.addEventListener('keydown', async (e) => {
            if (e.ctrlKey && e.key === 'c') {
                e.preventDefault();
                
                if (currentMoreOutput) {
                    const cancelMsg = currentMoreOutput.cancel();
                    addOutput(SIGNALS.SIGINT);
                    currentMoreOutput = null;
                    updatePrompt();
                    return;
                }
                
                if (isCommandRunning && currentCommand?.canInterrupt) {
                    currentCommand = null;
                    displaySignal(SIGNALS.SIGINT);
                    updatePrompt();
                    return;
                }
                
                if (!isCommandRunning) {
                    displaySignal(SIGNALS.SIGINT);
                    updatePrompt();
                    return;
                }
            }

            if (e.ctrlKey && e.key === 'z') {
                e.preventDefault();
                displaySignal(SIGNALS.SIGTSTP);
                return;
            }

            if (e.ctrlKey && e.key === '\\') {
                e.preventDefault();
                displaySignal(SIGNALS.SIGQUIT);
                return;
            }

            if (currentMoreOutput) {
                e.preventDefault();
                
                if (e.key === 'Escape' || e.key === 'q') {
                    const cancelMsg = currentMoreOutput.cancel();
                    addOutput(cancelMsg);
                    currentMoreOutput = null;
                    updatePrompt();
                    return;
                }
                
                if (e.code === 'Space' || e.key === 'PageDown') {
                    const more = currentMoreOutput.next();
                    if (more !== null) {
                        addOutput(more);
                    } else {
                        updatePrompt();
                    }
                    return;
                }
                
                if (e.key === 'Enter') {
                    const more = currentMoreOutput.next();
                    if (more !== null) {
                        addOutput(more, true);
                    } else {
                        updatePrompt();
                    }
                    return;
                }
            }
        });

       commandInput.addEventListener('keydown', async (e) => {
    if (e.key === 'Enter') {
        e.preventDefault();
        const commandLine = commandInput.textContent.trim();
        commandInput.textContent = '';
        
        if (commandLine) {
            commandHistory.push(commandLine);
            historyIndex = -1;
            addOutput(`${promptSpan.textContent}${commandLine}`);
            
            const result = await executeCommand(commandLine);
            if (result) {
                addOutput(result);
            }
            
            if (!currentMoreOutput) {
                terminal.scrollTop = terminal.scrollHeight;
            }
        }
    } else if (e.key === 'ArrowUp') {
        e.preventDefault();
        if (historyIndex === -1) {
            currentInput = commandInput.textContent;
        }
        if (historyIndex < commandHistory.length - 1) {
            historyIndex++;
            commandInput.textContent = commandHistory[commandHistory.length - 1 - historyIndex];
            // Place cursor at end
            const range = document.createRange();
            const sel = window.getSelection();
            range.selectNodeContents(commandInput);
            range.collapse(false);
            sel.removeAllRanges();
            sel.addRange(range);
        }
      commandInput.style.setProperty('--cursor-offset', '0ch');
    } else if (e.key === 'ArrowDown') {
        e.preventDefault();
        if (historyIndex > 0) {
            historyIndex--;
            commandInput.textContent = commandHistory[commandHistory.length - 1 - historyIndex];
        } else if (historyIndex === 0) {
            historyIndex = -1;
            commandInput.textContent = currentInput;
        }
        // Place cursor at end
        const range = document.createRange();
        const sel = window.getSelection();
        range.selectNodeContents(commandInput);
        range.collapse(false);
        sel.removeAllRanges();
        sel.addRange(range);
       commandInput.style.setProperty('--cursor-offset', '0ch');
    }
});
    
    
     function setCrt (color) {
        let thisColor=""; let bkColor="";let cmdColor="";
        if (color=="invert") {
        document.documentElement.style.filter = ((document.documentElement.style.filter)=="invert()")?"":"invert()";
        
       // document.documentElement.style.webkitFilter = ((document.documentElement.style.webkitFilter)=="invert()")?"":"invert()";
        
        } else if (color=="lineprinter") {
         document.documentElement.style.filter ="";
        document.getElementById("terminal").style.background="";
         document.getElementById("terminal").style.backgroundImage="linear-gradient(0deg, #380936 25%, #000000 25%, #000000 50%, #380936 50%, #380936 75%, #000000 75%, #000000 100%)";
        document.getElementById("terminal").style.backgroundSize= "160px 160px";
        document.getElementById("terminal").style.filter="invert()";
        document.getElementById("terminal").style.backgroundAttachment="local";
         thisColor = "#e5e5e5";  bkColor="#3f3f3f"; cmdColor="#fff"; 
              document.querySelector(':root').style.setProperty("--bgcolor",bkColor); document.querySelector(':root').style.setProperty("--textcolor",thisColor); document.querySelector(':root').style.setProperty("--commandcolor",cmdColor);
  
        }
     else {
      document.documentElement.style.filter ="";
     document.getElementById("terminal").style.background="";
     document.getElementById("terminal").style.filter="";
     switch(color) {
          
        case 'amber':  thisColor = "#FFBF00";  bkColor="#1a1a1a";  cmdColor="#ffd6aa"; break;
        case 'green':  thisColor = "#00CC00";  bkColor="#1a1a1a"; cmdColor="#00FF00"; break;
        case 'invert':  thisColor = "#1a1a1a";  bkColor="#c0c0c0"; cmdColor="#000"; break;
        case 'bw':
        case 'gray':
        case 'white':
        case 'monochrome':  thisColor = "#c0c0c0";  bkColor="#1a1a1a"; cmdColor="#fff"; break;
        case 'commodore': 
        case 'commodore64': 
        case 'c64':   thisColor = "#CCF";     document.getElementById("terminal").style.background="#006"; bkColor="#CCF"; cmdColor="#CCF"; break;}
        
        document.querySelector(':root').style.setProperty("--bgcolor",bkColor); document.querySelector(':root').style.setProperty("--textcolor",thisColor); document.querySelector(':root').style.setProperty("--commandcolor",cmdColor);}}
    function addDicts() /* yep, I hid more down here */ {let result= 
                 { "latin":{"retrocedere":"back","ostende":"display","imaginum":"images","imago":"image","purgare":"clear","terminus":"exit","sequi":"follow","progredi":"forward","grep":"grep","auxilium":"help","admonitus":"hint","nexuum":"links","plus":"more","optiones":"options","refice":"reload","versionum":"versions","ubi":"where"},
                 "short":{"<":"back",".":"display","*":"load","^":"exit","%":"images",",":"image","-":"clear","!":"follow",">":"forward","/":"grep","?":"help",";":"hint","+":"links","&":"more","~":"options","=":"reload","#":"versions","@":"where"},
                 "pirate":{"astern":"back","board":"load","hoist":"display","jettison":"clear","avast":"exit","tack":"follow","mate":"image","mateys":"images","ahead":"forward","plunder":"grep","shanties":"help","dubloon":"hint","booty":"links","spyglass":"more","armaments":"options","arrrr":"reload","manifest":"versions","ahoy":"where"}};return result;}
   
      function calcAssistance() {
    // Create full-screen overlay
    const overlay = document.createElement('div');
    overlay.style.position = 'fixed';
    overlay.style.top = '0';
    overlay.style.left = '0';
    overlay.style.width = '100%';
    overlay.style.height = '100%';
    overlay.style.backgroundColor = 'black';
    overlay.style.zIndex = '9999';
    overlay.style.display = 'flex';
    overlay.style.flexDirection = 'column';
    overlay.style.alignItems = 'center';
    overlay.style.justifyContent = 'center';
    overlay.style.fontFamily = 'monospace';
    overlay.style.color = 'white';

    // container
    const contentDiv = document.createElement('div');
    contentDiv.style.width = '80%';
    contentDiv.style.maxWidth = '800px';
    contentDiv.style.textAlign = 'center';
    overlay.appendChild(contentDiv);

    // num display
    const scoreDisplay = document.createElement('div');
    scoreDisplay.style.marginBottom = '10px';
    scoreDisplay.innerText = 'Score: 0';
    contentDiv.appendChild(scoreDisplay);

    // board
    const board = document.createElement('pre');
    board.style.margin = '0';
    board.style.lineHeight = '1';
    board.style.backgroundColor = 'black';
    board.style.border = '1px solid white';
    board.style.padding = '10px';
    contentDiv.appendChild(board);

    // Instructions
    const instructions = document.createElement('div');
    instructions.innerHTML = 'Use arrow keys to move<br>Press ESC to exit';
    instructions.style.marginTop = '20px';
    contentDiv.appendChild(instructions);

    // Add overlay to document
    document.body.appendChild(overlay);

    // state
    const width = 40;
    const height = 20;
    let snArray = [{x: Math.floor(width/2), y: Math.floor(height/2)}];
    let direction = {x: 1, y: 0};
    let targeted = {};
    let score = 0;
    let ended = false;
    let timingLoop;

    // characters
    const START_TRACE = '@';
    const END_TRACE = 'O';
    const TARGET = '*';
    const EMPTY = ' ';
    const BORDER = '#';

    // Initialize targeted position
    createTarget();

    // Handle keyboard input
    document.addEventListener('keydown', handleKeyPress);

    // Start loop
    timingLoop = setInterval(update, 150);

    // Function to place targeted at random position
    function createTarget() {
      targeted = {
        x: Math.floor(Math.random() * (width - 2)) + 1,
        y: Math.floor(Math.random() * (height - 2)) + 1
      };

      // Make sure targeted doesn't spawn on snArray
      for (const segment of snArray) {
        if (segment.x === targeted.x && segment.y === targeted.y) {
          createTarget();
          break;
        }
      }
    }

    // Function to update state
    function update() {
      if (ended) return;

      // Calculate new head position
      const head = {
        x: snArray[0].x + direction.x,
        y: snArray[0].y + direction.y
      };

      // Check for collisions
      if (
        head.x < 0 || head.x >= width ||
        head.y < 0 || head.y >= height ||
        snArray.some(segment => segment.x === head.x && segment.y === head.y)
      ) {
        ended = true;
        clearInterval(timingLoop);
        board.innerText = `\n\n\n       ACTIVITY COMPLETED\n\n    Your score: ${score}\n\n  Press any key to exit`;
        document.addEventListener('keydown', exitStage, {once: true});
        return;
      }

      // Add new head
      snArray.unshift(head);

      // Check if targeted eaten
      if (head.x === targeted.x && head.y === targeted.y) {
        score += 100000;
        scoreDisplay.innerText = `Score: ${score}`;
        createTarget();
      } else {
        // Remove tail if no targeted acquired
        snArray.pop();
      }

      // Render 
      render();
    }

    // Function to render 
    function render() {
      let grid = Array(height).fill().map(() => Array(width).fill(EMPTY));

      // Draw border
      for (let x = 0; x < width; x++) {
        grid[0][x] = BORDER;
        grid[height-1][x] = BORDER;
      }
      for (let y = 0; y < height; y++) {
        grid[y][0] = BORDER;
        grid[y][width-1] = BORDER;
      }

      // Draw targeted
      grid[targeted.y][targeted.x] = TARGET;

      // Draw snArray
      for (let i = 1; i < snArray.length; i++) {
        grid[snArray[i].y][snArray[i].x] = END_TRACE;
      }
      grid[snArray[0].y][snArray[0].x] = START_TRACE;

      // Update board
      board.innerText = grid.map(row => row.join('')).join('\n');
    }

    // Function to handle key presses
    function handleKeyPress(e) {
      switch (e.key) {
        case 'ArrowUp':
          if (direction.y === 0) direction = {x: 0, y: -1};
          break;
        case 'ArrowDown':
          if (direction.y === 0) direction = {x: 0, y: 1};
          break;
        case 'ArrowLeft':
          if (direction.x === 0) direction = {x: -1, y: 0};
          break;
        case 'ArrowRight':
          if (direction.x === 0) direction = {x: 1, y: 0};
          break;
        case 'Escape':
          exitStage();
          break;
      }
    }

    // Function to exit game and remove overlay
    function exitStage() {
      clearInterval(timingLoop);
      document.removeEventListener('keydown', handleKeyPress);
      document.body.removeChild(overlay);
    }

    // Initial render
    render();
  }

  
    
 async function initialize() {

    toggleInputLine(false);
    let startUrl = document.referrer;
    if (!startUrl || startUrl.split('/')[2].split(':')[0].split(':')[0]!="michaelkupietz.com"){
        startUrl = window.location.origin;
    }
    originalURL = startUrl;
    await doCommand("load")(theseOptions["starturl"]?theseOptions["starturl"]:startUrl);
   /* addOutput("Enter 'display' to view the page or 'links' to see available links."); */
    toggleInputLine(true);
}
    
    
     async function checkForUpdates() {
            try {
                // Get the current version history from the page
                const currentScript = document.getElementById('global-settings');
                const currentVersionMatch = currentScript.textContent.match(/(let|var|const) versionhistory=`([\s\S]*?)`[ \r\t]*;/)||currentScript.textContent.match(/(let|var|const) versionhistory='([\s\S]*?)'[ \r\t]*;/)||currentScript.textContent.match(/(let|var|const) versionhistory="([\s\S]*?)"[ \r\t]*;/);
                const currentVersion = currentVersionMatch ? currentVersionMatch[2] : '';
console.log("local version",currentVersion);
                // Fetch the latest version from GitHub
                const response = await fetch(`https://raw.githubusercontent.com/${githubInfo.username}/${githubInfo.reponame}/${githubInfo.branch}/${githubInfo.filename}`);
                const text = await response.text();
                
                // Extract version history from the fetched content
                const latestVersionMatch = text.match(/(let|var|const) versionhistory=`([\s\S]*?)`[ \r\t]*;/)||text.match(/(let|var|const) versionhistory='([\s\S]*?)'[ \r\t]*;/)||text.match(/(let|var|const) versionhistory="([\s\S]*?)"[ \r\t]*;/);
                const latestVersion = latestVersionMatch ? latestVersionMatch[2] : '';
console.log("latestVersion",latestVersion);
                // Compare versions
                if (githubInfo.updateChecking && latestVersion && currentVersion !== latestVersion) {
                   /*  document.getElementById('updateBanner').style.display = 'block'; */
                addOutput ("\nThe version of this browser on Github is different from this local version. You may want to check the repo at "+`https://github.com/${githubInfo.username}/${githubInfo.reponame}/${githubInfo.filename}`+" and download the more recent version if necessary.\n");
                }
            } catch (error) {
              if (githubInfo.updateChecking) {  addOutput ("There was an error checking for updates: "+error+"\n");}
                console.log('Error checking for updates:', error);
            }
        }


function doCommand(command) {  /* this is a command interpreter to pick the appropriate command structure to return as I transition to using objects containing manpage and other meta info instead of just functions for the commands. */
return commands[command]["do"] || commands[command] ; }

        initialize();
     
        // Check for updates every hour
        checkForUpdates();
     /*   setInterval(checkForUpdates, 60 * 60 * 1000); */
    
    </script>
</body>
</html>
