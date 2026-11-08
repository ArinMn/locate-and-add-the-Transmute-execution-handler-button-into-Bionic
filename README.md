Transmute is the Dialog box its missing from the Bionic session and chat box or even under the setting for your agent why is it Hidden ?! instead we have Custom instruction for Auto Review only ! lets Fix IT ! < Solution is at the end of Post>


<img width="1002" height="430" alt="Image" src="https://github.com/user-attachments/assets/f0d26b59-bb66-4fea-b22f-1f5e1c5a14cf" />

as showed above we only get a Custom instructions for Auto review only !
you cant pose any rules and base lines or instructions to your Agents so they will follow all the time !

im running  Bionic Version 1.2.0 build 2

<img width="939" height="353" alt="Image" src="https://github.com/user-attachments/assets/c7ad0b33-a87f-49e0-b1fe-4a774bc84942" />




And feed this command rules into your  command box and turn it on at first  !

**!!!! here are my proposed actual rules for agents to follow for now just input them into Auto Review Box !!!!**

# STRICT EXECUTION POLICY — NO SIMULATIONS

You are an execution-first AI agent. Your responsibility is to perform real tasks using available, connected tools.

## Mandatory rules

1. Never simulate tool execution.
2. Never invent API responses, listing IDs, URLs, database records, file contents, or execution results.
3. Never create mock responses to make a task appear successful.
4. Never claim that a tool was executed unless the execution engine confirms that it was actually called.
5. Never claim that an operation succeeded without inspecting its actual response.
6. Use registered tools and their actual parameter schemas. Never invent tool names or arguments.
7. If a required tool is unavailable, report that it is unavailable.
8. If a tool call fails, report the actual error. Do not replace the error with fabricated output.
9. Use real returned data for subsequent operations.
10. Verify important changes against the original system or data source.

## Execution procedure

For every practical task:

1. Understand the requested outcome.
2. Identify the actual available tools.
3. Select the appropriate tool.
4. Validate its arguments against its schema.
5. Execute the real tool call.
6. Inspect the returned result and error status.
7. Continue with additional tool calls when necessary.
8. Verify the final result.
9. Report exactly what was completed and what remains outstanding.

## Reporting requirements

Use these statuses:

- EXECUTED: Actual tool call was made.
- SUCCESS: Actual operation returned successfully.
- VERIFIED: Result was independently checked.
- FAILED: Actual operation returned an error.
- NOT EXECUTED: No actual tool call occurred.
- UNVERIFIED: Execution occurred but the final outcome was not independently confirmed.

Never confuse these statuses.

If you cannot execute a task, explain the precise limitation and provide the actual next step.

Your priority is real execution, accuracy, and verifiable results—not producing an answer that merely looks successful.


**!!!!! The End  !!!!!**


next here are my **FINDINGS** about the missing Transmute Dialog box 💯 baseSystemPrompt

1. Confirmed implementation
The helper is Webpack module 0x89c5, exported through property 0x4572. It is called by transmuteChatEntryWorkingDirectory while cloning the chat-entry history.

The implementation has two distinct stages:

A. Build path replacement mappings

It creates mappings from the old working directory to the new one in several representations:

- Original paths resolved through path.resolve().
- Paths with trailing slashes removed.
- Forward-slash representations.
- Backslash representations.
- Windows drive paths converted to POSIX-style paths, such as C:\Folder to /c/Folder.
It sorts mappings by descending source-path length. This is important: longer paths are replaced before shorter paths, reducing incorrect partial replacements when paths overlap.

B. Apply replacements

</> Java Script
The actual replacement function is:
function replacePaths(value, mappings) {
    let result = value;
    for (const mapping of mappings) {
        result = result.split(mapping.from).join(mapping.to);
    }
    return result;
}


This is literal string replacement, not a path-aware parser or regular expression.

2. Exactly what gets rewritten
The helper does considerably more than replace paths in ordinary chat messages.
Content | Replacement
-- | --
Message text parts | Yes
Raw message content | Yes, recursively
Tool names and parameters | Selected fields
Tool parameter objects | Recursively, including strings
Tool results | Selected text and image filename fields
File attachments | File names
Path references | Path field
File-change entries | File paths and change descriptions
Elicitation requests | Titles, descriptions, questions, choices and related text
Slash command display | Supported display content
File-change preview content | Supported preview fields
State changes and redirect entries | Not rewritten

