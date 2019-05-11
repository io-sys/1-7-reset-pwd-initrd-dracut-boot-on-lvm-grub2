# -*- mode: ruby -*-
# vim: set ft=ruby :
home = 'C:/Users/io.sys/Downloads/VirtualBox VMs/lab04-bash/'
# ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :linuxbash => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.101',
    :disks => {
        :sata1 => {
            :dfile => home + '/VirtualBox VMs/sata1.vdi',
            :size => 1024,
            :port => 1
        },
=begin
        :sata2 => {
            :dfile => home + '/VirtualBox VMs/sata2.vdi',
            :size => 1024, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => home + '/VirtualBox VMs/sata3.vdi',
            :size => 1024, # Megabytes
            :port => 3
        },
        :sata4 => {
            :dfile => home + '/VirtualBox VMs/sata4.vdi',
            :size => 1024,
            :port => 4
        }
=end
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            #box.vm.network "private_network", ip: boxconfig[:ip_addr]
			box.vm.network "public_network", bridge: 'Intel(R) Dual Band Wireless-AC 3165', type: "dhcp"
  
            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
                    needsController = false
            boxconfig[:disks].each do |dname, dconf|
                unless File.exist?(dconf[:dfile])
                  vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                  needsController =  true
                            end
  
            end
                    if needsController == true
                       vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                       boxconfig[:disks].each do |dname, dconf|
                           vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                       end
                    end
            end
  
        box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            yum install -y \
			redhat-lsb-core \
			wget \
			make \
			gcc \
			mock \
			mc \
			vim\
			tree \
			rpmdevtools \
			rpm-build \
			createrepo \
			yum-utils
			yum provides curl
			rpmdev-setuptree
			cd ~/rpmbuild/SRPMS/
			yumdownloader --source curl-7.29.0
			rpm -ivh ~/rpmbuild/SRPMS/curl-7.29.0-51.el7.src.rpm
			mkdir /usr/src/OpenSSL && cd /usr/src/OpenSSL
			wget https://www.openssl.org/source/openssl-1.0.2r.tar.gz
			tar -xvf openssl-1.0.2r.tar.gz --directory=/opt/
			sed -i.bak01 's@--without-ssl@--with-ssl=/opt/openssl-1.0.2r@' ~/rpmbuild/SPECS/curl.spec
			yum-builddep -y ~/rpmbuild/SPECS/curl.spec
			rpmbuild -bb ~/rpmbuild/SPECS/curl.spec
			yum localinstall -y ~/rpmbuild/RPMS/x86_64/curl-7.29.0-51.el7.centos.x86_64.rpm
			curl --version
			curl https://ya.ru | grep "Яндекс"
			yum install -y epel-release  
			yum install -y nginx
			systemctl start nginx
			systemctl enable nginx
			createrepo /usr/share/nginx/html   
			echo -e " \
			[otus]\n \
			name=Otus-Linux\n \
			baseurl=http://10.10.11.18/\n \
			enabled=1\n \
			gpgcheck=0" > /etc/yum.repos.d/otus.repo			
			rm -f /usr/share/nginx/html/*.png & rm -f /usr/share/nginx/html/*.html
			yum repolist enabled
			rm -f /usr/share/nginx/html/*.png & rm -f /usr/share/nginx/html/*.html
			yum repolist enabled
			cp ~/rpmbuild/RPMS/x86_64/curl-7.29.0-51.el7.centos.x86_64.rpm /usr/share/nginx/html/     
			createrepo /usr/share/nginx/html/
			yum repolist enabled             
			yum search curl
			sed -i.bak01 '/^ *location \/ {/a \\t\tautoindex on;' /etc/nginx/nginx.conf
			nginx -t
			nginx -s reload
			cp ~/rpmbuild/RPMS/x86_64/curl-debuginfo-7.29.0-51.el7.centos.x86_64.rpm /usr/share/nginx/html/
			createrepo /usr/share/nginx/html 
			yum repolist enabled | grep Otus
          SHELL
  
        end
    end
  end
  
