# Short notes on howto compile a kernel for Ubuntu with Secure Boot enabled

## Acquire kernel
### Download the kernel from: https://www.kernel.org/
tar -xvf linux-X.Y.Z.tar.gz
cd linux-X.Y.Z
   
## Set configuration & compile
cp /boot/config-$(uname -r) .config
make olddefconfig # Make new config based on the old one, set new config values to default.
make -j$(nproc) bindeb-pkg # This will take a while

## Install newly compiled kernel & copy .config
export KERNEL_RELEASE=$(make kernelrelease)
cd ..
sudo dpkg -i linux-headers-$KERNEL_RELEASE.deb linux-image-$KERNEL_RELEASE.deb linux-libc-$KERNEL_RELEASE.deb
cd linux-$KERNEL_RELEASE
sudo cp .config /usr/src/linux-headers-$KERNEL_RELEASE/.config

## Sign the kernel 
sudo sbsign --key /var/lib/shim-signed/mok/MOK-Kernel.priv --cert /var/lib/shim-signed/mok/MOK-Kernel.pem --output /boot/vmlinuz-$KERNEL_RELEASE /boot/vmlinuz-$KERNEL_RELEASE

## Sign the kernel modules
find /lib/modules/$(make kernelrelease) -type f -name '*.ko' | while read -r mod; do
    echo Signing $mod
    sudo /usr/src/linux-headers-$(make kernelrelease)/scripts/sign-file sha256 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der "$mod"
done
