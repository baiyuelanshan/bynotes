This script can only be used when the offset stored in zookeeper

reset_offset.sh

```shell
#!/bin/bash
zookeeper=$1
group=$2
topic=$3
partitions=$4

while [[ ${partitions} -ge 1 ]]
do
    let partitions-=1
    echo ${partitions}
    zookeeper-client -server ${zookeeper} set /consumers/${group}/offsets/${topic}/${partitions} 0
done

```
