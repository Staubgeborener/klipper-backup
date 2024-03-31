## Moonraker update manager
To add this repository to moonraker's update manager for easy updating, you need to add the following into ```moonraker.conf```
commits.  
    
!!! warning "Important Note"
    This is just for these repositories updates, not updates of your config backups. Those do not get added to moonraker.

```yaml
[update_manager klipper-backup]
type: git_repo
path: ~/klipper-backup
origin: https://github.com/Staubgeborener/klipper-backup.git
managed_services: moonraker
primary_branch: main
```

## Update via shell/console

If the script is executed (e.g. with `./script.sh`) and a new update is available, you will be notified with the following message
> NEW klipper-backup version available!

To pull the update navigate to the klipper-backup folder and run `git pull`:
```shell
cd ~/klipper-backup && git pull
```

!!! warning "Important Note" 
    Editing most files (with the exception of .env) inside of the `klipper-backup` folder, can lead to a ['dirty' repository](https://docs.mainsail.xyz/setup/updates/update-manager#dirty){:target="_blank"}, which will cause moonraker or the local git repository to be unable to pull updates until resolved. For Pull requests see [pull request](https://github.com/Staubgeborener/klipper-backup/pulls){:target="_blank"} or you are welcome to open a [feature request](https://github.com/Staubgeborener/klipper-backup/issues){:target="_blank"}.

There are different ways to start your backup, either [manually](manual.md) or [automatically](automation.md).