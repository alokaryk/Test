If a machine with a separate `/boot` partition gets stuck in `grub rescue>` again in the future, you do not need to restore from a backup. You can fix it manually right from the screen in less than 5 minutes.

Because the `/boot` partition (`/dev/sda1`) is standard and completely healthy, the files are all there—GRUB has just lost its path pointers.

Here is the exact rescue manual to get it booted and fixed:

---

## Step 1: The Manual Boot Sequence (At the Rescue Screen)

Run these commands one by one directly at the `grub rescue>` prompt:

**1. List the available partitions:**

```bash
grub rescue> ls

```

*(You will see options like `(hd0)`, `(hd0,msdos1)`, `(hd0,msdos2)`, `(hd1)`, `(hd1,msdos1)`)*

**2. Find your separate boot partition:**
Check the partitions until you find the one that lists your kernels. Since it is a separate partition, look directly inside the root of the partition (do **not** type `/boot/grub`, just `/grub`):

```bash
grub rescue> ls (hd0,msdos1)/grub

```

*If you see files ending in `.mod` or a file named `grub.cfg`, you have found it! If it says "unknown filesystem", try `(hd0,msdos2)/grub`, etc.*

**3. Manually set the paths to that partition:**
Tell GRUB exactly where to look (assuming it was `hd0,msdos1`):

```bash
grub rescue> set prefix=(hd0,msdos1)/grub
grub rescue> set root=(hd0,msdos1)

```

**4. Load the normal boot menu module:**

```bash
grub rescue> insmod normal

```

**5. Launch the standard boot loader screen:**

```bash
grub rescue> normal

```

The standard blue kernel selection menu will instantly pop up on your VMware console. Press **Enter**, and the OS will boot normally straight to your login screen.

---

## Step 2: Fix It Permanently (Once Logged In)

Once the operating system is booted normally and you are logged in as `root`, you must lock the configuration down immediately so it doesn't happen on the next reboot.

Run the exact three commands we executed today to re-align the boot sector and update the initial RAM disk:

```bash
# 1. Force rewrite the Master Boot Record to the primary disk header
grub2-install --target=i386-pc /dev/sda

# 2. Re-compile the menu entry paths to clear out bad update references
grub2-mkconfig -o /boot/grub2/grub.cfg

# 3. Force rebuild the initramfs ramdisk with the active LVM metadata drivers
dracut --regenerate-all --force

```

### Save This Playbook

Keep these commands handy in your technical documentation. If an automated patch or VMware disk sync causes a path misalignment in the future, this sequence will save you from having to do time-consuming backup restorations.
