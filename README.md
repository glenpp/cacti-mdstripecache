# Linux md (RAID5/6) Stripe Cache monitoring on Cacti vi SNMP

Most of the time Linux RAID gets setup and left alone, but when performance is critical you might like to consider some extra tuning. As is often the case, tuning is specific to the workload and in most real-world systems that varies with a lot of factors.

This grew off my Home Lab project where I have 2 main arrays: a RAID6 of 7x large SATA and RAID5 of 3x small SSDs for LVM2 Cache. Depending on what PoC / experiments are running, the system gets wildly varying workloads, and very often the arrays get maxed for periods. When this happens it's useful to know what is going on to apply feedback into tuning the arrays.

A specific area where I've already looked at was Stripe Cache Tuning which improved rebuild performance massively. This quick template provides continuous monitoring / graphing of this to see Stripe Cache usage for real workloads. Essentially, you want to adjust the size of the Stripe Cache to ensure that the array is not starved of cache and forced to re-read stripes. Remember, RAID5 & 6 need to read from all the stripes in order to calculate parity when just one has been written to - that's a big hit on performance, so correct cache tuning is critical.

## Data Collection

For this I've chosen to collect percentile data for the time preceding a sample. This means that you need to schedule collection to run for the sample period before (normally 5 minutes) so the data is ready when Cacti polls. For this I use the CRON setup described in SNMP Basics with data file in /var/local/snmp/ or wherever is appropriate for you.

Put the CRON script **mdstripecache\_cron.py** somewhere appropriate, I use **/etc/snmp/** and run this from cron every 5 minutes (or to match your Cacti polling):

```sh
# collect md stripe cache data
/etc/snmp/mdstripecache_cron.py /var/local/snmp/mdstripecache.csv 295 2 &
```

This stores the data in **/var/local/snmp/mdstripecache.csv**, samples for 295 seconds (enough to ensure the data is in place comfortably before the 5 minute / 300 second Cacti sampling), and during that period samples the Stripe Cache usage every 2 seconds. You can vary these as appropriate for your setup.

## snmpd Extension

Next we need to get the data out via snmpd. Add extension script **mdstripecache\_stats.py** to /etc/snmp/ and lines from snmpd.conf-mdstripecache to /etc/snmp/snmpd.conf

Then restart snmpd to start using the config.

At this point you should be able to retrieve this data with snmpwalk.

## Cacti setup

This will need the SNMP Query .xml file **mdstripecache.xml** adding - in my case into **/usr/local/share/cacti/resource/snmp_queries/** with the other local queries. Then load the Cacti Template **cacti_data_query_md_stripe_cache.xml** after which you should be able to add the data query to hosts and add graphs accordingly.
