<h1>Endlessh_Fail2ban_HoneyTrap</h1>



<h2>Description of the project</h2>

This project is a filter for Fail2ban to ban anyone accessing a working Endlessh-Server!<br>




<h2>Project status</h2>

The project itself is complete and ready for use. 



<h2>Requirements</h2>

Requirements for running the webui are:

- Linux-Distro <br>

- Running Endlessh Server with Logging to a logfile (https://github.com/skeeto/endlessh) <br>

- Running Fail2ban-Service (https://github.com/fail2ban/fail2ban)<br>


<h2>Installation</h2>

1. Install endleesh. <br>
   For Debian/Ubuntu Distros open Console and run <code>sudo apt install endlessh</code> <br>
   For other Distros look here (https://github.com/skeeto/endlessh) <br>
2. Configure endleesh <br>
   Run <code>sudo editor /etc/endleesh/config</code> <br>
   Add  
   ```
    Port 2222

    Delay 10000

    MaxLineLength 32

    MaxClients 4096

    LogLevel 2

    BindFamily 0
    ```
3. Start Endlessh <br>
   Run <code>sudo endlessh -v >endlessh.log 2>endlessh.err</code>
   Or a service for Distros with Systemd: Run <code>sudo editor /lib/systemd/system/endlessh.service</code> <br>
   
   Add <br>
   ```
    [Unit]
    Description=Endlessh SSH Tarpit
    Documentation=man:endlessh(1)
    Requires=network-online.target
    [Service]
    Type=simple
    Restart=always
    RestartSec=30sec
    ExecStart=/usr/bin/endlessh
    KillSignal=SIGTERM
    StartLimitInterval=5min
    StartLimitBurst=4
    StandardOutput=journal
    StandardError=journal
    StandardInput=null
    PrivateTmp=true
    PrivateDevices=true
    ProtectSystem=full
    ProtectHome=true
    InaccessiblePaths=/run /var
    PrivateUsers=true
    NoNewPrivileges=true
    ConfigurationDirectory=endlessh
    ProtectKernelTunables=true
    ProtectKernelModules=true
    ProtectControlGroups=true
    MemoryDenyWriteExecute=true
    StandardOutput=file:/var/log/endlessh.log
    StandardError=file:/var/log/endlessh.err
    [Install]
    WantedBy=multi-user.target
   ```
   Run <code>sudo systemctl enable endleesh</code> <br>
   Run <code>sudo systemctl start endleesh</code> <br>
   
 4. Install fail2ban <br>
    Run <code>sudo apt install fail2ban</code> <br>
      
 5. Add to <code>sudo editor /etc/fail2ban/jail.conf</code> <br>
   ```
    [endlessh]
    enabled     = true
    maxretry    = 1
    bantime     = 2419200
    findtime    = 432000
    port        = 0:65535
    logpath     = /var/log/endlessh.log
    filter      = endlessh
    ignoreip    =
   ```
6. Add filter <code>sudo editor /etc/fail2ban/filter.d/endlessh.conf</code> <br>
   ```
    [Definition]
    failregex = ^.* ACCEPT host=::ffff:<HOST> port=.*
   ```
7. Run <code>sudo systemctl enable fail2ban</code> <br>
   Run <code>sudo systemctl start fail2ban</code> <br>

8. Test with an SSH-Client by connecting to the fake SSH-Server (Caution you might ban yourself). <br>
   Add ignoreip in <code>/etc/fail2ban/jail.conf</code> 


<h2>Additional notes:</h2>

- If you want to run the endlessh Server on Port 22 run <code> echo 'net.ipv4.ip_unprivileged_port_start=0' > /etc/sysctl.d/50-unprivileged-ports.conf
and sysctl --system</code>





