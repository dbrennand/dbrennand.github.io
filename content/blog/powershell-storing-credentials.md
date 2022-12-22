---
title: "PowerShell: Storing Credentials Securely"
date: 2022-12-12T20:18:10+01:00
draft: false
tags: ["PowerShell", "Credentials"]
showToc: true
---

Recently I've been working on several PowerShell scripts that require credentials to access REST APIs. In this blog post, I will showcase two approaches for storing credentials securely for use in PowerShell scripts.

# Encrypted Password File 🔒

The encrypted password file leverages the [Windows Data Protection API (DPAPI)](https://learn.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)) to encrypt the password as a `System.Security.SecureString`:

```PowerShell
$Credentials = Get-Credential
$Credentials.Password
System.Security.SecureString
$Credentials.Password | ConvertFrom-SecureString
01000000d08c9ddf0115d1118c7a00c04fc297eb01000000f5ab85d7ee9da048ae4ae797ee7eaf0a000000000200000000001066000000010000200000008c4a03d2f0731e0e7661d695fda8b441eaff31e75724931f31374a0c8292b636000000000e800000000200002000000028da885828bd627480178382ce9a1b477819e7703546ce41819d37f4e63d33ba20000000ab2c4401635ec24db9f20071e18dea0b79ce16ba38b5503ec9937b7fbc849dcf40000000155053a793c210998ef7317b0161e7344c2174b904b527c0cf24e7bbf2243b99e936df3ab67bc9e285a1be33aed37c7604fb07f5d0c44ceb7d6334ca30b0a610
```

By default DPAPI uses the current user context to generate an encryption key. This encryption key is then used to encrypt the `PSCredential.Password` property as a `System.Security.SecureString` (as shown above). You can provide your own encryption key, and you can read more about this on <cite>Travis Gan's blog [^1]</cite>.

It's also worth noting there are several caveats to this approach:

1. You **must** encrypt the password file as the user which will be accessing it.
2. DPAPI is specific to the device which you encrypt the password file on. You **cannot** decrypt the password file on another system with the same user.

## Generating and using the Encrypted Password File

To store the password securely in a file, we can use the [`Export-Clixml`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/export-clixml?view=powershell-7.3) cmdlet which will store the `System.Management.Automation.PSCredential` object in XML format:

```PowerShell
$Credentials = Get-Credential
$Credentials | Export-Clixml -Path "$(pwd)\EncryptedCreds.xml"
```

Once created, the `EncryptedCreds.xml` file will contain the XML representation of the `PSCredential` object. As shown below, the `Password` property is stored as a `SecureString`:

```Xml
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">User</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000f5ab85d7ee9da048ae4ae797ee7eaf0a000000000200000000001066000000010000200000008c4a03d2f0731e0e7661d695fda8b441eaff31e75724931f31374a0c8292b636000000000e800000000200002000000028da885828bd627480178382ce9a1b477819e7703546ce41819d37f4e63d33ba20000000ab2c4401635ec24db9f20071e18dea0b79ce16ba38b5503ec9937b7fbc849dcf40000000155053a793c210998ef7317b0161e7344c2174b904b527c0cf24e7bbf2243b99e936df3ab67bc9e285a1be33aed37c7604fb07f5d0c44ceb7d6334ca30b0a610</SS>
    </Props>
  </Obj>
</Objs>
```

We can then re-create the `PSCredential` object for use in PowerShell scripts using the [`Import-Clixml`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-clixml?view=powershell-7.3) cmdlet:

```PowerShell
$Credentials = Import-Clixml -Path "$(pwd)\EncryptedCreds.xml"
$Credentials

UserName                     Password
--------                     --------
User     System.Security.SecureString

$Credentials.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     PSCredential                             System.Object

Write-Output -InputObject "Username: ""$($Credential.UserName)"" - Password: ""$($Credential.GetNetworkCredential().Password)"""
Username: "User" - Password: "Password123!"
```

Tada! ✨

This approach can be good for one off scripts but, can quickly become a maintainance burden when you're doing this for many scripts. For this type of use-case, the next approach is probably more suitable.

# SecretManagement & SecretStore Modules 🔐

The SecretManagement and SecretStore modules are a great choice for storing credentials securely. To begin, install the modules:

```PowerShell
Install-Module -Name "Microsoft.PowerShell.SecretManagement", "Microsoft.PowerShell.SecretStore" -Verbose
```

## Creating a SecretVault

Next, we need to create a `SecretVault` to store secrets using the [Register-SecretVault](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretmanagement/register-secretvault?view=ps-modules) cmdlet 🔐:

```PowerShell
Register-SecretVault -Name "MySecretVault" -ModuleName "Microsoft.PowerShell.SecretStore" -DefaultVault -Description "Secret vault storing my secrets." -PassThru

Name          ModuleName                       IsDefaultVault
----          ----------                       --------------
MySecretVault Microsoft.PowerShell.SecretStore True
```

## Storing Secrets

Nice! :thumbsup: Now lets store a `PSCredential` in the `SecretVault` with the [Set-Secret](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretmanagement/set-secret?view=ps-modules) cmdlet. When creating the first secret you will be prompted for a password which will be used to encrypt the `SecretVault`:

> ℹ You can store a `SecureString`, `HashTable`, `String` or `Byte[]` in a secret as well!

```PowerShell
$RestAPICreds = Get-Credential
Set-Secret -Name "REST API Creds" -Vault "MySecretVault" -Secret $RestAPICreds

Creating a new MySecretVault vault. A password is required by the current store configuration.
Enter password:
********
Enter password again for verification:
********
```

## Retrieving Secrets

Now that the secret is stored securely, we can retrieve it in another script using the [Get-Secret](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretmanagement/get-secret?view=ps-modules) cmdlet:

```PowerShell
$RestAPICredentialsSecret = Get-Secret -Name "REST API Credentials" -Vault "MySecretVault"
$RestAPICredentialsSecret

UserName                     Password
--------                     --------
test     System.Security.SecureString

# Bonus! Get information about a secret too!
# https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretmanagement/get-secretinfo?view=ps-modules
Get-SecretInfo -Name "REST API Credentials" -Vault "MySecretVault"

Name                 Type         VaultName
----                 ----         ---------
REST API Credentials PSCredential MySecretVault

# As it's a PSCredential object, we can use the GetNetworkCredential method to finally get the password!
$RestAPICredentialsSecret.GetNetworkCredential().Password
```

Awesome! 😎 We've stored the first secret securely! ✨🚀

However, if you use the [Get-Secret](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretmanagement/get-secret?view=ps-modules) cmdlet in a new session, or after a certain amount of time has elapsed, you may notice that you're prompted for the password to decrypt the `SecretVault` again! :scream:

```PowerShell
Get-Secret -Name "REST API Credentials" -Vault "MySecretVault"
Vault MySecretVault requires a password.
Enter password:
```

Obviously this is an issue if we want our scripts to work without any interaction!

## Retrieving Secrets Non-Interactively

To fix this we can leverage the first approach and store our `SecretVault` decryption password in an encrypted password file and retrieve it later! So... lets do it! :smile:

```PowerShell
$SecretVaultPassword = Get-Credential
$SecretVaultPassword | Export-Clixml -Path "$(pwd)\SecretVaultPassword.xml"
Get-Content -Path "$(pwd)\SecretVaultPassword.xml"

...
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
    <Props>
      ...
      <SS N="Password">700061007300730077006f0072006400</SS>
    </Props>
  </Obj>
```

Once done, we can use the [Unlock-SecretStore](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretstore/unlock-secretstore?view=ps-modules) cmdlet in our scripts to unlock the `SecretVault` non-interactively:

```PowerShell
$SecretVaultPassword = (Import-Clixml -Path "$(pwd)\SecretVaultPassword.xml").Password
Unlock-SecretStore -Password $SecretVaultPassword
```

Automation FTW! ⚙ 💪

## Modifying SecretVault Configuration

Finally, you may also want to modify the configuration of a `SecretVault`. You can do this using the SecretStore cmdlets [Get-SecretStoreConfiguration](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretstore/get-secretstoreconfiguration?view=ps-modules) and [Set-SecretStoreConfiguration](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretstore/set-secretstoreconfiguration?view=ps-modules).

Let's check the `SecretVault` configuration with the [Get-SecretStoreConfiguration](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretstore/get-secretstoreconfiguration?view=ps-modules) cmdlet:

```PowerShell
Get-SecretStoreConfiguration

      Scope Authentication PasswordTimeout Interaction
      ----- -------------- --------------- -----------
CurrentUser       Password             900      Prompt
```

We can see that the `SecretVault` configuration is set to prompt for a password, and the timeout before being prompted for the password again is 900 seconds (15 minutes) by default.

The timeout can be changed using the [Set-SecretStoreConfiguration](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.secretstore/set-secretstoreconfiguration?view=ps-modules) cmdlet:

```PowerShell
Set-SecretStoreConfiguration -PasswordTimeout 1800 -Confirm:$false
# Check the configuration again
Get-SecretStoreConfiguration

      Scope Authentication PasswordTimeout Interaction
      ----- -------------- --------------- -----------
CurrentUser       Password            1800      Prompt
```

## Conclusion

Finished! :smile: Hopefully you learned something new and found this post helpful! :slight_smile: There is a lot that I didn't cover when it comes to the SecretManagement module. For example, extensions which lets you use the module with third-party secret management products like Azure KeyVault, KeePass, HashiCorp Vault and more!

Until the next one! 👋

## References

[^1]: https://www.travisgan.com/2015/06/powershell-password-encryption.html
