# STEPS TO SETUP EXPERIMENT
KVM:
# Install QEMU KVM
sudo apt-get install qemu-kvm

OSv guest:
# Get OSv Source Code
git clone https://github.com/cloudius-systems/osv.git
# Install OSv Build Dependency
cd osv
python scripts/setup.py
git submodule update --init --recursive
# Build OSv Image
make
scripts/build
# Build netperf app
cd apps/netperf
make
# Include netserver in OSv image
cp apps/netperf/netserver.so tools/
# Add `/tools/netserver.so: tools/mkfs/netserver.so` to usr.manifest
scripts/build # Rebuild image
# Start netserver
sudo ./scripts/run.py  -e "/tools/netserver.so -D -4 -f" -c 4 --api

Ubuntu guest:
# Create image
qemu-img create ubuntu.img 10G
# Download Ubuntu net install
http://archive.ubuntu.com/ubuntu/dists/trusty-updates/main/installer-amd64/current/images/netboot/mini.iso
# Start KVM guest with mini.iso as CD-ROM
qemu-system-x86_64 -hda ubuntu.img -cdrom mini.iso -net nic -net user
# Install Ubuntu as usual
# ...
# Start KVM guest again without CD-ROM
qemu-system-x86_64 -hda ubuntu.img -net nic -net user
# Install netperf inside guest Ubuntu
sudo apt-get update
sudo apt-get install build-essential
wget ftp://ftp.netperf.org/netperf/netperf-2.7.0.tar.bz2
tar xf netperf-2.7.0.tar.bz2
cd netperf-2.7.0
./configure
make
# Start netserver
cd netperf-2.7.0/src
./netserver -4

Network configuration (Port mapping):
# Start KVM guest with port mapping
# TCP 12865 is Netperf control channel
# TCP 12866 is used as Netperf TCP data channel, use -P 12866 to specify 
# UDP 12866 is used as Netperf UDP data channel, use -P 12866 to specify
qemu-system-x86_64 -hda ubuntu.img -net nic -net user -redir tcp:12865::12865  -redir tcp:12866::12866  -redir udp:12866::12866
# For OSv, use the same -redir options by changing scripts/run.py
# Inside scripts/run.py change `args += ["-redir", "tcp:8000::8000"]`
# To `args += ["-redir", "tcp:8000::8000", "-redir", "tcp:12865::12865", "-redir", "tcp:12866::12866", "-redir", "udp:12866::12866"]`
sudo ./scripts/run.py  -e "/tools/netserver.so -D -4 -f" -c 4 --api

STEPS TO RUNNING EXPERIMENT
Auto Test Running

Python script.py (as attachment)
// the line “cmd = './netperf -t UDP_STREAM -H 192.168.0.1 -- -P 12866 -r %s  -m 3100' % i “ is subject to change for TCP_STREAM we use -s instead of -r for our socket size.

Graph Plot

We firstly analyze the experiment by python plot but the graph is ugly so that we used excel to plot.



