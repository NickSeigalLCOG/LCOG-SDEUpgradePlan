# ArcSDE Upgrade Plan


## What an Upgrade Means

The current definition of "upgrade" with regards to ArcSDE is to "upgrade the geodatabase system tables and install updated stored procedures, types, and functions". There is no requirement to install new or newer-versioned applications in order to upgrade a database: all ArcSDE properties are contained within the database itself or the user application that interfaces with it (e.g. ArcGIS Desktop, Server).

## Geodatbase-only Upgrade

As of ArcSDE 10.0, upgrades must be done via the "Upgrade Geodatabase" tool in the Geodatabase Administration Toolset. Using the sdesetup command-line tool will not work.

### Steps for Upgrading a geodatabase.

These are based on Esri's recommended procedure for upgrading a geodatabase, amended to our system and preferences.

Be sure that you have the most current version of ArcGIS Desktop, Engine Runtime, or Server installed on the machine you will be administrating the upgrade from. Post 10.1, you can only install an ArcSDE version that is concurrent or older than the program you are running the Upgrade Geodatabase tool from.

Test upgrade in a development environment; test regional uses (registering, versioning, versioned views, edit tracking, topology).

Choose a date for upgrading, and provide notification to all likely users.

On the upgrade date, for each database being upgraded:

  Create a backup of the database.

  Remove any custom functionality you may have added to the geodatabase system tables outside ArcGIS such as triggers, 
participation in SQL Server replication, or additional indexes. The upgrade procedure cannot take into account customizations you make to the system tables. If such customizations prevent the alteration of a system table's schema, the upgrade will fail.

  If you are using an ArcSDE service, do the following: Stop (do not pause) the service and delete it.

  Pre 10.1: Uninstall the old version of ArcSDE.
  Post 10.1: Uninstall the old version of ArcSDE application server (if in use), and install the new version.



Connect directly to the geodatabase. In most cases, you will connect as the geodatabase administrator.
Open the Geodatabase Properties dialog box, click the General tab, then click Upgrade Geodatabase. This opens the Upgrade Geodatabase geoprocessing tool dialog box.
Run the Upgrade Geodatabase tool.
On Windows, re-create the ArcSDE service (if used) using the sdeservice and sdemon commands. On UNIX or Linux, start an ArcSDE service (if used) using the sdemon command. NoteNote:
The sdeservice and sdemon ArcSDE administration commands are installed with the ArcSDE application server.



Backup Plan
Downtime
Services/Native Client
Regression Plan
Minimum Database Version
Command Line Tools
Versioned Views
Geometry



## Complete Upgrade (SQL Server upgrade) - TBW


## Sources
(1) A quick tour of enterprise geodatabase upgrades
http://resources.arcgis.com/en/help/main/10.2/index.html#/A_quick_tour_of_enterprise_geodatabase_upgrades/002q0000005p000000/

An overview of the Geodatabase Administration toolset
http://resources.arcgis.com/en/help/main/10.2/index.html#/An_overview_of_the_Geodatabase_Administration_toolset/001700000009000000/
