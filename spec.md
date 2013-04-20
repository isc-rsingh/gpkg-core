# GeoPackage Core Specification

A self-contained, single-file, cross-platform, serverless, transactional, open source RDBMS container is desired to simplify production, distribution and use of GeoPackages and guarantee the integrity of the data they contain.  Self-contained means that container software requires very minimal support from external libraries or from the operating system. Single-file means that a GeoPackage consists of a single file as viewed in the file systems supported by common operating systems. Cross-platform means that manipulation and use of a GeoPackage on one computer platform does not mess up its use on another, irrespective of operating system, file system, or byte order (endian) conventions.  Serverless means that the RDBMS container is implemented without any intermediary server process, and accessed directly by application software. Transactional means that all changes to data in the container are Atomic, Consistent, Isolated, and Durable (ACID) despite program crashes, operating system crashes, and power failures.

The mandatory core capabilities defined in sub clauses and requirement statements of this clause SHALL be implemented by every GeoPackage.
NOTE: Grouping and dependency relationships of conformance classes for clause 6 and 7 requirements are shown in Figure 3 in Annex A clause A.2.

## 6 File Format

The GeoPackage file format SHALL be an SQLite [9] database file with the version 3 file format [10][11]. A GeoPackage file shall be named with a `.geopackage` extension to enable operating system level handlers to determine that it is a GeoPackage without opening it. 

NOTE1: SQLite has been used as the base for a number of vector, raster and tile storage specifications, and commercial and open-source implementations. It is deployed and supported by Google on Android [B1] and Apple on IOS [B2] mobile devices.  Testing on a laptop indicates that its performance scales well for databases in excess of 200GB containing vector and raster tables of more than 4 million rows.  

The maximum size of a GeoPackage file is about 140TB. In practice a lower size limit may be imposed by the filesystem to which the file is written. Many mobile devices require external memory cards to be formatted using the FAT32 file system which imposes a maximum size limit of 4GB

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/file_format|
|REQ 1.     |The GeoPackage file format shall be a SQLite database version 3 format file, the first 16 bytes of which contain “SQLite format 3” in ASCII|

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/file_format_extension|
|REQ 2.     |The GeoPackage file name SHALL have a `.geopackage` extension|

## 7 Data Format

This document does not specify a data format. However, for a GeoPackage to be useful it must contain data. Therefore, Every implementation SHALL implement at least one of the GeoPackage data format extensions. 

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/data_format|
|REQ 3.     |Implementation of this core specification is not sufficient to create a valid GeoPackage|

## 8 Global Tables

### 8.1 GeoPackage Contents Table

The purpose of the GeoPackage `geopackage_contents` table is to provide identifying and descriptive information that an application can display to a user in a menu of geospatial data that is available for access and/or update.

A GeoPackage SHALL contain a `geopackage_contents` table or view as defined in this clause. The geopackage_contents table or view SHALL contain one row record for each tile table, raster table and vector features table [change this to "geospatial data table"?] in the GeoPackage.  The geopackage_contents table or view SHALL NOT contain row records for any other type of table in a GeoPackage (see clause 6.3.4.4 below).

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/geopackage_contents_table_file_name|
|REQ 4.     |The `geopackage_contents` table or view SHALL be named geopackage_contents|

**Table 2** - Geopackage `geopackage_contents` Table or View Definition

|Column Name|Column Type|Column Description|Null|Default|Key|
|-----------|-----------|------------------|----|-------|---|
|table_name |text       |The name of the tiles, raster or feature table|no| |PK|
|data_type  |text       |Type of data stored in the table. Must be one of features, featuresWithRasters, rasters or tiles|no| | |
|identifier |text       |A human-readable identifier (i.e. short name) for the table_name|no| | |
|description|text       |A human-readable description for the table_name|no|""| |
|last_change |text       |timestamp value in ISO 8601 format as defined by the strftime function `'%Y-%m-%dT%H:%M:%fZ'` format string applied to the current time|no|`strftime` `('%Y-%m-%dT%H:%M:%fZ',` ` CURRENT_TIMESTAMP)`| |
|min_x       |double     |Bounding box for all content in table_name|no|-180.0| |
|min_y       |double     |Bounding box for all content in table_name|no|-90.0| |
|max_x       |double     |Bounding box for all content in table_name|no|180.0| |
|max_y       |double     |Bounding box for all content in table_name|no|90.0| |
|srid        |integer    |Spatial Reference System ID: spatial_ref_sys.srid|no|0|FK|

