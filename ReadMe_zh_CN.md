[English](./ReadMe.md)

# 具有ADFS身份验证的Reporting Services

# 注意!

对**Microsoft SQL Server 2016 Reporting Services **的支持尚不起作用，请参阅： https://github.com/RegionOrebroLan-Lab/.NET-ReportingServices-Extensions/issues/1/

但是，您仍能够安装**Microsoft SQL Server 2017 Reporting Services**并使用*SQL-Server-2016-Reporting-Services-databases*进行配置。

此外 - 当此解决方案被认为是稳定的时，此存储库将移至 https://github.com/RegionOrebroLan/.

进入正文
=======================================

[**SSRS-13**]: https://social.technet.microsoft.com/wiki/contents/articles/50902.sql-server-2016-install-and-configure-ssrs.aspx
[**SSRS-14**]: https://www.microsoft.com/download/details.aspx?id=55252
[**PBIRS-15**]: https://www.microsoft.com/en-us/download/details.aspx?id=56722

这是使用ADFS身份验证设置Reporting Services的指南。 本指南适用于：
- [**Microsoft SQL Server 2016 Reporting Services**](https://social.technet.microsoft.com/wiki/contents/articles/50902.sql-server-2016-install-and-configure-ssrs.aspx) - 在本文档中引用为**SSRS-13**
- [**Microsoft SQL Server 2017 Reporting Services**](https://www.microsoft.com/download/details.aspx?id=55252) -  在本文档中引用为 **SSRS-14**
- [**Power BI Report Server**](https://www.microsoft.com/en-us/download/details.aspx?id=56722) -  在本文档中引用为**PBIRS-15** 

## 有关Reporting Services的自定义安全性的帮助/信息
- [Configure Custom or Forms Authentication on the Report Server](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-custom-or-forms-authentication-on-the-report-server/)
- https://github.com/Microsoft/Reporting-Services/tree/master/CustomSecuritySample/
- [How to install custom security extensions](https://docs.microsoft.com/en-us/sql/reporting-services/extensions/security-extension/how-to-install-custom-security-extensions/)

## 环境
本指南使用以下环境编写：
- **ADFS**-url: https://adfs.local.net/adfs/ls/
- **Reporting Services** urls:
  - https://reports.local.net/ReportServer/
  - https://reports.local.net/Reports/
- 安装路径（默认）:
  - [**SSRS-13**]: *C:\Program Files\Microsoft SQL Server\MSRS13.MSSQLSERVER\Reporting Services*
  - [**SSRS-14**]: *C:\Program Files\Microsoft SQL Server Reporting Services\SSRS*
  - [**PBIRS-15**]: *C:\Program Files\Microsoft Power BI Report Server\PBIRS*

**重要！**配置**AD FS **和**Web.config **时，https://reports.local.net/ReportServer/中的尾部斜杠非常重要。

要检查您的版本，您可以检查以下程序集的版本：

- [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin\Microsoft.ReportingServices.Interfaces.dll

要检查* .dll的版本：
1. 右键单击* .dll
2. 选择“属性”
3. 选择“详细信息” - 选项卡
4. 查看属性“文件版本”

在本指南的环境中：
- [**SSRS-13**] -> **13.0.5026.0**
- [**SSRS-14**] -> **14.0.600.594**
- [**PBIRS-15**] -> **15.0.1.130**

典型值:
- [**SSRS-13**] -> **13.X.X.X**
- [**SSRS-14**] -> **14.X.X.X**
- [**PBIRS-15**] -> **15.X.X.X**

## 使用自定义组件
- **RegionOrebroLan.ReportingServices.dll**: https://github.com/RegionOrebroLan-Lab/.NET-ReportingServices-Extensions

## 部署和配置

### 1 确保已设置使用SSL的Reporting Services

#### 1.1 安装
- **SSRS-13** - [SQL Server 2016: Install and Configure SSRS](https://social.technet.microsoft.com/wiki/contents/articles/50902.sql-server-2016-install-and-configure-ssrs.aspx)
- **SSRS-14** - [Install SQL Server Reporting Services (2017 and later)](https://docs.microsoft.com/en-us/sql/reporting-services/install-windows/install-reporting-services/)
- **PBIRS-15** - [Install Power BI Report Server](https://docs.microsoft.com/en-us/power-bi/report-server/install-report-server/)

#### 1.2 SSL
- [Configure SSL Connections on a Native Mode Report Server](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-ssl-connections-on-a-native-mode-report-server/)

### 2 确保Reporting Services可以访问ADFS
1. 在**AD FS **服务器上，打开**AD FS管理**
2. 选择**添加依赖方信任... **
3. **声明意识**
4. **手动输入有关信赖方的数据**
5. 输入**显示名称**
6. 令牌加密证书可以选择跳过
7. **启用对WS-Federation Passive协议的支持**并输入https://reports.local.net/ReportServer/ as **Relying party WS-Federation Passive protocol URL**
8. 跳过其他依赖方信任标识符
9. 允许每个人
10. 添加索赔发放政策
11. 添加规则
12. 将LDAP属性作为声明发送
13. 声明规则名称：* LDAP-attributes *
14. 属性存储：* Active Directory *
15. 映射：*用户 - 主体 - 名称*  - > * UPN *

### 3 停止服务
停止windows服务:
- [**SSRS-13**] & [**SSRS-14**]: **SQL Server Reporting Services (MSSQLSERVER)**
- [**PBIRS-15**]: **Power BI Report Server**

### 4 Machine-keys
如何使用IIS生成计算机密钥: [Easiest way to generate MachineKey](https://blogs.msdn.microsoft.com/amb/2012/07/31/easiest-way-to-generate-machinekey/)

#### 4.1 [**SSRS-13**]

##### 4.1.1 Web.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\web.config

添加以下内容：

```xml
<configuration>
    ...
    <system.web>
        ...
        <machineKey decryption="AES" decryptionKey="[YOUR-DECRYPTION-KEY]" validation="AES" validationKey="[YOUR-VALIDATION-KEY]" />
        ...
    </system.web>
    ...
</configuration>
```

##### 4.1.2 Microsoft.ReportingServices.Portal.WebHost.exe.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\RSWebApp\Microsoft.ReportingServices.Portal.WebHost.exe.config

添加以下内容：

```xml
<configuration>
    ...
    <system.web>
        <machineKey decryption="AES" decryptionKey="[YOUR-DECRYPTION-KEY]" validation="AES" validationKey="[YOUR-VALIDATION-KEY]" />
    </system.web>
    ...
</configuration>
```

#### 4.2 [**SSRS-14**] & [**PBIRS-15**]
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rsreportserver.config

添加以下子元素到： /Configuration:

```xml
<Configuration>
    ...
    <MachineKey Decryption="AES" DecryptionKey="[YOUR-DECRYPTION-KEY]" Validation="AES" ValidationKey="[YOUR-VALIDATION-KEY]" />
    ...
</Configuration>
```

**属性的外壳很重要！**

### 5 RSReportServer.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rsreportserver.config

#### 5.1 /Configuration/Authentication/AuthenticationTypes
Change from

```xml
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
```

to

```xml
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
```

#### 5.2 /Configuration/Extensions/Authentication
Change from

```xml
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
```

to

```xml
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
```

#### 5.3 /Configuration/UI (cookies to pass through)
将以下内容作为第一个子项添加到/Configuration/UI:

##### 5.3.1 生产环境 (SSL)

```xml
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
```

##### 5.3.2 开发环境 (如果未使用 SSL)

```xml
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
```

### 6 RSSrvPolicy.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\rssrvpolicy.config

通过在** IMembershipCondition **  - 嵌套代码组的元素下添加以下元素，使用类*“FirstMatchCodeGroup”*，允许完全信任[** Files.zip **](./Files.zip)中的程序集：

```xml
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
```

这是** IMembershipCondition **  - 元素，带有区域*“MyComputer”*，位于嵌套的** CodeGroup **  - 元素下，带有描述* *"This code group grants MyComputer code Execution permission."*. 像这样:

```xml
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
```

**PublicKeyBlob **  - 值取决于您用于签署程序集的强名称密钥文件。 要从强名称密钥文件中获取公钥：

1. 在文件系统的某个位置创建一个目录
2. 将* .snk文件复制到创建的目录
3. 在同一目录中创建“GetPublicKey.cmd”文件
4. 在文件系统中搜索“sn.exe”并复制其中一个命中的路径（例如“C：\Program Files\Microsoft SDKs\Windows\v7.1\Bin\sn.exe”）
5. 打开“GetPublicKey.cmd”文件（右键单击并选择“编辑”）
6. 添加以下内容：

    ```powershell
    @ECHO OFF
    SET sn="C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\sn.exe"
    %sn% -p SomeName.snk SomeName.PublicKey
    %sn% -tp SomeName.PublicKey
    PAUSE
    ```

7. 保存“GetPublicKey.cmd”
8. 运行“GetPublicKey.cmd”文件（双击它）
9. 复制显示的键并粘贴，然后删除所有换行符

要从程序集中获取公钥：

```powershell
    @ECHO OFF
    SET sn="C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\sn.exe"
    %sn% -Tp "C:\Folder\Assembly.dll"
    PAUSE
```

### 7 部署文件
下载文件[**Files.zip **](/ Files.zip)并解压缩。 它具有以下内容：
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

如果直接将文件下载到Reporting Services服务器，则可能会阻止这些文件，并且不会授予程序集访问权限。 要解决此问题，您可以右键单击每个下载的文件并选择属性。 如果它说：*此文件来自另一台计算机，可能会被阻止以帮助保护此计算机。*，取消阻止它。

#### 7.1 通常 - 所有版本
将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\ReportServer:
- log4net.config

将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- log4net.dll
- StructureMap.dll
- StructureMap.Net4.dll

#### 7.2 [**SSRS-13**]
将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- **13**\RegionOrebroLan.ReportingServices.dll

将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\RSWebApp:
- log4net.config

将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\RSWebApp\bin:
- log4net.dll
- **13**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

#### 7.3 [**SSRS-14**]
将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- **14**\RegionOrebroLan.ReportingServices.dll

将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\Portal:
- log4net.config
- log4net.dll
- **14**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

#### 7.4 [**PBIRS-15**]
将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin:
- **15**\RegionOrebroLan.ReportingServices.dll

将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\Portal:
- log4net.config
- log4net.dll
- **15**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

将以下文件复制到 [\[INSTALLATION-PATH\]](#environment)\PowerBI:
- log4net.config
- log4net.dll
- **15**\RegionOrebroLan.ReportingServices.dll
- StructureMap.dll
- StructureMap.Net4.dll

Copy [\[INSTALLATION-PATH\]](#environment)\ReportServer\bin\Microsoft.ReportingServices.Authorization.dll to [\[INSTALLATION-PATH\]](#environment)\PowerBI.

### 8 Web.config
**Path:** [\[INSTALLATION-PATH\]](#environment)\ReportServer\web.config

要获取指纹值（/configuration/system.identityModel/identityConfiguration/issuerNameRegistry/trustedIssuers/add @thumbprint），请在ADFS服务器上运行以下PowerShell命令：

```powershell
Write-Host (Get-AdfsCertificate -CertificateType "Token-Signing").Thumbprint.ToLower();
```

将以下部分添加到Web.config（将XX更改为13,14或15，具体取决于版本）：

```xml
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
```

### 9 启动服务
启动windows服务:
- [**SSRS-13**] & [**SSRS-14**]: **SQL Server Reporting Services (MSSQLSERVER)**
- [**PBIRS-15**]: **Power BI Report Server**