## Basic File Integrity Monitor (FIM) in PowerShell

## Brief Objective
In this project, I reproduced a basic File Integrity Monitor (FIM) using PowerShell ISE, emulating the core 
functionality typically used in enterprise environments to monitor and ensure the integrity of critical files. The 
script employs hash-based techniques to detect unauthorized changes, deletions, or modifications of files. By 
recreating this script, I aimed to develop my understanding of FIM and its importance in security monitoring.

## Skills Learned
- Core understanding of file integrity monitoring (FIM)
- Implemented hashing algorithms in PowerShell
- Improved knowledge of cybersecurity best practices in detecting potential threats through file integrity checks
- Gained insight into security auditing techniques

## Tools Used
- Windows PowerShell ISE

## Flow Diagram
![Screenshot 2025-03-20 144327](https://github.com/user-attachments/assets/9421a39c-b7d6-4285-9c0a-11b126b53a5e)


## Script
```powershell
Write-Host ""
Write-Host "What would you like to do?"
Write-Host ""
Write-Host "A) Collect new Baseline?"
Write-Host "B) Begin monitoring files with saved Baseline?"
Write-Host ""

$response = Read-Host -Prompt "Please enter 'A' or 'B'"
Write-Host ""

Function Calculate-File-Hash($filepath) {
    $filehash = Get-FileHash -Path $filepath -Algorithm SHA512
    return $filehash
}

Function Erase-Baseline-If-Already-Exists() {
    $baselineExists = Test-Path -Path .\baseline.txt

    if ($baselineExists) {
        # Delete it
        Remove-Item -Path .\baseline.txt
    }
}

if ($response -eq "A".ToUpper()) {
    #Delete baseline.txt if it already exists
    Erase-Baseline-If-Already-Exists

    # Calculate Hash from the target files and store in baseline.txt
    
    # Collect all files in the target folder
    $files = Get-ChildItem -Path .\Files
    
    #for each file, calculate the hash, and write to baseline.txt
    foreach ($f in $files) {
        $hash = Calculate-File-Hash $f.Fullname
        "$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append
    }
}
elseif ($response -eq "B".ToUpper()) {
    
    $fileHashDictionary = @{}

    # Load file|hash from baseline.txt and store them in a dictionary
    $filePathsAndHashes = Get-Content -Path .\baseline.txt

    foreach ($f in $filePathsAndHashes) {
        $fileHashDictionary.add($f.Split("|")[0],$f.Split("|")[1])
    }

    $fileHashDictionary
    
    # Begin (continuously) monitoring files with saved Baseline
    while ($true) {
        Start-Sleep -Seconds 1
        
        $files = Get-ChildItem -Path .\Files
    
        #for each file, calculate the hash, and write to baseline.txt
        foreach ($f in $files) {
            $hash = Calculate-File-Hash $f.Fullname
            #"$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append

            #Notify if a new file has been created
            if ($fileHashDictionary[$hash.Path] -eq $null) {
                # A file has been created!
                Write-Host "$($hash.Path) has been created!" -ForegroundColor Green 
            }
            else {

                # Notify if a new file has been changed
                if ($fileHashDictionary[$hash.Path] -eq $hash.Hash) {
                    # The file has not changed
                }
                else {
                    # File file has been compromised!, notify the user
                    Write-Host "$($hash.Path) has changed!!!" -ForegroundColor Red
                }
            }
        }

        foreach ($key in $fileHashDictionary.Keys) {
                $baselineFileStillExists = Test-Path -Path $key
                if (-Not $baselineFileStillExists) {
                    # One of the baseline files must have been deleted, notify the user
                    Write-Host "$($key) has been deleted!" -ForegroundColor DarkRed
            }
        }
    }
}
```

## Demonstration
- Notification that a file has been deleted/moved out of the folder
- Notification that a file has been created
- Notification that text within the file has been changed
