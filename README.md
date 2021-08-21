
USAGE:

run logs.ps1 in powershell as administrator. The script contains parts which needs to be run under elevated client. To prevent errors, run the file as an administrator.

If you have set default powershell Execution Policy, add parameter -ExecutionPolicy Bypass, otherwise the script will not run at all.

Example of usage: open cmd as and administrator and run:

powershell -ExecutionPolicy Bypass -File logs.ps1



EXPLANATION:

It is possible to pull out every script from file and run it indemepndently.

The logs.ps1 contains:

Adding powershell ccapability to use Get-AdGroupMember
Get-WindowsCapability -Online | Where-Object {$_.Name -like "*ActiveDirectory.DS-LDS*"} | Add-WindowsCapability -Online;

Create logfiles folder in Desktop under currently signed user profile.
New-Item -Path "c:\Users\$env:username\Desktop\" -Name "logfiles" -ItemType "directory";

Exporting Windows/LocalSessionManager logs that contains connections info to RDP service.
get-winevent -logname "Microsoft-Windows-TerminalServices-LocalSessionManager/Operational" | Export-Csv c:\Users\$env:username\Desktop\logfiles\1_RDP_log.txt -Encoding UTF8;

List of active RDP sessions
Qwinsta | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\2_RDP_sessionlist.txt;

Details about active RDP sessions 1-10. If created files are empty, there is no active session atm.
qprocess /id:1 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session1.txt;
qprocess /id:2 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session2.txt;
qprocess /id:3 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session3.txt;
qprocess /id:4 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session4.txt;
qprocess /id:5 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session5.txt;
qprocess /id:6 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session6.txt;
qprocess /id:7 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session7.txt;
qprocess /id:8 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session8.txt;
qprocess /id:9 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session9.txt;
qprocess /id:10 | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\3_RDP_Session10.txt;

OS architecture info + computer name
(Get-WMIObject win32_operatingsystem).name, (Get-WmiObject Win32_OperatingSystem).OSArchitecture, (Get-WmiObject Win32_OperatingSystem).CSName  | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\4_OSinfo.txt;

(New-Object System.DirectoryServices.DirectorySearcher("(&(objectCategory=User)(samAccountName=$($env:username)))")).FindOne().GetDirectoryEntry().memberOf | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\5_Usersandgroups2.txt;
Get-AdGroupMember -identity "Administrators" | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\5_Usersandgroups.txt;
Get-AdGroupMember -identity "Administrators" | select name | Export-csv -path c:\Users\$env:username\Desktop\logfiles\6_admin_members.csv -NoTypeInformation;
TASKLIST /v /fi "STATUS eq running" | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\7_running_tasks.txt;
TASKLIST /svc | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\8_all_tasks.txt;
Get-Process | Format-List * | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\9_processes_formatlist.txt;
Get-Process | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\10_processes.txt;
netstat -anob | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\11_netstat.txt;
netsh advfirewall export "c:\Users\$env:username\Desktop\logfiles\12_advfirewallpolicy.wfw";
net accounts | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\12_netaccounts.txt;
Get-SmbConnection | Out-File -FilePath c:\Users\$env:username\Desktop\logfiles\13_smbconnections.txt;
Compress-Archive -Path c:\Users\$env:username\Desktop\logfiles -DestinationPath c:\Users\$env:username\Desktop\logfiles.zip;
Remove-Item -LiteralPath "c:\Users\$env:username\Desktop\logfiles" -Force -Recurse;
