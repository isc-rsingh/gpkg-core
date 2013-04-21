###############################
GeoPackage Core Specification
###############################

A self-contained, single-file, cross-platform, serverless, transactional, open source RDBMS container is desired to simplify production, distribution and use of GeoPackages and guarantee the integrity of the data they contain.  Self-contained means that container software requires very minimal support from external libraries or from the operating system. Single-file means that a GeoPackage consists of a single file as viewed in the file systems supported by common operating systems. Cross-platform means that manipulation and use of a GeoPackage on one computer platform does not mess up its use on another, irrespective of operating system, file system, or byte order (endian) conventions.  Serverless means that the RDBMS container is implemented without any intermediary server process, and accessed directly by application software. Transactional means that all changes to data in the container are Atomic, Consistent, Isolated, and Durable (ACID) despite program crashes, operating system crashes, and power failures.

The mandatory core capabilities defined in sub clauses and requirement statements of this clause SHALL be implemented by every GeoPackage.

NOTE: Grouping and dependency relationships of conformance classes for clause 6 and 7 requirements are shown in Figure 3 in Annex A clause A.2.

************************
6 File Format
************************

The GeoPackage file format SHALL be an SQLite [9] database file with the version 3 file format [10][11]. A GeoPackage file shall be named with a `.geopackage` extension to enable operating system level handlers to determine that it is a GeoPackage without opening it. 

NOTE1: SQLite has been used as the base for a number of vector, raster and tile storage specifications, and commercial and open-source implementations. It is deployed and supported by Google on Android [B1] and Apple on IOS [B2] mobile devices.  Testing on a laptop indicates that its performance scales well for databases in excess of 200GB containing vector and raster tables of more than 4 million rows.  

The maximum size of a GeoPackage file is about 140TB. In practice a lower size limit may be imposed by the filesystem to which the file is written. Many mobile devices require external memory cards to be formatted using the FAT32 file system which imposes a maximum size limit of 4GB

============= ===================================================================
Requirement 1 http://www.opengis.net/spec/GPKG/1.0/req/core/file_format
------------- -------------------------------------------------------------------
The GeoPackage file format shall be a SQLite database version 3 format file, the first 16 bytes of which contain “SQLite format 3” in ASCII
=================================================================================

============= ===================================================================
Requirement 2 http://www.opengis.net/spec/GPKG/1.0/req/core/file_format_extension
------------- -------------------------------------------------------------------
The GeoPackage file name SHALL have a `.geopackage` extension
=================================================================================

************************
7 Data Format
************************

This document does not specify a data format. However, for a GeoPackage to be useful it must contain data. Therefore, Every implementation SHALL implement at least one of the GeoPackage data format extensions. 

============= ===================================================================
Requirement 3 http://www.opengis.net/spec/GPKG/1.0/req/core/data_format
------------- -------------------------------------------------------------------
Implementation of this core specification is not sufficient to create a valid GeoPackage
=================================================================================

************************
8 Global Tables
************************

=====================================
8.1 GeoPackage Contents Table
=====================================

The purpose of the GeoPackage `geopackage_contents` table is to provide identifying and descriptive information that an application can display to a user in a menu of geospatial data that is available for access and/or update.

A GeoPackage SHALL contain a `geopackage_contents` table or view as defined in this clause. The geopackage_contents table or view SHALL contain one row record for each tile table, raster table and vector features table [change this to "geospatial data table"?] in the GeoPackage.  The geopackage_contents table or view SHALL NOT contain row records for any other type of table in a GeoPackage (see clause 6.3.4.4 below).

============= ===================================================================
Requirement 4 http://www.opengis.net/spec/GPKG/1.0/req/core/geopackage_contents_table_file_name
============= ===================================================================
The `geopackage_contents` table or view SHALL be named geopackage_contents
=================================================================================

**Table 2** - Geopackage `geopackage_contents` Table or View Definition

.. csv-table:: Table 2 Geopackage `geopackage_contents` Table or View Definition
	:header: "Column Name", "Column Type", "Column Description", "Null", "Default", "PK"
	:widths: 10, 10, 60, 5, 10, 5
	
	table_name,text,"The name of the tiles, raster or feature table",no,,PK
	data_type,text,"Type of data stored in the table. Must be one of features, featuresWithRasters, rasters or tiles",no,, 
	identifier,text,A human-readable identifier (i.e. short name) for the table_name,no,, 
	description,text,A human-readable description for the table_name,no," ", 
	last_change,text,"timestamp value in ISO 8601 format as defined by the strftime function `'%Y-%m-%dT%H:%M:%fZ'` format string applied to the current time",no,"`strftime` `('%Y-%m-%dT%H:%M:%fZ',` ` CURRENT_TIMESTAMP)`", 
	min_x,double,Bounding box for all content in table_name,no,-180.0, 
	min_y,double,Bounding box for all content in table_name,no,-90.0, 
	max_x,double,Bounding box for all content in table_name,no,180.0, 
	max_y,double,Bounding box for all content in table_name,no,90.0, 
	srid,integer,Spatial Reference System ID: spatial_ref_sys.srid,no,0,FK

The `geopackage_contents` table is intended to provide a list of all geospatial data sets in the GeoPackage. The `data_type` specifies the type of content. The bounding box (`min_x`, `min_y`, `max_x`, `max_y`) provides an informative bounding box (not necessarily minimum bounding box) of the data set.  If the `srid` column value references a *geographic* coordinate reference system (CRS), then the min/max x/y values are in decimal degrees; otherwise, the srid references a *projected* CRS and the min/max x/y values are in the units specified by that CRS.

NOTE1: the "0" default value for the srid column is for an undefined geographic CRS. See clause 6.3.2.2 below.

Values of the `geopackage_contents` table or view `last_change` column SHALL be in ISO 8601 format containing a complete date plus UTC hours, minutes, seconds and a decimal fraction of a second, with a ‘Z’ (‘zulu’) suffix indicating UTC.

NOTE2: The following statement selects such a timestamp value:

.. code:: sql
	SELECT (strftime('%Y-%m-%dT%H:%M:%fZ','now')).

See Annex B: Table Definition SQL clause B.1 geopackage_contents.
