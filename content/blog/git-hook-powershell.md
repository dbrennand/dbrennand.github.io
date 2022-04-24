---
title: "Using Git Hooks to lint PowerShell"
date: 2022-04-24T15:55:08+01:00
draft: false
tags: ["PowerShell", "Git", "Hook", "Pre-Commit", "Lint", "Linters"]
showToc: true
---

Hi there! 👋

Recently I discovered [Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks). Git Hooks provide a way of running custom scripts when a certain git action occurs. In this post, I want to share a pre-commit Git Hook I've written to lint PowerShell code using the [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer) module.

# What is a Linter? 🕵️‍♂️

A linter analyses code to identify common errors, bugs and stylistic issues. Their aim is to improve code quality. Linters perform static analysis meaning they check code without executing it. Some well known linters for other languages include ESLint and Pylint.

# What is PSScriptAnalyzer?

PSScriptAnalyzer is a linter for PowerShell modules and scripts. It runs a set of rules against PowerShell code. These rules are based on best practices identified by the PowerShell team at Microsoft and the community. The full set of rules can be found [here](https://github.com/PowerShell/PSScriptAnalyzer/tree/master/docs/Rules).

# What is a pre-commit Git Hook?

A pre-commit Git Hook executes when running `git commit`. The contents of the `pre-commit` file located at `.git/hooks/pre-commit` is executed and if the script has an exit code of 1 (job failed), then the commit is aborted. Pre-commit hooks provide an excellent use case for linting code changes before pushing to a remote repository.

# Linting PowerShell code using the pre-commit Git Hook 🔎

To use this pre-commit Git Hook you must have PowerShell 7, Git and the PSScriptAnalyzer module installed.

Install PSScriptAnalyzer:

```PowerShell
Install-Module -Name "PSScriptAnalyzer" -Verbose
```

## Pre-commit Git Hook

```PowerShell
#!/usr/bin/env pwsh
Import-Module -Name "PSScriptAnalyzer"

$DiffNames = git --no-pager diff --name-only --staged --line-prefix="$(git rev-parse --show-toplevel)/"
$Results = @()

foreach ($DiffName in $DiffNames) {
    Write-Output -InputObject "Analysing ""$($DiffName)"""
    $Output = Invoke-ScriptAnalyzer -Path $DiffName
    $Results += $Output
}

if ($Results.Count -gt 0) {
    Write-Warning -Message "PSScriptAnalyzer identified one or more files with linting errors. Commit aborted. Fix them before committing or use 'git commit --no-verify' to bypass this check."
    foreach ($Result in $Results) {
        Write-Error -Message "$($Result.ScriptName) - Line $($Result.Line) - $($Result.Message)"
    }
    exit 1
}
```

## Installation

### Linux

1. Create `.git/hooks/pre-commit`:

    ```bash
    touch .git/hooks/pre-commit
    ```

1. Paste the PowerShell snippet above into `.git/hooks/pre-commit`

2. Make `.git/hooks/pre-commit` executable:

    ```bash
    chmod +x .git/hooks/pre-commit
    ```

### Windows

1. Create `.git/hooks/pre-commit.ps1` and `.git/hooks/pre-commit`:

    ```PowerShell
    New-Item -Path ".git\hooks\pre-commit.ps1", ".git\hooks\pre-commit" -ItemType "File"
    ```

1. Paste the PowerShell snippet above into `.git/hooks/pre-commit.ps1`

2. Paste the snippet below into `.git/hooks/pre-commit`:

    ```sh
    #!/bin/sh
    pwsh -File "$(git rev-parse --show-toplevel)\.git\hooks\pre-commit.ps1"
    ```

## Usage

Now lets see if it's working! :smile:

1. Create a directory and initialise a new repository:

    ```bash
    mkdir ~/git-hook-pwsh
    cd ~/git-hook-pwsh
    git init
    ```

2. Follow the installation steps above depending on your environment.

3. Create a file called `git-hook-pwsh.ps1` which uses `Write-Host`:

    ```bash
    echo 'Write-Host -Object "Oh no! Not the dreaded Write-Host!"' > git-hook-pwsh.ps1
    ```

4. Stage `git-hook-pwsh.ps1` and attempt to commit it:

    ```bash
    git add git-hook-pwsh.ps1; git commit -m "Is my pre-commit git hook working?"
    ```

If all goes well you should see an output similar to below in your terminal:

```bash
➜  git-hook-pwsh git:(master) ✗ git commit -m "Is my pre-commit git hook working?"
Analysing "/home/dab/git-hook-pwsh/git-hook-pwsh.ps1"
WARNING: PSScriptAnalyzer identified one or more files with linting errors. Commit aborted. Fix them before committing or use 'git commit --no-verify' to bypass this check.
Write-Error: git-hook-pwsh.ps1 - Line 1 - File 'git-hook-pwsh.ps1' uses Write-Host. Avoid using Write-Host because it might not work in all hosts, does not work when there is no host, and (prior to PS 5.0) cannot be suppressed, captured, or redirected. Instead, use Write-Output, Write-Verbose, or Write-Information.
```

Awesome! It worked! :star:

I've also posted this pre-commit Git Hook as a GitHub [gist](https://gist.github.com/dbrennand/9ff23c19369d256bd043c766555ef2ec) so others can find it! :smiley:

Enjoy! :sparkles: Until next time! :smile:
