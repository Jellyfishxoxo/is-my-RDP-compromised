
## USAGE

run logs.ps1 in powershell as administrator. 
>The script contains parts which needs to be run under elevated client. To prevent errors, run the file as an administrator.

If you have set default powershell Execution Policy, add parameter -ExecutionPolicy Bypass, otherwise the script will not run at all.

__Example of usage:__

>open cmd as and administrator and run:

```
powershell -ExecutionPolicy Bypass -File logs.ps1
```


## EXPLANATION:

It is possible to pull out every script from file and run it indemepndently.

The logs.ps1 contains:

__Adding powershell capability to use Get-AdGroupMember__
```
Get-WindowsCapability -Online | Where-Object {$_.Name -like "*ActiveDirectory.DS-LDS*"} | Add-WindowsCapability -Online;
```

__Create logfiles folder in Desktop under currently signed user profile.__
```
New-Item -Path "c:\Users\$env:username\Desktop\" -Name "logfiles" -ItemType "directory";
```

__Exporting Windows/LocalSessionManager logs that contains connections info to RDP service.__

>1_RDP_log.txt
```
get-winevent -logname "Microsoft-Windows-TerminalServices-LocalSessionManager/Operational" | Export-Csv c:\Users\$env:username\Desktop\logfiles\1_RDP_log.txt -Encoding UTF8;
```

__List of active RDP sessions__
>2_RDP_sessionlist.txt
```
Qwinsta | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\2_RDP_sessionlist.txt;
```

__Details about active RDP sessions 1-10. If created files are empty, there is no active session atm.__
>3_RDP_Session.txt
```
qprocess /id:1 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session1.txt;
qprocess /id:2 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session2.txt;
qprocess /id:3 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session3.txt;
```
__OS info + account priviledges__
>4_OSinfo.txt 5_Usersandgroups 5_Usersandgroups2 6_admin_members.csv
```
(Get-WMIObject win32_operatingsystem).name, (Get-WmiObject Win32_OperatingSystem).OSArchitecture, (Get-WmiObject Win32_OperatingSystem).CSName  | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\4_OSinfo.txt;
(New-Object System.DirectoryServices.DirectorySearcher("(&(objectCategory=User)(samAccountName=$($env:username)))")).FindOne().GetDirectoryEntry().memberOf | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\5_Usersandgroups2.txt;
Get-AdGroupMember -identity "Administrators" | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\5_Usersandgroups.txt;
Get-AdGroupMember -identity "Administrators" | select name | Export-csv -path c:\Users\$env:username\Desktop\logfiles\6_admin_members.csv -NoTypeInformation;
```
__Processes and tasks__
>7_running_tasks.txt 8_all_tasks.txt 9_processes_formatlist.txt 10_processes.txt
```
TASKLIST /v /fi "STATUS eq running" | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\7_running_tasks.txt;
TASKLIST /svc | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\8_all_tasks.txt;
Get-Process | Format-List * | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\9_processes_formatlist.txt;
Get-Process | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\10_processes.txt;
```

__Netastat and Firewall__
>11_netstat.txt 12_advfirewallpolicy.wfw 12_netaccounts.txt 13_smbconnections.txt
```
netstat -anob | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\11_netstat.txt;
netsh advfirewall export "c:\Users\$env:username\Desktop\logfiles\12_advfirewallpolicy.wfw";
net accounts | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\12_netaccounts.txt;
Get-SmbConnection | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\13_smbconnections.txt;
```

__Final archive creation__
>logfiles.zip
```
Compress-Archive -Path c:\Users\$env:username\Desktop\logfiles -DestinationPath c:\Users\$env:username\Desktop\logfiles.zip;
Remove-Item -LiteralPath "c:\Users\$env:username\Desktop\logfiles" -Force -Recurse;
```
