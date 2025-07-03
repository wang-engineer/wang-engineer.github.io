---
layout: default
title: "2025-07-02: Displaying Git Branch in Your Shell Prompt"
tags: [Git]
permalink: /blogger/posts/2025-07-02-display-git-branch/
---

# Displaying Git Branch in Your Shell Prompt

When you're deep in a Git project, constantly checking which branch you're on with `git branch` can slow you down. Wouldn't it be awesome if your shell prompt could show the current branch name, highlighted in a bright, eye-catching color? This simple tweak saves time and keeps you focused on coding. In this friendly guide, we'll show you how to set up your shell prompt to display the Git branch in <span style="color: cyan;">cyan</span> for Zsh, Bash, and PowerShell. We'll provide the code, explain how to configure it, break down each line, and show example outputs so you know exactly what to expect.

## Zsh Setup

### Configuration File
To customize the Zsh prompt, modify the `~/.zshrc` file, which is the configuration file for Zsh.

### Code
Add the following code to your `~/.zshrc`:

```zsh
function parse_git_branch() {
    git branch 2> /dev/null | sed -n -e 's/^\* \(.*\)/[\1]/p'
}
COLOR_DEF=$'%f'
COLOR_GIT=$'%F{51}'
setopt PROMPT_SUBST
export PROMPT='%n@%m %1~ ${COLOR_GIT}$(parse_git_branch)${COLOR_DEF} %# '
```

### Line-by-Line Explanation
- `function parse_git_branch() { ... }`: Defines a function named `parse_git_branch` to retrieve the current Git branch.
- `git branch 2> /dev/null`: Runs the `git branch` command and redirects error output (e.g., when not in a Git repository) to `/dev/null` to suppress it.
- `| sed -n -e 's/^\* \(.*\)/[\1]/p'`: Pipes the output to `sed`, which extracts the current branch (marked with `*`) and wraps it in square brackets (e.g., `[main]`). The `-n` flag suppresses default output, and `p` prints only matching lines.
- `COLOR_DEF=$'%f'`: Sets the default color reset code to restore the terminal’s default text color.
- `COLOR_GIT=$'%F{51}'`: Sets the color for the Git branch to <span style="color: cyan;">cyan</span> (color code 51 in Zsh’s 256-color palette).
- `setopt PROMPT_SUBST`: Enables command substitution in the prompt, allowing the `parse_git_branch` function to be evaluated each time the prompt is displayed.
- `export PROMPT='%n@%m %1~ ${COLOR_GIT}$(parse_git_branch)${COLOR_DEF} %# '`: Defines the prompt format:
  - `%n`: Username.
  - `@`: Literal `@` symbol.
  - `%m`: Hostname (machine name).
  - `%1~`: Current directory (abbreviated home directory with `~`).
  - `${COLOR_GIT}$(parse_git_branch)${COLOR_DEF}`: Displays the Git branch in <span style="color: cyan;">cyan</span>, resetting the color afterward.
  - `%#`: Displays `%` for regular users or `#` for root.

### Setup Instructions
1. Open your `~/.zshrc` file in a text editor (e.g., `nano ~/.zshrc`).
2. Append the code above to the file.
3. Save and close the file.
4. Reload the Zsh configuration by running `source ~/.zshrc` or restarting your terminal.

### Example Output
If you're in the `~/your-project` directory on the `main` branch, your prompt will look like this (with `[main]` in <span style="color: cyan;">cyan</span>):

```
user@machine ~/your-project [main] %
```

## Bash Setup

### Configuration File
To customize the Bash prompt, modify the `~/.bashrc` file (or `~/.bash_profile` on some systems).

### Code
Add the following code to your `~/.bashrc`:

```bash
parse_git_branch() {
  git branch 2>/dev/null | sed -n -e 's/^\* \(.*\)/[\1]/p'
}
COLOR_DEF="\[\033[0m\]"
COLOR_GIT="\[\033[38;5;51m\]"
export PS1="\u@\h \w ${COLOR_GIT}\$(parse_git_branch)${COLOR_DEF} \$ "
```

### Line-by-Line Explanation
- `parse_git_branch() { ... }`: Defines a function to get the current Git branch.
- `git branch 2>/dev/null`: Runs `git branch` and suppresses errors by redirecting them to `/dev/null`.
- `| sed -n -e 's/^\* \(.*\)/[\1]/p'`: Uses `sed` to extract the current branch and wrap it in brackets (e.g., `[main]`).
- `COLOR_DEF="\[\033[0m\]"`: Resets the terminal text color using ANSI escape codes.
- `COLOR_GIT="\[\033[38;5;51m\]"`: Sets the Git branch color to <span style="color: cyan;">cyan</span> using ANSI 256-color code 51.
- `export PS1="\u@\h \w ${COLOR_GIT}\$(parse_git_branch)${COLOR_DEF} \$ "`: Defines the prompt:
  - `\u`: Username.
  - `\h`: Hostname.
  - `\w`: Current directory (full path).
  - `${COLOR_GIT}\$(parse_git_branch)${COLOR_DEF}`: Displays the Git branch in <span style="color: cyan;">cyan</span>, resetting the color afterward.
  - `\$`: Displays `$` for regular users or `#` for root.

