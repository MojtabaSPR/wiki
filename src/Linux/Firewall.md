# Firewall


- IPtables
    
    
    **To see if iptables is actually running, we can check that the iptables modules are loaded and use the -L switch to inspect the currently loaded rules:**
    
    # lsmod | grep  ip_tables
    
    # sudo iptables -L
    
- Delete Rule
    
    
    **Get iptables rules line number:**
    
    # sudo iptables -L
    --line-numbers
    
    **Delete specific rule by rule number:**
    
    # sudo iptables -D
    INPUT 3
    
- Remove ufw chains
    
    **#This is a bash script that will remove all ufw chains:**
    
    **#From [https://gist.github.com/funkjedi/88c31179d455b9c6edb2b31b9564ede1](https://gist.github.com/funkjedi/88c31179d455b9c6edb2b31b9564ede1)**
    
    #!/bin/bash
    for ufw in `iptables -L | grep 'Chain ufw' | awk '{ print $2 }'`; do iptables -F $ufw; done
    for ufw in `iptables -L | grep 'Chain ufw' | awk '{ print $2 }'`; do
    iptables -D INPUT -j $ufw
    iptables -D FORWARD -j $ufw
    iptables -D OUTPUT -j $ufw
    done
    for ufw in `iptables -L | grep 'Chain ufw' | awk '{ print $2 }'`; do iptables -X $ufw; done