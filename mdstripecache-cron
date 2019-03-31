#!/usr/bin/env python3
"""CRON job for md Stripe Cache Monitoring

See: https://www.pitt-pladdy.com/blog/_20160309-081028_0000_Linux_md_RAID5_6_Stripe_Cache_monitoring_on_Cacti_vi_SNMP/


Copyright (C) 2016  Glen Pitt-Pladdy

   This program is free software: you can redistribute it and/or modify
   it under the terms of the GNU General Public License as published by
   the Free Software Foundation, either version 3 of the License, or
   (at your option) any later version.

   This program is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
   GNU General Public License for more details.

   You should have received a copy of the GNU General Public License
   along with this program.  If not, see <http://www.gnu.org/licenses/>.
"""


import sys
import os
import time
import glob
import re
import csv
import math


def main(argv):
    """Grab the stats to cache file
    """
    # check args
    if len(argv) != 4:
        sys.exit("Usage: {} <statsfile path> <run for seconds> <sample interval seconds>\n".format(argv[0]))
    statsfile = argv[1]
    tempfile = '{}.TMP-{:d}'.format(statsfile, os.getpid())
    runfor = float(argv[2])
    interval = float(argv[3])

    # work out timing
    starttime = time.time()
    endtime = starttime + runfor

    # discover storage to watch and stripe_cache_size (constant)
    mddevices = [os.path.dirname(f).split('/')[3] for f in glob.glob('/sys/block/md*/md/stripe_cache_size')]
    cachesizes = {}
    cacheactives = {}
    for dev in mddevices:
        with open(os.path.join('/sys/block', dev, 'md/stripe_cache_size')) as f_size:
            cachesizes[dev] = int(f_size.read())
        cacheactives[dev] = []

    # read /proc/mdstat for descriptions
    description = {}
    with open('/proc/mdstat', 'rt') as f_mdstat:
        dev = None
        for line in f_mdstat:
            # see if we get a device line
            devmatch = re.match(r'^(md\d+)\s*:\s*(\w+\s+raid\S+)\s', line)
            if devmatch:
                dev = devmatch.group(1)
                description[dev] = devmatch.group(2) + ' '
                continue
            # we use the second line in description
            if dev != None:
                description[dev] += line.strip()
                dev = None    # reset
                continue

    # sample loop
    nexttime = starttime
    while nexttime < endtime:
        for dev in mddevices:
            with open(os.path.join('/sys/block', dev, 'md/stripe_cache_active')) as f_active:
                cacheactives[dev].append(int(f_active.read()))
        nexttime = nexttime + interval
        time.sleep(nexttime - time.time())

    # output percentiles
    with open(tempfile, 'wt') as f_cache:
        csvw = csv.writer(f_cache)
        csvw.writerow(['Device', 'Description', '95th', '99th', 'max'])
        for dev in mddevices:
            data = sorted(cacheactives[dev])
            i95 = int(math.ceil(float(len(data) - 1) * 0.95))
            i99 = int(math.ceil(float(len(data) - 1) * 0.99))
            csvw.writerow(
                [
                    dev,
                    description[dev],
                    float(data[i95]) / cachesizes[dev] * 100.,
                    float(data[i99]) / cachesizes[dev] * 100.,
                    float(data[-1]) / cachesizes[dev] * 100.,
                ]
            )
    os.rename(tempfile, statsfile)

if __name__ == '__main__':
    main(sys.argv)

