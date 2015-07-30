##**Update Office 2013 or Office 2016 using SCCM**

The script automates the configuration  updating of Office 365 Pro Plus through Microsoft System Center Configuration Manager (SCCM) and ensure the PC where Office 365 Pro Plus is installed gets Office 365 Pro Plus updates from the closest SCCM Distribution Point (DP).

###**Pre-Requisites:**

Before running this script, the following conditions have to be met

1. .Net Framework 3.5 SP1 must be installed on client machines.
2. A operational SCCM environment.
3. Office 2013 or Office 2016 is already installed on client machines. 
4. Office Auto Updates have been Disabled on the client machines preferably via Group Policy

###**Running the script**

1. The script should ideally be run on a SCCM server.  If you do not run it from a SCCM server ensure you always specify the -Path parameter.
2. Open a Elevated PowerShell Console(see, [Starting Windows PowerShell](https://technet.microsoft.com/en-us/library/hh857343.aspx)):

		From the Run dialog type PowerShell.

3. Change directory to the location where the PowerShell Script is saved.   This directory must contain the files that are in the *Setup-SCCMOfficeUpdates* folder.

		Example: cd C:\PowerShellScripts

4. Type the following in the elevated PowerShell Session

		 . .\Setup-SCCMOfficeUpdates.ps1
         
         By including the additional period before the relative script path you are 'Dot-Sourcing' 
		 the PowerShell functions in the script into your PowerShell session which will allow you to 
		 run the function from the console.

5. The first thing you must do is download the Office update files to a staging location to make them available for SCCM. From the existing PowerShell session type the command below.

		Download-OfficeUpdates -Path (Optional) -Version (Optional)
        
	If you specify the *-Path* parameter then the script will download the Office updates to that path. The path must be a valid UNC path. Specifying the *-Version* parameter will cause the script to download a specific version of the Office updates.
    
    If you do not specify any parameters the script will create a local folder name 'OfficeUpdates' on the SystemDrive.  It will then share the folder using a hidden share name 'OfficeUpdates$'. This share will be used to store the Office update files. If you are not running the script on a SCCM server it is important that you specify the -Path parameter with all functions so the local share will not be created.
    
6. Now that the Office update files have been downloaded to a share on the network you can run the setup function to configure SCCM. A SCCM  collection must be specified to use this function. The collection specified should contain the workstations that you want configured.  If there are no Distribution Point Groups added to the collection then you will also have to use the parameter *-DistributionPointGroupName*

		Setup-SCCMOfficeUpdates -Collection CollectionName -DistributionPointGroupName DPGroupName

7. The function *Setup-SCCMOfficeUpdates* will create a SCCM Package that is configured to run the *SCO365PPTrigger.exe* executable on the client machines.  After the package is created the cmdlet *Start-CMContentDistribution* is run in order to start the process to distribute the Package contents to the Distribution Points. Before proceeding to the next step you should monitor and wait until the content distribution process is complete.  Clients deployments will fail until the content is distributed to their Distribution Point.

8. Once the content distribution is complete you can run the final function.  The *Deploy-SCCMOfficeUpdates* function will deploy the package.  You must provide the name of the collection.  This should be the same collection you used with the previous function.

		Deploy-SCCMOfficeUpdates -Collection CollectionName





