# Reporting-Services-With-ADFS-Authentication

## Help/information about custom security for Reporting Services
- [Configure Custom or Forms Authentication on the Report Server](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-custom-or-forms-authentication-on-the-report-server/)
- https://github.com/Microsoft/Reporting-Services/tree/master/CustomSecuritySample

## Custom assemblies used

- **RegionOrebroLan.IdentityModel.dll**: https://github.com/RegionOrebroLan/.NET-IdentityModel-Extensions
- **RegionOrebroLan.ReportingServices.dll**: https://github.com/RegionOrebroLan/.NET-ReportingServices-Extensions

## Deployment and configuration

### 1 Windows Identity Foundation 3.5
In **RegionOrebroLan.IdentityModel.dll** SAML-tokens are converted to a "impersonatable" WindowsIdentity. To be able to create an "impersonatable" WindowsIdentity from a user-principal-name, by calling:

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

### 2 RSReportServer.config
**Path:** *C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\rsreportserver.config*

#### 2.1 /Configuration/Authentication/AuthenticationTypes
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

#### 2.2 /Configuration/Extensions/Authentication
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

#### 2.3 /Configuration/UI (cookies to pass through)
Add the following as the first child to /Configuration/UI:

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

### 3 RSSrvPolicy.config
**Path:** *C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\rssrvpolicy.config*

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
								    Name="RegionOrebroLan-StrongName"
								    class="UnionCodeGroup"
								    Description="This code group grants RegionOrebroLan code full trust."
								    PermissionSetName="FullTrust"
								    version="1"
							    >
								    <IMembershipCondition class="StrongNameMembershipCondition" PublicKeyBlob="{RegionOrebroLan-StrongName}" version="1" />
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

### 4 Web.config
**Path:** *C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\web.config**

### 5 Deploy assemblies
Copy the following dll's
- Microsoft.IdentityModel.dll
- RegionOrebroLan.IdentityModel.dll
- RegionOrebroLan.ReportingServices.dll

to:
  - C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\bin
  - C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\Portal