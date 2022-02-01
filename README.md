# Progenitor
Progenitor is a Powershell module that provides a set of functions that allows Infrastructure developers to extract the configuration from existing Windows computers and turn the configuration into Infrastructure as Code. Progenitor uses Powershell Desired State Configuration to reverse engineer the configuration of existing Windows computers and outputs:

1. ANSIBLE scripts
2. DSC Scripts

Progenitor uses the following DSC resources to extract the configuration from the Windows computer:

1. cNtfsPermissionEntry
2. MSFT_xRegistryResource
4. MSFT_xPackageResource
5. MSFT_xWebAppPool
6. MSFT_xWebsite
7. MSFT_xWebApplication
8. MSFT_xWebVirtualDirectory
9. MSFT_xWindowsFeature

The Get-ANSIBLEPlayBooks.ps1 script in the test folder provides an example that shows how Progenitor can be used to extract the configuration from an existing Windows computer and turn it into ANSIBLE scripts. The following environmental variables must be set before running the script:

1. $env:ServerInstance = This set to either the name or the ip address of the Windows computer
2. $env:MyUsername = The name of the user
3. $env:MyPassword = The password for the user
4. $env:ConvertTo = The type of output to create and this can be set to either 'AnsiblePlayBook' or 'DSC'


```Powershell
<#
    This script uses the progenitor module to reverse engineer an existing 
    server configuration and turn it into a set of ANSIBLE Scripts  function

    Nigel Thomas

    May 31, 2021

    New-ItemProperty -Name LocalAccountTokenFilterPolicy -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -PropertyType DWord -Value 1
    Set-ItemProperty -Name LocalAccountTokenFilterPolicy -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System  -Value 0
    Get-WSManInstance winrm/config/listener -Enumerate
    Get-Item wsman:\localhost\Client\TrustedHosts
    Start-Service -Name WinRM
    Set-Item WSMan:\localhost\Client\TrustedHosts -Concatenate -Value 172.31.248.15 
    Clear-Item -Path WSMan:\localhost\Client\TrustedHosts -Force 
    
#>

Import-Module C:\Downloads\development\progenitor -Force

#Use environmental variables to configure the session information
#$env:ServerInstance = 'The name or ip address of the server'
#$env:MyUsername = 'The user name'
#$env:MyPassword = 'The user password'
#$env:ConvertTo = 'AnsiblePlayBook'
#$env:ConvertTo = 'DSC'

#Array of DSC Resources that we want to tyurn iinto ANSIBLE Scripts
$DSCResourceNames = 'MSFT_xWindowsFeature', 'MSFT_xPackageResource', 'MSFT_xWebAppPool','MSFT_xWebsite','MSFT_xWebApplication', 'MSFT_xWebVirtualDirectory', 'cNtfsPermissionEntry', 'MSFT_xRegistryResource'

$RegistryKeys = 'HKLM:\SOFTWARE\ODBC\ODBC.INI', 'HKLM:\SOFTWARE\WOW6432NODE\ODBC\ODBC.INI'

$OuputPath = 'C:\Downloads\development\progenitor\Output\Test'

if (!$DSCResourceNames ) {
 $message = 'The variable DSCResourceName is not set.'
 Write-Error -Message $message -Category InvalidData 
 return
}

if (!$env:ServerInstance) {
 $message = 'The environment variable ServerInstance is not set.'
 Write-Error -Message $message -Category InvalidData 
 return
}

$ServerInstance = $env:ServerInstance

if ($env:MyUserName) {
    $Username = $env:MyUserName
}
else {
    $Username = $null
}

if ($env:MyPassword) {
    $Password = $env:MyPassword
}
else {
    $Password = $null
}

if ($Username -and $Password) {
    $secpassword = ConvertTo-SecureString $Password -AsPlainText -Force
    $DSCCredentials = New-Object -typename System.Management.Automation.PSCredential -argumentlist $UserName, $secpassword
}

if ($DSCCredentials) {
    $s = Connect-ServerInstance -ServerInstance $ServerInstance -ServerCredential $DSCCredentials -TrustedHost:$true
    #$s
}
else {
    $s = Connect-ServerInstance -ServerInstance $ServerInstance -TrustedHost:$true
    #$s
}

if (!$s) {
    return
}

if ($s) {

    foreach ($DSCResourceName in $DSCResourceNames) {

        if (!$DSCResourceName) {
            continue
        }

        if ($DSCResourceName.ToString().ToLower().Equals('msft_xregistryresource')) {

            $AllResult = Get-DSCResourceSetting -Session $s -DSCResourceName $DSCResourceName -Name $RegistryKeys
        }
        else {

            $AllResult = Get-DSCResourceSetting -Session $s -DSCResourceName $DSCResourceName
        }

        

        # Convert to the desired format
        switch ($env:ConvertTo) {

            'AnsiblePlayBook' {

                if ($AllResult) {
                    
                    $tempfilename = "{0}-{1}-{2}.yml" -f $DSCResourceName, $ServerInstance, $((Get-Date -UFormat "%Y-%m-%d_%I-%M-%S_%p").tostring())

                    $message = "Saving results for {0} to {1}\{2} `r`n " -f $DSCResourceName, $OuputPath, $tempfilename

                    Write-Host $message -Verbose

                    ConvertTo-AnsiblePlayBook -DSCInputObject $AllResult -DSCResourceName $DSCResourceName | Out-file "$OuputPath\$tempfilename"
                }
            }
            
            'DSC' {}

        }
    }

    Remove-PSSession -Session $s


}


Remove-Module progenitor
#Remove-Module powershell-yaml

```
