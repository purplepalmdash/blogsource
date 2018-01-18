+++
title = "WorkingtipsOnhelmpackageing"
date = "2018-01-18T18:01:00+08:00"
description = "WorkingtipsOnhelmpackageing"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Change the tgz files
Via following scripts:    

```
for i in `ls *.tgz`
do
	echo $i
	tar xzvf $i
	mv $i $i.bak
	cd wordpress
	sed -i 's|serviceType: LoadBalancer|serviceType: ClusterIP|g' values.yaml
	cd ..
	helm package wordpress
	rm -rf wordpress
done
```
Then copy all of the helmed tgz file to a new place.    

### Get sha256sum value
Get the sha256sum,   

```
sha256sum * | tac | awk {'print $1'}>sha256sum.txt
```
manually adjust its sequencees.  Then form a file like following:    

```
    digest: 6dbf2c9394e216e691116ea5135cd5c83b55ce1bd199f23ab40ec6a8c426a7ea
    digest: a017ad98dc9f727ea6c422be74f077557b8af35470033b61a7c7c11d6b2a1194
    digest: fad50a4f5dcae6ce8af05634cb14fadb19b2018fe30521669d0e19a68192d2ec
    digest: ae45690c85c733bd66db97cd859c40174c88932ff8e944852afe762dc62c47d0
    digest: 5528dfda0b68cf7f00eecab4b56933e6935a01d680989f6e690a281492efce64
    digest: 071f4af604b580c13ec1b53bf5ebf898d3d198bbecd616d309d71d94d7635b25
    digest: 12de12908f3c8b61a1f68ea529a06ad402903bd4dea458cf335bf8e9dda3f290

.......
```

### Get the index.yaml digest
Via following command:    

```
# cat index.yaml | grep digest>kkkk.txt
# cat kkkk.txt| more
    digest: b33a1646e90963a387406f4857f312476b4615222a36e354f7b9e62a2971ce9e
    digest: d54b28981ce97485a59d78acd43182e6d275efd451245c3420129d3cf2b1bfb5
    digest: 88337f823951e048118e44dc7b7e1e6508dcac138612edff13bc63a3ec90a997
    digest: 07f2323bc6559c973bd4f8deb3b5c83acb99c5aac93d2135b4f1690bf9a3be85
    digest: 26d71a30ee0f845ff45854d96a69f18ded56038c473f13dcc43a572a7e9cd92d
```

### Replace all of the digest
Python scripts for doing this thing:    

```
findlines = open('originsha256sum.txt').read().split('\n')
replacelines = open('modifiedsha256sum.txt').read().split('\n')
find_replace = dict(zip(findlines, replacelines))

with open('index.yaml') as data:
    with open('newindex.yaml', 'w') as new_data:
        for line in data:
            for key in find_replace:
                if key in line:
                    line = line.replace(key, find_replace[key])
            new_data.write(line)

new_data.close()
data.close()

```
Using `python2 replace.py` you could get a new `newindex.yaml`, use this one
for replacing your original `index.yaml` in your web-server, and also replace
all of the charts file with your generated tgz files.    

Now deploy the apps, you will get an ClusterIP equited rather than load
balance equipped apps.    
