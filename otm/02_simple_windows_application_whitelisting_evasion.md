# Simple Windows Application Whitelisting Evasion

Often deployed as the new way to prevent malware and unauthorized execution, application whitelisting has been billed as a way to contain and prevent advanced threats. "Deploy application whitelisting technology that allows systems to run software only if it is included on the whitelist and prevents execution of all other software on the system." So goes the guidance of the Critical Security Controls. Is this guidance effective? Are there practical ways to circumvent whitelisting technology. If so, what are these techniques?  

Adversaries adapt. Eventually, like we see in the biological world (weeds, mosquitoes), adversaries become resistant or inoculated against our defenses. We have developed a catalog of bypass techniques we would like to share. These techniques, while focused on the Windows Operating Systems, may have application to other areas.  

Application Whitelisting is becoming a prevalent defensive counter-measure in many environments.  The goal being that new or unknown or unapproved binaries or scripts will not execute on the system.  As a part of this research we developed a catalog of evasion techniques that allow us to circumvent these types of controls.  The guiding principle is that we will only use those files that pre-exist and that may be trusted to in order to achieve this evasion.  The following document summarizes these techniques.

## Script execution

The goals is to execute `.bat`, `.vbs`, or `.ps1` scripts.  The first tactic is not to use these file extensions at all for the content of the script you wish to execute. Using .txt, or other non-script related file extensions is sufficient.  Second, we will use redirection to pipe contents to the script execution environment.
 
    .bat        cmd.exe /k < script.txt  
    .vbs		cscript.exe //E:vbscript script.txt  
    .ps1		Get-Content script.txt | iex  

The ability to detect and block script execution is difficult since the content of the script can be derived from a number of sources.

## .NET execution

The goals is to be able to execute a .NET assembly. The technique here is to find Sponsors. A Sponsor is defined as a trusted executable that executes another file. By influencing the execution target, we can achieve evasion here. We have identified at least four Sponsors, there are likely others. These are:

    InstallUtil.exe
    IEExec.exe
    DFsvc.exe
    PresentationHost.exe

By using specially crafted .NET assemblies, we are able to have these Sponsors execute the assembly on our behalf.  Our perspective here is not so much what the original intention of the Sponsor is.  But rather, how can we influence the binary to achieve our desired actions.  

> There is a fundamental difference between the approach taken by a development team and that taken by someone attacking an application. A development team typically approaches an application based on what it is intended to do. In other words, they are designing an application to perform specific tasks based on documented functional requirements and use cases. An attacker, on the other hand, is more interested in what n application can be made to do and operates on the principle that "any action not specifically denied, is allowed" - **OWASP Secure Coding Practices Guide v2**[^1]

One of the principle reasons that execution of .NET assemblies is missed, has to do with how many vendors detect and block the execution events. The guidelines presented in the Kernel Data and Filtering Support guide, describes how vendors might attempt to block execution:  

> Released Windows operating system versions do support notification of image loading operations (which does meet some ISV requirements), but do not provide the ability to block the loading operation.  For that reason, Microsoft investigated whether a new API would be needed to support this requirement, but ultimately concluded that existing supported functionality could be used to achieve the desired module load blocking behavior.  In particular, a file system mini-filter can be utilized to block the loading of both modules in both user mode (e.g., DLLs) and kernel mode (e.g., device drivers).  Intercepting `IRP_MJ_ACQUIRE_FOR_SECTION_SYNCHRONIZATION` and returning `STATUS_ACCESS_DENIED` when sections are loaded for `PAGE_EXECUTE` permission is an appropriate approach." - **Kernel Data Filtering and Support**[^2]

The difficulty is that this is not how a .NET assembly is loaded.  

> The root issue here is that at the point at which execute operations are detected (`CreateFileMapping`->`NtCreateSection`), only read-only access to the section is requested, so it is not processed as an execute operation.  Later, execute access is requested in the file mapping (MapViewOfFile->NtMapViewOfSection), which results in the image being mapped as `EXECUTE_WRITECOPY` and subsequently allows unchecked execute access." - **Chris Lord, Bit9**

We believe that this may affect a number of applications that rely on the pattern to detect and block execution events.    

## Native Image Execution

There are a couple of techniques that can be used to execute Native images.  For example, you can attempt to execute the file via `rundll32` or `pcwutl.dll`.

    C:\Windows\System32\rundll32.exe" C:\Windows\system32\pcwutl.dll,CreateAndRunTask -path "C:\rouge.exe

However this execution event is likely to be blocked. The best technique we have found is to utilize a custom Portable Executable(PE) loader to execute the native binary from memory.  

In order to achieve this we will leverage the work done by other researchers to mimic the PE loader provided by the operating system.

We built a custom tool called **Malwaria**, which was used to execute a native DLL in memory.  We used the techniques described earlier to achieve code execution.  This tool embeds the dll as a resource, and then executes the file from a byte array in memory. 

An excellent tool has been developed by the PowerSploit team.  `Invoke-ReflectivePEInjection`[^7].  This tool is an amazing example of how PowerShell and the .NET framework can be leveraged to execute arbitrary PE files.  Since many environments trust the `powershell.exe` binary, it is a natural evolution for adversaries to leverage this framework.  By creating an exploit for CVE-2014-4113, and embedding that into the `Invoke-ReflectivePEInjection`[^7] script.  We were able to achieve privilege escalation, even an environment with strict whitelisting enabled.  

## Conclusion:

Adversaries adapt.  In response to an application whitelisting defense, we believe that adversaries will adapt a "Living Off The Land" approach.  This involves using pre-existing trusted tools to achieve lateral movement and privilege escalation.  When it is necessary to execute a custom binary, it is likely that adversaries can leverage approaches described above in order to achieve action on objectives.



## References

* [^1] https://www.owasp.org/images/0/08/OWASP_SCP_Quick_Reference_Guide_v2.pdf  
* [^2] http://www.bitnuts.de/KernelBasedMonitoring.pdf  
* [^3] Kernel Data and Filtering Support   -http://download.microsoft.com/download/4/4/b/44bb7147-f058-4002-9ab2-ed22870e3fe9/Kernal%20Data%20and%20Filtering%20Support%20for%20Windows%20Server%202008.doc  
* [^4] https://github.com/stephenfewer/ReflectiveDLLInjection  
* [^5] https://github.com/fancycode/MemoryModule  
* [^6] https://github.com/Scavanger/MemoryModule.net  
* [^7] https://github.com/clymb3r/PowerShell/tree/master/Invoke-ReflectivePEInjection  
* [^8] http://opensecuritytraining.info/LifeOfBinaries.html  
* [^9] PE/COFF Specification - http://download.microsoft.com/download/e/b/a/eba1050f-a31d-436b-9281-92cdfeae4b45/pecoff.doc  
* [^10] Living Off the Land: A Minimalist’s Guide to Windows Post-Exploitation – Christopher Campbell & Matthew Graeber  

#### Metadata

**Primary Author Name**: Casey Smith  	
**Primary Author Twitter**: @subTee

