# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
 config.vm.synced_folder ".", "/vagrant", disabled: true

 config.vm.define "jenkins" do |jenkins|
    jenkins.vm.box = "../sbeliakou-vagrant-centos-7.3-x86_64-minimal.box"
    jenkins.vm.network "private_network", ip: "192.168.56.10"
    jenkins.vm.hostname = "jenkins"
    jenkins.vm.network :forwarded_port, guest: 80, host: 8080, auto_correct: true
    jenkins.vm.network :forwarded_port, guest: 8080, host: 8090, auto_correct: true
    jenkins.vm.synced_folder "../master/", "/opt/jenkins/master"
    jenkins.vm.synced_folder "../soft/", "/opt/jenkins/bin" 
    jenkins.vm.provider "virtualbox" do |v|
	v.memory = "2048"
	v.name = "jenkins"
    end
    jenkins.vm.provision "shell", inline: <<-SHELL
	setenforce 0;
	chattr -i /etc/selinux/config;
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config;
	systemctl stop firewalld
	systemctl disable firewalld
	yum install net-tools nginx -y
	systemctl enable nginx

	###NGINX configuring###
	cd /etc/nginx
	rm -rf nginx.conf
	wget https://raw.githubusercontent.com/untiro/vagrant/master/nginx.conf -a /var/log/wget.log
	cd /etc/nginx/conf.d
	wget https://raw.githubusercontent.com/untiro/vagrant/master/server.conf -a /var/log/wget.log
	systemctl start nginx	

	###JAVA install ###
	cd /opt/
#	wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz" -a /var/log/wget.log
#	echo "wget command finished - you may chack /var/log/wget.log for results"
	tar xzf /opt/jenkins/bin/jdk-8u131-linux-x64.tar.gz -C /opt
	cd /opt/jdk1.8.0_131/
	alternatives --install /usr/bin/java java /opt/jdk1.8.0_131/bin/java 2
	alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_131/bin/jar 2
	alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_131/bin/javac 2
	alternatives --set jar /opt/jdk1.8.0_131/bin/jar
	alternatives --set javac /opt/jdk1.8.0_131/bin/javac
	echo "export JAVA_HOME=/opt/jdk1.8.0_131" >>/etc/environment
	echo "export JRE_HOME=/opt/jdk1.8.0_131/jre" >>/etc/environment
	echo "export PATH=$PATH:/opt/jdk1.8.0_131/bin:/opt/jdk1.8.0_131/jre/bin" >>/etc/environment
	
	###JENKINS install###
#	mkdir -p /opt/jenkins/{bin,master}
#	groupadd jenkins
#	useradd -g vagrant -d /opt/jenkins/master jenkins
	echo "export JENKINS_HOME=/opt/jenkins/master" >>/etc/environment
	export JENKINS_HOME=/opt/jenkins/master
	echo "export JENKINS_DIR=/opt/jenkins/bin" >>/etc/environment
	export JENKINS_DIR=/opt/jenkins/bin
#	cd $JENKINS_DIR && wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war -a /var/log/wget.log
#	echo "wget command finished - you may chack /var/log/wget.log for results"
	#init script#
#	touch /opt/jenkins/jnk-stup.sh
#	echo "sudo -u jenkins java -jar $JENKINS_DIR/jenkins.war > /var/log/jenkins-startup 2> /var/log/jenkins-startup2 &" > /opt/jenkins/jnk-stup.sh
#	chmod +x /opt/jenkins/jnk-stup.sh
#	chown -R jenkins:jenkins /opt/jenkins/
	cd /etc/systemd/system
        wget https://raw.githubusercontent.com/untiro/vagrant/master/jenkins.service -a /var/log/wget.log
	systemctl daemon-reload
	systemctl enable jenkins.service
	systemctl start jenkins.service
	sleep 8s
	echo "Admin Password: " && cat /opt/jenkins/master/.jenkins/secrets/initialAdminPassword
    SHELL
  end
end
