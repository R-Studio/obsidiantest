# Linux
**VIM**
Copy line: `yy + p`
Copy 3 lines: `3 + yy + p`
Undo: `u`
Remove line: `dd`
Remove character: `x`
New line a& insert mode: `o`

## Storage
List block devices: `lsblk`

### LVM
Physical Volume = **PV**
Volume Group = **VG**
Logical Volume = **LV**

List volume group: `vgs`
List physical volumes: `pvs`
List logical volumes: `lvs`

**Create Volume**
```bash
pvcreate /dev/<DEVICE_NAME>
vgcreate <VG_Name> /dev/<DEVICE_NAME>
lvcreate -n <LV_NAME> -L<SIZE_IN_GB>g <VG_NAME>
mkfs.ext4 /dev/<VG_NAME>/<LV_NAME>
mkdir /mnt/<LV_NAME>
mount /dev/<VG_NAME>/<LV_NAME> /mnt/<LV_NAME>/

#fstab:
sudo vim /etc/fstab
/dev/<VG_NAME>/<LV_NAME> /mnt/<LV_NAME> ext4 defaults 0 0
mount -a
```


**Extend Volume (incl. Filesystem)**
```bash
lvresize -r -L+1g /dev/robin/robin-1
lvresize -r -l+100%FREE /dev/robin/robin-1
```


**New Volume in VG with all free space**
```bash
lvcreate -n robin_2 -l100%FREE robin
mkfs.xfs /dev/robin/robin_2
```



## SED
#### Regex using only the captured group: 
```bash
echo KUBECONFIG=Your_Content-That Your want! | sed -n "s/^KUBECONFIG.*=\(\S*\)/\1/p"

# Output:
Your_Content-That Your want!
```

##### Explanation
```
-n          suppress printing
s           substitute
^           starts with
KUBECONFIG  initial search match
.*          any character (zero, one or multiple times)
=           character "="
\(          start capture group
\S*         capture any non-white space character (word)
\)          end capture group
/\1         substitute everything with the 1st capture group
p           print it
```

**-> [SED Cheatsheet](https://quickref.me/sed)**