# Remediation off the Land.
  
Remediationn off the Land (RotL) is a simple tool that converts a list of artifacts from a malware infection into commands that can be executed on the system to delete/remove those artifacts.

## Installation

``pip3 install rotl``

## The rotl.py script

When installed, a commannd line script named 'rotl' is supplied that can be used to convert the remediation scripts into remediation files. Currently only windows remediations are supported.

```
$ rotl.py -h
usage: rotl.py [-h] [-w {win}] [-f REMEDIATION] [-t {win}] [-o OUTFILE]

Remediation off the Land: Write remediation files to execute

optional arguments:
  -h, --help            show this help message and exit
  -w {win}, --write-template {win}
                        write a remediation template file to local dir.
  -f REMEDIATION, --remediation REMEDIATION
                        the remediation file describing the infection
  -t {win}, --os-type {win}
                        remediation type (operating system)
  -o OUTFILE, --outfile OUTFILE
                        name of output file to write.
```

## The Remediation File

You can use the rotl script to print a copy of the remediation template file that can be used to describe a malicious infection. 

```
$ rotl.py -w win
+ Wrote remediate.ini
```

Now, you can edit the remediate.ini file to reflect the infection.

```

$ cat remediate.ini 
## Example remediate routine file.
##  All keys are commented out under their respective sections by default.

# Specify full paths to files that you want to delete.
#  ex: file1=c:\programdata\lemontrack installer\winserv.exe
[files]
;file1=
;file2=
;file3=

# Specify processes that you want to kill by name. All processes matching the name will be killed
#  ex: proc1=winserv.exe
[process_names]
;proc1=
;proc2=
;proc3=

# Delete a scheduled task
#  ex: task1=DHCP Monitor Task
[scheduled_tasks]
;task1=
;task2=

# SC delete services by their name
[services]
;service1=
;service2=
 
# Delete entire directories
#  ex: directory1=C:\ProgramData\LemonTrack Installer
[directories]
;directory1=
;directory2=

# Delete processes by their ID
#  ex: pid1=2664
[pids]
;pid1=
;pid2=

# delete individual registry key-values
#  ex: reg1=HKU\S-1-5-21-1660022851-2357930215-3100199371-1001\Software\Microsoft\Windows\CurrentVersion\Run\LemonTrack
#  This translates to: REG DELETE "HKU\S-1-5-21-1660022851-2357930215-3100199371-1001\Software\Microsoft\Windows\CurrentVersion\Run" /v LemonTrack /f
[registry_values]
;reg1=
;reg2=

# delete all values behing a key
#  ex: reg1=HKLM\Software\Microsoft\Windows\CurrentVersion\Run
#  REG DELETE HKLM\Software\Microsoft\Windows\CurrentVersion\Run /f
[registry_keys]
;reg1=
;reg2=
```

### Example

Example remediate file describing a Qbot infection:

```
$ cat remediate.ini 
[files]
file1=C:\WINDOWS\TEMP\iajzq.mkt
file2=C:\Documents and Settings\Administrator\Application Data\Microsoft\Iajzq\iajzq.exe

[process_names]
proc1=cscript.exe
proc2=iajzq.exe
proc3=wscntfy.exe

[scheduled_tasks]
task1=mxsiajzqupd

[services]
service1=fehjgnzjh
 
[directories]
directory1=C:\documents and settings\administrator\application data\microsoft\iajzq

[pids]

[registry_values]
reg1=HKU\S-1-5-21-1549631456-1210741653-3294372961-500\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\lcmkfq

[registry_keys]
```

Create the batch file:

```
$ rotl.py -f remediate.ini 
+ Wrote 'remediation.bat'
```

Now you this file was executed with admin rights on the infected system to remove the infection.

```
$ cat remediation.bat 
taskkill /IM "cscript.exe" /F
taskkill /IM "iajzq.exe" /F
taskkill /IM "wscntfy.exe" /F
REG DELETE "HKU\S-1-5-21-1549631456-1210741653-3294372961-500\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "lcmkfq" /f
del "C:\WINDOWS\TEMP\iajzq.mkt"
del "C:\Documents and Settings\Administrator\Application Data\Microsoft\Iajzq\iajzq.exe"
cd "C:\documents and settings\administrator\application data\microsoft\iajzq" && DEL /F /Q /S * > NUL && cd .. && RMDIR /Q /S "C:\documents and settings\administrator\application data\microsoft\iajzq"
schtasks /Delete /TN "mxsiajzqupd" /F
net stop "fehjgnzjh" && SC DELETE "fehjgnzjh"
```

