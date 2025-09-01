# DumpMe Lab — CTF writeup

- Platform: [Cyberdefenders.org](https://cyberdefenders.org/)
- Category: Endpoint Forensics
- Difficulty: Medium  
- Solver: Nubboss

## Summary
- Scenario: A SOC analyst captured a memory dump from a machine infected with Meterpreter. - Your task: analyze the dump, extract IOCs and answer 16 questions.  


## Artifacts
- Provided files: Triage-Memory.mem 
- Screenshots: /screenshots  

## Tools & Environment
- OS: Windows 10 VM  
- Tools: Volatility 2,3 , sha1sum

---

## P.S 
My first step in every Volatility task was to run windows.malfind and I found some IOCs.

How:
```powershell
.\volatility_2.6_win64_standalone.exe -f  .\Triage-Memory.mem malfind
```
## We have some interesting process "UWkpjFjDzM"

## Q1 — What is the SHA1 hash of Triage-Memory.mem (memory dump)?

How:
```powershell
Get-FileHash -Algorithm SHA1 .\Triage-Memory.mem
```

## A1 - C95E8CC8C946F95A109EA8E47A6800DE10A27ABD

## Q2 — What volatility profile is the most appropriate for this machine?(ex:Win10x86_14393)

How:
```powershell
.\volatility_2.6_win64_standalone.exe -f  .\Triage-Memory.mem imageinfo
#In output we have Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_23418
```

## A2 - Win7SP1x64

## Q3 — What was the process ID of notepad.exe?
How:
```powershell
.\volatility_2.6_win64_standalone.exe -f .\Triage-Memory.mem --profile=Win7SP1x64 pslist | Select-String "notepad"
```

## A3 - 3032

## Q4 — Name the child process of wscript.exe.

How:
First get pid wscript
```powershell
.\volatility_2.6_win64_standalone.exe -f .\Triage-Memory.mem --profile=Win7SP1x64 pslist | Select-String "wscript"
```
Pid = 5116
Second filter for pid 
```powershell
.\volatility_2.6_win64_standalone.exe -f .\Triage-Memory.mem --profile=Win7SP1x64 pstree | Select-String "5116"
```

## A4 - UWkpjFjDzM.exe

## Q5 — What was the IP address of the machine at the time the RAM dump was created?

How:
```powershell
.\volatility_2.6_win64_standalone.exe -f .\Triage-Memory.mem --profile=Win7SP1x64 netscan
```
In "Local address" we can saw our local ip

## A5 - 10.0.0.101

## Q6 — Based on the answer regarding the infected PID, can you determine the IP of the attacker?

Pid "some interesting procces is UWkpjFjDzM 3496"
How:
```powershell
 .\volatility_2.6_win64_standalone.exe -f .\Triage-Memory.mem --profile=Win7SP1x64 netscan | Select-String "3496"
```

## A6 - 10.0.0.106

## Q7 — How many processes are associated with VCRUNTIME140.dll?


How:
```powershell
 .\volatility_2.6_win64_standalone.exe -f .\Triage-Memory.mem --profile=Win7SP1x64 dlllist | Select-String "VCRUNTIME140"
```
I only had one process, but that was incorrect. 
Then I went to my Debian system and used Volatility 3.
How:
```cmd
    python3 vol.py -f Triage-Memory.mem windows.dlllist | grep VCRUNTIME140
```
And i had correct output
 
## A7 - 5

## Q8 — After dumping the infected process, what is its md5 hash?

How:
```powershell
 ..\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 procdump -p 3496 --dump-dir=dump
 Get-FileHash -Algorithm MD5 .\executable.3496.exe

```
 
## A8 - 690ea20bc3bdfb328e23005d9a80c290

## Q9 — What is the LM hash of Bob's account?

How:
```powershell
 .\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 hashdump
```
 
## A9 - aad3b435b51404eeaad3b435b51404ee

## Q10 — What memory protection constants does the VAD node at 0xfffffa800577ba10 have?

How:
    In this case, I saved the previous output as a .txt file so it’s easier to find the needed section.

```powershell
 .\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 vadinfo > vad.txt
```
    In txt file i used find and found neded section

# VAD (Virtual Address Descriptor) is like a note the operating system keeps that says which parts of a program’s virtual memory are in use. It records things such as:
    - where the memory region starts and ends,
    - what the region is used for (private data, a file mapping, or program code),
    - and what protections apply (read, write, execute).
 

## A10 - PAGE_READONLY

## Q11 — What memory protection did the VAD starting at 0x00000000033c0000 and ending at 0x00000000033dffff have?

How:
    In txt file i used find and found neded section
 
## A11 - aad3b435b51404eeaad3b435b51404ee

## Q12 — There was a VBS script that ran on the machine. What is the name of the script? (submit without file extension)

How:
```powershell
   .\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 cmdline | Select-String "wscript"
```

## A12 - vhjReUDEuumrX

## Q13 — An application was run at 2019-03-07 23:06:58 UTC. What is the name of the program? (Include extension)

How:
```powershell
    .\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 shimcache |Select-String"2019-03-07 23:06:58"
```

## A13 - Skype.exe

## Q14 — What was written in notepad.exe at the time when the memory dump was captured?

How:
```powershell
     .\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 memdump -p 3032 --dump-dir=dump
     .\strings64.exe  .\3032.dmp  | Select-String "flag<"
```

## A14 - flag<REDBULL_IS_LIFE>

## Q15 — What is the short name of the file at file record 59045?

How:
```powershell
    .\volatility_2.6_win64_standalone -f .\Triage-Memory.mem --profile=Win7SP1x64 mftparser > mft.txt
```
    In txt file i used find and found neded section

## A15 - EMPLOY~1.XLS

## Q16 — What is the short name of the file at file record 59045?

in Q6 we have PID

## A16 - 3496


