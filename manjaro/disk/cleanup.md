# manjaro/disk/cleanup

When working with Rust, I face the same issue from time to time: my disk runs
out of storage. I still haven't figured out the exact reason why this happens,
but it always occurs when running tarpaulin and a MongoDB Docker container
(from Docker Desktop).

Maybe this is just a coincidence, but I've had that happen to me when those 3
are involved with each other.

I usually open the "Disks" and "Disk Usage Analytics" programs to know what is
going on, and most of the time it's just a lot of pacman caches.

So, how can we clean the caches without `rm -rf /var/cache/pacman/pkg/*`?

First, let's take a look into out orphan packages, which are those that are no
longer required:

```sh
pacman -Qdtq
```

- `-Q`: query the package database.
- `-d`: lists the packages installed as dependencies.
- `-t`: lists unrequired packages (orphans).
- `-q`: strips the version names out of the output.

Then, uninstall them:

```sh
pacman -Rns $(pacman -Qdtq)
```

- `-R`: remove packages.
- `-n`: nosave. Ignores `pacman`'s file backups.
- `-s`: remove targets recursively.

Shorthand for cleaning cache:

```sh
pacman -Scc
```

- `-S`: synchronizes remote repositories.
- `-cc`: one removes packages that are no longer installed from cache. Two
  removes all files from the cache. Both remove packages from the database
  as well.

## References

Documentation:

- https://pacman.archlinux.page/pacman.8.html
- https://gist.github.com/HFTrader/4fb15d461d86634fd1cba5d251ca7925

Some articles:

- https://www.linuxactionshow.com/pacman-clear-cache/
- https://sathyasays.com/clean-up-pacman-cache-automatically-on-arch-linux/
