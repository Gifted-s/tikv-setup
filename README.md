# tikv-setup local


1. Use TiUP Playground (Easiest)

This is the fastest way to get a full TiKV + PD + TiDB cluster running on your laptop.

# Install TiUP (if not already)
curl --proto '=https' --tlsv1.2 -sSf https://tiup.io/install.sh | sh

# Start a TiDB playground with TiKV
tiup playground --db 0 --pd 1 --kv 1


--db 0 → no TiDB SQL layer (so you interact directly with TiKV).

--pd 1 → 1 Placement Driver instance.

--kv 1 → 1 TiKV node.

This will run everything locally with default configs. You can then modify TiKV configs with --kv.config <file>.

2. Run Only TiKV (Standalone Mode)

If you just want TiKV + PD, without TiDB SQL:

Clone TiKV repo:
```
git clone https://github.com/tikv/tikv.git
cd tikv
make release
```

Start a PD server (required):
```
./bin/pd-server --name=pd --data-dir=pd --client-urls="http://127.0.0.1:2379" --peer-urls="http://127.0.0.1:2380"
```

Start a TiKV node:
```
./bin/tikv-server --pd="127.0.0.1:2379" --addr="127.0.0.1:20160" --data-dir=tikv
```

This will use RocksDB under the hood.

3. Modify RocksDB Config for Research

TiKV exposes RocksDB tuning knobs in its config file. Example (tikv.toml):
```
[rocksdb]
max-background-jobs = 8
max-sub-compactions = 4
write-buffer-size = "128MB"

[rocksdb.defaultcf]
block-cache-size = "512MB"
```

You can then start TiKV with:
```
./bin/tikv-server --pd="127.0.0.1:2379" --config=./tikv.toml
```

4. Testing / Benchmarking

You can benchmark TiKV using go-ycsb
 or sysbench
Example (YCSB load):

```
go-ycsb load tikv -P workloads/workloada -p tikv.pd="127.0.0.1:2379"
```

👉 Question for you: do you want to

(A) Run just TiKV locally with minimal setup for RocksDB profiling,
or

(B) Run the whole TiDB cluster but focus only on TiKV tuning?

That will change whether I show you how to strip things down to TiKV + PD only or keep TiDB SQL layer around.

TiKV always requires at least one PD (Placement Driver) instance running, even in standalone mode. Here’s how you can run it locally:

1. Build (or download) PD

If you already cloned TiKV, you also need PD. You can either:

Option A – Download prebuilt binary with TiUP (easier):

tiup install pd


Option B – Build from source:
```
git clone https://github.com/tikv/pd.git
cd pd
make
```

This will give you a bin/pd-server executable.

2. Start PD locally

Run it on 127.0.0.1:2379 (client port) and 127.0.0.1:2380 (peer port):
```
./bin/pd-server \
  --name=pd \
  --data-dir=pd-data \
  --client-urls="http://127.0.0.1:2379" \
  --peer-urls="http://127.0.0.1:2380" \
  --initial-cluster="pd=http://127.0.0.1:2380"
```

--name=pd → identifier of the PD node.

--data-dir=pd-data → local storage for PD metadata.

--client-urls → where clients (like TiKV) connect.

--peer-urls → communication between PD nodes (for a cluster, but still needed in standalone).

--initial-cluster → bootstrap info (must point to itself in standalone).

3. Verify PD is running

Once it’s up, you can check:

curl http://127.0.0.1:2379/pd/api/v1/members


You should see JSON describing the PD cluster with one member.

4. Start TiKV and connect to PD

Now launch TiKV and point it at PD:
```
./bin/tikv-server \
  --addr="127.0.0.1:20160" \
  --data-dir=tikv-data \
  --pd="127.0.0.1:2379"
```


⚡ From here you can start modifying tikv.toml configs for RocksDB tuning and restart tikv-server to apply changes.


https://chatgpt.com/share/68bb86f6-3ab8-8005-b602-8d2ee0bc83b7
