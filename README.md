# anki-project

This is a project in Haskell using Codespaces. I will connect to a word
synonyms API to generate synonyms to a given word and input those synonyms
into an Anki study set.

## Haskell Extension Setup
Once I created this project using Stack, I installed the Haskell extension in
VScode to enable syntax highlighting and to see syntax errors through the HLS.
Initially, I ran into difficulty while trying to run the HLS. I resolved the
issues by adjusting certain Haskell settings in the settings.json file.

The first error I encountered with the extension occurred after I refreshed my
VSCode window once I installed the extension. The error message said:
"Project requires GHCup but it isn't installed". This was unexpected because
installing GHCup was the very first step I took after beginning this project.
Clearly, the extension could not find GHCup. The only GitHub issue related to
this error is detailed here: https://github.com/haskell/haskell-language-server/issues/3032
A user in the thread resolved this issue by altering their settings.json file
to (1) use the path for HLS management, (2) specify an explicit GHCup location,
and (3) specify an explicit HLS location. I tried this approach, changing the
locations to the correct locations on my codespace:
```json
{
    "haskell.manageHLS": "PATH",
    "haskell.serverExecutablePath": "~/.ghcup/bin/haskell-language-server-9.6.1~1.10.0.0",
    "haskell.ghcupExecutablePath": "~/.ghcup/bin/ghcup"
}
```

Unfortunately, this solution led to the second error.

The second error I encountered displayed the message "Connection to server got 
closed. Server will restart." in the output. After revisiting the thread above,
I realized that the reason for this was that I was directly invoking the language
server binary instead of the wrapper binary. I modified my solution to:
```json
{
    "haskell.manageHLS": "PATH",
    "haskell.serverExecutablePath": "~/.ghcup/bin/haskell-language-server-wrapper-1.10.0.0",
    "haskell.ghcupExecutablePath": "~/.ghcup/bin/ghcup"
}
```
This too led to an extension error: "Failed to find executable "ghc" in $PATH for 
this Default project."

Fortunately, this was a bug that seemed like it could be resolved. There was previous documentation 
on the fix written on the GitHub page for this extension: https://github.com/haskell/vscode-haskell#stackcabalghc-can-not-be-found
I added the "haskell.serverEnvironment" property to my settings file and retested:
```json
{
    "haskell.manageHLS": "PATH",
    "haskell.serverExecutablePath": "~/.ghcup/bin/haskell-language-server-wrapper-1.10.0.0",
    "haskell.ghcupExecutablePath": "~/.ghcup/bin/ghcup",
    "haskell.serverEnvironment": {
        "PATH": "${HOME}/.ghcup/bin:$PATH"
    }
}
```

Still, the issue persisted. "ghc" could not be found. I later realized that the
"manageHLS" property was supposed to be "GHCup" to give GHCup permission to
provide any binaries (ie. ghc and the HLS) to the extension. This meant that I needed
to change the "manageHLS" property from "PATH" to "GHCup". In addition, since
the initial error I ran into was that the extension could not find GHCup, I still 
needed to explicitly provide the location of GHCup in the settings file. I 
hypothesized that I either needed (1)  the "ghcupExecutablePath" property or 
(2) the "serverEnvironment" property, so I tested both. I tried the "ghcupExecutablePath" 
property first and removed the "serverEnvironment" property:
```json
{
    "haskell.manageHLS": "GHCup",
    "haskell.ghcupExecutablePath": "~/.ghcup/bin/ghcup",
}
```

If you evaluate a solution based on whether it produces errors in the output,
this attempt will fool you. When I ran this solution, the HLS loaded up fine,
but it displayed an error in on the first line of every Haskell file. The
error said: "<command line>: cannot satisfy -package anki-project-0.1.0.0 (use 
-v for more information)"

I didn't know how to interpret this message, so I could not debug it. I decided
to try to second settings configuration, which kept the "serverEnvironment"
property but discarded the "ghcupExecutablePath" property:
```json
{
     "haskell.manageHLS": "GHCup",
    "haskell.serverEnvironment": {
        "PATH": "${HOME}/.ghcup/bin:$PATH"
    }
}
```

At last, this solution worked. The HLS initialized without running into any
errors, and the cryptic message at the header of each file was gone. It appeared 
to underline basic syntax errors correctly, so I was convinced that the solution
worked!