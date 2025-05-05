# REST source authentication

### EntraID Authentication (per-source)

winget, since some specific version, supports REST sources that require EntraID authentication.

The `GET /information` api path responds with something like:

```json
{
  "Data": {
    "SourceIdentifier": "<SOME-STRING>",
    "ServerSupportedVersions": [
      "1.1.0",
      "1.4.0",
      "1.5.0",
      "1.6.0",
      "1.7.0",
      "1.9.0"
    ],
    "Authentication": {
      "AuthenticationType": "microsoftEntraId",
      "MicrosoftEntraIdAuthenticationInfo": {
        "Resource": "<GUID>",
        "Scope": "user_impersonation"
      }
    }
  }
}
```

The "Resource" GUID has to be the Application ID of an EntraID Enterprise Application in the Tenant of the users who you will be authenticating (I guess authenticating users from multiple tenants is not supported?).

That application has to be set up specifically for use by winget, the process can be seen in Microsofts script [`New-MicrosoftEntraIdApp.ps1`](https://github.com/microsoft/winget-cli-restsource/blob/main/Tools/PowershellModule/src/Library/New-MicrosoftEntraIdApp.ps1):

```powershell
$Name = '<SOME-STRING> winget source'

## Normalize Microsoft Entra Id app name
$NormalizedName = $Name -replace "[^a-zA-Z0-9-()_.]", ""
if ($Name -cne $NormalizedName) {
    $Name = $NormalizedName
    Write-Warning "Removed special characters from the Microsoft Entra Id app name (New Name: $Name)."
}

## Creating a line break from previous steps
Write-Information "Microsoft Entra Id app name to be created: $Name"

$App = New-AzADApplication -DisplayName $Name -AvailableToOtherTenants $false
if (!$App) {
    Write-Error "Failed to create Microsoft Entra Id app. Name: $Name"
    return $Return
}

## Add App Id Uri
$AppId = $App.AppId
$App = Update-AzADApplication -ApplicationId $AppId -IdentifierUri "api://$AppId" -ErrorVariable ErrorUpdate
if ($ErrorUpdate) {
    Write-Error "Failed to add App Id Uri. Error: $ErrorUpdate"
    return $Return
}

## Add Api scope
$ScopeId = [guid]::NewGuid().ToString()
$ScopeName = "user_impersonation"
$Api = @{
    oauth2PermissionScopes = @(
        @{
            adminConsentDescription = "Sign in to access $Name WinGet rest source"
            adminConsentDisplayName = "Access WinGet rest source"
            userConsentDescription  = "Sign in to access $Name WinGet rest source"
            userConsentDisplayName  = "Access WinGet rest source"
            id = $ScopeId
            isEnabled = $true
            type = "User"
            value = $ScopeName
        }
    )
}
$App = Update-AzADApplication -ApplicationId $AppId -Api $Api -ErrorVariable ErrorUpdate
if ($ErrorUpdate) {
    Write-Error "Failed to add Api scope. Error: $ErrorUpdate"
    return $Return
}

## Add authorized clients
$Api = @{
    preAuthorizedApplications = @(
        @{
            # "App Installer"
            appId = "7b8ea11a-7f45-4b3a-ab51-794d5863af15"
            delegatedPermissionIds = @($ScopeId)
        }
        @{
            # "Microsoft Azure CLI"
            appId = "04b07795-8ddb-461a-bbee-02f9e1bf7b46"
            delegatedPermissionIds = @($ScopeId)
        }
        @{
            # "Microsoft Azure PowerShell"
            appId = "1950a258-227b-4e31-a9cf-717495945fc2"
            delegatedPermissionIds = @($ScopeId)
        }
    )
}
$App = Update-AzADApplication -ApplicationId $AppId -Api $Api -ErrorVariable ErrorUpdate
if ($ErrorUpdate) {
    Write-Error "Failed to add authorized clients"
    return $Return
}

Write-Output "Application Id (Resource): $AppId, Scope: $ScopeName"
```

If the REST source responded correctly on the `/information` API endpoint (easy) the winget client will prompt for authentication (sign-in) on any operations queryting the source, such as `winget search -q SEARCHTERM -s source-with-auth`.
If you were able to complete the EntraID login, winget sends an "Authorization" header with a EntraID JWT to the REST source in its API request(s) (standard format; "Authorization: Bearer JWT").
If you were not able to complete the EntraID login, the enterprise application was likely not set up correctly or your user was not assigned to the app or maybe it's conditional access.

### EntraID Authentication (per-package)

TODO: Look into it, the 1.10 manifest schema looks like this is a feature now but didn't immediately find any announcement or docs on it.
