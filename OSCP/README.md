---
layout: page
title: Documentation
description: >
  Here you should be able to find everything you need to know to accomplish the most common tasks when blogging with Hydejack.
hide_description: true
sitemap: false
permalink: /OSCP/
---

Here you should be able to find everything you need to know to accomplish the most common tasks when blogging with Hydejack.

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

While this manual tries to be beginner-friendly, as a user of Jekyll it is assumed that you are comfortable running shell commands and editing text files.
{:.note}



## 00_방법론_및_체크리스트
### Linux_Methodology
* [Enum_Cheatsheet]{:.heading.flip-title} --- How to install and run Hydejack.
* [Interactive_Shell_(shell_안정화)]{:.heading.flip-title} --- You can skip this if you haven't used Hydejack before.
* [Linux_Automated_Enum]{:.heading.flip-title} --- Once Jekyll is running you can start editing your config file.
{:.related-posts.faded}

### Windows_Methodology
* [Kerberos_기법]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Other
* [Check_List_(Standalones)]{:.heading.flip-title} --- How to install and run Hydejack.
* [Check_List_(AD)]{:.heading.flip-title} --- You can skip this if you haven't used Hydejack before.
{:.related-posts.faded}


## 01_정찰_및_정보_수집
### Service_Enumeration
* [NFS_(2049)]{:.heading.flip-title} --- How to install and run Hydejack.
* [RPC_(135)]{:.heading.flip-title} --- You can skip this if you haven't used Hydejack before.
* [SMB_(139,_445)]{:.heading.flip-title} --- Once Jekyll is running you can start editing your config file.
* [SNMP_(161)]{:.heading.flip-title} --- Once Jekyll is running you can start editing your config file.
* [SNMP_Example_diagram]{:.heading.flip-title} --- Once Jekyll is running you can start editing your config file.
{:.related-posts.faded}

### User_Discovery
* [kerbrute_for_usernames]{:.heading.flip-title} --- How to install and run Hydejack.
* [Names_to_username_AD]{:.heading.flip-title} --- How to install and run Hydejack.
* [netexec]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Other
* [Network_Scanning]{:.heading.flip-title} --- How to install and run Hydejack.
* [Web_Content_Discovery]{:.heading.flip-title} --- You can skip this if you haven't used Hydejack before.
{:.related-posts.faded}


## 02_초기_공격_및_익스플로잇
### Common_Techniques
* [기존_파일_수정,_간단_리버스쉘(bash)]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Linux
* [pgp_파일_해독]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Web_Attacks
* [SSTI]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Windows_Attacks
#### Get Shell
* [evil-winrm]{:.heading.flip-title} --- How to install and run Hydejack.
* [wmiexec]{:.heading.flip-title} --- How to install and run Hydejack.
* [psexec]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

#### Impacket_Tool
* [GetNPUsers]{:.heading.flip-title} --- How to install and run Hydejack.
* [GetST]{:.heading.flip-title} --- How to install and run Hydejack.
* [GetUserSPNs]{:.heading.flip-title} --- How to install and run Hydejack.
* [lookup_sid]{:.heading.flip-title} --- How to install and run Hydejack.
* [secretsdump]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Other
* [jenkins]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

## 03_권한_상승
### Linux
* [Hijacking_Python_Library]{:.heading.flip-title} --- How to install and run Hydejack.
* [Wildcard_Injection]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

### Windows
#### AD 개체 권한
* [AllowedToDelegate]{:.heading.flip-title} --- How to install and run Hydejack.
* [GPO]{:.heading.flip-title} --- How to install and run Hydejack.
* [RBCD_(Resource-Based_Constrained_Delegation)]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

#### Dumping_Hashes
* [mimikatz]{:.heading.flip-title} --- How to install and run Hydejack.
* [쓰레기통_뒤지기]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

#### Privileges_&_groups
* [DnsAdmins_(Group)]{:.heading.flip-title} --- How to install and run Hydejack.
* [SeBackupPrivilege]{:.heading.flip-title} --- How to install and run Hydejack.
* [SeDebugPrivilege]{:.heading.flip-title} --- How to install and run Hydejack.
* [SeImpersonatePrivilege]{:.heading.flip-title} --- How to install and run Hydejack.
* [Server_Operator_(Group)]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

#### Service_Attacks
* [Insecure_Service_Permissions]{:.heading.flip-title} --- How to install and run Hydejack.
* [Unquoted_Service_Path]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}

#### Other
* [AD_CS_(Active_Directory_Certificate_Services)]{:.heading.flip-title} --- How to install and run Hydejack.
* [AlwaysInstallElevated_(feat._PrivescCheck.ps1)]{:.heading.flip-title} --- How to install and run Hydejack.
* [Force_Change_Password_attack]{:.heading.flip-title} --- How to install and run Hydejack.
* [Golden_Ticket_Attack]{:.heading.flip-title} --- How to install and run Hydejack.
{:.related-posts.faded}








