## Hướng dẫn cài đặt và cấu hình dịch vụ Glance trong OpenStack

Trước khi cài đặt và cấu hình dịch vụ Image, chúng ta cần tạo cơ sở dữ liệu, các dịch vụ cần thiết và các API endpoints.

#####1. Tạo cơ sở dữ liệu, thực hiện theo các bước sau:

  a. Sử dụng cơ sở dữ liệu truy cập người dùng kết nối với cơ sở dữ liệu máy chủ:

    ```sh
    $ mysql -u root -p
    ```
    
  b. Tạo cơ sở dữ liệu glance:
    
    ```sh
    CREATE DATABASE glance;
    ```
    
  c. Cấp quyền truy cập thích hợp cho cơ sở dữ liệu glance:
    
```sh
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
      IDENTIFIED BY 'GLANCE_DBPASS';
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
      IDENTIFIED BY 'GLANCE_DBPASS';
```    
  'GLANCE_DBPASS': password thích hợp
  
  Thoát khỏi cơ sỡ dữ liệu truy cập người dùng
  
#####2. Chuyển sang quyền admin để truy cập CLI:
```sh
   $ source admin-openrc.sh
```
#####3. Tạo các chính sách dịch vụ
 a. Tạo user tên là glance:
 ```sh
    $ keystone user-create --name glance --pass GLANCE_PASS
```
  GLANCE_PASS: điền pass thích hợp
  
  b. Gán quyền admin cho người dùng glance:
  
```sh
    $ keystone user-role-add --user glance --tenant service --role admin
```
  c. Tạo đối tượng dịch vụ cho glance:

```sh
    $ keystone service-create --name glance --type image \
      --description "OpenStack Image Service" 
```
#####4. Tạo các API endpoints cho dịch vụ Image: 
```sh
    $ keystone endpoint-create \
      --service-id $(keystone service-list | awk '/ image / {print $2}') \
      --publicurl http://controller:9292 \
      --internalurl http://controller:9292 \
      --adminurl http://controller:9292 \
      --region regionOne
```

####Cài đặt và cấu hình các thành phần trong dịch vụ Image


######1. Cài đặt các gói:

```sh
# apt-get install glance python-glanceclient 
```
    

######2. Sửa file /etc/glance/glance-api.conf :

  a.  [database]:
  
```sh
[database]
...
connection = mysql://glance:GLANCE_DBPASS@controller/glance
```
Chú ý: GLANCE_DBPASS là pass của cơ sở dữ liệu của dịch vụ Image đã chọn trước đó.
  
  b. [keystone_authtoken] và [paste_deploy]:
  
```sh
    [keystone_authtoken]
    ...
    auth_uri = http://controller:5000/v2.0
    identity_uri = http://controller:35357
    admin_tenant_name = service
    admin_user = glance
    admin_password = GLANCE_PASS
     
    [paste_deploy]
    ...
    flavor = keystone
```
  Chú ý: GLANCE_PASS là pass của người dùng glance
  
c. [glance_store]:
  
```sh
    [glance_store]
    ...
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/
```
d.  [DEFAULT]
  
```sh
    [DEFAULT]
    ...
    notification_driver = noop
``` 
e. (Optional)
  
```sh
    [DEFAULT]
    ...
    verbose = True
```
#####3. Gắn vào cơ sở dữ liệu dịch vụ Image:
 
```sh
# su -s /bin/sh -c "glance-manage db_sync" glance
```

####Hoàn thành cài đặt

#####1. Khởi động lại dịch vụ  Image:

```sh
# service glance-registry restart
# service glance-api restart
```

#####2. Theo mặc định, các gói Ubuntu tạo ra SQLite cơ sở dữ liệu: 

  Vì theo chúng ta sử dụng SQL, nên bạn có thể di xóa SQLite :
```sh
# rm -f /var/lib/glance/glance.sqlite

```
