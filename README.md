# File-Integrity-Monitor-Project

The Mission: In this lab we will create File Integrity Monitor (FIM) in Windows PowerShell to confirm the integrity of files to uphold the confidentiality and integrity of files in the CIA Triad.

The process will look like this at a high-level. The FIM will first ask what the user wants to do. A. Collect a new Baseline?
B. Begin monitoring files with saved Baseline?

1. Collect New Baseline
A. Calculate HASH value from target files
B. Store the file hash pairs in baseline.txt

2. Begin monitoring files with saved Baseline
A. Load file hash pairs from baseline.txt

2A. Continuously monitor file integrity
A If a file's hash is detected as different than the recorded baseline, print to screen (color), if a file has been changed or deleted (integrity is compromised)

Let's break down this script into sections and explain it step by step in a simple way.
-------------------------------------------------------------------
Phase One. Function: Calculate-File-Hash

Function Calculate-File-Hash($filepath) {
    $filehash = Get-FileHash -Path $filepath -Algorithm SHA512
    return $filehash
}

What it does:
This function takes a file's path as input and calculates its hash using the SHA-512 algorithm. A hash is like a unique digital fingerprint for the file. This helps check if the file has been altered later.
---------------------------------------------------------------------------------------
Phase Two. Function: Erase-Baseline-If-Already-Exists

Function Erase-Baseline-If-Already-Exists() {
    $baselineExists = Test-Path -Path .\baseline.txt

    if ($baselineExists) {
        Remove-Item -Path .\baseline.txt
    }
}

What it does:
This function checks if a file called baseline.txt exists in the current folder. If it does, the function deletes it. This is to make sure we start fresh with a new baseline.
-----------------------------------------------
Phase Three. User Interaction 

Write-Host "What would you like to do?"
Write-Host "    A) Collect new Baseline?"
Write-Host "    B) Begin monitoring files with saved Baseline?"
$response = Read-Host -Prompt "Please enter 'A' or 'B'"

What it does:
The script asks the user what they want to do:

Option A: Create a new baseline.
Option B: Start monitoring files using an existing baseline. The user's input is stored in the variable $response.
--------------------------------------------------------
Phase Four. Option A: Collect New Baseline

if ($response -eq "A".ToUpper()) {
    Erase-Baseline-If-Already-Exists

    $files = Get-ChildItem -Path .\Files

    foreach ($f in $files) {
        $hash = Calculate-File-Hash $f.FullName
        "$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append
    }
}

What it does:

If the user chooses "A", the script deletes the old baseline.
It then collects all files in the Files folder, calculates each file's hash, and saves this information in baseline.txt.
Each line in baseline.txt has the file's path and its hash, separated by a |.
----------------------------------------------------------------
Phase Five. Option B: Begin Monitoring

elseif ($response -eq "B".ToUpper()) {
    $fileHashDictionary = @{}

    $filePathsAndHashes = Get-Content -Path .\baseline.txt
    
    foreach ($f in $filePathsAndHashes) {
         $fileHashDictionary.add($f.Split("|")[0],$f.Split("|")[1])
    }

What it does:

If the user chooses "B", the script loads the baseline data from baseline.txt into a dictionary (fileHashDictionary), where each file's path is the key and its hash is the value.
---------------------------------------------
Phase Six. Monitoring Files

    while ($true) {
        Start-Sleep -Seconds 1
        $files = Get-ChildItem -Path .\Files

        foreach ($f in $files) {
            $hash = Calculate-File-Hash $f.FullName

            if ($fileHashDictionary[$hash.Path] -eq $null) {
                Write-Host "$($hash.Path) has been created!" -ForegroundColor Green
            }
            else {
                if ($fileHashDictionary[$hash.Path] -eq $hash.Hash) {
                    # The file has not changed
                }
                else {
                    Write-Host "$($hash.Path) has changed!!!" -ForegroundColor Yellow
                }
            }
        }

        foreach ($key in $fileHashDictionary.Keys) {
            $baselineFileStillExists = Test-Path -Path $key
            if (-Not $baselineFileStillExists) {
                Write-Host "$($key) has been deleted!" -ForegroundColor DarkRed -BackgroundColor Gray
            }
        }
    }
}

What it does:

The script continuously monitors the files in the Files folder.
It checks if any files have been added, changed, or deleted compared to the baseline:
New file: If a file isn’t in the baseline, it prints a message that a new file was created.
Changed file: If a file’s hash doesn’t match the baseline, it prints that the file has changed.
Deleted file: If a file in the baseline is missing, it prints that the file was deleted.

In Summary
Collect Baseline: Stores the "fingerprints" of all files.
Monitor Files: Watches for changes, additions, or deletions, alerting the user.
This script is useful for tracking if files in a folder are modified, added, or deleted over time.
