# ili-kataloge

## Fragen
- Umgang mit Baskets, Datasets? Ist das notwendig? Erstellen wir einmalig zu Beginn den Katalog und dann ist gut? Der Anwender kann in QGIS ja immer weitere Werte erfassen (auch wenn das nicht gewollt ist). Bei einem Modellupdate könnten dann die zusätzlichen Werte im Katalog schnell vergessen gehen.
- Was muss kommentiert werden? Nur das Attribut in der Klassen? (Nicht die Katalog-Konstrukte?)

## Test

### PostgreSQL

```
docker run --rm --name editdb -p 54321:5432 -e POSTGRES_PASSWORD=mysecretpassword -e POSTGRES_DB=edit -e PG_READ_PWD=dmluser -e PG_WRITE_PWD=gretl sogis/oereb-db:latest
```

```
java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --dbschema awjf_biotopbaeume --schemaimport
```

... Katalog erfassen und exportieren ...

```
java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --dbschema awjf_biotopbaeume --export ch.so.awjf.biotopbaeume.katalog.xtf
```

... Schema anlegen und Katalog importieren ...

```
java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --dbschema awjf_biotopbaeume --schemaimport

java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --dbschema awjf_biotopbaeume --import ch.so.awjf.biotopbaeume.katalog.xtf
```

... Daten erfassen und exportieren ...

```
java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --dbschema awjf_biotopbaeume --export ch.so.awjf.biotopbaeume.xtf
```

... Publikationsschema anlegen ...

```
java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_Publikation_20991231 --dbschema awjf_biotopbaeume_pub --schemaimport
```

... Datenumbau ...

```
INSERT INTO 
    awjf_biotopbaeume_pub.biotopbaeume_biotopbaum 
    (
        geometrie,
        baum_id,
        baumart,
        merkmal_code,
        merkmal_name 
    )
SELECT 
    baum.geometrie, 
    baum.baum_id, 
    baum.baumart, 
    katalog.acode, 
    katalog.aname 
FROM 
    awjf_biotopbaeume.biotopbaeume_biotopbaum AS baum
    LEFT JOIN awjf_biotopbaeume.kataloge_kat_hauptmerkmal AS katalog 
    ON baum.merkmal = katalog.t_id 
;
```

... Pub-Daten exportieren ...

```
java -jar /Users/stefan/apps/ili2pg-4.6.0/ili2pg-4.6.0.jar --dbhost localhost --dbport 54321 --dbdatabase edit --dbusr ddluser --dbpwd ddluser --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_Publikation_20991231 --dbschema awjf_biotopbaeume_pub --export ch.so.awjf.biotopbaeume_pub.xtf
```

### GPKG (edit only)

... Schema anlegen und Daten mit Katalog importieren ...

```
java -jar /Users/stefan/apps/ili2gpkg-4.6.0/ili2gpkg-4.6.0.jar --dbfile fubar.gpkg --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --schemaimport

java -jar /Users/stefan/apps/ili2gpkg-4.6.0/ili2gpkg-4.6.0.jar --dbfile fubar.gpkg --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --import ch.so.awjf.biotopbaeume.xtf
```

... Daten inkl. Katalog exportieren ...

```
java -jar /Users/stefan/apps/ili2gpkg-4.6.0/ili2gpkg-4.6.0.jar --dbfile fubar.gpkg --nameByTopic --strokeArcs --defaultSrsCode 2056 --createFk --createFkIdx --createGeomIdx --modeldir ".;https://models.geo.admin.ch" --models SO_AWJF_Biotopbaeume_20991231 --export fubar.xtf
```
