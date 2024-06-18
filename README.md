

cp mdata-fetch-NO-TMP /opt/local/lib/svc/method/mdata-fetch-NO-TMP
cp mdata-NO-TMP.xml /opt/local/lib/svc/manifest/mdata-NO-TMP.xml

svcadm stop svc:/smartdc/mdata
svcadm disable svc:/smartdc/mdata
svccfg delete svc:/smartdc/mdata

svccfg import /opt/local/lib/svc/manifest/mdata-NO-TMP.xml

umount /tmp
umount -f /tmp

vim /etc/vfstab # remove line for /tmp or comment it out

rmdir /tmp
ln -s /var/tmp /tmp

svcadm enable svc:/smartdc/mdata

