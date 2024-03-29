THIS FILE IS UNDER CONSTRUCTION.  please mail stevef@pcbi.upenn.edu with questions....

This file is a guide to installing a mysql server by a regular user (not an administrator).
It installs mysql in the user's space.  

It is intended to serve users who:
  -  do not already have a mysql available, and cannot get an admin to install one
  -  do have one already installed, but will not be able to use it or reconfigure it


## INSTALLATION OVERVIEW

I.      General Requirements
II.	Installing from General Linux/Unix Binary Packages
III.	Creating a Database and User Account
IV.     Installing Required PERL modules for MySQL
V.      Optimizing MySQL
VI.     Troubleshooting Installation Issues
VII.    Removing Your MySQL Installation


### I. General Requirements  

    -  MySQL server 5.1 or greater
    -  MySQL DBI and DBD driver modules for PERL (available at CPAN)
    -  Unix/Linux or MacOS 10.0.1 or greater



### II. Installing MySQL From Linux/Unix Binary Packages (Root access not required)

   1. Go to  http://www.mysql.com/downloads/
       a) click on "Download" under MySQL Community Server

       b) for linux: click on "Linux (non RPM packages)"

       c) otherwise, find the .tar package for your OS

       d) choose the download for your platform
          - use the 'uname -a' command if you don't know your platform
          - 'cat /proc/cpuinfo | grep model' will list your processor type (ie.
             AMD or Intel, 32 or 64 bit) if you are running Linux. 

       e) either login/register or click "No thanks, just take me to the downloads!"

       f) choose a mirror near you, and download


   2. Create a directory for your installation, and move the file to it.
       a) This directory needs to be on a volume that has *adequate space* for all the data. 
          - Please see the UserGuide.txt to estimate how much disk space you will need.

       b) (See step 6 below if you want to relocate the mysql data to a separate volume.)


   3. Unzip the downloaded file into the directory:
  
		tar -xzvf mysql-standard-5.1.34-linux-i686-glibc23.tar.gz
                rm mysql-standard-5.1.34-linux-i686-glibc23.tar.gz   (to save disk space)

   4. Give the new directory a shortcut name

	 	ln -s mysql-standard-5.1.34-linux-i686-glibc23 mysql
    

   5. Change to the mysql directory and configure mysql using the sample config file provided in orthomcl
      download:

               cd mysql
               cp my_orthomcl_dir/doc/OrthoMCLEngine/Main/mysql.cnf .

       a) set basedir= to the full path of your new mysql directory

       b) set datadir= to the full path of the /data subdir in your new mysql
          directory
           - this is the directory that will hold all the data.
           - use the df -h command to see how much space you have
           - you will need at least 5x the size of the file made by
             orthomclBlastParser
           - (revisit this after you have run orthomclBlastParser)

       c) now see the mysqlConfigurationGuide.txt for important optimization
          configuration items

   6. Set up the default MySQL databases:

	        
                ./scripts/mysql_install_db --defaults-file=mysql.cnf 
    
        NOTE: The script will inform that you that need to set a root password. 
              Don't worry about this for now; you will perform this task in another step.


   7. You are now ready to start your MySQL server as a background process. To do so, from within your mysql directory, run:

           ./bin/mysqld_safe --defaults-file=mysql.cnf &
        
         NOTE: You *must* run this command from the mysql directory.
               You should see something similar to the following:

 		[1] 67786% Starting mysqld daemon with databases from home/youraccountname/mysql/data

   8. At this point your MySQL password is still blank. Use the following command to set a new                     
      root password:

		./bin/mysqladmin --defaults-file=mysql.cnf -u root password "yourpasswordhere"

      NOTE: DO NOT FORGET THIS PASSWORD.  write it down someplace that you won't forget.


### III. Create a New Database and User Account

   1. Log in to your mysql server as root. If you are logging in to an existing MySQL server, 
      use any existing account that can create a user and grant privileges: 

		./bin/mysql --defaults-file=mysql.cnf -u root -p

      Enter the root password you set in Step II.8 when prompted.

   2. Once logged in as root, create the database and user (schema) that you will use for OrthoMCL
      (we use orthomcl as an example here), and grant the user account the necessary privileges:
 
              mysql> CREATE DATABASE orthomcl;
	
	      mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE VIEW,CREATE, INDEX, DROP on orthomcl.* TO orthomcl@localhost;
	
 	      mysql> set password for orthomcl@localhost = password('yourpassword');

      NOTE: DO NOT FORGET THIS PASSWORD.  write it down someplace that you won't forget

      NOTE: if you want to play with the data in the database, you can get into it like this:
               ./bin/mysql --defaults-file=mysql.cnf -u orthomcl


