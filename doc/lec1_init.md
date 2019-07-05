# System Pre-configuration Checks

1. Update yum
```bash
sudo yum update -y
```
2. Change the run level to multi-user text mode
```bash
sudo systemctl isolate multi-user.target
sudo systemctl isolate runlevel3.target
```
3. Disable SE Linux
```bash
sudo vi /etc/sysconfig/selinux # SELINUX=disabled 수정
```
4. Disable firewall
```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```
5. Check vm.swappiness and update permanently as necessary.
```bash
cat /proc/sys/vm/swappiness
sudo vi /etc/sysctl.conf # vm.swappiness=1 추가
```
6. Disable transparent hugepage support permanently [https://www.cloudera.com/documentation/enterprise/5-12-x/topics/cdh_admin_performance.html]
```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
sudo vi /etc/default/grub # transparent_hugepage=never 내용 추가
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot
cat /proc/cmdline # 
```
7. Check to see that nscd service is running 
```bash
sudo yum install nscd -y
sudo systemctl start nscd
sudo systemctl enable nscd
sudo systemctl status nscd
```
8. Check to see that ntp service is running
```bash
sudo systemctl stop chronyd
sudo systemctl disable chronyd
sudo yum install ntp -y
sudo systemctl start ntpd
sudo systemctl enable ntpd
```
9. Disable IPV6 
```bash
ip addr | grep inet6
sudo vi /etc/default/grub # ipv6.disable=1 내용 추가
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
# sudo reboot
ip addr | grep inet6 
```
10. setup a private/public key
```bash
ssh-keygen -t rsa

.ssh/id_rsa
.ssh/id_rsa.pub

chmod 600 .ssh/id*
```
11. update /etc/hosts
```bash
  172.31.13.181 cm.my.prac cm
  172.31.1.236  d1.my.prac d1
  172.31.1.125  d2.my.prac d2
  172.31.1.58   d3.my.prac d3
  172.31.3.197  m1.my.prac m1
```
12. change each hostname
```bash
sudo vi /etc/hostname
sudo reboot
```
13. add known hosts
```bash
ssh centos@cm
ssh centos@d1
ssh centos@d2
ssh centos@d3
ssh centos@m1
```
