# Progenitor
Progenitor is a Powershell module that provides a set of functions that allows Infrastructure developers to extract the configuration from existing Windows computers and turn the configuration into Infrastructure As Code. Progenitor uses Powershell Desired State Configuration to reverse engineer the configuration of existing Windows computers and outputs:

1. ANSIBLE scripts
2. DSC Scripts

Progenitor uses the following DSC resources to extract the configuration from the Windiows computer:

1. cNtfsPermissionEntry
2. MSFT_xRegistryResource
4. MSFT_xPackageResource
5. MSFT_xWebAppPool
6. MSFT_xWebsite
7. MSFT_xWebApplication
8. MSFT_xWebVirtualDirectory
9. MSFT_xWindowsFeature

