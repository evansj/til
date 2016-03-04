# Split `~/.ssh/config` into multiple files

I split out my `~/.ssh/config` into multiple files, one per client / project. I have a script which assembles them all back into `~/.ssh/config`, and (in OS X) I have this script configured to run automatically whenever one of the files changes. This is how I do it:

1. Back up your original `~/.ssh/config`!

    ```bash
    $ cp ~/.ssh/config ~/.ssh/config.bak
    ```

2. Create a new directory `~/.ssh/config.d/`

    ```bash
    $ mkdir ~/.ssh/config.d
    ```


3. Split out `Host` entries from your original config file into new files beneath `~/.ssh/config.d/`. The files are read in alphabetical order so use a name like `00-defaults` for the first file. For example:

    `~/.ssh/config.d/00-general.conf`
    ```
    Host *
        Compression no
        ForwardX11 no
        ServerAliveInterval 10
        ServerAliveCountMax 3
        ControlMaster auto
        ControlPath ~/.ssh/master-%r@%h:%p
        ControlPersist yes
    ```

    `~/.ssh/config.d/10-project-1.conf`
    ```
    Host some.example.server
        HostName ssh1.example.com
        User ec2-user
        IdentityFile ~/.ssh/ec2/project1.pem
        IdentitiesOnly yes
    ```

    `~/.ssh/config.d/20-project-2.conf`
    ```
    Host another.example.server
        HostName ssh2.example.com
        User ec2-user
        IdentityFile ~/.ssh/ec2/project2.pem
        IdentitiesOnly yes
    ```

    etc.

4. Create a file `~/.ssh/build_config.sh` containing the following:

    ```sh
    #!/bin/sh

    # cat ~/.ssh/config.d/* > ~/.ssh/config

    # cat all files, with a blank line between each file
    awk 'FNR==1{print ""}{print}' ~/.ssh/config.d/* > ~/.ssh/config
    ```

5. Make it executable

    ```bash
    $ chmod +x ~/.ssh/build_config.sh
    ```

6. Run the script to create `~/.ssh/config` (you did back up `~/.ssh/config` first, right? See step 1!)

    ```bash
    $ ~/.ssh/build_config.sh
    ```

7. On OS X, configure the script to run automatically every time you change anything in the `~/.ssh/config.d/` directory by creating a file in `~/Library/LaunchAgents/`

    `~/Library/LaunchAgents/info.evansweb.ops.ssh-config.plist`
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC -//Apple Computer//DTD PLIST 1.0//EN
    http://www.apple.com/DTDs/PropertyList-1.0.dtd>
    <plist version="1.0">
    <dict>
        <key>Label</key>
        <string>info.evansweb.ops.ssh-config</string>
        <key>ProgramArguments</key>
        <array>
            <string>/Users/YOUR_USERNAME/.ssh/build-config.sh</string>
        </array>
        <key>WatchPaths</key>
        <array>
            <string>/Users/YOUR_USERNAME/.ssh/config.d/</string>
        </array>
        <key>ThrottleInterval</key>
        <integer>1</integer>
        <key>ProcessType</key>
        <string>Background</string>
    </dict>
    </plist>
    ```

    **Change _YOUR_USERNAME_ to your correct username. You can't use `~` to refer to your home directory in this file.**

8. Load the Launch Daemon with the following command:

    ```bash
    launchctl load ~/Library/LaunchAgents/info.evansweb.ops.ssh-config.plist
    ```

Now, your `~/.ssh/config.d/` will be monitored and the script will run automatically if you add, delete or modify any files. Monitoring will also continue automatically after a reboot. If you want to disable monitoring, execute this:

```bash
launchctl unload ~/Library/LaunchAgents/info.evansweb.ops.ssh-config.plist
```
