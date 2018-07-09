# Reporting Services with ADFS-authentication

# IMPORTANT!

Support for **Microsoft SQL Server 2016 Reporting Services** does not work yet, see: https://github.com/RegionOrebroLan-Lab/.NET-ReportingServices-Extensions/issues/1/

However, you should be able to install **Microsoft SQL Server 2017 Reporting Services** and configure it with your *SQL-Server-2016-Reporting-Services-databases*.

Also - when this solution is considered stable this repository will move to https://github.com/RegionOrebroLan/.

=======================================
=======================================

[**SSRS-13**]: https://social.technet.microsoft.com/wiki/contents/articles/50902.sql-server-2016-install-and-configure-ssrs.aspx
[**SSRS-14**]: https://www.microsoft.com/download/details.aspx?id=55252
[**PBIRS-15**]: https://www.microsoft.com/en-us/download/details.aspx?id=56722

This is a guide to set up Reporting Services with ADFS-authentication. This guide applies to:
- [**Microsoft SQL Server 2016 Reporting Services**](https://social.technet.microsoft.com/wiki/contents/articles/50902.sql-server-2016-install-and-configure-ssrs.aspx) - referenced as **SSRS-13** in this document
- [**Microsoft SQL Server 2017 Reporting Services**](https://www.microsoft.com/download/details.aspx?id=55252) - referenced as **SSRS-14** in this document
- [**Power BI Report Server**](https://www.microsoft.com/en-us/download/details.aspx?id=56722) - referenced as **PBIRS-15** in this document

## Help/information about custom security for Reporting Services
- [Configure Custom or Forms Authentication on the Report Server](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-custom-or-forms-authentication-on-the-report-server/)
- https://github.com/Microsoft/Reporting-Services/tree/master/CustomSecuritySample/
- [How to install custom security extensions](https://docs.microsoft.com/en-us/sql/reporting-services/extensions/security-extension/how-to-install-custom-security-extensions/)

## Environment
This guide is written using the following environment:
- **ADFS**-url: https://adfs.local.net/adfs/ls/
- **Reporting Services** urls:
  - https://reports.local.net/ReportServer/
  - https://reports.local.net/Reports/
- Installation path (default):
  - [**SSRS-13**]: *C:\Program Files\Microsoft SQL Server\MSRS13.MSSQLSERVER\Reporting Services*
  - [**SSRS-14**]: *C:\Program Files\Microsoft SQL Server Reporting Services\SSRS*
  - [**PBIRS-15**]: *C:\Program Files\Microsoft Power BI Report Server\PBIRS*

**Important!** The trailing slash in https://reports.local.net/ReportServer/ is important when configuring **AD FS** and in **Web.config**.

To check your version you can check the version of the following assembly:
- [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin\Microsoft.ReportingServices.Interfaces.dll

To check the version of a *.dll:
1. Right-click the *.dll
2. Choose "Properties"
3. Choose "Details"-tab
4. Look at the property "File version"

In the environment for this guide:
- [**SSRS-13**] -> **13.0.5026.0**
- [**SSRS-14**] -> **14.0.600.594**
- [**PBIRS-15**] -> **15.0.1.130**

Generally:
- [**SSRS-13**] -> **13.X.X.X**
- [**SSRS-14**] -> **14.X.X.X**
- [**PBIRS-15**] -> **15.X.X.X**

## Custom assemblies used
- **RegionOrebroLan.ReportingServices.dll**: https://github.com/RegionOrebroLan-Lab/.NET-ReportingServices-Extensions

## Deployment and configuration

### 1 Ensure Reporting Services with SSL is setup

#### 1.1 Installation
- **SSRS-13** - [SQL Server 2016: Install and Configure SSRS](https://social.technet.microsoft.com/wiki/contents/articles/50902.sql-server-2016-install-and-configure-ssrs.aspx)
- **SSRS-14** - [Install SQL Server Reporting Services (2017 and later)](https://docs.microsoft.com/en-us/sql/reporting-services/install-windows/install-reporting-services/)
- **PBIRS-15** - [Install Power BI Report Server](https://docs.microsoft.com/en-us/power-bi/report-server/install-report-server/)

#### 1.2 SSL
- [Configure SSL Connections on a Native Mode Report Server](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-ssl-connections-on-a-native-mode-report-server/)

### 2 Ensure Reporting Services has access to ADFS
1. On the **AD FS** server, open **AD FS Management**
2. Select **Add Relying Party Trust...**
3. **Claims aware**
4. **Enter data about the relying party manually**
5. Enter a **Display name**
6. Skip optional token encryption certificate if you want
7. **Enable support for the WS-Federation Passive protocol** and enter https://reports.local.net/ReportServer/ as **Relying party WS-Federation Passive protocol URL**
8. Skip additional relying party trust identifiers
9. Permit everyone
10. Add claims issuance policy
11. Add rule
12. Send LDAP Attributes as Claims
13. Claim rule  name: *LDAP-attributes*
14. Attribute store: *Active Directory*
15. Mappings: *User-Principal-Name* -> *UPN*

### 3 Stop the service
Stop the windows service:
- [**SSRS-13**] & [**SSRS-14**]: **SQL Server Reporting Services (MSSQLSERVER)**
- [**PBIRS-15**]: **Power BI Report Server**

### 4 Machine-keys
Howto generate a machine-key with IIS: [Easiest way to generate MachineKey](https://blogs.msdn.microsoft.com/amb/2012/07/31/easiest-way-to-generate-machinekey/)

#### 4.1 [**SSRS-13**]

##### 4.1.1 Web.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\web.config

Add the following:

    <configuration>
        ...
        <system.web>
            ...
            <machineKey decryption="AES" decryptionKey="[YOUR-DECRYPTION-KEY]" validation="AES" validationKey="[YOUR-VALIDATION-KEY]" />
            ...
        </system.web>
        ...
    </configuration>

##### 4.1.2 Microsoft.ReportingServices.Portal.WebHost.exe.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\RSWebApp\Microsoft.ReportingServices.Portal.WebHost.exe.config

Add the following:

    <configuration>
        ...
        <system.web>
            <machineKey decryption="AES" decryptionKey="[YOUR-DECRYPTION-KEY]" validation="AES" validationKey="[YOUR-VALIDATION-KEY]" />
        </system.web>
        ...
    </configuration>

#### 4.2 [**SSRS-14**] & [**PBIRS-15**]
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rsreportserver.config

Add the following child to /Configuration:

    <Configuration>
        ...
        <MachineKey Decryption="AES" DecryptionKey="[YOUR-DECRYPTION-KEY]" Validation="AES" ValidationKey="[YOUR-VALIDATION-KEY]" />
        ...
    </Configuration>

**The casing of the attributes is important!**

### 5 RSReportServer.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rsreportserver.config

#### 5.1 /Configuration/Authentication/AuthenticationTypes
Change from

    <Configuration>
	    ...
	    <Authentication>
		    <AuthenticationTypes>
			    <RSWindowsNTLM />
		    </AuthenticationTypes>
		    ...
	    </Authentication>
	    ...
    </Configuration>

to

    <Configuration>
	    ...
	    <Authentication>
		    <AuthenticationTypes>
			    <Custom />
		    </AuthenticationTypes>
		    ...
	    </Authentication>
	    ...
    </Configuration>

#### 5.2 /Configuration/Extensions/Authentication
Change from

    <Configuration>
        ...
        <Extensions>
            ...
            <Authentication>
                <Extension Name="Windows" Type="Microsoft.ReportingServices.Authentication.WindowsAuthentication, Microsoft.ReportingServices.Authorization" />
            </Authentication>
            ...
        </Extensions>
        ...
    </Configuration>

to

    <Configuration>
        ...
        <Extensions>
            ...
            <Authentication>
                <Extension Name="Windows" Type="RegionOrebroLan.ReportingServices.Authentication.WindowsAuthentication, RegionOrebroLan.ReportingServices" />
            </Authentication>
            ...
        </Extensions>
        ...
    </Configuration>

#### 5.3 /Configuration/UI (cookies to pass through)
Add the following as the first child to /Configuration/UI:

##### 5.3.1 Production-environment (SSL)

    <Configuration>
        ...
        <UI>
            <CustomAuthenticationUI>
                <PassThroughCookies>
                    <PassThroughCookie>.ASPXAUTH</PassThroughCookie>
                    <PassThroughCookie>FedAuth</PassThroughCookie>
                    <PassThroughCookie>FedAuth1</PassThroughCookie>
                </PassThroughCookies>
            </CustomAuthenticationUI>
            ...
        </UI>
        ...
    </Configuration>

##### 5.3.2 Development-environment (if you are not using SSL)

    <Configuration>
        ...
        <UI>
            <CustomAuthenticationUI>
                <PassThroughCookies>
                    <PassThroughCookie>.ASPXAUTH</PassThroughCookie>
                    <PassThroughCookie>FedAuth</PassThroughCookie>
                    <PassThroughCookie>FedAuth1</PassThroughCookie>
                </PassThroughCookies>
                <UseSSL>False</UseSSL>
            </CustomAuthenticationUI>
            ...
        </UI>
        ...
    </Configuration>

### 6 RSSrvPolicy.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rssrvpolicy.config

Allow full trust for the assemblies in [**Files.zip**](/Files.zip) by adding the following elements under the **IMembershipCondition**-element of the nested code-group with class *"FirstMatchCodeGroup"*:

    <CodeGroup
	    Name="Log4Net-StrongName"
	    class="UnionCodeGroup"
	    Description="This code group grants Log4Net code full trust."
	    PermissionSetName="FullTrust"
	    version="1"
    >
	    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="0024000004800000940000000602000000240000525341310004000001000100297dcac908e28689360399027b0ea4cd852fbb74e1ed95e695a5ba55cbd1d075ec20cdb5fa6fc593d3d571527b20558d6f39e1f4d5cfe0798428c589c311965244b209c38a02aaa8c9da3b72405b6fedeeb4292c3457e9769b74e645c19cb06c2be75fb2d12281a585fbeabf7bd195d6961ba113286fc3e286d7bbd69024ceda" version="1" />
    </CodeGroup>
    <CodeGroup
	    Name="RegionOrebroLan-StrongName"
	    class="UnionCodeGroup"
	    Description="This code group grants RegionOrebroLan code full trust."
	    PermissionSetName="FullTrust"
	    version="1"
    >
	    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="0024000004800000940000000602000000240000525341310004000001000100d5b5f1623c455dd2ce7a24fcb9ad5db0f32a8793bf2925b82d4b4d43ea1058f5cfd6b8136b8cb850715e921a0256ea4188cfc3b257021125f2cf36b3a584eb6caa674831da70eba16f154ae4ca0dc4cd29dc02d8422e5a72416aeb6bcda9b2e9c06f19df1edb5f5403677345c8f06f2612d571628ef43d2bf6d877c91c94e4d3" version="1" />
    </CodeGroup>
    <CodeGroup
	    Name="StructureMap-StrongName"
	    class="UnionCodeGroup"
	    Description="This code group grants StructureMap code full trust."
	    PermissionSetName="FullTrust"
	    version="1"
    >
	    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="00240000048000009400000006020000002400005253413100040000010001008d9a2a76e43cd9b1b1944b1f3b489a046b33f0bcd755b25cc5d3ed7b18ded38240d6db7578cd986c72d3feb4f94a7ab26fcfa41e3e4f41cf2c029fba91159db05c44d63f0b2bfac24353a07f4a1230dd3d4240340adafa2275277fa083c75958062cd0e60016701db6af7ae718efdf1e802a840595b49c290964255b3c60c494" version="1" />
    </CodeGroup>

That is the **IMembershipCondition**-element, with zone *"MyComputer"*, under the nested **CodeGroup**-element with description *"This code group grants MyComputer code Execution permission."*. Like this:

    <configuration>
	    <mscorlib>
		    <security>
			    <policy>
				    <PolicyLevel version="1">
					    <CodeGroup class="FirstMatchCodeGroup" PermissionSetName="Nothing" version="1">
						    ...
						    <CodeGroup class="FirstMatchCodeGroup" Description="This code group grants MyComputer code Execution permission. " PermissionSetName="Execution" version="1">
							    <IMembershipCondition class="ZoneMembershipCondition" version="1" Zone="MyComputer" />
							    <CodeGroup
								    Name="Log4Net-StrongName"
								    class="UnionCodeGroup"
								    Description="This code group grants Log4Net code full trust."
								    PermissionSetName="FullTrust"
								    version="1"
							    >
								    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="0024000004800000940000000602000000240000525341310004000001000100297dcac908e28689360399027b0ea4cd852fbb74e1ed95e695a5ba55cbd1d075ec20cdb5fa6fc593d3d571527b20558d6f39e1f4d5cfe0798428c589c311965244b209c38a02aaa8c9da3b72405b6fedeeb4292c3457e9769b74e645c19cb06c2be75fb2d12281a585fbeabf7bd195d6961ba113286fc3e286d7bbd69024ceda" version="1" />
							    </CodeGroup>
							    <CodeGroup
								    Name="RegionOrebroLan-StrongName"
								    class="UnionCodeGroup"
								    Description="This code group grants RegionOrebroLan code full trust."
								    PermissionSetName="FullTrust"
								    version="1"
							    >
								    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="0024000004800000940000000602000000240000525341310004000001000100d5b5f1623c455dd2ce7a24fcb9ad5db0f32a8793bf2925b82d4b4d43ea1058f5cfd6b8136b8cb850715e921a0256ea4188cfc3b257021125f2cf36b3a584eb6caa674831da70eba16f154ae4ca0dc4cd29dc02d8422e5a72416aeb6bcda9b2e9c06f19df1edb5f5403677345c8f06f2612d571628ef43d2bf6d877c91c94e4d3" version="1" />
							    </CodeGroup>
							    <CodeGroup
								    Name="StructureMap-StrongName"
								    class="UnionCodeGroup"
								    Description="This code group grants StructureMap code full trust."
								    PermissionSetName="FullTrust"
								    version="1"
							    >
								    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="00240000048000009400000006020000002400005253413100040000010001008d9a2a76e43cd9b1b1944b1f3b489a046b33f0bcd755b25cc5d3ed7b18ded38240d6db7578cd986c72d3feb4f94a7ab26fcfa41e3e4f41cf2c029fba91159db05c44d63f0b2bfac24353a07f4a1230dd3d4240340adafa2275277fa083c75958062cd0e60016701db6af7ae718efdf1e802a840595b49c290964255b3c60c494" version="1" />
							    </CodeGroup>
							    ...
						    </CodeGroup>
						    ...
					    </CodeGroup>
				    </PolicyLevel>
			    </policy>
		    </security>
	    </mscorlib>
    </configuration>

The **PublicKeyBlob**-value depends on what strong-name-key-file you used to sign your assembly. To get the public-key from a strong-name-key-file:

1. Create a directory somewhere on your filesystem
2. Copy your *.snk file to the created directory
3. Create a "GetPublicKey.cmd" file in the same directory
4. Search your filesystem for "sn.exe" and copy the path of one of your hits (eg. "C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\sn.exe")
5. Open the "GetPublicKey.cmd" file (right click and choose Edit)
6. Add the following to it:

        @ECHO OFF
        SET sn="C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\sn.exe"
        %sn% -p SomeName.snk SomeName.PublicKey
        %sn% -tp SomeName.PublicKey
        PAUSE

7. Save "GetPublicKey.cmd"
8. Run the "GetPublicKey.cmd" file (double click it)
9. Copy the displayed key and paste it, then remove any line breaks

To get the public-key from an assembly:

        @ECHO OFF
        SET sn="C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\sn.exe"
        %sn% -Tp "C:\Folder\Assembly.dll"
        PAUSE

### 7 Deploy files
Download the file [**Files.zip**](/Files.zip) and extract it. It has the following content:
  - **13**
    - RegionOrebroLan.ReportingServices.dll
  - **14**
    - RegionOrebroLan.ReportingServices.dll
  - **15**
    - RegionOrebroLan.ReportingServices.dll
- log4net.config
- log4net.dll
- StructureMap.dll
- StructureMap.Net4.dll

If you download the files to the Reporting Services server directly they might be blocked and the assemblies will not be granted access. To resolve it you can right-click each downloaded file and choose properties. If it says: *This file came from another computer and might be blocked to help protect this computer.*, unblock it.

#### 7.1 Generally - all versions
Copy the following files to [\[INSTALLATION-PATH\]](#environment)\ReportServer:
- log4net.config

Copy the following files to [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- log4net.dll
- StructureMap.dll
- StructureMap.Net4.dll

#### 7.2 [**SSRS-13**]
Copy the following files to [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- **13**\RegionOrebroLan.ReportingServices.dll

Copy the following files to [\[INSTALLATION-PATH\]](#environment)\RSWebApp:
- log4net.config

Copy the following files to [\[INSTALLATION-PATH\]](#environment)\RSWebApp\bin:
- log4net.dll
- **13**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

#### 7.3 [**SSRS-14**]
Copy the following files to [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- **14**\RegionOrebroLan.ReportingServices.dll

Copy the following files to [\[INSTALLATION-PATH\]](#environment)\Portal:
- log4net.config
- log4net.dll
- **14**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

#### 7.4 [**PBIRS-15**]
Copy the following files to [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- **15**\RegionOrebroLan.ReportingServices.dll

Copy the following files to [\[INSTALLATION-PATH\]](#environment)\Portal:
- log4net.config
- log4net.dll
- **15**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

Copy the following files to [\[INSTALLATION-PATH\]](#environment)\PowerBI:
- log4net.config
- log4net.dll
- **15**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

Copy [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin\Microsoft.ReportingServices.Authorization.dll to [\[INSTALLATION-PATH\]](#environment)\PowerBI.

### 8 Web.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\web.config

To get the thumbprint value (/configuration/system.identityModel/identityConfiguration/issuerNameRegistry/trustedIssuers/add @thumbprint) run the following PowerShell-command on the ADFS-server:

    Write-Host (Get-AdfsCertificate -CertificateType "Token-Signing").Thumbprint.ToLower();

Add the following parts to Web.config (change XX to 13, 14 or 15, depending on version):

    <configuration>
        ...
        <configSections>
            <section name="structureMap" type="RegionOrebroLan.ReportingServices.StructureMap.Configuration.Section, RegionOrebroLan.ReportingServices, Version=XX.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
            <section name="system.identityModel" type="System.IdentityModel.Configuration.SystemIdentityModelSection, System.IdentityModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
            <section name="system.identityModel.services" type="System.IdentityModel.Services.Configuration.SystemIdentityModelServicesSection, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
        </configSections>
        ...
        <runtime>
            ...
            <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
                ...
                <dependentAssembly>
                    <assemblyIdentity name="StructureMap" culture="neutral" publicKeyToken="e60ad81abae3c223" />
                    <bindingRedirect newVersion="3.1.9.463" oldVersion="0.0.0.0-3.1.9.463" />
                </dependentAssembly>
                ...
            </assemblyBinding>
            ...
        </runtime>
        ...
        <structureMap>
            <registries>
                <add type="RegionOrebroLan.ReportingServices.StructureMap.Registry, RegionOrebroLan.ReportingServices, Version=XX.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
            </registries>
        </structureMap>
        ...
        <system.identityModel>
            <identityConfiguration>
                <audienceUris>
                    <add value="https://reports.local.net/ReportServer/" />
                </audienceUris>
                <certificateValidation certificateValidationMode="PeerOrChainTrust" />
                <claimsAuthenticationManager type="RegionOrebroLan.ReportingServices.Security.Claims.WindowsFederationClaimsAuthenticationManager, RegionOrebroLan.ReportingServices, Version=XX.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />    
                <issuerNameRegistry type="System.IdentityModel.Tokens.ConfigurationBasedIssuerNameRegistry, System.IdentityModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089">
                    <trustedIssuers>
                        <add name="https://adfs.local.net/adfs/services/trust/" thumbprint="[THUMBPRINT-VALUE]" />
                    </trustedIssuers>
                </issuerNameRegistry>
            </identityConfiguration>
        </system.identityModel>
        ...
        <system.identityModel.services>
            <federationConfiguration>
                <cookieHandler requireSsl="true" />
                <wsFederation issuer="https://adfs.local.net/adfs/ls/" passiveRedirectEnabled="true" realm="https://reports.local.net/ReportServer/" requireHttps="true" />
            </federationConfiguration>
        </system.identityModel.services>
        ...
        <system.web>
            ...
            <authorization>
                <deny users="?" />
            </authorization>
            ...
            <httpModules>
                <clear />
                <add name="BootstrapperModule" type="RegionOrebroLan.ReportingServices.Web.BootstrapperModule, RegionOrebroLan.ReportingServices, Version=XX.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
                <add name="CustomErrorHandlerModule" type="RegionOrebroLan.ReportingServices.Web.ErrorHandlerModule, RegionOrebroLan.ReportingServices, Version=XX.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
                <add name="ErrorHandlerModule" type="System.Web.Mobile.ErrorHandlerModule, System.Web.Mobile, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
                <add name="OutputCache" type="System.Web.Caching.OutputCacheModule" />
                <add name="SessionAuthenticationModule" type="System.IdentityModel.Services.SessionAuthenticationModule, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
                <!-- Must be declared after the SessionAuthenticationModule. -->
                <add name="FederationAuthenticationModule" type="RegionOrebroLan.ReportingServices.Web.FederationAuthenticationModule, RegionOrebroLan.ReportingServices, Version=XX.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
            </httpModules>
            ...
            <identity impersonate="false" /><!-- Or remove the identity-element. -->
            ...
        </system.web>
        ...
    </configuration>

### 9 Start the service
Start the windows service:
- [**SSRS-13**] & [**SSRS-14**]: **SQL Server Reporting Services (MSSQLSERVER)**
- [**PBIRS-15**]: **Power BI Report Server**