The `geopackage_contents` table is intended to provide a list of all geospatial data sets in the GeoPackage. The `data_type` specifies the type of content. The bounding box (`min_x`, `min_y`, `max_x`, `max_y`) provides an informative bounding box (not necessarily minimum bounding box) of the data set.  If the `srid` column value references a *geographic* coordinate reference system (CRS), then the min/max x/y values are in decimal degrees; otherwise, the srid references a *projected* CRS and the min/max x/y values are in the units specified by that CRS.

NOTE1: the "0" default value for the srid column is for an undefined geographic CRS. See clause 6.3.2.2 below.

Values of the `geopackage_contents` table or view `last_change` column SHALL be in ISO 8601 format containing a complete date plus UTC hours, minutes, seconds and a decimal fraction of a second, with a ‘Z’ (‘zulu’) suffix indicating UTC.

NOTE2: The following statement selects such a timestamp value:
```SQL
SELECT (strftime('%Y-%m-%dT%H:%M:%fZ','now')).
```

See Annex B: Table Definition SQL clause B.1 geopackage_contents.

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/geopackage_contents_table_schema|
|REQ 5.     |A GeoPackage SHALL include a `geopackage_contents` table or updateable view with columns and constraints defined in **Table 2**|

### 8.2 Spatial Reference Systems Table

A table or updateable view named `spatial_ref_sys` is the first component of the standard SQL schema for simple features described in clause 8.3.1 below. The coordinate reference system definitions it contains are referenced by the GeoPackage's `geopackage_contents`, `geometry_columns` and `raster_columns` tables to relate the vector, raster, and tile data in user tables to locations on the earth.

The `spatial_ref_sys` table includes at a minimum the columns specified in OGC 06-104r4 [13] section 7.1.2.2 containing data that defines spatial reference systems.  This table or view may include additional columns to meet the requirements of implementation software or other specifications. 
This table SHALL contain:
- a record with an `auth_name` value of "EPSG" or "epsg" and `auth_srid` value of "4326" [17][18] for WGS-84 [19]
- a record with an `auth_name` value of "NONE" and `auth_srid` value of "-1", and srtext “undefined” for undefined Cartesian coordinate reference systems
- and a record with an `auth_name` value of "NONE" and `auth_srid` value of "0", and srtext “undefined” for undefined *Cartesian* coordinate reference systems
- and a record with an SRID of 0, an auth_name of “NONE”, an auth_srid of  0, and srtext “undefined” for undefined *geographic* coordinate reference systems.

It SHALL also contain records to define all other spatial reference systems used by the features, rasters and tiles in the GeoPackage.  It is recommended that it contain records for all coordinate reference systems in the latest EPSG database [B3] when storage space permits.

