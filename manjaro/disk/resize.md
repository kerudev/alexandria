# manjaro/disk/resize

> [!NOTE] TLDR
> Use gparted, click on Resize/Move the selected partition. Boot with
> proprietary drivers if gparted's GUI freezes. Don't be impatient and wait for
> gparted to finish to avoid losing data.

When I installed Manjaro (over a year and a half ago), I reserved almost half
of my disk for it (170GB).

My Windows partition (or the "forbidden partition", as I like to call it) used
to have a whopping 290GB, but one day I felt... inspired and got the guts to
free 130GB of space (disabled pagination for hibernation or whatever dumb 
Winbloat feature), but didn't assign that to Manjaro.

It's finally the time to resize the Manjaro partition, so I figured I'd make
myself a guide to check out when I finally get rid of the "forbidden partition"
(the day will come eventually).

## 2026-06-20

The last part was written before the resize, which was quite the experience.

I booted Manjaro from a USB and opened `gparted` (manages disk partitions), but
accidentally assigned the free space to the "forbidden partition".

I was forced to boot Windows... but I figured it'd be a good time to do some
cleaning (I freed 50GB more!). When I opened `diskmgmt.msc`, drive C didn't
seem to have the extra 130GB...

So again, I booted the USB, opened `gparted` and unassigned space from Windows.
I left it running for about 2 hours, but the progress bar didn't move (it got
stuck on "calibrating"). I clicked cancel, but the operation got applied
correctly... weird, right? (more on this later).

The 200GB of unassigned space was between `dev/sdb4` (Windows) and `dev/sdb6`
(Manjaro). I clicked on `Resize/Move the selected partition`. 

After 20 minutes or so, I clicked on `Details` as there was no progress. It
seemed like the GUI was frozen, as I had to press Enter or Space to open a
modal to save the details to a file.

When I opened the file, I read that the operation was tried and cancelled...

## 2026-06-20

I tried something different on the USB boot menu. Instead of booting with open
source drivers, I booted with proprietary drivers. This marked the difference
for `gparted`, as now `Resize/Move the selected partition` did its job and even
the GUI was not frozen anymore.

The operation took around 2h. I read on some forums that `gparted` took more
than 50h to finish, but that wasn't my case at all.

## References

- https://medium.com/@itsnibhatt/easy-guide-to-resizing-linux-partitions-using-gparted-3567d60bf660
