# Short notes on howto compile a kernel for Ubuntu with Secure Boot enabled

## Acquire kernel
### Download the kernel from: https://www.kernel.org/
`tar -xvf linux-X.Y.Z.tar.gz`<br>
`cd linux-X.Y.Z`<br>
   
## Set configuration & compile
`cp /boot/config-$(uname -r) .config` <br>
`make olddefconfig` # Make new config based on the old one, set new config values to default. <br>
`make -j$(nproc) KCFLAGS='-march=native -O3 -pipe' bindeb-pkg` # This will take a while <br>

## Install newly compiled kernel & copy .config
`export KERNEL_RELEASE=$(make kernelrelease)` <br>
`cd ..` <br>
`sudo dpkg -i linux-headers-$KERNEL_RELEASE.deb linux-image-$KERNEL_RELEASE.deb linux-libc-$KERNEL_RELEASE.deb` <br>
`cd linux-$KERNEL_RELEASE` <br>
`sudo cp .config /usr/src/linux-headers-$KERNEL_RELEASE/.config` <br>

## Sign the kernel 
```bash
sudo sbsign --key /var/lib/shim-signed/mok/MOK-Kernel.priv --cert /var/lib/shim-signed/mok/MOK-Kernel.pem --output /boot/vmlinuz-$KERNEL_RELEASE /boot/vmlinuz-$KERNEL_RELEASE
```

## Sign the kernel modules
```bash
find /lib/modules/$KERNEL_RELEASE -type f -name '*.ko' | while read -r mod; do
    echo Signing $mod
    sudo /usr/src/linux-headers-$(make kernelrelease)/scripts/sign-file sha256 /var/lib/shim-signed/mok/MOK-Kernel.priv /var/lib/shim-signed/mok/MOK-Kernel.der "$mod"
done
```
