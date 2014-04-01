# ArcSDE Upgrade Plan


### What an Upgrade Means

The current definition of "upgrade" with regards to ArcSDE is to "upgrade the geodatabase system tables and install updated stored procedures, types, and functions". There is no requirement to install new or newer-versioned applications in order to upgrade a database: all ArcSDE properties are contained within the database itself or the user application that interfaces with it (e.g. ArcGIS Desktop, Server).

### Geodatbase-only Upgrade

As of ArcSDE 10.0, upgrades must be done via the "Upgrade Geodatabase" tool in the Geodatabase Administration Toolset. Using the sdesetup command-line tool will not work.

#### Steps for Upgrading a geodatabase.

These are based on Esri's recommended procedure for upgrading a geodatabase, amended to our system and preferences.

Be sure that you have the most current version of ArcGIS Desktop, Engine Runtime, or Server installed on the machine you will be administrating the upgrade from. Post 10.1, you can only install an ArcSDE version that is concurrent or older than the program you are running the Upgrade Geodatabase tool from.

Also check to see that your database server meets the minimum system requirements (3).

Test upgrade in a development environment; test regional uses (registering, versioning, versioned views, edit tracking, topology).

Choose a date for upgrading, and provide notification to all likely users.

On the upgrade date, for each database being upgraded:

1. Create a backup of the database.
2. Remove any custom functionality you may have added to the geodatabase system tables outside ArcGIS such as: triggers, participation in SQL Server replication, or additional indexes. The upgrade procedure cannot take into account customizations you make to the system tables. If such customizations prevent the alteration of a system table's schema, the upgrade will fail.
3. Disconnect remaining users from the geodatabase, and block any new connections (right-click > Properties > Connections).
4. If you are using an ArcSDE service:
  i.   Stop (do not pause) the service, via Windows Services.
  ii.  Uninstall the application server (pre-10.1: uninstall ArcSDE).
  iii. Delete the service when prompted by uininstaller.
5. If you are using the ArcSDE command-line tools, uninstall them.
6. Run the Upgrade Geodatabase tool, by either:
  i.  Open the Upgrade Geodatabase tool via the toolbox or a Python script.
  ii. Connect directly to the geodatabase with a login of sufficient administrative permissions. Right-click on the connection file select Properties > General > Upgrade Geodatabase. This opens the Upgrade Geodatabase geoprocessing tool.
