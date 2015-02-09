# Setup-OpenStack
## DOC hướng dẫn cài đặt OpenStack bằng tiếng việt và có giải thích các bước trong quá trình cài đặt
[Mục lục]

[1. Lời mở đâu](#lmd)

[2. Chuẩn bị mô hình](#cbmh)

- [a. Thông tin cấu hình máy](#ttch)
	
- [b. Dựng mô hình cài đặt](#dmhcd)
	
- [c. Đổi các thông số cần thiết](#ttct)
	
- [d. Cài đặt gói NTP](#ntp)
	
- [e. Cài đặt gói Openstack packet](packet)
<a name="lmd"></a>
### 1. Lời mở đầu
Xin chào các bạn. Hôm nay tôi sẽ viết bài nói về quá trình cài đặt OpenStack bản Juno trên mô hình 3 node. Và được LAB trên môi trường VMware
<a name='cbmh'></a>
### 2. Chuẩn bị mô hình
<a name="ttch"></a>
#### a. Thông tin cấu hình máy
Đầu tiên các bạn xác định thông số cấu hình tối thiểu cho máy ảo cho 3 node
- Node controller : Ram 2G
- Node network :    Ram 1G
- Node compute :    Ram 4G

<a name="dmhcd"></a>
#### b. Dựng mô hình cài đặt
Bước tiếp theo các bạn cần dưng 3 node theo thông số đã được chọn và tạo được mô hình như sau:

<img class="image__pic js-image-pic" src="http://i.imgur.com/GPih2OH.png" alt="" id="screenshot-image">

<a name="ttct"></a>
#### c. Đổi các thông số cần thiết
Thêm một bước trong quá trình cài đặt rất quan trong là các bạn hãy chỉnh sửa file host trong các node theo file sau:
```
127.0.0.1   lookback
10.10.10.180    controller
10.10.10.181    network
10.10.10.182    compute
```
**Lưu ý: Thông tin trong file hosts tôi chỉnh theo trong sơ đồ. Nếu trong mô hình cài đặt của các bạn có địa chỉ IP khác thì các bạn cần sửa lại tương ứng nhé**

Sau đó các bạn hay đứng trên các node để ping giữa các node để đảm bảo chắc chắn rằng các node đã được thông với nhau

<a name="ntp"></a>
#### d. Cài gói NTP
Mục địch của việc cài gói NTP để đảm bảo việc đồng bộ thời gian giữa các node với nhau. Nếu thời gian không được đồng bộ thì các bạn cũng biết rồi đó, hệ thống của chúng ta sẽ die ngay lập tức

Trên tất các các node dùng lệnh sau để tải gói NTP về:
```
apt-get install ntp 
```
- Trên node Controller chỉnh sửa trong file cấu hình `/etc/ntp.conf`:
Các bạn hay comment tất cả các dòng có từ server và thêm dòng server sau vào:
```
server 0.asia.pool.ntp.org
server 1.asia.pool.ntp.org
server 2.asia.pool.ntp.org
server 3.asia.pool.ntp.org
```
Và comment 2 dòng sau:
```
restrict -4 default kod notrap nomodify nopeer noquery
restrict -6 default kod notrap nomodify nopeer noquery
```
Sau đó thêm 2 dòng sau vào
```
restrict -4 default kod notrap nomodify
restrict -6 default kod notrap nomodify
```
- Trên các node còn lại các bản hay comment tất cả các dòng có chữ server trong file cấu hình `/etc/ntp.conf` và thêm dòng sau đây:
```
server controller iburst
```
Mục đích của các bước là trên là để trên node Controller lấy thời gian chuẩn xác từ máy chủ và các node còn lại sẽ lấy thông tin thời gian của node Controller giúp cho hệ thống của chúng ta luôn được đồng bộ và đúng với thời gian với bên ngoài
- Sau đó reset lại ntp `service ntp restart ` trên tất cả các node. Và kiểm tra, ta dùng lệnh `ntpq -c peers`
Nếu kết quả trên node Controller hiện ra tương tự như sau

<img class="image__pic js-image-pic" src="http://i.imgur.com/HjFGpaU.png" alt="" id="screenshot-image">

Trên các node còn lại hiện ra như sau thì các bạn đã làm đúng

<img class="image__pic js-image-pic" src="http://i.imgur.com/3GnDUOQ.png" alt="" id="screenshot-image">

<a name="packet"></a>
#### e. Cài đặt gói Openstack packet
Cài đặt gói OpenStack packet ( thực hiện trên tất cả các Node).Trong quá trình cài đặt OPENSTACK chúng ta cần cài đặt bước 3 để quá trình cài đặt OPENSTACK được thành công
	```
apt-get install ubuntu-cloud-keyring
	```
```
echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/juno main" > /etc/apt/sources.list.d/cloudarchive-juno.list
```
Sau đó update Ubuntu
```
apt-get update && apt-get dist-upgrade
```
#### f. Cài đặt Database ( *thực hiện trên node Controller*)

Ta cần cài đặt DATABASE cho hệ thống để lưu trữ thông tin
	# apt-get install mariadb-server python-mysqldb

+ Chỉnh sửa thông tin trong file cấu hình /etc/mysql/my.cnf
```
[mysqld]
bind-address = 0.0.0.0 
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
```
Ý nghĩa của đoạn chỉnh sửa là cho phép máy khác có thể truy câp vào cơ sở dữ liệu
Sau đó reset lại mysql
`service mysql restart`
Thêm một thao tác qua đó sẽ giúp chúng ta bảo vệ tốt hơn DATABASE của mình
Dùng lệnh `mysql_secure_installation`
Sau đó hệ thống sẽ hiện ra các thông tin và chúng ta chọn xóa nhưng tài khoản nặc danh đi,vv..


#### f. Cài đặt gói Rabbit-server
Cài đặt gói rabbit-server với mục đích dùng để truyền thông giữa các service với nhau.Sử dụng câu lệnh sau để cài đặt gói rabbitmq-server
`apt-get install rabbitmq-server`
Tiếp theo chỉnh sửa lại mật khẩu cho tài khoản guest
`rabbitmqctl change_password guest hoainam123`
Mặc định trong rabbit-server có tài khoản guest và mật khẩu mặc định. Ta nên thay đổi phần mật khẩu cho tài khoản guest
để các service trong Openstack sử dụng tài khoản guest và mật khẩu đã tạo
**Lưu ý:** 
- Ta phải cấu hình cho mọi dịch vụ trong Openstack sử dụng tài khoản và mk đã đặt ra
- Tên `hoainam123` chỉnh là phần chỉnh sửa lại mật khẩu cho tài khoản guest trong Rabbit-server*

**===== Kết thúc quá trình chuẩn bị cho việc cài OpenStack =====**

### 3. Cài đặt OpenStack
#### a. Cài đặt dich vụ Identity
Đầu tiên các bạn phải hiểu các thuật ngữ trong phần Identity như:
- Tennant: Có thể bao gồm nhóm tài nguyên như nova,neutron hoặc cũng có thể là một nhóm người dùng.
- Endpoint: Thương là địa chỉ khả dụng như url để truy cập vào các dịch vụ như nova,neutron 
- Role: là các rule đối với từng tennant hoặc người dùng: VD như đói với tài khoản admin sẽ có toàn quyền trong một dịch vụ.
- Token: Nó là một chuỗi số dùng để xác thực cho các yêu cầu

**Bắt đầu quá trình cài đặt**

**Bước 1: Tạo một database keystone trong mysql và gán toàn quyền cho database keystone**
+ Các bạn truy cập vào DATABASE với tài khoản root bằng câu lệnh `mysql -u root -p` và tạo ra database keystone và gán toàn quyền cho nó bằng việc sử dụng câu lệnh
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';
```
Với KEYSTONE_DBPASS chính là password của keystone. Và tôi sẽ thay bằng pass là **hoainam123**

**Bước 2: Cài đặt dịch vụ keystone:**
Tải gói keystone về sửa dụng câu lệnh
```
apt-get install keystone python-keystoneclient
```
Sau đó chỉnh sửa trong file cấu hình `/etc/keystone/keystone.conf` như sau
```
[DEFAULT]
admin_token = ADMIN_TOKEN 
verbose = True 
[database]
connection = mysql://keystone:KEYSTONE_DBPASS@controller/keystone 
[token]
provider = keystone.token.providers.uuid.Provider
driver = keystone.token.persistence.backends.sql.Token
```
Phần ADMIN_TOKEN chính là token dành cho admin trong keystone và tôi thay bằng pass **hoainam123**
Và KEYSTONE_DBPASS chính là pass của database keystone là **hoainam123**
Sau khi chỉnh sửa xong các bạn dùng lệnh `cat /etc/keystone/keystone.conf | grep -v ^# | grep -v ^$`. File cấu hình keystone sẽ ra như sau

<img class="image__pic js-image-pic" src="http://i.imgur.com/hApBWZ3.png" alt="" id="screenshot-image">

Sau đó Update nhưng thông tin của dịch vụ keyston vào trong database keyston
`su -s /bin/sh -c "keystone-manage db_sync" keystone`

Restart lại dịch vụ keystone
`service keystone restart`

Xóa tài khoản keystone.db
`rm -f /var/lib/keystone/keystone.db`

Sử dụng thêm lệnh
```
(crontab -l -u keystone 2>&1 | grep -q token_flush) || \
echo '@hourly /usr/bin/keystone-manage token_flush >/var/log/keystone/keystone-tokenflush.log 2>&1' \
>> /var/spool/cron/crontabs/keystone
```

**Bước 3: Tạo ra các tenant, user, role**
Đầu tiên ta phải chạy với quyền admin với admin_token khai báo trong phần keystone
```
export OS_SERVICE_TOKEN=hoainam123
export OS_SERVICE_ENDPOINT=http://controller:35357/v2.0 
```
Với 2 lệnh này các thao tác của chúng ta sẽ được chạy với quyền admin ( quyền cao nhất trong hệ thống) vào chạy với endpoint trên port 35357

Sau đó tạo một tenant là admin và tạo một người dùng là admin và tạo role là admin ( với role sẽ có toàn quyền đối với các dịch vụ)

+ Tao tenant admin
`keystone tenant-create --name admin --description "Admin Tenant"`
+ Tạo user là admin
`keystone user-create --name admin --pass ADMIN_PASS --email EMAIL_ADDRESS`
Với ADMIN_PASS là pass của user admin ( pass này sẽ sử dụng khi đăng nhâp vào dashboard ) tôi thay bằng **hoainam123**
	 
+ Tạo role dành cho tenant admin
`keystone role-create --name admin`

**Bước 4: Cài đặt dịch vụ keyston vào trong phần keyston và API để truy nhập vào dịch vụ**
+ Tạo dịch vụ keyston trong phần KEYSTONE
`keystone service-create --name keystone --type identity --description "OpenStack Identity"`
+ Tạo API cho dịch vụ keystron để có thể truy nhập thông qua URL với port 5000 bằng việc chạy lệnh sau:
```
keystone endpoint-create \
--service-id $(keystone service-list | awk '/ identity / {print $2}') \
--publicurl http://controller:5000/v2.0 \
--internalurl http://controller:5000/v2.0 \
--adminurl http://controller:35357/v2.0 \
--region regionOne
```
**Bước 5: Tạo một scripts để khi sử dụng lại các quyền admin**
Sử dụng lệnh sau để tạo file admin-openrc.sh 
`vi admin-openrc.sh`
sau đó thêm các dòng sau vào trong file
```
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS # Phần này tôi thay ADMIN_PASS là hoainam123
export OS_AUTH_URL=http://controller:35357/v2.0
```
Với ADMIN_PASS là pass cho tài khoản admin và tôi thay bằng **hoainam123**

Sử dụng lệnh sau để tạo file demo-openrc.sh
`vi demo-openrc.sh`
Ghi vào trong file dòng sau
```
export OS_TENANT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS # Phần này tôi thay bằng tài khoản demo là hoainam123
export OS_AUTH_URL=http://controller:5000/v2.0
```
**Sau đó các bạn hay kiểm tra lại ( phần này các bạn xem thêm DOC để kiểm tra nhé =)) )**


**===== Kết thúc quá trình cài đặt dịch vụ Identity =====**

#### b. Cài đặt dịch vụ Image ( Cài đặt trên node Controller )

**Bước 1: Tạo DATABASE glance trong DATABASE**
+ Truy cập vào Mysql tạo DATABASE glance và gán toàn quyền cho DATABASE glance
```
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';
```
Với GLANCE_DBPASS là mật khẩu cho DATABASE glance để ta có thể truy cập và tôi thay bằng **hoainam123**

**Bước 2: Tạo dịch vụ Glance để quá trình xác thực trong dịch vụ xác thực KEYSTONE**
+ Chạy câu lệnh sau để sử dụng với quyền admin
`source admin-openrc`
+ Tạo người dùng glance vào trong Phần keyston
`keystone user-create --name glance --pass hoainam123`  
	Với **hoainam123** là password cho glance trong KEYSTONE

+ Gán quyền admin cho glance để keyston cấp toàn quyền cho glance
`keystone user-role-add --user glance --tenant service --role admin`
+ Tạo dich vụ glance 
`keystone service-create --name glance --type image --description "OpenStack Image Service"`
+ Tạo API cho dịch vụ glance để có thể truy nhập thông qua URL với port 9292 với câu lệnh sau:
```
keystone endpoint-create \
--service-id $(keystone service-list | awk '/ image / {print $2}') \
--publicurl http://controller:9292 \
--internalurl http://controller:9292 \
--adminurl http://controller:9292 \
--region regionOne
```

**Bước 3: Cài đặt gói Glance**
+ Cài đặt
`apt-get install glance python-glanceclient`
+ Chỉnh sửa file /etc/glance/glance-api.conf thêm các thông tin vào file như sau
```
[DEFAULT]
verbose = True
[keystone_authtoken]
auth_uri = http://controller:5000/v2.0
identity_uri = http://controller:35357
admin_tenant_name = service
admin_user = glance
admin_password = GLANCE_PASS   # Thay thế phần GLANCE_DBPASS là passwork của glance
[paste_deploy]
flavor = keystone
[glance_store]
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```
Sau đó ta update những thông tin vào trong DATABASE bằng lệnh
`su -s /bin/sh -c "glance-manage db_sync" glance`
Khi tạo mặc định glance sẽ lưu thông tin DATABASE và trong file glance.sqlite. Lúc này không cần thiết nữa nên chúng ta sẽ xóa file đó đi
`rm -f /var/lib/glance/glance.sqlite`

**Bước 4: Kiểm tra dịch vụ Image bằng việc tải image của hệ điều hành cirros và Up lên glance**
+ Tạo thư mục chứa image tải về
`mkdir /tmp/images`
`cd /tmp/images`
+ Dowload image
`wget http://cdn.download.cirros-cloud.net/0.3.3/cirros-0.3.3-x86_64-disk.img`
Và upload lên dịch vụ Image của OPENSTACK
```
glance image-create --name "cirros-0.3.3-x86_64" --file cirros-0.3.3-x86_64-disk.img --disk-format qcow2 \
	--container-format bare --is-public True --progress
```
+ Kiểm tra lại xem image đã up lên Image trong OPENSTACK
`glance image-list`
Kết quả chúng ta sẽ được như hình sau:

<img class="image__pic js-image-pic" src="http://i.imgur.com/7vCRBg2.png" alt="" id="screenshot-image">

**===== Kết thúc quá trình cài đặt dịch vụ Image =====**
