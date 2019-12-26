# minio-cluster
Developed Distributed Minio Cluster On CentOs-7

# MINIO Documentation

What is Minio ?

Minio is a self-hosted AWS S3 drop-in replacement/AWS S3-compatible, object storage server written in Go. It can be used to store objects such as photos, videos,log files, backups, etc .

Installation Of Distributed Minio on CentOs – 7

We Create 4 Node Distributed Minio Cluster On CentOs -7

Prerequisites :
      
      4 nodes with Centos Installation
      Hostname set for each node in Some format like Following and Do

      Entry in /etc/hosts file :
              # vi /etc/hosts
                192.168.xx.xx mnode1
                192.168.xx.xx mnode2
                192.168.xx.xx mnode3
                192.168.xx.xx mnode4
                
In this Installation, we’ll install Minio to /usr/local/bin, and will configure it to run
as a systemd service. Because Minio is in Binary file

# Step 1: Download Minio

Download Minio binary

          # wget https://dl.min.io/server/minio/release/darwin-amd64/minio
          # chmod +x minio
          # mv minio /usr/local/bin

You can Check Now Minio Version Using Following Command
          # minio version
          
 # Step 2: Run Distributed Minio

To start a distributed MinIO instance, you just need to pass drive locations as parameters to the minio server command. Then, you’ll need to run the same command on all the participating nodes.

Notes :

        • All the nodes running distributed MinIO need to have same access key and
        secret key for the nodes to connect. To achieve this, it is mandatory to
        export access key and secret key as environment
        variables, MINIO_ACCESS_KEY and MINIO_SECRET_KEY, on all the
        nodes before executing MinIO server command.
        • All the nodes running distributed MinIO setup are recommended to be in
        homogeneous environment, i.e. same operating system, same number of
        disks and same network interconnects.
        • MinIO distributed mode requires fresh directories. If required, the drives can
        be shared with other applications. You can do this by using a sub-directory
        exclusive to MinIO. For example, if you have mounted your volume
        under /export, pass /export/data as arguments to MinIO server.
        • The IP addresses and drive paths below are for demonstration purposes only,
        you need to replace these with the actual IP addresses and drive
        paths/folders.
        • Servers running distributed MinIO instances should be less than 15 minutes
        apart. You can enable NTP service as a best practice to ensure same times
        across servers.
        • Running Distributed MinIO on Windows operating system is experimental.
        Please proceed with caution.
        • MINIO_DOMAIN environment variable should be defined and exported if
        domain is needed to be set.
        
    
Example 1: Start distributed MinIO instance on 32 nodes with 32 drives each
mounted at /export1 to /export32 (pictured below), by running this command on all
the 32 nodes:



# Step 3: Prepare Object Storage Disk

After Minio is downloaded, let’s prepare a block device that we’ll use to store
objects. The path used can just be a directory inside your file system root.

For convenience and reliability, I’m using a secondary disk in my server.

I’m Assume that You have two Partition on each node with same size and Mount
on same path.

Example: 2 Disks like /dev/sdb1 mount on /shivam/data1  /dev/sdb2 mount on /shivam/data2


# Step 4 : Managing Minio Service With Systemd

For running a system with systemd create a user and group for running Minio service.

        # sudo groupadd --system minio
        # sudo useradd -s /sbin/nologin --system -g minio minio

Give minio user ownership for the /shivam directory.

        # sudo chown -R minio:minio /shivam

Create a Systemd service unit file for minio

         # vim /etc/systemd/system/minio.service
Add below contents to the file
----------------------------------------------------------------------------------------------------
                          [Unit]
                          Description=MinIO
                          Documentation=https://docs.min.io
                          Wants=network-online.target
                          After=network-online.target
                          AssertFileIsExecutable=/usr/local/bin/minio
                          [Service]
                          WorkingDirectory=/shivam
                          User=minio
                          Group=minio
                          EnvironmentFile=-/etc/default/minio
                          ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo
                          \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
                          ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_ACCESS_KEY}\" ]; then echo
                          \"Variable MINIO_ACCESS_KEY not set in /etc/default/minio\"; exit 1; fi"
                          ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_SECRET_KEY}\" ]; then echo
                          \"Variable MINIO_SECRET_KEY not set in /etc/default/minio\"; exit 1; fi"
                          ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
                          # Let systemd restart this service always
                          Restart=always# Specifies the maximum file descriptor number that can be opened by this
                          process
                          LimitNOFILE=65536
                          # Disable timeout logic and wait until process is stopped
                          TimeoutStopSec=infinity
                          SendSIGKILL=no
                          [Install]
                          WantedBy=multi-user.target
                          # Built for ${project.name}-${project.version} (${project.name})
--------------------------------------------------------------------------------------------------


Create minio environment file and put the following contents :

      # vi /etc/default/minio

--------------------------------------------------------------------------------------------------

          # Volume to be used for Minio server.
          MINIO_VOLUMES=http://mnode{1...4}/shivam/data{1...2}
          # Use if you want to run Minio on a custom port.
          MINIO_OPTS="--address :9000"
          # Access Key of the server.
          MINIO_ACCESS_KEY=shivam
          # Secret key of the server.
          MINIO_SECRET_KEY=shivamdeshpande
----------------------------------------------------------------------------------------------------


# Step 4: Add Port 9000 in firewalld and Restart minio Service.
            # firewall-cmd –permanent –add-port=9000/tcp
            # firewall-cmd --reload
            # systemctl daemon-reload
            # systemctl enable minio
            # systemctl restart minio
            # systemctl status minio

References :

https://docs.min.io/docs/distributed-minio-quickstart-guide.html

Minio Client :    https://docs.min.io/docs/minio-client-quickstart-guide.html

Minio with s3cmd :  https://docs.min.io/docs/s3cmd-with-minio.html
