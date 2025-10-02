# ignition

1. Write butane config
2. Convert butane to ignition config
3. Host coreos files using HTTP server on DiskStation:
  
```
fedora-coreos-42.20250914.3.0-live-initramfs.x86_64.img
fedora-coreos-42.20250914.3.0-live-kernel.x86_64
fedora-coreos-42.20250914.3.0-live-rootfs.x86_64.img
```

4. Write ipxe embed script pointing ignition file and to files on DiskStation
5. Clone ipxe repo:
   
```
 git clone https://github.com/ipxe/ipxe.git
```
6. Build ipxe with embedded script:
```
cd ipxe/src
make bin-x86_64-efi/ipxe.efi EMBED=myscript.ipxe
```
7. Put ipxe.efi on TFTP server
8. Point to ipxe.efi using DHCP server
9. Start Host and boot from PXE
