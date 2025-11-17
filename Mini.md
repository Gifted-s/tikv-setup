### 1. TiKV Setup (Local)

If you just want **TiKV + PD**, without TiDB SQL:

### Clone TiKV repo

```bash
git clone https://github.com/tikv/tikv.git
cd tikv
export CMAKE_ARGS="-DCMAKE_POLICY_VERSION_MINIMUM=3.5" && export CMAKE_POLICY_VERSION_MINIMUM=3.5 && make release
```

### Start a TiKV node
```bash
./bin/tikv-server --pd="127.0.0.1:2379" --addr="127.0.0.1:20160" --data-dir=tikv
```


## TiKV always requires at least one PD (Placement Driver)

Build from source
```bash
git clone https://github.com/tikv/pd.git
cd pd
make
```

This will give you a `bin/pd-server` executable.

---

### 2. Start PD locally
Run it on `127.0.0.1:2379` (client port) and `127.0.0.1:2380` (peer port):
```bash
./bin/pd-server   --name=pd   --data-dir=pd-data   --client-urls="http://127.0.0.1:2379"   --peer-urls="http://127.0.0.1:2380"   --initial-cluster="pd=http://127.0.0.1:2380"
```

- `--name=pd` → identifier of the PD node.  
- `--data-dir=pd-data` → local storage for PD metadata.  
- `--client-urls` → where clients (like TiKV) connect.  
- `--peer-urls` → communication between PD nodes (for a cluster, but still needed in standalone).  
- `--initial-cluster` → bootstrap info (must point to itself in standalone).

---

### 3. Verify PD is running
Once it’s up, check:
```bash
curl http://127.0.0.1:2379/pd/api/v1/members
```

You should see JSON describing the PD cluster with one member.

---

### 4. Start TiKV and connect to PD
```bash
./bin/tikv-server   --addr="127.0.0.1:20160"   --data-dir=tikv-data   --pd="127.0.0.1:2379"
```

To restart, you need to run the following in the TiKV directory; otherwise it won't detect the new cluster ID created by the Placement Driver if it restarts:
```bash
rm -r ./target/release/tikv-data
```
