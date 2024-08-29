# Implement link latency to a specific destination

## Centos/Rocky - install netem

```
yum install kernel-modules-extra
modprobe sch_netem
```

## Setup env vars

```
IFACE=ens5
DELAY=100ms
DSTIP="172.17.0.5" # optional, all if empty
DSTPORT="3000"     # optional, all if empty
SRCPORT="3000"     # optional, all if empty

[ "${DSTIP}" != "" ] && DST="match ip dst ${DSTIP}" || DST=""
[ "${SRCPORT}" != "" ] && SOURCEPORT="match ip sport ${SRCPORT} 0xffff" || SOURCEPORT=""
[ "${DSTPORT}" != "" ] && DESTPORT="match ip dport ${DSTPORT} 0xffff" || DESTPORT=""
```

## Add link latency

```
tc qdisc del dev ${IFACE} root # Ensure you start from a clean slate
tc qdisc add dev ${IFACE} root handle 1: prio
tc qdisc add dev ${IFACE} parent 1:3 handle 30: netem delay ${DELAY}
tc filter add dev ${IFACE} protocol ip parent 1:0 prio 3 u32 ${DST} ${SOURCEPORT} ${DESTPORT} flowid 1:3
```

## Clear link latency

```
tc qdisc del dev ${IFACE} root
```