### IV. Installing the Required PERL Modules for MySQL 

   1. Check to see if the DBI and DBD::mysql PERL modules
      are installed:

          $ perl -MDBI -e 1
          $ perl -MDBD::mysql -e 1

      - If you receive no output, then the module *is* installed and you
        can continue to section V. However, if you receive an error message for
        either, continue to step 2 and install the missing module(s).

      - It is often the case that the DBI module is installed, but DBD:mysql
        is not.


   2. If you have root access, the easiest way to install Perl modules on Unix/Linux 
      is to perform a system-wide install using CPAN:

          $ perl -MCPAN -e shell
          cpan> o conf makepl_arg "mysql_config=/path_to_your_mysql_dir/bin/mysql_config"
          cpan> install Data::Dumper
          cpan> install DBI
          cpan> force install DBD::mysql


   3. Installing modules as a standard user

      - Follow the steps below to install modules as a non-root user. We
        assume /myperl in your home directory is your custom perl directory.

         1. In your home directory, create the PERL and CPAN directories, and 
            a blank CPAN config module:
                
                $ mkdir myperl
                $ mkdir .cpan
                $ mkdir .cpan/CPAN
                $ echo "\$CPAN::Config = {}"> ~/.cpan/CPAN/MyConfig.pm
 
         2. Configure your environment by adding the following to your
            
            .bash_profile file in your home directory:
            ```bash
               ######################################
               if [ -z "$PERL5LIB" ]
                 then
	          # If PERL5LIB wasn't previously defined, set it...
	          PERL5LIB=~/myperl/lib
                else
	          # ...otherwise, extend it.
 	          PERL5LIB=$PERL5LIB:~/myperl/lib
               fi

               MANPATH=$MANPATH:~/myperl/man

               export PERL5LIB MANPATH
               ######################################
            ```

          3. Create the necessary directories and process your .bash_profile:
               ```bash
                $ mkdir -p ~/myperl/lib
                $ mkdir -p ~/myperl/man/man{1,3}
                $ source ~/.bash_profile
                ```
 
          4. Confirm that your custom per5lib paths have been set:

               ``` $ perl -wle'print for grep /myperl/, @INC'```

              - You should see pathing relative to your home directory. If not,
                repeat steps 1-3.
        
          5. Invoke the CPAN shell and complete CPAN configuration:

                ```$ perl -MCPAN -we shell```

             - CPAN will request that you set your config. Accepting the
               default (type <ENTER) is ok for most questions, but watch for 
               and change "Your choice" for two options; the Makefile.Pl
               and the UNINST options:
 

                 Parameters for the 'perl Makefile.PL' command?
                 Typical frequently used settings:

                       PREFIX=~/perl       non-root users (please see manual for more hints)
                       
                 Your choice:  [] 
                 ```
                 PREFIX=~/myperl LIB=~/myperl/lib INSTALLMAN1DIR=~/myperl/man/man1 INSTALLMAN3DIR=~/myperl/man/man3
                 ```
                Parameters for the 'make install' command?
                Typical frequently used setting:

                        UNINST=1         to always uninstall potentially conflicting files

                Your choice:  [] UNINST=0
               


           1. Set your mysql PATH, and install the modules that you have determined 
              you need:
 
                 ```bash
                 $ PATH=$PATH:/path_to_your_mysql_directory/bin
                 $ export PATH
                 $ perl -MCPAN -we shell
                 cpan> install Data::Dumper
                 cpan> install DBI
                 cpan> force install DBD:mysql
                 ```
                            

### V. MySQL Server Optimization
   
   - please see the mysqlConfigurationGuide.txt document provided in the orthomcl download.



### VI. Troublshooting Installation Issues

   The MySQL server logs all status and error messages in a file called
yourhost.err, where yourhost is the name of your machine. The file is located 
in the mysql/data directory and contains useful information for debugging problems 
with your MySQL server.

Below are some common installation issues and resolutions:

 (1)
     ISSUE: Your MySQL installation is conflicting with another install

        - You may be conflicting with an existing MySQL install if you see an
          error in your log similar to the following when running mysql_install_db, 
          mysqladmin, or mysql:

              Installing MySQL system tables...
              090515 13:32:49 [Warning] Can't create test file
              /var/lib/mysql/localhost.lower-test
              090515 13:32:49 [Warning] Can't create test file
              /var/lib/mysql/localhost.lower-test
              ERROR: 1005  Can't create table 'db' (errno: 13)
              090515 13:32:49 [ERROR] Aborting

          
     RESOLUTION:

         - A path is set incorrect in your mysql.cnf if you see a reference to
           /var/lib in your error log. Check to ensure that you have correctly 
           set the mysql_sock and port parameters in mysql.cnf.

         -  - to see if 3307 is in use, type this command:
                    netstat -a | grep tcp | grep 3307
           - if so, set port=3308 (or 3309, etc., if 3308 is already used) 

 (2)
     ISSUE: Unspecified or Misconfigured mysql.cnf file 

        - If only the first numeric line appears (you do not see a "Starting
          mysqld daemon..." message) when you execute ./bin/mysqld_safe, you
          probably entered at least one incorrect path in your mysql.cnf file, or
          you did not specify --defaults-file=mysql.cnf when starting MySQL.


     RESOLUTION:

        - Check to ensure that:  

           - You specified the --defaults-file=mysql.cnf when running mysql or
             a mysqladmin command.
        
           - Your mysql.cnf parameters correctly reflect your MySQL
             installation path. 

                


### VI. Removing Your MySQL Installation

  - If you wish to remove your MySQL installation, this can be performed in
    two simple steps:

       (1) Shutdown your MySQL server:

            cd mysql
            ./bin/mysqladmin --defaults-file=mysql.cnf -u root -p shutdown

     
       (2) Change to the parent directory of your MySQL installation, and remove 
           the mysql directory and symbolic link:

       
            rm -r -f mysql-standard-5.1.34-linux-i686-glibc23 
            rm mysql


            NOTE: Always use caution when using the rm -r -f command! This
                  command deletes an entire directory structory with no request
                  for confirmation, so be sure the correct directory is specified.







          
	    

		
 




