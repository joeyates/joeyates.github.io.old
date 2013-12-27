--- 
layout: post
typo_id: 24
title: Setting up PostGIS
---
<h1>
	Install</h1>
<p>
	You can install PostGIS as a package or from sources. Sometimes the version available from Linux distributions is a bit out of step with the PostgreSQL version that gets installed (i.e. Ubuntu currently installs PostgreSQL 8.4, but only has PostGIS 8.3)</p>

<!--more-->

<h2>From Sources</h2>

<p>
	<a href="http://postgis.refractions.net/download/">Choose which version to download</a></p>

<pre>$ wget http://postgis.refractions.net/download/postgis-1.4.1.tar.gz
$ tar -xkzf postgis-1.4.1.tar.gz
$ cd postgis-1.4.1
$ ./configure
$ make
$ sudo make install
</pre>
<p>
	On my system (Ubuntu), the net result is that you get various files copied into /usr/share/postgresql/8.4/contrib:</p>
<pre>$ ls /usr/share/postgresql/8.4/contrib
postgis.sql                   postgis_upgrade_13_to_14.sql  postgis_upgrade.sql  uninstall_postgis.sql
postgis_upgrade_12_to_14.sql  postgis_upgrade_14_minor.sql  spatial_ref_sys.sql</pre>
<h1>
	Create a PostGIS Template Database</h1>
<p>
	If you create a specific template with PostGIS, and use it instead of template1 (the default) template, you will be able to create PostGIS-enabled databases more quickly.</p>
<pre>$ sudo -u postgres createdb template_postgis -E UTF-8
</pre>
<p>
	Set the Template Flag</p>
<pre>UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'template_postgis';</pre>
<p>
	Make sure pl/pgsql is installed:</p>
<pre>$ sudo -u postgres createlang -d template_postgis plpgsql</pre>
<p>
	Insert PostGIS functions and tables:</p>
<pre>$ sudo -u postgres psql template_postgis < /usr/share/postgresql/8.4/contrib/postgis.sql
$ sudo -u postgres psql template_postgis < /usr/share/postgresql/8.4/contrib/spatial_ref_sys.sql
</pre>
<h4>
	Correct Permissions</h4>
<p>
	The PostGIS tables need to be writeable by the owner of the database:</p>
<pre>$ sudo -u postgres psql template_postgis
# GRANT ALL ON geometry_columns TO PUBLIC;
# GRANT ALL ON spatial_ref_sys TO PUBLIC;
</pre>
<h1>
	Create a PostGIS-Enabled Database</h1>
<pre>$ sudo -u postgres createuser [USER NAME]
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) n
Shall the new role be allowed to create more new roles? (y/n) n
$ sudo -u postgres createdb [DATABASE NAME] -O [USER NAME] -T template_postgis</pre>
<h1>
	Check it Works</h1>
<pre>$ psql -U [USER NAME] [DATABASE NAME]
</pre>
<p>
	First let's create a 'locations' table:</p>
<pre># CREATE TABLE locations (id serial, name character varying);
# SELECT AddGeometryColumn('', 'locations', 'geom', 32661, 'POINT', 2);
</pre>
<p>
	Insert a record:</p>
<pre># INSERT INTO "locations" ("name", "geom") VALUES(E'Tokyo International Airport', ST_GeomFromText('POINT(139.783333 35.55)', 32661)) RETURNING "id";
 id
----
  1
(1 row)

INSERT 0 1</pre>
<p>
	Find all results in a given box:</p>
<pre># SELECT * FROM "locations" WHERE ("locations"."geom" && SetSRID(E'BOX3D(130.0 30.0, 140.0 40.0)'::box3d, 32661));
 id |            name             |                        geom
----+-----------------------------+----------------------------------------------------
  1 | Tokyo International Airport | 0101000020957F0000151C5E10117961406666666666C64140
(1 row)</pre>
