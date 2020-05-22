#Стенд ДЗ NFS

	1. Создан файл Vagrant, в котором поднимаются 2 ВМ: NFSSRV и NFSCLNT
	2. Шарим на сервере папку Upload с правом записи:
		2.1 Ставим пакет yum install nfs-utils -y
		2.2 Создаём шару mkdir /home/vagrant/Upload
		2.3 Задаём права на папку chmod -R 755 /home/vagrant/Upload
		2.4 Задаём пользователя для папки chown vagrant:vagrant /home/vagrant/Upload
		2.5 Активируем сервисы и стартуем их:
			systemctl enable rpcbind
			systemctl enable nfs-server
			systemctl enable nfs-lock
			systemctl enable nfs-idmap
			systemctl start rpcbind
			systemctl start nfs-server
			systemctl start nfs-lock
			systemctl start nfs-idmap
		2.6 Расшариваем директорию через сеть:
			vi /etc/exports
```
			/home/vagrant/Upload    192.168.11.* (rw,sync,no_root_squash,no_all_squash)
			exportfs -a
		exportfs: No options for /home/vagrant/Upload 192.168.11.*: suggest 192.168.11.*(sync) to avoid warning
		exportfs: No host name given with /home/vagrant/Upload (rw,async,no_root_squash,no_all_squash,no_subtree_check), suggest *(rw,async,no_root_squash,no_all_squash,no_subtree_check) to avoid warning

```
		2.7 Рестарт NFS службы
```
			systemctl restart nfs-server
```
		2.8 Переопределяем порты NFS службы в файерволе 
```
			firewall-cmd --permanent --zone=public --add-service=nfs
			firewall-cmd --permanent --zone=public --add-service=mountd
			firewall-cmd --permanent --zone=public --add-service=rpc-bind
			firewall-cmd --reload


			[root@NFSSRV vagrant]# systemctl start firewalld.service
			[root@NFSSRV vagrant]# firewall-cmd --permanent --zone=public --add-service=nfs
			success
			[root@NFSSRV vagrant]# firewall-cmd --permanent --zone=public --add-service=mountd
			success
			[root@NFSSRV vagrant]# firewall-cmd --permanent --zone=public --add-service=rpc-bind
			success
			[root@NFSSRV vagrant]# firewall-cmd --reload
			success

```
	3. Настройка клиента NFS
	4. Проверяем доступность по сети сервера NFS пингом
```
			$ ping 192.168.11.104
			PING 192.168.11.104 (192.168.11.104) 56(84) bytes of data.
			64 bytes from 192.168.11.104: icmp_seq=1 ttl=64 time=1.47 ms
			64 bytes from 192.168.11.104: icmp_seq=2 ttl=64 time=0.749 ms
```
	5. Установка yum install nfs-utils
	6. Создаём директории для монтирования шары с NFS сервера:
		mkdir -p /mnt/nfs/Upload
	7. Прописываем монтирование на клиенте сетевой директории
		mount -t nfs 192.168.11.104:/home/vagrant/Upload /mnt/nfs/Upload
```
		mount.nfs: timeout set for Fri May 22 05:55:13 2020
		mount.nfs: trying text-based options 'vers=4.1,addr=192.168.11.104,clientaddr=192.168.11.102'
```
	8. Пропишем монтирование в fstab
```
		echo "192.168.11.104:/home/vagrant/Upload /mnt/nfs/Upload nfs defaults 0 0">>/etc/fstab
```
	9. Для общей проверки перезапустил клиент и сервер nfs, создал файл в примонтированной директории и проверил его наличие на сервере. 