### Setup Instructions
1. Open your `~/.bashrc` file in a text editor (e.g., `nano ~/.bashrc`).
2. Append the code above to the file.
3. Save and close the file.
4. Reload the Bash configuration by running `source ~/.bashrc` or restarting your terminal.

### Example Output
If you're in the `~/your-project` directory on the `main` branch, your prompt will look like this (with `[main]` in <span style="color: cyan;">cyan</span>):

```
user@machine ~/your-project [main] $
```

## PowerShell Setup

### Configuration File
To customize the PowerShell prompt, modify your PowerShell profile file. On Windows, this is typically located at `~\Documents\WindowsPowerShell\profile.ps1` or `~\Documents\PowerShell\profile.ps1` (for PowerShell 7). If the profile file doesn’t exist, you can create it.

### Code
Add the following code to your PowerShell profile file:

```powershell
function Get-GitBranch {
    try {
        $branch = git branch --show-current 2>$null
        if ($branch) {
            return "[$branch]"
        }
        return ""
    } catch {
        return ""
    }
}
$COLOR_DEF = "`e[0m"
$COLOR_GIT = "`e[38;5;51m"
function Prompt {
    $user = $env:USERNAME
    $host = $env:COMPUTERNAME
    $path = (Get-Location).Path
    $gitBranch = Get-GitBranch
    "${user}@${host} ${path} ${COLOR_GIT}${gitBranch}${COLOR_DEF} $ "
}
```

### Line-by-Line Explanation
- `function Get-GitBranch { ... }`: Defines a function to retrieve the current Git branch.
- `try { ... } catch { ... }`: Wraps the Git command in a try-catch block to handle errors gracefully (e.g., when not in a Git repository).
- `$branch = git branch --show-current 2>$null`: Uses `git branch --show-current` to get the current branch name, redirecting errors to `$null`.
- `if ($branch) { return "[$branch]" }`: If a branch is found, wraps it in brackets (e.g., `[main]`).
- `return ""`: Returns an empty string if no branch is found or an error occurs.
- `$COLOR_DEF = "`e[0m"`: Defines the ANSI escape code to reset text color.
- `$COLOR_GIT = "`e[38;5;51m"`: Defines the <span style="color: cyan;">cyan</span> color for the Git branch using ANSI 256-color code 51.
- `function Prompt { ... }`: Overrides the default PowerShell prompt function.
- `$user = $env:USERNAME`: Gets the username from the environment variable.
- `$host = $env:COMPUTERNAME`: Gets the hostname from the environment variable.
- `$path = (Get-Location).Path`: Gets the full current directory path, using Windows-style paths (e.g., `C:\Users\user\your-project`).
- `$gitBranch = Get-GitBranch`: Calls the `Get-GitBranch` function to get the branch name.
- `"${user}@${host} ${path} ${COLOR_GIT}${gitBranch}${COLOR_DEF} $ "`: Constructs the prompt:
  - Username and hostname (e.g., `user@machine`).
  - Full directory path (e.g., `C:\Users\user\your-project`).
  - Git branch in <span style="color: cyan;">cyan</span> (e.g., `[main]`), if available.
  - A `$` symbol at the end.

### Setup Instructions
1. Determine your PowerShell profile file location by running `echo $PROFILE` in PowerShell.
2. If the file doesn’t exist, create it with `New-Item -Path $PROFILE -Type File -Force`.
3. Open the profile file in a text editor (e.g., `notepad $PROFILE`).
4. Append the code above to the file.
5. Save and close the file.
6. Reload the profile by running `. $PROFILE` or restarting PowerShell.

### Example Output
If you're in the `C:\Users\user\your-project` directory on the `main` branch, your prompt will look like this (with `[main]` in <span style="color: cyan;">cyan</span>):

```
user@machine C:\Users\user\your-project [main] $
```

## Summary
This tutorial showed you how to add a splash of color and functionality to your shell prompt by displaying the current Git branch in <span style="color: cyan;">cyan</span> for Zsh, Bash, and PowerShell. We provided the code for each shell, explained how to set it up in the appropriate configuration files (`~/.zshrc`, `~/.bashrc`, or the PowerShell profile), broke down each line, and included example outputs to show what your prompt will look like in action. Note that you may need to tweak the code slightly to fit your environment, such as adjusting the color codes (e.g., changing 51 to another number for a different color) or prompt format to match your preferences or terminal settings. With these steps, you can streamline your workflow and keep your Git branch front and center, all while adding a touch of style to your terminal!