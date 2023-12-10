# Firewall

- IPtables

    To see if iptables is actually running, we can check the iptables modules are loaded and use the -L switch to inspect the currently loaded rules:

    ```bash
    ~: lsmod | grep  ip_tables
    ~: iptables -L
    ```

- Delete Rule

    Get iptables rules line number:

    ```bash
    ~: iptables -L --line-numbers
    ```

    Delete specific rule by rule number:

    ```bash
    ~: iptables -D INPUT 3
    ```

- Remove ufw chains

    This is a bash script that will remove all ufw chains:

    ```bash
    #!/bin/bash
    for ufw in `iptables -L | grep 'Chain ufw' | awk '{ print $2 }'`; do iptables -F $ufw; done
    for ufw in `iptables -L | grep 'Chain ufw' | awk '{ print $2 }'`; do
    iptables -D INPUT -j $ufw
    iptables -D FORWARD -j $ufw
    iptables -D OUTPUT -j $ufw
    done
    for ufw in `iptables -L | grep 'Chain ufw' | awk '{ print $2 }'`; do iptables -X $ufw; done
    ```
