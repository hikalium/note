# Optane DC Persistent memory

# ipmctl

```
$ sudo ipmctl show -dimm
 DimmID | Capacity    | LockState | HealthState | FWVersion    
===============================================================
 0x0020 | 126.422 GiB | Disabled  | Healthy     | 01.02.00.5355
 0x0120 | 126.422 GiB | Disabled  | Healthy     | 01.02.00.5355
 0x1020 | 126.422 GiB | Disabled  | Healthy     | 01.02.00.5355
 0x1120 | 126.422 GiB | Disabled  | Healthy     | 01.02.00.5355
```

```
$ sudo ipmctl show -region
 SocketID | ISetID             | PersistentMemoryType | Capacity    | FreeCapacity | HealthState 
=================================================================================================
 0x0000   | 0xd170eeb8be5b2444 | AppDirect            | 252.000 GiB | 252.000 GiB  | Healthy
 0x0001   | 0xd70ceeb813572444 | AppDirect            | 252.000 GiB | 252.000 GiB  | Healthy
```

# ndctl

```
$ sudo ndctl list --regions
[
  {
    "dev":"region1",
    "size":270582939648,
    "available_size":270582939648,
    "max_available_extent":270582939648,
    "type":"pmem",
    "iset_id":-2950721181468646332,
    "persistence_domain":"memory_controller"
  },
  {
    "dev":"region0",
    "size":270582939648,
    "available_size":270582939648,
    "max_available_extent":270582939648,
    "type":"pmem",
    "iset_id":-3354919245155982268,
    "persistence_domain":"memory_controller"
  }
]
```

https://docs.pmem.io/ndctl-user-guide/managing-namespaces

```
$ sudo ndctl create-namespace -m fsdax
{
  "dev":"namespace1.0",
  "mode":"fsdax",
  "map":"dev",
  "size":"248.06 GiB (266.35 GB)",
  "uuid":"0cf929b9-b27f-4c0c-afdc-4ed5f9705b9c",
  "sector_size":512,
  "align":2097152,
  "blockdev":"pmem1"
}
```

# mount

```
cd /mnt
sudo mount /dev/pmem1 /mnt/pmem1
```



