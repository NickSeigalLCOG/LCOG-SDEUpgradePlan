# ArcSDE Upgrade Plan


### What an Upgrade Means

The current definition of "upgrade" with regards to ArcSDE is to "upgrade the geodatabase system tables and install updated stored procedures, types, and functions". There is no requirement to install new or newer-versioned applications in order to upgrade a geodatabase: as of version 10.1 all ArcSDE properties are contained within the database itself or the user application that interfaces with it (e.g. ArcGIS Desktop, Server). So in effect, our "upgrade" is to move each geodatabase to the current version of ArcSDE internally, and to consider alterations of our geodatabase standards and practices.

### Geodatbase-only Upgrade

As of ArcSDE 10.0, upgrades must be done via the "Upgrade Geodatabase" tool in the Geodatabase Administration Toolset. Using the sdesetup command-line tool will no longer work.

#### Steps for Upgrading a geodatabase.

These steps are based on Esri's recommended procedure for upgrading a geodatabase, amended to our system and preferences.

##### Before the Upgrade Window

1. Ensure that the most current version of an ArcGIS client application (e.g. Desktop) is installed on the machine you will be administrating the upgrade *from*. Post 10.1, you can only install an ArcSDE version that is concurrent or older than the application running the Upgrade Geodatabase tool.
2. Check to see that your database server meets the minimum system requirements (see link 3 below).
3. Duplicate the geodatabase, and perform a test-upgrade with the duplicate. Test functionality as exists in the geodatabase (registering, versioning, versioned views, edit tracking, topology, etc.).
4. Choose a date for upgrading, and provide notification to all likely users.

##### During the Upgrade Window

For each database being upgraded:

1. Create backups of the geodatabase (see 'Backup Plan' under 'Considerations' below).
2. Remove any custom functionality you may have added to the geodatabase system tables outside ArcGIS such as: triggers, participation in SQL Server replication, or additional indexes. The upgrade procedure cannot take into account customizations you make to the system tables. If such customizations prevent the alteration of a system table's schema, the upgrade will fail.
  - Note: LCOG doesn't have any of these customizations.
3. Disconnect remaining users from the geodatabase, and block any new connections (right-click > Properties > Connections).
4. If you are using an ArcSDE service:
  i.   Stop (do not pause) the service, via Windows Services.
  ii.  Uninstall the application server (pre-10.1: uninstall ArcSDE).
  iii. Delete the service when prompted by uininstaller.
5. If you are using the ArcSDE command-line tools, uninstall them.
6. Run the Upgrade Geodatabase tool.
  - Be sure to leave "Perform-Pre-Requisite Check" enabled. This double-checks that there are no other active connections, you have sufficient privileges, the database can support XML columns, and all datasets can be opened. If any prerequisites are not met, the tool will terminate before upgrading.
7. Any network datasets, parcel fabrics, or mosaic datasets will not upgrade to the new version along with the rest of the geodatabase. Upgrading these is optional, but functionality will be limited to what was available at the SDE version it remains at. Upgrades are done via the Upgrade Dataset tool.
  - Note: I do not believe LCOG or our partners have implemented any of these at the enterprise-level.
9. If needing a service for client access, install the new version of the ArcSDE application server. Then re-create the service using the sdeservice and sdemon command-line tools (included in the application server install).
10. If needing command-line tools for administration automation, install the new version.


### Considerations


#### Downtime

Geodatabase upgrades need to happen during a period of time when no users will need access for a decently-sized period of time. For that reason, it would be best to perform the upgrade over a weekend. The only required access needed on weekends is for the LCOG ETL processing. The ETLs can easily be paused, then activated after the upgrade and initial testing is complete. For that purpose, I recommend that the ETL geodatabases get priority (LCOGGeo, RLIDGeo, Staging).

##### Recommended Action

1. Notify staff and partners of downtimes, with assurances of plan and regression precautions. Notify by April 11.
2. Upgrade weekend April 26-27.


#### Backup Plan