7. Be sure to leave "Perform-Pre-Requisite Check" enabled. This double-checks: no other active connections, you have sufficient privileges, the database can support XML columns, all datasets can be opened. If any prerequisites are not met, the tool terminates.
8. Any network datasets, parcel fabrics, or mosaic datasets will not upgrade to the new version along with the rest of the geodatabase. Upgrading these is optional, but functionality will be limited to what was available at the SDE version it remains at. Upgrades are done via the Upgrade Dataset tool.
9. If needing a service for client access, install the new version of the ArcSDE application server. Then re-create the service using the sdeservice and sdemon command-line tools (included with .
10. If needing command-line tools for administration automation, install the new version.

### Considerations


#### Downtime

Geodatabase upgrades need to happen in a period of time where no users will need access for a decently-sized amount of time. For that reason, it would be best to perform the upgrade over a weekend. The only required access provided at that time is for the LCOG ETL processing. The ETLs can easily be paused, then activated after the upgrade & initial testing is complete. For that purpose, I recommend that the primary ETL geodatabases get priority (LCOGGeo, RLIDGeo, Staging).

##### Action Now
Pick a weekend for the upgrade. I will be around, willing, and family-less on April 18-20 and 25-27. We could upgrade the primary ETL geodatabases the first weekend, then proceed with the others as time permits the same weekend or the next.

Also, notify staff & partners of downtimes, with assurances of the plan.


#### Backup Plan

We currently back up a selection of geodatabases on gisrv106 (EugeneGeo, LCOGGeo, Regional) every night after 7pm. Our process currently uses the sdeexport command-line tool to make .sdx files for each geodatabase object. Since the command-line tools will be deprecated after ArcSDE 10.2, we should be looking for another backup method. It is also implied that the .sdx backup format will deprecate along with sdeexport & sdeimport. Suggested backup methods include copying, XML workspace exports, and SQL Server backup methods.

##### Action Now
For the purposes of an upgrade, I recommend we compress and back up each geodatabase immediately preceding its upgrade. Backups should be done not only via our standard backup method, but also via duplicating the database on the server. This will allow for faster regression, as the duplicate will essentially be the geodatabase at the older version with a slightly different name.

##### Action Later
We will need to determine a new best-fit backup plan for our needs before moving beyond SDE 10.2.


#### Regression Plan

There is no formal mechanism to downgrade a geodatabase to a older version. If we need to move back to an older version of the geodatabase, we'll need to restore from backup. The simplest method of doing this would be to take the duplicate copy of the geodatabase made before the upgrade, and rename it to the standard name (see Backup Plan).


#### Application Server - Services/Native Client

Since we are running ArcSDE 10.0, we are therefore also running the application server, which provides services access to geodatabases rather than direct connections. From 10.1 on, the application server is a separately-installed application from ArcSDE; in fact ArcSDE is wholly contained in the geodatabase (and client, to be complete) sans application. LCOG has been insisting on direct connections to our geodatabases for a while now, which removes the need for running services and installing the application server.

##### Action Now
I recommend we turn off services and do not reinstall the application server after the upgrade. If any users have yet to switch to direct connections, this will finally push them to take what is a simple step.


#### Command-Line Tools

The ArcSDE command-line tools will be deprecated after 10.2. Functionality that uses them will need to find other methods to serve needs. We currently have three primary uses for the command-line tools:
(1) Managing users/locks. We may have some shell scripts that use these, but for the most part we have moved to using ArcCatalog, ArcToolbox, and SQL Server to take care of our needs.
(2) Exporting backup files (see Backup Plan).
(3) Write-offs of shapefiles and coverages in various ETLs using sde2cov & sde2shp. These are mostly only in place for write-offs to Eugene's app staging data.

##### Action Now
Install the ArcSDE command-line tools.

##### Action Later
Follow recommendations under Backup Plan. Refactor the ETLs that use sde2cov/sde2shp to use other methods.


#### Versioned Views

Versioned Views (VVs)are views of datasets that allow not only reading the default version across delta tables, but also allow editing versioned datasets outside of an ArcGIS editing session. At ArcSDE 10.1, the former nomenclature Multiversioned View (MVV) was replaced with Versioned Views. Previously existing MVVs will continue to work after an upgrade, but it's recommended to replace them eventually. VVs are automatically created when a dataset is registered as versioned. If data was already registered before upgrading beyond 10.1, VVs will only be created for each table/feature class/feature dataset via their right-click > Manage > Enable SQL Access. MVVs and VVs are not visible in ArcCatalog, but are present & valid for viewing & editing.

"Multiversioned views are implemented at the ArcSDE level. This means multiversioned views do not work with functionality implemented at the geodatabase level. For this reason, they should not be used to edit data that participates in geodatabase behavior." (5)

##### Action Now
We currently have only a few multiversioned views, relating to land use and site address updates pushed from RLID to maintenance datasets. I would recommend either:
1. Enable VVs on our versioned datasets, but leave the MVVs we have in place. Eventually we'll replace the MVVs with the VVs as we refactor the ETLs they participate in.
2. Or we do away with pushing data from publication to maintenance in this way. There are two types of data that need to be in the maintenance dataset: directly-edited data and reference data. These are likely reference data for editors being pushed, and this need could be met with relationship classes or joins in the editing MXD to a cross-reference table that wouldn't require version-editing by the ETL.

That said, I recommend that we activate the versioned views for all versioned datasets after we upgrade the geodatabase.


#### Geometry

Since we began our migration to enterprise geodatabases in the ArcSDE 9.x era, nearly all of our geometry is stored in the SDEBinary type. As of ArcSDE 10.1, geometry storage defaults to the native SQL Servef geometry type. However, if the geodatabase configuration was hard-coded to SDEBinary, new feature classes will stick with that. Also, already-existing feature classes remain with the geometry storage they already had.

There are a number of advantages to using the native SQL Server type.
1. Feature classes become single-table, rather than referencing the geometry in another location in the database.
2. Access to the geometry can be done outside ArcGIS clients.
3. Backup plans can be done through SQL Server, without direct knowledge of SDE access (since table = feature class).
4. SQL Server implements a subset of the OGC spatial methods plus custom extensiosn, allowing geoprocessing (e.g. clip, near, etc.) in SQL statements.
5. Post-10.1, views are automatically spatial if they detect a native geometry type. Also, you cannot create a spatial view with SDEBinary types except via the sdetable command-line tool.

In fact, the only disadvantage the native type seems to have is that Shape.area & Shape .length have been replaced by Shape.STArea() and Shape.STLength(), which ArcGIS doesn't quite clean up in exports/copies. Also, older SQL Server versions have trouble with the native geometry. The only SQL Server setup we have that is old enough is gisrv112, which Bob & I have developed a workaround (views excluding the geometry, plus knowing gisrv112 is going away soon).

There is a Migrate Storage tool that converts geometry or raster storage on a chosen dataset.

##### Action Now
I recommend that right away we migrate the geometry types at the very least for RLIDGeo. All new datasets in the Staging geodatabase are already using native geometry, and work just fine. If RLIDGeo were migrated, we could perform the Monday toggle in SQL itself, making the toggle blazing fast.

##### Action Later
I also recommend migrating datasets in other geodatabases to native geometry, though this can be done gradually. Having all datasets on native geometry will simplify making views and ETLs in the future. At the least, I would like to convert internal LCOG datasets, while recommending partners allow us to do the same.


## Sources
(1) Upgrading a geodatabase in SQL Server.
http://resources.arcgis.com/en/help/main/10.2/index.html#/A_quick_tour_of_enterprise_geodatabase_upgrades/002q0000005p000000/

(2) Geodatabase Administration toolset.
http://resources.arcgis.com/en/help/main/10.2/index.html#/An_overview_of_the_Geodatabase_Administration_toolset/001700000009000000/

(3) ArcGIS 10.2.x for Server system requirements.
http://resources.arcgis.com/en/help/main/10.2/index.html#/Preparing_to_upgrade_a_geodatabase_in_SQL_Server/002q000000m6000000/

(4) What are versioned views?
http://resources.arcgis.com/en/help/main/10.1/index.html#//006z0000000q000000

(5) What are multiversioned views? (ArcGIS Desktop 10 Help).
http://help.arcgis.com/en/arcgisdesktop/10.0/help/index.html#//006z0000000q000000

(6) Maintaining an enterprise geodatabase in SQL Server.
http://resources.arcgis.com/en/help/main/10.2/index.html#/What_type_of_maintenance_is_needed_for_a_geodatabase/002q00000051000000/

(7) Do This, Not That! – Alternatives to using SDE command line tools http://blogs.esri.com/esri/supportcenter/2013/10/04/do-this-not-that-alternatives-to-using-sde-command-line-tools/