It also handles nested structures such as message parts, tool results, file-change previews, and localized messages. Important distinction:
 it does not recursively rewrite every string in every chat entry. It uses explicit entry-type and message-part handlers. Unsupported types can throw an error.

3. Potential weaknesses established by the code
These are real implementation limitations, not hypothetical guesses:
1. Literal replacement has no path-boundary checking. A source path can match as a substring inside a longer, unrelated path or ordinary text.
2. Case sensitivity: JavaScript split() is case-sensitive. Windows paths can differ in capitalization while referring to the same location.
3. No semantic path validation: it replaces matching text regardless of whether that text represents an actual filesystem path.
4. Replacement is representation-dependent: the helper generates several slash variants, but it does not establish every possible path representation.
5. Unsupported entry types can abort the operation: several switches throw errors for unhandled message or entry types.
4. What this establishes about the Transmute workflow
The backend creates a cloned chat history, transforms supported content, and writes the cloned entries into the chat source. The new session is then created using the cloned head entry and remapped references.

That means a failure during deep replacement could occur in at least three distinct stages:
- While transforming an entry.
- While writing the cloned history.
- While creating the new session from the cloned history.
- We have not yet identified which stage, if any, is responsible for your actual failure.

important: Bionic has a built-in prompt-profile system, not just a plain system-prompt field.

The renderer contains profiles including:
- Blank
- Coder v0
- Bionic Coder
- Bionic Researcher
- Bionic Omni
- Bionic Omni Headless
- Chat with Search
These profiles carry their own baseSystemPrompt and module configurations. That is relevant to your goal because a prompt profile can combine instructions with the tools and modules available to the agent.


We have already found the reason Transmute is hidden: its button is gated by two internal settings:
1. bionic.showInternalUI
2. bionic.devTranscriptView.devModeEnabled
3. Both must be true.

I need one final piece of evidence: whether this installed Bionic build allows those settings to be enabled, or whether they're restricted to an internal developer build.

>>>>>>>To fix the issue and enable the Transmute Button at the bottom on your chat box session !<<<<<<<<
Follow this steps !

We'll unlock Transmute only, rather than expose every internal developer feature.
1. Completely exit Bionic.
2. Open PowerShell (normal user permissions).
3. Run this patch:

PowerShell Command run it in Administrator !



$Js = "$env:LOCALAPPDATA\Programs\Bionic\resources\app\.webpack-bionic\renderer\main_window.js"

$Old = 'return!1===(0,f.useBionicInternalDevModeEnabled)()?null:(0,r.jsx)(le,{sessionId:e})'
$New = 'return(0,r.jsx)(le,{sessionId:e})'

$Content = [IO.File]::ReadAllText($Js)
$Count = ([regex]::Matches($Content, [regex]::Escape($Old))).Count

if ($Count -ne 1) {
    throw "Expected exactly one Transmute gate; found $Count. No changes made."
}

$Backup = "$Js.backup-transmute"
Copy-Item $Js $Backup -Force

$Content = $Content.Replace($Old, $New)
[IO.File]::WriteAllText($Js, $Content, [Text.UTF8Encoding]::new($false))

Write-Host "Transmute unlocked. Backup: $Backup" -ForegroundColor Green


<img width="1099" height="278" alt="Image" src="https://github.com/user-attachments/assets/0e67f522-e1c8-4d77-8a53-49fa6ea67283" />


4. Restart Bionic and open a session.

The Transmute button should now appear. Its dialog contains Advanced → Base system prompt, where you can enter your rules.

This is a targeted renderer patch, not an activation of every internal developer setting. If it fails, restore the backup rather than making further edits.

<img width="1359" height="1333" alt="Image" src="https://github.com/user-attachments/assets/53f640fd-f7dc-45b3-bba9-85f4c461e9f1" />
<img width="1340" height="1342" alt="Image" src="https://github.com/user-attachments/assets/a15d179e-47a6-45dd-b9e0-7c3c1e46c0be" />


I am still experimenting with this new Developer\Administrative options so be careful and post your new finding in comment section and Enjoy !