We currently back up a selection of the geodatabases on gisrv106 (EugeneGeo, LCOGGeo, Regional) every night after 7 pm. Our process currently uses the sdeexport command-line tool to make .sdx files for each geodatabase object. Since the command-line tools will be deprecated after ArcSDE 10.2.x, we should be looking for another backup method. It is also implied that the .sdx backup format will deprecate along with sdeexport and sdeimport. Suggested backup methods include copying datasets to other workspaces, XML workspace exports, and whatever SQL Server backup methods are at our disposal. There are three backup concepts to consider: dataset backups (currently done via sdeexport), whole database backups (I see 3 days of recent backups for most GDBs on gisrv106 occurring around 11pm), and server snapshots (not done for gisrv106 unless we have a plan I don't know about).


##### Recommended Action

1. For the purposes of an upgrade, I recommend we compress and back up each geodatabase immediately preceding its upgrade.
2. Backups should be done not only via our standard backup method, but also via duplicating the database on the server. This will allow for faster regression, as the duplicate will essentially be the geodatabase still at the older version but with a slightly different name (e.g. RLIDGeo_Backup100).
3. Later, we need to determine a new best-fit backup plan for our needs before moving beyond SDE 10.2.


#### Regression Plan

There is no formal mechanism to downgrade a geodatabase to an older version. If we need to move back to an older version of the geodatabase, we'll need to restore from backup. The simplest method of doing this would be to take the duplicate copy of the geodatabase made before the upgrade, and rename it to the standard name (see 'Backup Plan').


#### Application Server - Services/Native Client

Since we are running ArcSDE 10.0, we are therefore also running the application server, which provides services access to geodatabases rather than direct connections. From 10.1 onward, the application server is a separately-installed application from ArcSDE. In fact, ArcSDE is wholly contained in the geodatabase (and client, to be complete) sans application. LCOG has been insisting on direct connections to our geodatabases for a while now, which removes the need for running services and installing the application server.

##### Recommended Action

1. I recommend we turn off services and not reinstall the application server after the upgrade. If any users have yet to switch to direct connections, this will finally push them to take what is a fairly simple step (creating direct connections and updating map documents and layer files).
2. Offer technical support in the form of arcpy scripting to users needing to update mxd or lyr files to use direct connections.


#### Command-Line Tools

The ArcSDE command-line tools will be deprecated after 10.2.x. Functionality that uses them will need to find other methods to serve those needs. We currently have three primary uses for the command-line tools:
1. Managing users/locks. We may have some shell scripts that still use these, but for the most part we have moved to using ArcCatalog, ArcToolbox, and SQL Server to take care of our needs.
2. Exporting backup files (see 'Backup Plan').
3. Write-offs of shapefiles and coverages in various ETLs using sde2cov and sde2shp. These are mostly only in place for write-offs to Eugene's app staging data.

##### Recommended Action

1. Install the ArcSDE command-line tools at upgrade.
2. Later, follow recommendations under 'Backup Plan'. Refactor the ETLs that use sde2cov and sde2shp to use other methods.


#### Versioned Views

Versioned Views (VVs) are views of versioned datasets that allow one to not only read a version across SDE delta tables by a non-ArcGIS client, but also allow editing versioned datasets in this manner. At ArcSDE 10.1, the former nomenclature Multiversioned View (MVV) was replaced with Versioned Views. I'm sure there's a technical difference as well. Previously existing MVVs will continue to work after an upgrade, but it's recommended to replace them eventually. VVs are automatically created when a dataset is registered as versioned (SDE names them <datasetname>_vw). If data was already registered before upgrading beyond 10.1, VVs must be created for each table/feature class/feature dataset manually via a right-click > Manage > Enable SQL Access toggle. MVVs and VVs are not visible in ArcCatalog, but are present & valid for viewing & editing.

LCOG has very few multiversioned views, relating to land use and site address updates pushed from RLID to maintenance datasets.

##### Recommended Action

1. Enable VVs on our versioned datasets, but leave the MVVs we have in place. 
2. Eventually, replace the MVVs with the VVs as we refactor the ETLs they participate in.
3. Explore doing away with pushing data from publication to maintenance in this way. The only data that actually needs to be on a maintenance dataset is directly-maintained data. These data pushes are likely for maintainers' reference, and could be related or joined to the maintained data in lookup tables that wouldn't require version editing to update.


#### Geometry

Since we began our migration to enterprise geodatabases in the ArcSDE 9.x era, nearly all of our geometry is stored in the SDEBinary type. As of ArcSDE 10.1, geometry storage defaults to the native SQL Server geometry type. However, if the geodatabase configuration was hard-coded to SDEBinary, new feature classes will stick with that. Also, already-existing feature classes remain with the geometry storage they had when created.

There are a number of advantages to using the native SQL Server type.
1. Feature classes become single-table, rather than referencing the geometry at another location in the database.
2. Access to the geometry can be done outside ArcGIS clients (e.g. SQL Server Management Studio).
3. Dataset backup plans can be done through SQL Server, without direct knowledge of SDE access (since table = feature class).
4. SQL Server implements a subset of the OGC spatial methods (plus some custom extensions), allowing geoprocessing (e.g. buffer, clip, merge, near, etc.) in SQL statements.
5. Post-10.1, views are automatically spatial if they detect a native geometry type. Also, you can no longer create a spatial view with SDEBinary types except via the sdetable command-line tool (which will soon deprecate).

In fact, the only disadvantage the native type seems to have is that Shape.area and Shape.length have been replaced by Shape.STArea() and Shape.STLength() (the equivalent OGC methods), which ArcGIS doesn't omit correctly or cleanly rename in exports/copies. Also, older SQL Server versions (2005 and earlier) can't handle native geometry. The only SQL Server setup we have that is that old is gisrv112, which Bob and I have developed a workaround for (views excluding the geometry, plus knowing gisrv112 is going away soon).

There is a Migrate Storage tool that converts geometry or raster storage on a chosen dataset.

Migration is already occuring ad-hoc on many of our GDB feature classes: the geometry storage configuration is set to "default", and some of our recent feature classes have been created with ArcGIS 10.1 or 10.2 (so far, these ad-hoc feature classes are mostly in the Staging database).

##### Recommended Action

1. Ensure that the geometry storage configuration is set to "default" on all GDBs. This will ensure that future feature classes use the preferred storage type.
2. For the most part, leave geometry storage on existing feature classes be. I do recommend that we migrate Staging and RLIDGeo's at the least. All new datasets in the Staging geodatabase are already using native geometry, and work just fine. If RLIDGeo were migrated, we could perform the Monday toggle in SQL itself, making the toggle blazingly fast.


#### SDE User

Another artifact of having long-lived SDE geodatabases is that most of our geodatabases have the SDE schema tables owned by a user called 'sde'. This used to be the default when SDE was more centered on the application server. Currently however, the default setup is to have "dbo" own the schema tables (except for multi-database SDE, which we do not implement). Most but not all the geodatabases on gisrv106 have the 'sde' user owning schema tables. The exceptions are: Reporting (a new GDB I created), RLIM (an old DB I added SDE to recently), and EugeneGeo.

Changing the schema table owner requires creating an entirely new geodatabase, then porting all the properties (datasets, views, stored procedures, users, roles) from the old to the new. There may be some applications that may streamline this, but it would take lots of time.

##### Recommended Action

1. Upgrade without getting involved in this for now. Everything will continue to work just fine this way.
2. Fix on internal/developer geodatabases (e.g. Staging), as these have few users/roles and can be constructed over time without de-syncing datasets.


#### Partner Notification & Testing

Upgrading a geodatabase should be completely invisible to the end-users. The only outward-facing issue is that we will need to have downtime while the upgrade is occurring. If anyone is querying via the Shape.area/Shape.length methods, there may be some issues with the geometry storage transition (though I believe Arc will internally respect Shape.area/Shape.length as a part of the geometry object created within the application).

A more careful consideration needs to be undertaken for LCOG- or partner-loading of data to other repositories. I know of two cases:
1. RLID loads data from our geoprocessing ETLs. There is a known issue with gisrv112 with geometry storage, but Bob and I have created a workaround for the limited life of that server. Also since both Bob and I can adjust the loading script, we can easily correct any of the problems before or as they occur.
2. EWEB loads RLIDGeo datasets weekly to their own SDE database. I've already been in contact with Jeff Schenck and Mike Pulley, and I'm satisfied that their system & method of loading will continue to work after the upgrades. Still, I've promised Jeff and Mike that they can make a loading run at a test version of RLIDGeo 10.2.x prior to the true upgrade.

##### Recommended Action

1. Send notifications out to either all geodatabase users or their area managers as soon as we choose an upgrade window.
2. Get test duplicates of each geodatabase upgraded, and offer primary stakeholders in each geodatabase testing access.


#### Geodatabases

* Maintenance
  - EugeneGeo: Upgrade GDB & datasets, enable versioned views, check geometry config.
  - LCOGAppliedGeo: Upgrade GDB & datasets, enable versioned views, check geometry config.
    - Question: Is this being used much? If not, could retire or replace with wholly new version migrated dataset geometry & DBO schema owner.
  - LCOGGeo: Upgrade GDB & datasets, enable versioned views, check geometry config.
  - Regional: Upgrade GDB & datasets, enable versioned views, check geometry config.
* Publication:
  - Reporting: Already upgraded (new GDB).
  - RLIDGeo: Upgrade GDB & datasets, enable versioned views, check geometry config, migrate dataset geometry.
* LCOG Dev
  - Development: Upgrade GDB & datasets, enable versioned views, check geometry config, migrate dataset geometry.
    - Option: Could be retired & recreated when needed.
  - RLIM: To be retired ASAP.
  - Staging: Upgrade GDB & datasets, enable versioned views, migrate dataset geometry, change schema owner.
  - Test: Upgrade GDB & datasets, enable versioned views, check geometry config, migrate dataset geometry.
    - Option: Could be retired & recreated when needed, or at the least merged into Development.

### Sources
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
