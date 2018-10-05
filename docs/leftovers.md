### Exploring Catalog Tables and User Tables

In Firefox, go to the HBase Master’s web interface. Open the Firefox browser and go to port 60010 on your instance's site.

Under the Tables section, click on the tab for ‘Catalog Tables’, then click on the entry for `hbase:meta`.

>Note: the information that is shown for `hbase:meta`. It shows which RegionServer is serving the `hbase:meta` The table is not split into more regions because the `hbase:meta` table is never split.

Go back to the previous page and click on the ‘User Tables’ tab.

Recreate the tablesplit table in the HBase shell as shown earlier in the exercise.

```console
	hbase> create 'tablesplit', 'cf1', 'cf2', {SPLITS => ['A', 'M', 'Z']}
```

In the web interface click on the entry for tablesplit. You might have to refresh the screen in order to see the newly created table listed.

Look at the Table Regions section. Note that since you pre-split the table, the regions for the table are shown here. Looking closer at the regions, notice that each region shows the RegionServer serving that region. Each row also shows the start and stop key for every region.

Finally, the Regions by Region Server section shows which RegionServers are responsible for the various portions of the table’s regions.

Click on the link for the RegionServer to view the RegionServer’s web interface.

The RegionServer’s web interface shows metrics about its current state. It shows a more detailed breakdown of each region’s metrics, and also shows information about the configuration and status of the Block Cache.

Scroll down to the Regions section where you will see the four splits you created for the tablesplit.