In OGC 06-104r4 [INSTEAD OF OGC DOC #S HOW ABOUT SHORT NAMES THAT REFERENCE THE STANDARD, E.G. SFSQL] only the primary key column of this table is defined to be NOT NULL.  In a GeoPackage, all columns in this table or updateable view SHALL be defined to be NOT NULL.

NOTE1:  See [B19] chapter 6 for a discussion of NULL column values.

**Table 3** - `spatial_ref_sys` Table or View Definition

|Column Name|Column Type|Column Description|Key|
|-----------|-----------|------------------|---|
|srid       |integer    |Unique identifier for each Spatial Reference System within a database|PK|
|auth_name  |text       |the case-insensitive name of the standard or standards body that is being cited for this reference system, e.g. EPSG or epsg| |
|auth_srid  |integer    |the ID of the Spatial Reference System as defined by the Authority cited in AUTH_NAME| |
|srtext     |text       |Well-known Text Representation of the Spatial Reference System as specified in OGC 06-103r4 section 9| |

See Annex B: Table Definition SQL clause B.2 `spatial_ref_sys`.

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/vector_features/spatial_ref_sys_table|
|REQ 6.     |A GeoPackage SHALL include a spatial_ref_sys table or updateable view with at a minimum the columns defined in **Table 3**|

### 8.3 Manifest Table

#### 8.3.1 Introduction

The GeoPackage manifest serves as an extended table of contents and a data source access metadata store for the GeoPackage data container. It is intended to provide a standard interface between GeoPackage contents and OGC and otherWeb Services that have provided the content in a GeoPackage, or that in the future may use GeoPackages as an input or output format.  The GeoPackage manifest will facilitate two-way geosynchronization of GeoPackage contents with other data stores via web services.  GeoPackage manifest metadata may also be used to provide identifying and coverage area information for a GeoPackage and for the GeoPackages with data for adjacent areas.

#### 8.3.2 Manifest Content Model and Table Definition

A GeoPackage manifest is represented by a single XML document. The canonical GeoPackage manifest content model is an extension of the ATOM [45] encoding of the OGC® OWS Context (OWC) Document Conceptual Model [46].  Other XML document content models may also be used, but only one manifest XML document may be provided per content model. 

A GeoPackage SHALL have a manifest table as defined in table 4.

**Table 4** - `manifest` Table or View Definition

|Column Name|Column Type|Column Description|Null|Default|Key|
|-----------|-----------|------------------|----|-------|---|
|id         |text       |Identifier name or abbreviation of the manifest document content model|no|"OWC"|PK|
|manifest   |text       |XML manifest document fragment|no|" "| |

See Annex B: Table Definition SQL clause B.3 manifest.

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/manifest/manifest_table|
|REQ 7.     |There SHALL be a manifest table in a GeoPackage as defined in **Table 4**|

#### 8.3.3 Manifest XML Schema

The GeoPackage manifest is defined in the `geoPackageContext.xsd` XML schema document shown in table xx below as an extension of the OGC Context document Atom Encoding defined in http://schemas.opengis.net/owc/1.0/OWSContextCore.xsd by 12-084 OWS Context Atom Encoding [47].  The Manifest for a GeoPackage is encoded as an atom:feed element that contains elements describing the GeoPackage itself and GeoPackages for adjacent areas, and atom:entry elements which describe the contents and optionally the source of data in GeoPackage container tables.  The `geoPackageContext.xsd` XML schema defines elements and attributes that are used to extend those of Atom and OWS Context. See the documentation elements in the schema in table xx for descriptions of these elements and attributes. Their use in a manifest document is discussed in the next section.

The http://www.opengis.net/gpkg/1.0 namespace will be proposed to the OGC Naming Authority.

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/manifest/manifest_table/owc_content/xml|
|REQ 8.     |The manifest table in a GeoPackage SHALL contain a record with an “OWC” id value and XML-encoded manifest data that is schema-valid in accordance with the OWC GeoPackage Manifest content model as specified by `atom.xsd`, `OWSContextCore.xsd` and `geoPackageContext.xsd`.|

#### 8.3.4 Manifest Schematron Schema

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/manifest/manifest_table/owc_content/schematron|
|REQ 9.     |The manifest table in a GeoPackage SHALL contain a record with an “OWC” id value and XML-encoded manifest data that is schema-valid in accordance with the `geoPackageContext.sch` Schematron schema.|

#### 8.3.5 Sample Manifest XML Document
A sample manifest XML document and description of its content is provided in Annex H.

### 8.4 Metadata Tables

#### 8.4.1 Introduction

Two tables in a GeoPackage provide a means of storing metadata in the form of XML documents that are defined in accordance with any authoritative metadata specifications, and relating it to the features, rasters, and tiles data in a GeoPackage. These tables are intended to provide the support necessary to implement the hierarchical metadata model defined in ISO 19115 [42], Annex B B.5.25 MD_ScopeCode, Annex G and Annex H, so that as GeoPackage data is captured and updated, the most local and specific detailed metadata changes associated with the new or modified data may be captured separately, and referenced to existing global and general metadata.

The `xml_metadata` table that contains metadata is described in clause 8.4.2, and the `metadata_reference` table that relates metadata to GeoPackage data is described in clause 8.4.3.

These tables SHALL be defined in every GeoPackage. However, they may contain 0 row records. There is no GeoPackage requirement that such metadata be provided, or that defined metadata be structured in a hierarchial fashion with more than one level, only that if it is, these tables SHALL be used.  Such metadata and data that relates it to GeoPackage contents SHALL NOT be stored in other tables as defined in clause 6.3.4.4; see REQ 30 in clause xx.

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/metadata/xml_metadata_table|
|REQ 10.    |A GeoPackage SHALL have an `xml_metadata` table as defined in Table 5 |

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/metadata/xml_metadata_table/undefined|
|REQ 11.    |The `xml_metadata` table in a GeoPackage SHALL contain one row record for the ‘undefined’ scope as defined by the SQL in SQL statement xx.|

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/metadata/metadata_reference_table|
|REQ 12.    |There SHALL be a `metadata_reference` table in a GeoPackage with the columns defined in Table 7.|

|Requirement|Core|
|-----------|----|
|URI        |http://www.opengis.net/spec/GPKG/1.0/req/core/metadata/metadata_reference_table/undefined_parent_|
|REQ 13.    |A `metadata_reference` table in a GeoPackage with any rows SHALL contain at least one row record with an `md_parent_id` value of 0 that references the ‘undefined’ `xml_metadata` row record as defined by the SQL in SQL statement xx (see REQ 11).|

NOTE: Informative examples of hierarchical metadata are provided in Annex J.

#### 8.4.2 XML Metadata Table

[BREAK THIS OUT INTO A METADATA EXTENSION?]

The first component of GeoPackage metadata is the `xml_metadata` table that contains metadata in XML documents or document fragments.  This table may contain metadata structured in accordance with any authoritative metadata specification, such as ISO 19115 [42], ISO 19115-2 [B31], ISO 19139 [B32], Dublin Core [B33], CSDGM [B34], DDMS [B35], NMF/NMIS [B36], etc.  The GeoPackage interpretation of what constitutes “metadata” is a broad one that includes UML models [B37] encoded in XMI [B38], GML Application Schemas [B39], ISO 19110 feature catalogues [B40], OWL [B41] and SKOS [B42]taxonomies, etc.

**Table 5** - `xml_metadata` Table or View Definition

|Column Name|Column Type|Column Description|Null|Default|Key|
|-----------|-----------|------------------|----|-------|---|
|id         |integer    |Metadata primary key|no|       |PK |
|md_scope   |text       |Name of the data scope to which this metadata applies; see table 42 below|no|'undefined'| |
|md_standard_uri|text   |URI reference to the metadata structure definition authority|no|http://schemas.opengis.net/iso/19139| |
|metadata   |text       |XML metadata document or fragment|no|" "| |

The `md_scope` column in the `xml_metadata` table is the name of the applicable scope for the contents of the metadata column for a given row. The list of valid scope names and their definitions is provided in table 6 below. The initial contents of this table were obtained from the ISO 19115 [42], Annex B B.5.25 `MD_ScopeCode` code list, which was extended for use in the GeoPackage specification by addition of entries with “NA” as the scope code column in table 6.

**Table 6** - Metadata Scopes

|Name (md_scope) |Scope Code|Definition |
|undefined       |NA        |Metadata information scope is undefined|
|fieldSession    |012       |Information applies to the field session|
|collectionSession|004      |Information applies to the collection session|
|series          |006       |Information applies to the (dataset) series|
|dataset         |005       |Information applies to the (geographic feature) dataset|
|featureType     |010       |Information applies to a feature type (class)|
|feature         |009       |Information applies to a feature (instance)|
|attributeType   |002       |Information applies to the attribute class|
|attribute       |001       |Information applies to the characteristic of a feature (instance)|
|tile            |016       |Information applies to a tile, a spatial subset of geographic data|
|model           |015       |Information applies to a copy or imitation of an existing or hypothetical object|
|catalog         |NA        |Metadata applies to a feature catalog|
|schema          |NA        |Metadata applies to an application schema|
|taxonomy        |NA        |Metadata applies to a taxonomy or knowledge system|
|software        |013       |Information applies to a computer program or routine|
|service         |014       |Information applies to a capabilitiy which a service provider entity makes available to a service user entity through a set of interfaces that define a behaviour, such as a use case|
|collectionHardware|003     |Information applies to the collection hardware class|
|nonGeographicDataset|007   |Information applies to non-geographic data|
|dimensionGroup  |008       |Information applies to a dimension group|

NOTE1: The scope codes in Table 6 include a very wide set of descriptive information types as “metadata” to describe data.
NOTE2: The “catalog” md_scope may be used for Feature Catalog [B40] information stored as XML metadata that is linked to features stored in a GeoPackage.
NOTE3: The “schema” md_scope may be used for Application Schema [B37][B38][B39][B45] information stored as XML metadata that is linked to features stored in a GeoPackage.
NOTE4: The “taxonomy” md_scope may be used for taxonomy or knowledge system [B41][B42] “linked data” information stored as XML metadata that is linked to features stored in a GeoPackage.
A GeoPackage SHALL have an xml_metadata table.
The xml_metadata table SHALL contain at least the row defined by the SQL insert statement show in Table 49.
See Annex B: Table Definition SQL clause B.4 xml_metadata.

#### 8.4.3 Metadata Reference Table

The second component of GeoPackage metadata is the `metadata_reference` table or view that links metadata in the `xml_metadata` table to data in the feature, raster, and tiles tables.

**Table 7** - `metadata_reference` Table or View Definition

|Column Name|Column Type|Column Description|Null|Default|Key|
|-----------|-----------|------------------|----|-------|---|
|reference_scope|text   |Metadata reference scope; must be one of ‘table’, ‘column’,’row’,’row/col’|no|'table'| |
|table_name |text       |Name of the table to which this metadata reference applies; must be the name of a table in `geometry_columns` or `raster_columns` table|no|‘undefined’| |
|column_name|text       |Name of the column to which this metadata reference applies; ‘undefined’ for reference_scope of ‘table’ or ‘row’, or the name of a column in the `table_name` table for reference_scope of ‘column’ or ‘row/col’|no|‘undefined’| |
|row_id_value|int       |0 for reference_scope of ‘table’ or ‘column’, or the rowed of a row record in the table_name table for reference_scope of ‘row’ or ‘row/col’|no|0| |
|timestamp  |text       |timestamp value in ISO 8601format as defined by the strftime function `'%Y-%m-%dT%H:%M:%fZ'` format string applied to the current time|no|`strftime` `('%Y-%m-%dT%H:%M:%fZ',` ` CURRENT_TIMESTAMP)`| |
|md_file_id |int        |`xml_metadata` table id column value for the metadata to which this metadata_reference applies|no|0|FK|
|md_parent_id|int       |`xml_metadata` table id column value for the hierarchical parent metadata for the metadata to which this `metadata_reference` applies|no|0|FK|

Every GeoPackage SHALL contain a `metadata_reference` table with columns as defined in table 7   Every GeoPackage metadata_reference table that contains any rows SHALL contain at least one row record with an `md_parent_id` value of 0 that references the ‘undefined’ `xml_metadata` row record as defined by the SQL in SQL statement xx. Such record(s) establish the metadata reference to the “root” of a metadata hierarchy.

NOTE1:  Such a metadata hierarchy may have only one level of defined metadata.

NOTE2: A 0 `row_id_value` in the `metadata_reference` tables indicates table or column scope and no corresponding row for the metadata reference. So a data row with a rowed value of 0 cannot be the subject of a metadata reference.  GeoPackage applications should therefore avoid directly setting rowed values in feature, raster, or tile data tables to any values less than 1 to avoid assignment of 0 `rowid` values to data rows that could not be linked to metadata.

NOTE3: In an SQLite implementation, the `rowid` value is always equal to the value of a single-column primary key on an integer column [B30] and is not changed by a database reorganization performed by the VACUUM SQL command.

Values of the `metadata_reference` table timestamp column SHALL be in ISO 8601 format containing a complete date plus UTC hours, minutes, seconds and a decimal fraction of a second, with a ‘Z’ (‘zulu’) suffix indicating UTC.

NOTE4: The following statement selects such a timestamp value:
```SQL
“SELECT (strftime('%Y-%m-%dT%H:%M:%fZ','now'))”.
```

See Annex B: Table Definition SQL clause B.5 metadata_reference.

