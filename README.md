# Minimalis Linux AArch64
<h3>Cara bikinnya</h3>
<ol>
<li>Download kernel linux versi 6.6.6 (sesuai selera) <a href="https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.6.6.tar.xz">di sini</a></li>
<li>Ekstrak pake perintah <code>tar -xvf linux-6.6.6.tar.xz</code></li>
<li>Masuk ke direktori linux-6.6.6</li>
<li>Terus ketik <pre>make ARCH=arm64 defconfig
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image</pre>tunggu ampe selesai proses buildnya</li>
<li>Kalo udah selesai, bikin <code>rootfs</code>, build <code>BusyBox</code><pre>cd .. 
mkdir rootfs
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xjf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig</pre> di tunggu, abis itu <code>make menuconfig</code> di dalem <code>menuconfig</code> navigasi ke <code>Networking Utilities ---></code> cari <code>tc</code> dan <b>disable</b>, aktifin <i>Build static binary</i>, pilih <code>Busybox Settings -> Build static binary (no shared libs) → [X]</code>, terus simpen dan keluar terus build
<pre>make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
make CONFIG_PREFIX=../rootfs install</pre> Terus bikin <code>/init</code> shellscript
<pre>cd ../rootfs
cat << 'EOF' > init
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
exec /bin/sh
EOF
chmod +x init</pre>
Tambahin folder yang di perluin
<pre>mkdir -p proc sys dev
cd ..
</pre></li>
<li>Copy <code>rootfs</code> ke direktori <code>linux-6.6.6</code>
<pre>sudo cp -R rootfs ./linux-6.6.6</pre> 
Abis itu ke direktori <code>rootfs</code>
<pre>cd rootfs</pre> Bikin <code>rootfs.cpio.gz</code>
<pre>find . | cpio -H newc -o | gzip > ../rootfs.cpio.gz
</pre></li>
<li>Coba booting, caranya <code>cd ..</code>
<pre>
qemu-system-aarch64 \
  -M virt -cpu cortex-a57 -m 512M \
  -kernel arch/arm64/boot/Image \
  -initrd rootfs.cpio.gz \
  -nographic \
  -append "console=ttyAMA0 init=/init"
</pre> Hasilnya kurleb kek gini kalo ga kernel panic
<pre>
[    1.316436] rtc-pl031 9010000.pl031: registered as rtc0
[    1.318902] rtc-pl031 9010000.pl031: setting system clock to 2025-05-13T03:59:28 UTC (1747108768)
[    1.324229] i2c_dev: i2c /dev entries driver
[    1.353384] sdhci: Secure Digital Host Controller Interface driver
[    1.353870] sdhci: Copyright(c) Pierre Ossman
[    1.356904] Synopsys Designware Multimedia Card Interface Driver
[    1.360385] sdhci-pltfm: SDHCI platform and OF driver helper
[    1.367445] ledtrig-cpu: registered to indicate activity on CPUs
[    1.380479] usbcore: registered new interface driver usbhid
[    1.381122] usbhid: USB HID core driver
[    1.398981] hw perfevents: enabled with armv8_pmuv3 PMU driver, 7 counters available
[    1.421074] NET: Registered PF_PACKET protocol family
[    1.423823] 9pnet: Installing 9P2000 support
[    1.424911] Key type dns_resolver registered
[    1.469410] registered taskstats version 1
[    1.471967] Loading compiled-in X.509 certificates
[    1.517256] input: gpio-keys as /devices/platform/gpio-keys/input/input0
[    1.536443] clk: Disabling unused clocks
[    1.537716] ALSA device list:
[    1.538128]   No soundcards found.
[    1.543504] uart-pl011 9000000.pl011: no DMA platform data
[    1.615939] Freeing unused kernel memory: 9152K
[    1.618614] Run /init as init process
/bin/sh: can't access tty; job control turned off
~ # uname -a
Linux (none) 6.6.6 #1 SMP PREEMPT Tue May 13 09:24:39 WIB 2025 aarch64 GNU/Linux
~ #
</pre></li>
<li>Semoga sukses dan ga ada yang miss tutorialnya</li>
