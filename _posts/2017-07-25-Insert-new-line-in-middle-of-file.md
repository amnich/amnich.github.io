---
title: "2017-07-25 Insert new line in middle of file"
date: 2017-07-25
tags: [PowerShell]
---

A simple way to add new line to a text file after finding a line matches a pattern.    
    
    $fileName = "D:\PoSH\file.txt"
    $pattern = "Some pattern to look for"  # example "\[Wrapped Scripts\]" 
    $newLine = "New line"
    (Get-Content $fileName) | 
        Foreach-Object {
            $_ # send the current line to output
            if ($_ -match $pattern) 
            {
                #Add Line after the selected pattern 
                $newLine
            }
        } | Set-Content $fileName
