# Ryu-ML-SDN-
Ryu SDN ML Traffic Flow 
SDN LAB Servers DHCP, HTTPS/DNS, File server 


RUN Ryu Controller 

~/mininet/ryu run ml_routing.py


sudo mn --topo single,3 --mac --switch ovsk --controller=remote,ip=127.0.0.1,port=6653 --nat

HTTP 
h2 python3 -m http.server 80 &

DNS 
h2 echo "addres=/example.com/10.0.0.2" > /etc/dnsmasq.conf
mininet> h2 pkill dnsmasq
mininet> h2 dnsmasq --no-daemon --interface=h2-eth0 --listen-address=10.0.0.2 --bind-interfaces --address=/example.com/10.0.0.2 &

mininet> h2 service dnsmasq restart

 h2 netstat -tulpn | grep 53
mininet> h1 nslookup example.com 10.0.0.2


# Stop any running DHCP server first
mininet> h3 pkill dhcpd

# Create a clean DHCP configuration
mininet> h3 bash -c "cat > /etc/dhcp/dhcpd.conf << 'EOF'
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.10 10.0.0.50;
  option routers 10.0.0.3;
  option domain-name-servers 10.0.0.2;
}
EOF"

# Configure the interface
mininet> h3 bash -c "echo 'INTERFACESv4=\"h3-eth0\"' > /etc/default/isc-dhcp-server"


# Start DHCP server
mininet> h3 pkill dhcpd

mininet> h3 dhcpd -4 -cf /etc/dhcp/dhcpd.conf -lf /var/lib/dhcp/dhcpd.leases h3-eth0

 h1 ifconfig h1-eth0 0

 mininet> h1 dhclient h1-eth0


File Server 

mininet> h3 mkdir -p /srv/files
mininet> h3 echo "Welcome to File Server" > /srv/files/info.txt
mininet> h3 python3 -m http.server 8080 --directory /srv/files &



Testing 

# Check HTTP server on h2
mininet> h2 netstat -tulpn | grep :80

# Check DNS server on h2  
mininet> h2 netstat -tulpn | grep :53

# Check file server on h3
mininet> h3 netstat -tulpn | grep :8080

# Check DHCP server on h3
mininet> h3 netstat -ulnp | grep :67


# Test from h1
mininet> h1 curl -s http://10.0.0.2 | head -10

# Test with wget
mininet> h1 wget -q -O- http://10.0.0.2

# Test specific file download
mininet> h1 wget http://10.0.0.2/README.md


# Test DNS resolution for custom domain
mininet> h1 nslookup example.com 10.0.0.2

# Test reverse DNS
mininet> h1 nslookup 10.0.0.2 10.0.0.2

# Test external DNS resolution (should work through NAT)
mininet> h1 nslookup google.com 10.0.0.2

# Test file server homepage
mininet> h1 curl -s http://10.0.0.3:8080

# Download the test file
mininet> h1 wget -q -O- http://10.0.0.3:8080/info.txt

# Test file download and verify content
mininet> h1 wget -q http://10.0.0.3:8080/info.txt -O /tmp/testfile.txt
mininet> h1 cat /tmp/testfile.txt

# First, check current IP on h1
mininet> h1 ifconfig h1-eth0

# Release current IP and get new one via DHCP
mininet> h1 dhclient -r h1-eth0
mininet> h1 ifconfig h1-eth0 0.0.0.0
mininet> h1 dhclient -v h1-eth0

# Verify new IP (should be in range 10.0.0.10-50)
mininet> h1 ifconfig h1-eth0

# Check DHCP lease information
mininet> h3 cat /var/lib/dhcp/dhcpd.leases


# Test external DNS resolution
mininet> h1 nslookup google.com

# Test ping to external sites
mininet> h1 ping -c 3 google.com

# Test web access to external sites
mininet> h1 curl -I https://www.google.com

# Test apt-get (internet access)
mininet> h1 apt-get update


# Test ping between all hosts
mininet> pingall

# Test specific host connectivity
mininet> h1 ping -c 3 h2
mininet> h1 ping -c 3 h3
mininet> h2 ping -c 3 h3


# Iperf BW testing 
mininet> h3 iperf -u -c 10.0.0.2 -b 100M &

mininet> h1 iperf -u -c 10.0.0.2 -b 10M &
