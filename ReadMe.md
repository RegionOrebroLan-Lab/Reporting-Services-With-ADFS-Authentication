# Reporting-Services-With-ADFS-Authentication

## 1 Windows Identity Foundation 3.5
Install the **Windows Identity Foundation 3.5** feature.

## 2 RSReportServer.config
C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\rsreportserver.config

From

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

## 3 Web.config
C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\Web.config

## 4 Deploy assemblies
Copy the following dll's
- Microsoft.IdentityModel
- RegionOrebroLan.IdentityModel
to:
  - C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\bin
  - C:\Program Files\Microsoft SQL Server Reporting Services\SSRS\Portal