# Convert-ASP.NET-Core-Application-as-Linux-Service

In this article, i'm going to review the steps you need to know in order to convert your application and deploy to a Linux server running Apache

## I Modify Dot Net Core project:

1.1 ) modify all project files: <project name>.csproj

into section PropertyGroup:

       <PropertyGroup>
			<AssemblyTitle>MicroServiceBePro.StaticTables</AssemblyTitle>
			<TargetFramework>netcoreapp1.1</TargetFramework>
			<AssemblyName>StaticLoader</AssemblyName>
			<PackageId>StaticLoader</PackageId>
			<NetStandardImplicitPackageVersion>1.6.1</NetStandardImplicitPackageVersion>
			<PackageTargetFallback>$(PackageTargetFallback);dnxcore50</PackageTargetFallback>
			<GenerateAssemblyConfigurationAttribute>false</GenerateAssemblyConfigurationAttribute>
			<GenerateAssemblyCompanyAttribute>false</GenerateAssemblyCompanyAttribute>
			<GenerateAssemblyProductAttribute>false</GenerateAssemblyProductAttribute>
		</PropertyGroup>
  
  insert additional line :
  
		  <RuntimeIdentifiers>win7-x64;win7-x86;ubuntu.16.04-x64;</RuntimeIdentifiers>
		  <SuppressDockerTargets>True</SuppressDockerTargets>
		  
  New section look look like here:
	
	       <PropertyGroup>
			<AssemblyTitle>MicroServiceBePro.StaticTables</AssemblyTitle>
			<TargetFramework>netcoreapp1.1</TargetFramework>
			<AssemblyName>StaticLoader</AssemblyName>
			<PackageId>StaticLoader</PackageId>
			
			<PreserveCompilationContext>true</PreserveCompilationContext>
			<RuntimeIdentifiers>win7-x64;win7-x86;ubuntu.16.04-x64;</RuntimeIdentifiers>
		    <SuppressDockerTargets>True</SuppressDockerTargets>
			

			<GenerateAssemblyConfigurationAttribute>false</GenerateAssemblyConfigurationAttribute>
			<GenerateAssemblyCompanyAttribute>false</GenerateAssemblyCompanyAttribute>
			<GenerateAssemblyProductAttribute>false</GenerateAssemblyProductAttribute>
		</PropertyGroup>
	
1.2) Visual Studio 2017:

    Do operation:
       - Clean Solution
	   - Rebuild Solution
	   
	Look project errors
	   
1.2.3 - Tools -> Nuget Package Manager

	       find add remove package: 
		       Microsoft.ApplicationInsights.AspNetCore.
			This package using for Microsoft Azure server. 
			
1.2.4 Remove from Startup.cs ( Startup class ) next code:

            if (env.IsDevelopment())
            {
                // This will push telemetry data through Application Insights pipeline faster, allowing you to view results immediately.
                builder.AddApplicationInsightsSettings(developerMode: true);
            }		
			
1.2.5 Do it: Clean Solution, Rebuild All	
		
1.3 Run Command Prompt ( cmd )
     		
		dotnet publish -c Release -r ubuntu.16.04-x64
		
		

## II Modify Ubuntu Server.

2.1 Create application store foder

         sudo mkdir /var/www/beprotb
		 
2.2 copy all files 
	      from:   bin/Release/netcoreapp1.1/ubuntu.16.04-x64/publish	
            to:	  /var/www/beprotb	 


2.3  make a file executable and runnable on the linux server 

			sudo chmod +x BeproTB


2.4  test application

			sudo ./BeproTB


### Create linux service			
			
2.5  create service definition file:
	  
	       /lib/systemd/system/beprotb.service
		   
		   where beprotb - service name 

			using editor : nano
			
			sudo nano /lib/systemd/system/beprotb.service
			
file context:
			
				[Unit]
					Description= BeproTB
					
					[Service]
					ExecStart=/var/www/beprotb/BeproTB
					Restart=always
					RestartSec=60
					SyslogIdentifier=beprotb
					Environment=ASPNETCORE_ENVIRONMENT=Production

					[Install]
				WantedBy=bepro.target
	  
2.6  make folder /etc/systemd/system/bepro.target.wants

2.7  enable service :

			sudo  systemctl enable /lib/systemd/system/beprotb.service
			
2.8 start service :
				
			sudo service beprotb start

2.9 check service status:

			sudo service beprotb status	

					you can use next command: 

							sudo service beprotb stop

							sudo service beprotb start

							sudo service beprotb restart

							sudo service beprotb status


### Create appache Virtual Host 

2.10.1 Start by copying the file for the first domain:

					sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/beprotb.conf


2.10.2 Edit site definition
			
					cd /etc/apache2/sites-available

					sudo nano beprotb.conf
					
					file context:

						<VirtualHost *:80>
							ProxyPreserveHost On
							ProxyPass / http://http://127.0.0.1:5001/
							ProxyPassReverse / http://127.0.0.1:5001/
							ServerName beprotb.sgcombo.com
						</VirtualHost>

2.10.3 After you create virtual host file, you must enable them.

					sudo a2ensite beprotb.conf
					
2.10.4 Restart Apache

					sudo service apache2 restart

You can use command:

			sudo service apache2 stop

			sudo service apache2 start

			sudo service apache2 restart

			sudo service apache2 status






