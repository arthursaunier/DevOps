# TP03

##

#### ssh comd:
>ssh -i key/id_rsa centos@arthur.saunier.takima.cloud

ansible all -m yum -a “name=httpd state=present" --private-key=key/id_rsa -u centos --become

ansible all -m shell -a 'echo "<html><h1>Hello CPE</h1></html>" >> /var/www/html/index.html' --private-key=key/id_rsa -u centos --become

ansible all -m service -a "name=httpd state=started" --private-key=key/id_rsa -u centos --become