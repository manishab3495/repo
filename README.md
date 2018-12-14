This setup will install Apache, Tomcat and MySQL on the remote server
To run the setup, we have main.yml file with all required details.
This file will call below three roles one by one. 
1. Apache
2. Tomcat
3. MySQL

You need to run below command to run the setup
#ansible-playbook main.yml -u root

This command will run main.yml playbook with root user to make required changes on remote machine.
I have configured serverless ssh login with remote server from my host with root user, it means I don't need to type password or provide key all the time with command.

Above main.yml will call below three files one by one, which has the configuration details of the respective application.
1. roles/apache/tasks/main.yml
2. roles/tomcat/tasks/main.yml
3. roles/mysql/tasks/main.yml

Apache configuration tasks mentioned in the roles/apache/tasks/main.yml step by step: 
1. Installation of apache appliation
2. Enabling apache on server reboot
3. Enabling required packages to configure apache for loadbalancing with tomcat
4. Adding apache configuration file with loadbalancer changes
5. After all the configuration, restarting apache service

Tomcat configuration tasks mentioned in the roles/tomcat/tasks/main.yml step by step:
1. To make tomcat run, we always need java to be installed. Hence installing openjdk-7-jre
2. Creating tomcat user and group in next step.
3. To reduce the time to download the file from internet, I have copied tomcat tarball already on the machine using same tarball to copy on remote machine and extracting it under /opt
4. As per the requirement, two tomcat servers needs to run on same machine, we need two tomcat setup with different ports, to achive this, I have created tomcat1 and tomcat2 directory under /opt. In which, tomcat1 will run on port 8080 and tomcat2 will run on port 9080
5. To make port changes, different server.xml files are required. I made ONLY port changes in main server.xml and split it into two parts. One for tomcat1 and another one for tomcat2. Copying the files under conf directory of each tomcat.
6. To make database connection, under tomcat directory, conf/Catalina/localhost directory needs to be present. Creating the directory copying the ROOT.xml file which is required to show the database connectivity configuration under conf/Catalina/localhost
7. Another dependeny to connect to database is "JDBC" connector. I have downloaded the jar file from internet and copying it into lib directory to make use of it. 
8. To run tomcat without any hustle, I have changes the permission of catalina.sh and startup.sh files. Also, changed the ownership of the tomcat directory recursively. 
9. To start tomcat at server reboot, we need its init script to be present under /etc/init.d directory. Creating two different files for tomcat1 and tomcat2 and allowing tomcat to start at server reboot.
10. And finally, starting tomcat by running startup.sh script. I have used "nohup and &" to start the command because, ansible creates a shell session and run the instruction written in yml files on remote server only on that session. But once ansible is done will all the configuration changes, it closed the shell session which cause the startup.sh script to kill itself. To overcome this problem, I have used "nohup and &" in the command.

Tomcat configuration tasks mentioned in the roles/mysql/tasks/main.yml step by step:
1. MySQL asks for root password to setup at the time of its installation. If we do not provide it, MySQL fails to start. To configure root password, I have used debconf module which indicates the question and provides the value at the runtime. I am setting password as password for root in this task.
2. Installing mysql 5.6
3. Enabling mysql to start the server reboot
4. And finally restarting mysql after all setup. 


If you want to run single application at a time, you can use below command and replace the application's yml file as per the requirement.

To configure apache:
#ansible-playbook apache.yml -u root

To configure MySQL:
#ansible-playbook mysql.yml -u root

To configure tomcat:
#ansible-playbook tomcat.yml -u root

To get the details output on the screen, you can always use "-vvvv" flag in ansible-playbook command.