[Enum_Cheatsheet]: /00_방법론_및_체크리스트/Linux_Methodology/Enum_Cheatsheet.md
[Interactive_Shell_(shell_안정화)]: /00_방법론_및_체크리스트/Linux_Methodology/Interactive_Shell_(shell_안정화).md
[Linux_Automated_Enum]: /00_방법론_및_체크리스트/Linux_Methodology/Linux_Automated_Enum.md

[Kerberos_기법]: /00_방법론_및_체크리스트/Windows_Methodology/Kerberos_기법.md

[Check_List_(Standalones)]: /00_방법론_및_체크리스트/Check_List_(Standalones).md
[Check_List_(AD)]: /00_방법론_및_체크리스트/Check_List_(AD).md

[NFS_(2049)]: /01_정찰_및_정보_수집/Service_Enumeration/NFS_(2049).md
[RPC_(135)]: /01_정찰_및_정보_수집/Service_Enumeration/RPC_(135).md
[SMB_(139,_445)]: /01_정찰_및_정보_수집/Service_Enumeration/SMB_(139,_445).md
[SNMP_(161)]: /01_정찰_및_정보_수집/Service_Enumeration/SNMP_(161).md
[SNMP_Example_diagram]: /01_정찰_및_정보_수집/Service_Enumeration/SNMP_Example_diagram.md

[kerbrute_for_usernames]: /01_정찰_및_정보_수집/User_Discovery/kerbrute_for_usernames.md
[Names_to_username_AD]: /01_정찰_및_정보_수집/User_Discovery/Names_to_username_AD.md
[netexec]: /01_정찰_및_정보_수집/User_Discovery/netexec.md

[Network_Scanning]: /01_정찰_및_정보_수집/Network_Scanning.md
[Web_Content_Discovery]: /01_정찰_및_정보_수집/Web_Content_Discovery.md

[기존_파일_수정,_간단_리버스쉘(bash)]: /02_초기_공격_및_익스플로잇/Common_Techniques/기존_파일_수정,_간단_리버스쉘(bash).md

[pgp_파일_해독]: /02_초기_공격_및_익스플로잇/Linux/pgp_파일_해독.md

[SSTI]: /02_초기_공격_및_익스플로잇/Web_Attacks/SSTI.md

[evil-winrm]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Get_Shell/evil-winrm.md

[GetNPUsers]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/GetNPUsers.md
[GetST]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/GetST.md
[GetUserSPNs]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/GetUserSPNs.md
[lookup_sid]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/lookup_sid.md
[psexec]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/psexec.md
[secretsdump]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/secretsdump.md
[wmiexec]: /02_초기_공격_및_익스플로잇/Windows_Attacks/Impacket_Tool/wmiexec.md

[jenkins]: /02_초기_공격_및_익스플로잇/jenkins.md

[Hijacking_Python_Library]: /03_권한_상승/Linux/Hijacking_Python_Library.md
[Wildcard_Injection]: /03_권한_상승/Linux/Wildcard_Injection.md

[AllowedToDelegate]: /03_권한_상승/Windows/AD_개체_권한/AllowedToDelegate.md
[GPO]: /03_권한_상승/Windows/AD_개체_권한/GPO.md
[RBCD_(Resource-Based_Constrained_Delegation)]: /03_권한_상승/Windows/AD_개체_권한/RBCD_(Resource-Based_Constrained_Delegation).md

[mimikatz]: /03_권한_상승/Windows/Dumping_Hashes/mimikatz.md
[쓰레기통_뒤지기]: /03_권한_상승/Windows/Dumping_Hashes/쓰레기통_뒤지기.md

[DnsAdmins_(Group)]: /03_권한_상승/Windows/Privileges_&_groups/DnsAdmins_(Group).md
[SeBackupPrivilege]: /03_권한_상승/Windows/Privileges_&_groups/SeBackupPrivilege.md
[SeDebugPrivilege]: /03_권한_상승/Windows/Privileges_&_groups/SeDebugPrivilege.md
[SeImpersonatePrivilege]: /03_권한_상승/Windows/Privileges_&_groups/SeImpersonatePrivilege.md
[Server_Operator_(Group)]: /03_권한_상승/Windows/Privileges_&_groups/Server_Operator_(Group).md

[Insecure_Service_Permissions]: /03_권한_상승/Windows/Service_Attacks/Insecure_Service_Permissions.md
[Unquoted_Service_Path]: /03_권한_상승/Windows/Service_Attacks/Unquoted_Service_Path.md

[AD_CS_(Active_Directory_Certificate_Services)]: /03_권한_상승/Windows/AD_CS_(Active_Directory_Certificate_Services).md
[AlwaysInstallElevated_(feat._PrivescCheck.ps1)]: /03_권한_상승/Windows/AlwaysInstallElevated_(feat._PrivescCheck.ps1).md
[Force_Change_Password_attack]: /03_권한_상승/Windows/Force_Change_Password_attack.md
[Golden_Ticket_Attack]: /03_권한_상승/Windows/Golden_Ticket_Attack.md



[LICENSE]: ../LICENSE.md
[NOTICE]: ../NOTICE.md
[CHANGELOG]: ../CHANGELOG.md
