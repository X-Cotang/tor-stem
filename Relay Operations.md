# Intro
Mạng Tor dựa vào các tình nguyện viên để quyên góp băng thông
# Types of relays on the Tor network
## Guard and middle relay
#### (also known as non-exit relays)
- Rơle bảo vệ là rơle đầu tiên trong chuỗi 3 rơle xây dựng mạch Tor.
- Để trở thành rơle bảo vệ, rơle phải ổn định và nhanh (ít nhất là 2 MB / giây) nếu không nó sẽ vẫn là rơle giữa. 
-  Tất cả các rơle sẽ được liệt kê trong danh sách công khai của rơle Tor.
- Rơle không thoát không cho phép thoát trong chính sách thoát của nó.
## Exit relay
- Rơle thoát là rơle cuối cùng trong mạch Tor
- Không nên chạy rơle thoát Tor từ nhà
-  Các nhà khai thác chuyển tiếp thoát lý tưởng được liên kết với một số tổ chức, như trường đại học, thư viện, không gian tin tặc hoặc tổ chức liên quan đến quyền riêng tư.
- Chính sách pháp lí cho các nhà khai thác exit relay:https://community.torproject.org/relay/community-resources/
## Bridge
- Thiết kế của mạng Tor có nghĩa là địa chỉ IP của rơle Tor là công khai.
- Cầu Tor là các nút trong mạng không được liệt kê trong thư mục Tor công cộng, điều này khiến các ISP và chính phủ khó chặn chúng hơn.
- Giúp đỡ những người bị kiểm duyệt để vào tor.
-  Pluggable transports một loại cầu đặc biệt có thêm một lớp che giấu bổ sung.

# Relay requirements
## Bandwidth and Connections
# Technical considerations
# Cấu hình Middle/Guard relay cho Ubuntu/Debian
### Enable Tor Package Repository in Debian
```bash
apt install apt-transport-https
```
- Thêm các dòng sau vào file /etc/apt/sources.list hoặc file mới trong /etc/apt/sources.list.d/
```
deb https://deb.torproject.org/torproject.org <DISTRIBUTION> main
deb-src https://deb.torproject.org/torproject.org <DISTRIBUTION> main
```
- Thay thế ```<DISTRIBUTION>```  bằng Operating System codename. Run ```lsb_release -c``` or ```cat /etc/debian_version``` để xem.
- Sau đó thêm khóa gpg được sử dụng để ký các gói bằng cách chạy các lệnh sau tại dấu nhắc lệnh của bạn:
```bash
wget -qO- https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -
```
- Cài đặt tor
```bash
apt update
apt install tor deb.torproject.org-keyring
```
### Configuration File
Put the configuration file /etc/tor/torrc in place:
```conf
#change the nickname "myNiceRelay" to a name that you like
Nickname myNiceRelay
ORPort 443
ExitRelay 0
SocksPort 0
ControlSocket 0
# Change the email address bellow and be aware that it will be published
ContactInfo tor-operator@your-emailaddress-domain
```
- Restart lại service
```
systemctl restart tor@default
```
# Obfs4 bridge
- Yêu cầu:
    1. 24/7 Internet connectivity
    2. The ability to expose TCP ports to the Internet (make sure that NAT doesn't get in the way)

1. Cài đặt tor
```bash
sudo apt-get install tor
```
2. Install obfs4proxy
```bash
sudo apt-get install obfs4proxy
```
3. Edit file torcc
```conf
BridgeRelay 1

# Replace "TODO1" with a Tor port of your choice.
# This port must be externally reachable.
# Avoid port 9001 because it's commonly associated with Tor and censors may be scanning the Internet for this port.
ORPort TODO1

ServerTransportPlugin obfs4 exec /usr/bin/obfs4proxy

# Replace "TODO2" with an obfs4 port of your choice.
# This port must be externally reachable and must be different from the one specified for ORPort.
# Avoid port 9001 because it's commonly associated with Tor and censors may be scanning the Internet for this port.
ServerTransportListenAddr obfs4 0.0.0.0:TODO2

# Local communication port between Tor and obfs4.  Always set this to "auto".
# "Ext" means "extended", not "external".  Don't try to set a specific port number, nor listen on 0.0.0.0.
ExtORPort auto

# Replace "<address@email.com>" with your email address so we can contact you if there are problems with your bridge.
# This is optional but encouraged.
ContactInfo <address@email.com>

# Pick a nickname that you like for your bridge.  This is optional.
Nickname PickANickname
```
- Nếu dùng cổng nhỏ hơn 1024 thì cần cung cấp cho obfs4 các khả năng của CAP_NET_BIND_SERVICE để liên kết cổng với người dùng không phải là root
```bash
sudo setcap cap_net_bind_service=+ep /usr/bin/obfs4proxy
```
- Đặt NoNewPrivileges=no vào /lib/systemd/system/tor@default.service và /lib/systemd/system/tor@.service và sau đó chạy ```systemctl daemon-reload```
- Restart tor
```bash 
systemctl restart tor
```
- Xem log trong /var/log/tor/log hoặc /var/log/syslog
# Exit node 
- (chán chả muốn nói):https://community.torproject.org/relay/setup/exit/
# Reference
https://community.torproject.org/relay/setup/