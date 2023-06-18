The flows used for training originated from multiple sources:
1. Data collected during the simulation on Thursday 15, June 2023, from 10:00 am to 6:00 pm. 
2. Data collected during the last hour or so of the 2022 finals, provided by @oruamdalo
3. Data collected during the simulation on Friday 23, June 2023, from 10:00 am to 6:00 pm.
The provided files are the CSVs with the computed flows and a zip containing the single final pcap file from which it was generated. Due to GitHub file size lilmits, the zip files will be available through the release page.
All the traffic belongs to a single machine, which we had access to. Nevertheless, it classifies as a NIDS and not and HIDS since we aren't using any logs other than the network traffic data.
The data collection was adapted from and existing setup involving tcpdump and rsync. Due to the constraint of the VM hosting the services, we couldn't hold all the packets there. This meant cycling the files, overwriting them periodically. To avoid overwriting on my machine, every time I ran rsync it copied everything in a new numbered folder. I ended up with 570 directories, each with up to 500 pcap files of 1MB each. 
With the script merge_pcaps.ipynb, I merged all those into a number of larger pcap files, amounting to 131.8 GB of data. 
Then I removed duplicated packets in each file, cutting the size to 22.2 GB.
After merging and removing duplicates again, I had 9.9. GB remaining.
These were fed into CICFlowMeter (the modified version from Liu et al.), leaving me with 52.2 MiB of flow data. For comparison, CIC-IDS-2017 has 5 files, for a total of 1.1 GiB of data. CSE-CIC-IDS-2018 is at an even larger scale, with 33.6 GiB of data. 
Even then, we also should take into account that our dataset has plenty of flows on ports 22 and 3333 which belong to ssh and caronte. Those were not services, haven't been under attack, and should be cut removed from the dataset.
Beyond that, we should also consider that:
1. There  were both successful and unsuccesful attacks, since during the challenge we responded by patching the vulnerabilities we could.
2. Our dataset is probably less imbalanced than CIC-IDS-2017 and CSE-CIC-IDS-2018 (or maybe more imbalanced in the opposite direction), since most of the traffic on the machine was by other players trying to attack us and only some of the traffic was coming from simulated legitimate accesses by the gameserver. That also means it is less realistic since a real network configuration wouldn't have such an high ratio of attackers to legitimate users.
3. All the traffic was masked by the gameserver, so it is impossible to distinguish hosts by ip or port. This isn't a concern however, since that information will be discarded anyway in our data processing pipeline.
4. Our dataset is unlabeled. Without prior full knowledge of the exact attacks happening and of the IPs that were interacting with our machine, we can't rely of automatic labeling like for the CIC datasets. We also can't manually label the whole dataset (even if it's small compared to the other two, it's still 79820 flows only for the first file). This means we can't train with Random Forest or XGBoost, and we'll have to try Unsupervised Learning methods.
