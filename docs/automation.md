There are a couple different methods for automating backups you can choose to use one/both or none its up to you.

## Backup on boot
1. Create the service file in systemd
```shell 
sudo nano /etc/systemd/system/klipper-backup-on-boot.service
```  
2. Copy and paste the below text (be sure to uncomment the correct `after=` and `wants=` lines based on if your linux install is using network manager)  
```shell
[Unit]
Description=Klipper Backup On-boot Service
#Uncomment below lines if using network manager
#After=NetworkManager-wait-online.service
#Wants=NetworkManager-wait-online.service
#Uncomment below lines if not using network manager
#After=network-online.target
#Wants=network-online.target

[Service]
User={replace with your username}
Type=oneshot
ExecStart=/bin/bash -c 'bash $HOME/klipper-backup/script.sh "New Backup on boot $(date +%%D)"'

[Install]
WantedBy=default.target
```
3. Reload the service daemon, enable the service and start the service
```
sudo systemctl daemon-reload
sudo systemctl enable klipper-backup-on-boot.service
sudo systemctl start klipper-backup-on-boot.service
```

## Cron
You can also use cron jobs for scheduling backups.
!!! note
    While there is a `@reboot` option for cron jobs, creating the service from [above](#backup-on-boot) is a better approach for backups on boot. The service can be set to wait for your network to be online before attemping a backup where the cronjob will not and you will need to add a sleep command based on how long it takes for the network to become available.  

1. Edit the crontab file using:
```
crontab -e
```  

    !!! info
        if it's your first time running the command you will be asked which editor you would like to use, nano is the easiest  

2. Once the editor is open, at the very bottom of the file add the line:
```
0 */4 * * * $HOME/klipper-backup/script.sh
```
This tells cron to run the backup script every 4 hours. You can find other cron examples here: <a href="https://crontab.guru/examples.html" target="_blank">https://crontab.guru/examples.html</a>

## Backup on file changes
!!! warning "Important Info"
    The following service relies on the inotify-tools package. 
    
    To install the package run ```sudo apt-get install inotify-tools``` in your terminal.
2. Create the service file in systemd
```shell 
sudo nano /etc/systemd/system/klipper-backup-filewatch.service
```  
3. Copy and paste the below text (be sure to uncomment the correct `after=` and `wants=` lines based on if your linux install is using network manager)  
```shell
[Unit]
Description=Klipper Backup Filewatch Service
#Uncomment below lines if using network manager
#After=NetworkManager-wait-online.service
#Wants=NetworkManager-wait-online.service
#Uncomment below lines if not using network manager
#After=network-online.target
#Wants=network-online.target

[Service]
User={replace with your username}
Type=simple
ExecStart=/bin/bash -c '\
    exclude_pattern=".swp|.tmp|printer-[0-9]*_[0-9]*.cfg|.bak|.bkp"; \
    inotifywait -mrPq -e close_write -e move -e delete --exclude "$exclude_pattern" $HOME/printer_data/config/ | \
    while read -r path event file; do \
        echo "Event Type: $event, Watched Path: $path, File Name: $file"; \
        bash -c '\''bash $HOME/klipper-backup/script.sh "Files modified - $(date +\"%%x - %%X\")"'\'' > /dev/null 2>&1; \
    done'

[Install]
WantedBy=default.target
```
4. Reload the service daemon, enable the service and start the service
```
sudo systemctl daemon-reload
sudo systemctl enable klipper-backup-filewatch.service
sudo systemctl start klipper-backup-filewatch.service
```
5. When making large edits where you might want to stop the service you can go into your service manager and stop the service below is an example of its location in fluidd. 
![fluidd-service-manager](https://i.imgur.com/mQgVY7J.gif)