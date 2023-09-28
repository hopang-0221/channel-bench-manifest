## Porting Cheshire to channel-bench
Porting `cheshire` target includes modifieying and adding the following files:
```
1. channel-bench/kernel/src/plat/cheshire/*
    - `overlay-cheshire.dts` different from that in seL4 (No `Clint` configuration)
2. channel-bench/kernel/tools/dts/cheshire.dts (directly copied from seL4)
3. channel-bench/kernel/libsel4/sel4_plat_include/cheshire/* (directly copied from seL4)
4. channel-bench/tools/opensbi/platform/fpga/cheshire/* (directly copied from seL4)
5. channel-bench/projects/util_libs/libplatsupport/plat_include/cheshire/* (directly copied from seL4)
6. channel-bench/projects/util_libs/libplatsupport/src/plat/cheshire/* (directly copied from seL4)
7. channel-bench/projects/channel-bench/apps/side-bench/src/mastik_common/low.h
    - Add L3 cache confiuration for `CHESHIRE`
8. projects/channel-bench/include/channel-bench/bench_common.h
    - Add `BENCH_PMU_BITS` and `BENCH_PMU_COUNTERS` configuration for `CHESHIRE`
9. projects/include/arch/riscv/arch/machine.h
    - Add fencet for cheshire (`#if defined(CONFIG_PLAT_ARIANE) || defined(CONFIG_PLAT_CHESHIRE)`).
10. projects/channel-bench/apps/side-bench/src/mastik_riscv/llc_skd_spy.c
    - Change the number of lines in LLC to 256: `void *v = data + s * 64 + i * 64 * 256;`
11. kernel/src/api/syscall.c
    - Change the size of LLC to 128 KB: `char gadget[128*1024];`
    - Change the number of lines in LLC to 256: `void *v = gadget + s * 64 + i * 64 * 256;`
12. projects/channel-bench/apps/manager/src/convert.c
    - Change thread domain: `create_thread(&spy, 0)`;
```


## Run channel-bench
To initialize channel-bench, we should first execute:
```
> repo init -u https://github.com/niwis/channel-bench-manifest.git -m timing.xml
> repo sync
```

For `kernel` directory, add `http://github.com/niwis/seL4` as remote, and checkout to its `gadget` branch.

Switch `Opensbi` version to 0.9 for correct operation:
```
> cd tools/opensbi
> git checkout 234ed8e
```

To build image for `cheshire` platform, we need to execute the following commands:
```
> mkdir build-cheshire
> cd build-cheshire
> ../init-build.sh -DPLATFORM=cheshire
```

Edit `CMakeCache.txt`:
```
BenchUntypeSizeBits = 13
LibSel4UtilsCSpaceSizeBits = 8
ManagerCovertBench = ON
BenchCovert = ON
KernelSharedGadget = ON
KernelTimeSlice = 1
KernelTimerTickMS = 10
KernelNumDomains = 1
LibSel4CacheColourNumCacheColours = 16
NUM_DOMAINS = 1
LibSel4CacheColouring = OFF
KernelDomainMicroarchFlush = ON
SIMULATION = TRUE
```

After that, we need to configure the changes we make using `ccmake`:
```
> ccmake . 
```

Finally, to generate the image file, we run `ninja`:
```
> ninja
```

To get .dump file from image file, we can type this command, this is used when we need to debug:
```
> riscv64-unknown-elf-objdump -d ./images/manager-image-riscv-cheshire > images/manager-image-riscv-cheshire.dump
```