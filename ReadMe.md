# Reporting-Services-With-ADFS-Authentication

This is a guide to set up Reporting Services with ADFS authentication. This guide applies to [Microsoft SQL Server 2017 Reporting Services](https://www.microsoft.com/download/details.aspx?id=55252) or [Power BI Report Server](https://www.microsoft.com/en-us/download/details.aspx?id=56722).

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
  - **SQL Server Reporting Services**: *C:\Program Files\Microsoft SQL Server Reporting Services\SSRS*
  - **Power BI Report Server**: *C:\Program Files\Microsoft Power BI Report Server\PBIRS*

**Important!** The trailing slash in https://reports.local.net/ReportServer/ is important when configuring **AD FS** and in **Web.config**.

## Custom assemblies used
- **RegionOrebroLan.IdentityModel.dll**: https://github.com/RegionOrebroLan/.NET-IdentityModel-Extensions
- **RegionOrebroLan.ReportingServices.dll**: https://github.com/RegionOrebroLan/.NET-ReportingServices-Extensions

## Deployment and configuration

### 1 Ensure Reporting Services with SSL ís setup
Either **SQL Server Reporting Services** or **Power BI Report Server**.

#### 1.1 SQL Server Reporting Services
- [Install SQL Server Reporting Services (2017 and later)](https://docs.microsoft.com/en-us/sql/reporting-services/install-windows/install-reporting-services/)
- [Configure SSL Connections on a Native Mode Report Server](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-ssl-connections-on-a-native-mode-report-server/)

#### 1.2 Power BI Report Server
- [Quickstart: Install Power BI Report Server](https://docs.microsoft.com/en-US/power-bi/report-server/quickstart-install-report-server/)
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

### 3 Windows Identity Foundation 3.5
In **RegionOrebroLan.IdentityModel.dll** SAML-tokens are converted to an "impersonatable" WindowsIdentity. To be able to create an "impersonatable" WindowsIdentity from a user-principal-name, by calling:

    Microsoft.IdentityModel.WindowsTokenService.S4UClient.UpnLogon("firstname.lastname@company.com");

you need to configure the **Claims to Windows Token Service** (windows-service). First you need to install the windows-feature **Windows Identity Foundation 3.5** on the computer. Then you have to allow the Reporting-Services-process-account access to run it by adding the account to **C:\Program Files\Windows Identity Foundation\v3.5\c2wtshost.exe.config**:

    <?xml version="1.0"?>
        <configuration>
            <configSections>
                <section name = "windowsTokenService" type="Microsoft.IdentityModel.WindowsTokenService.Configuration.WindowsTokenServiceSection, Microsoft.IdentityModel.WindowsTokenService, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" />
            </configSections>
            <startup>
                <supportedRuntime version="v2.0.50727" />
                <supportedRuntime version="v4.0" />
            </startup>
            <windowsTokenService>
                <!-- By default no callers are allowed to use the Windows Identity Foundation Claims To NT Token Service. Add the identities you wish to allow below. -->
                <allowedCallers>
                    <clear />
                    <!-- <add value="NT AUTHORITY\Network Service" /> -->
                    <!-- <add value="NT AUTHORITY\Local Service" /> -->
                    <!-- <add value="NT AUTHORITY\System" /> -->
                    <!-- <add value="NT AUTHORITY\Authenticated Users" /> -->
                    <add value="NT Service\SQLServerReportingServices" />
                </allowedCallers>
            </windowsTokenService>
        </configuration>

Then restart the windows-service **Claims to Windows Token Service**.

### 4 Stop the service
Stop the windows service:
- **SQL Server Reporting Services (MSSQLSERVER)**, if running SQL Server Reporting Services
- **Power BI Report Server**, if running Power BI Report Server

### 5 RSReportServer.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rsreportserver.config

#### 5.1 /Configuration (machine-keys)
Howto generate a machine-key with IIS: [Easiest way to generate MachineKey](https://blogs.msdn.microsoft.com/amb/2012/07/31/easiest-way-to-generate-machinekey/)

Add the following child to /Configuration:

    <Configuration>
        ...
        <MachineKey Decryption="AES" DecryptionKey="[YOUR-DECRYPTION-KEY]" Validation="AES" ValidationKey="[YOUR-VALIDATION-KEY]" />
        ...
    </Configuration>

**The casing of the attributes is important!**

#### 5.2 /Configuration/Authentication/AuthenticationTypes
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

#### 5.3 /Configuration/Extensions/Authentication
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

#### 5.4 /Configuration/UI (cookies to pass through)
Add the following as the first child to /Configuration/UI:

##### 5.4.1 Production-environment (SSL)

    <Configuration>
        ...
        <UI>
            <CustomAuthenticationUI>
                <PassThroughCookies>
                    <PassThroughCookie>FedAuth</PassThroughCookie>
                    <PassThroughCookie>FedAuth1</PassThroughCookie>
                </PassThroughCookies>
            </CustomAuthenticationUI>
            ...
        </UI>
        ...
    </Configuration>

##### 5.4.2 Development-environment (if you are not using SSL)

    <Configuration>
        ...
        <UI>
            <CustomAuthenticationUI>
                <PassThroughCookies>
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

Allow RegionOrebroLan-StrongName full trust by adding the following section as the first element under the nested code-group with class "FirstMatchCodeGroup":

    <configuration>
	    <mscorlib>
		    <security>
			    <policy>
				    <PolicyLevel version="1">
					    <CodeGroup class="FirstMatchCodeGroup" PermissionSetName="Nothing" version="1">
						    ...
						    <CodeGroup class="FirstMatchCodeGroup" Description="This code group grants MyComputer code Execution permission. " PermissionSetName="Execution" version="1">
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

### 7 Web.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\web.config

        <?xml version="1.0" encoding="utf-8"?>
        <configuration>
            ...
            <configSections>
                <section name="structureMap" type="RegionOrebroLan.ReportingServices.StructureMap.Configuration.Section, RegionOrebroLan.ReportingServices, Version=1.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
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
                    <add type="RegionOrebroLan.ReportingServices.StructureMap.Registry, RegionOrebroLan.ReportingServices, Version=1.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
                </registries>
            </structureMap>
            ...
            <system.identityModel>
                <identityConfiguration>
                    <audienceUris>
                        <add value="https://reports.local.net/ReportServer/" />
                    </audienceUris>
                    <certificateValidation certificateValidationMode="PeerOrChainTrust" />
                    <issuerNameRegistry type="System.IdentityModel.Tokens.ConfigurationBasedIssuerNameRegistry, System.IdentityModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089">
                        <trustedIssuers>
                            <add name="https://adfs.local.net/adfs/services/trust/" thumbprint="ef3d1322792e8d32f4f20c921bdf259e069bf567" />
                        </trustedIssuers>
                    </issuerNameRegistry>
                    <securityTokenHandlers>
                        <remove type="System.IdentityModel.Tokens.SamlSecurityTokenHandler, System.IdentityModel, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
                        <add type="RegionOrebroLan.IdentityModel.Tokens.SamlImpersonatableSecurityTokenHandler, RegionOrebroLan.IdentityModel, Version=1.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead">
                            <samlSecurityTokenRequirement issuerCertificateRevocationMode="Online" issuerCertificateTrustedStoreLocation="LocalMachine" issuerCertificateValidationMode="PeerOrChainTrust" mapToWindows="true">
                                <nameClaimType value="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name" />
                                <roleClaimType value="schemas.microsoft.com/ws/2006/04/identity/claims/role" />
                            </samlSecurityTokenRequirement>
                        </add>
                    </securityTokenHandlers>
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
                ,,,
                <httpModules>
                    <clear />
                    <add name="BootstrapperModule" type="RegionOrebroLan.ReportingServices.Web.BootstrapperModule, RegionOrebroLan.ReportingServices, Version=1.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
                    <add name="CustomErrorHandlerModule" type="RegionOrebroLan.ReportingServices.Web.ErrorHandlerModule, RegionOrebroLan.ReportingServices, Version=1.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
                    <add name="ErrorHandlerModule" type="System.Web.Mobile.ErrorHandlerModule, System.Web.Mobile, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
                    <add name="OutputCache" type="System.Web.Caching.OutputCacheModule" />
                    <add name="SessionAuthenticationModule" type="System.IdentityModel.Services.SessionAuthenticationModule, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
                    <!-- Must be declared after the SessionAuthenticationModule. -->
                    <add name="FederationAuthenticationModule" type="RegionOrebroLan.ReportingServices.Web.FederationAuthenticationModule, RegionOrebroLan.ReportingServices, Version=1.0.0.0, Culture=neutral, PublicKeyToken=520b099ae7bbdead" />
                </httpModules>
                ...
            </system.web>
            ...
        </configuration>

### 8 Deploy files
Copy the following dll's
- Microsoft.IdentityModel.dll
- RegionOrebroLan.IdentityModel.dll
- RegionOrebroLan.ReportingServices.dll

to:
1. [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin
2. [\[INSTALLATION-PATH\]](#environment)\Portal

### 9 Start the service
Start the windows service:
- **SQL Server Reporting Services (MSSQLSERVER)**, if running SQL Server Reporting Services
- **Power BI Report Server**, if running Power BI Report Server