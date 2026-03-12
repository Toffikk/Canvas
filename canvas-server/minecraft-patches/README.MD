# Patch remapping

This branch is showcasing the remapping process. This is not for general use. It
requires special steps and special set up. Follow the instructions in this readme, don't ask for help if you haven't
read this entire readme and followed all of these steps.

Remapping patches requires knowledge of how to fix issues when patches don't or can't apply correctly. Patches not
applying correctly is not an issue with this repo or Weaver. You must fix them and continue the process manually
yourself. This is identical to any other upstream update.

### Setup
-----

This repo contains a mappings file (mojang+parchment -> mojang unobf) that will be used during the patch remapping.

> [!WARNING]
> Weaver only supports `Tiny` v1/v2 mappings for remapping.
>
> Additionally, by default the simple remapping type in Mercury is used by Weaver.
> If your mappings file contain changes to class names/packages, append the `--no-simple-remap` flag to the remap task.


Should you wish to remap to/from different mappings, place the mappings file, that should be used while remapping under `build-data` under the name of `remap-mappings.tiny`

> [!INFO]
> The directories below, such as 0001,0002 etc, refer to subdirectories under the `unmapped` directory.


If your repo contains any patches that shouldn't be remapped but are important for apply (such as generated additional upstreams) it is important to place them under `0001`.
Place the patches you want to remap in `0002`. Weaver will only remap patches placed in the last directory - for
example if there are directories `0001`, `0002`, and `0003`, only the patches in `0003` will be remapped.


> [!CAUTION]
> All of the directories should contain unmapped patches, as they will get applied on top of the unmapped source.


For example:

* If you are remapping `base` patches, place them in `0002`.
* If you are remapping `source` patches, place `base` patches in `0002`, and the `source` patches in `0003`.
* If you are remapping `feature` patches, place `base` patches in `0002`, `source` patches in `0003` and the `feature` patches in `0004`.

### Pre-remapping
-------------

The patch remap task has does not support remapping non-git and file patches.
Before you start trying to remap source patches, make sure to export them as a single feature patch.
This can be done by modifying the task's `baseRef` to start at `basepatches`. Example can be found in the root `build.gradle.kts` of this repo.

After that is done, rebuild it as a feature patch by running:

```shell
./gradlew rebuildMinecraftFeaturePatches
```

You can later apply the patch as a `feature` and squash it into the `file` tag to revert it back to the file patches format.

### Remapping
---------

To start remapping patches, run the `remapPatches` task:

```shell
./gradlew remapMinecraftPatches
```

This can take a very long time, hours even, as patch remapping is a very complex and CPU \& disk intensive task. How long
it takes depends entirely on how many patches there are to be remapped, and how powerful your computer is.


That said, while you may need to walk away during the process, the remapping process does require some amount of
hands-on work to get it to complete. If you leave for a while and come back to a patch failure, you'll need to fix the
issue and restart paperweight to pick up from where it left off, explained below.

### Fixing patch application issues

It is likely that patch remapping will succeed first try. 
Should the patch application fail, it is caused by your patches failing one way or another due
to minor differences in tooling, setup, etc. Again, it is your responsibility to handle patch application issues as they
come up just like any other upstream update.


While the process of fixing issues with patch application is the same, how to resume and complete the process is
different, as it hopefully goes without saying that patch remapping is a more complex process which requires extra work
vs just a simple patch update.

### Patch remap working directory setup

There are a variety of files and directories that weaver uses during patch remap, but only one directory that you
need to worry about. This is the directory where patches are applied, and code is remapped.

```
.gradle/caches/paperweight/patch-remap-input
```

That directory contains what would be the `src/minecraft/java` directory during the patch remapping process.

### Apply patch fixes

If a patch fails to apply, go to the `patch-remap-input` directory to fix the issue, like you normally would.
Once you have fixed the issues in patch application, and are ready to continue, run `am --continue` in the
`patch-remap-input` directory:

```shell
git am --continue
```

Each patch is applied in its own `am` session, so this won't cause any of the other patches to incidentally apply. After
the `am` command in the above `patch-remap-input` directory is completed, continue the remapping process with:

```shell
./gradlew remapMinecraftPatches --continue-remap
```

Due to the `--continue-remap` flag weaver will pick up where it left off. If the patch that failed is in a
pre-remap upstream patch set, it will simply continue applying the other patches. If the patch is in the remap patch
set, weaver will record the patch, remap it, and re-apply it on the remap branch as it normally would, then
continue applying the rest of the patches from there.

### Completed patch remap

After the process is complete and all patches have been remapped, the new remapped patches (only including the remap
patch set explained above) will be generated into `remapped`.

