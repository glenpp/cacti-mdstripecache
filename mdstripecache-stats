#!/usr/bin/env python3
"""
stats reader for md Stripe Cache Monitoring

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
import csv


def main(argv):
    """Grab the stat
    """
    # check args
    if len(argv) != 3:
        print("Usage: {} <statsfile path> <Device|Description|95th|99th|max>\n".format(sys.argv[0]), file=sys.stderr)
        sys.exit(1)
    statsfile = argv[1]
    metric = argv[2]

    # sanity check file
    timelimit = time.time() - 600
    if not os.path.isfile(statsfile) or os.stat(statsfile).st_mtime < timelimit:
        print("ERROR: stats file \"{}\" too old or doesn't exist\n".format(statsfile), file=sys.stderr)
        sys.exit(2)

    # read in data
    with open(statsfile, 'rt') as f_stats:
        for value in csv.DictReader(f_stats):
            print(value[metric])


if __name__ == '__main__':
    main(sys.argv)

