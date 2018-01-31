# GSuite-as-identity-Provider-IdP-for-Office-365-or-Azure-Active-Directory
Finally manage to sync GSuite account with azure active directory!!!

## Google Admin requirements
### Set up SAML app (choose Microsoft Office 365)

> [GSuite Admin SAML Apps Link](https://admin.google.com/AdminHome?fral=1#AppsList:serviceType=SAML_APPS)

![GSuite SAML Apps](https://i.imgur.com/qSIrLyN.png)

Note:

ACS URL: `https://login.microsoftonline.com/login.srf`

Entity ID: `urn:federation:MicrosoftOnline`


![GSuite Office 365 Settings](https://i.imgur.com/0yEnR5m.png)

### Configure Provisioning

> [GSuite Office 365 Provisioning settings Link](https://admin.google.com/AdminHome?fral=1#AppDetails:service=935556381546&flyout=provisioningSetupV2)

![GSuite Office 365 settings](https://i.imgur.com/giY8PmH.png)



## Azure Active Directory requirements (this is a pain in the a**)
Validate your domain on Azure:
https://portal.azure.com/?l=en.en-us#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Domains

And DON'T set this domain as Primary:

![](https://i.imgur.com/GhEaTXo.png)


Download the `GoogleIDPMetadata-{your-domain}.xml` file:

![GoogleIDPMetadata-{your-domain}.xml sample file](https://i.imgur.com/rNvQshH.png)

Then install all required tools (powershell tools)

![Required PowerShell tools](https://i.imgur.com/sSkF2vZ.png)

And start a powershell console:

type this:
```
$Msolcred = Get-credential
Connect-MsolService -Credential $MsolCred
```

Now edit my sample `dfs-pf-samlp.xml` file with your Google Ids:

- `GOOGLESAMLID` and 
- copy paste your certificate (from `GoogleIDPMetadata-{your-domain}.xml` file)

Then import the config into powershell:
```
$wsfed = Import-Clixml dfs-pf-samlp.xml
```

And Set the domain as federated:
```
Set-MsolDomainAuthentication -DomainName "{your-domain}" -FederationBrandName $wsfed.FederationBrandName -Authentication Federated -PassiveLogOnUri $wsfed.PassiveLogOnUri -ActiveLogOnUri $wsfed.ActiveLogonUri -SigningCertificate $wsfed.Signi
ngCertificate -IssuerUri $wsfed.IssuerUri -LogOffUri $wsfed.LogOffUri -PreferredAuthenticationProtocol "SAMLP"
```

And use this command to export your domain settings:
```
Get-MsolDomainFederationSettings -DomainName "{your-domain}" | Export-Clixml dfs-pf-samlp.xml
```

The command to view the config is:
```
Get-MsolDomainFederationSettings -DomainName "{your-domain}" | Format-List *
```

Next you have to assign a licence to all your users

and to set azure self service password reset to off:

https://portal.azure.com/?l=en.en-us#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/PasswordReset

Test the link with incognito mode or invite mode:
1. From Office 365: https://www.office.com/
2. From App launcher (Google App)

![Google App launcher](https://i.imgur.com/UfVOBQ9.png)
