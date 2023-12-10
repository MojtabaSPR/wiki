# User & Group

- Group Commands

    List all groups:

    ```bash
     ~: cat /etc/group
    ```

    List a group members:

    ```bash
     ~: getent group g1

     # /etc/group also shows group members
     ~: cat /etc/group
     ~: cat /etc/group | grep group1 
    ```

    Create group:

    ```bash
    ~: groupadd groupname
    ```

    Remove group:

    ```bash
    ~: groupdel groupname
    ```

- Users Commands

    Print current user name:

    ```bash
    ~: users
    ~: whoami
    ~: id -un
    ```

    List all users info:

    ```bash
    ~: cat /etc/passwd
    ~: getent passwd
    ```

    List all group membership of a user:

    ```bash
     ~: groups user1
     ~: id user1
    ```

    List users logged in:

    ```bash
    ~: who
    ~: who -a
    ~: who -u
    ```

    Modify user:

    ```bash
    # Add a User to a Group
    ~: usermod -a -G GROUP USER
    ~: usermod -aG docker mojtaba

    # Remove a user from a specific group
    ~: gpasswd -d username groupname
    ~: deluser username groupname

    # Change User Primary Group
    ~: usermod -g GROUP USER

    # Change user password file comment field
    ~: usermod -c "GECOS Comment" USER

    # Changing a User Home Directory
    ~: usermod -d DIR USER

    # Changing a User Name
    ~: usermod -l NEW_USER USER

    # Locking and Unlocking a User Account
    ~: usermod -L USER
    ~: usermod -U USER
    ```

    Create user:

    ```bash
    # When executed without any option, useradd creates a new user account using the default settings specified in the /etc/default/useradd file.
    # The command adds an entry to the /etc/passwd, /etc/shadow, /etc/group and /etc/gshadow files.
    ~: useradd user2
    ~: passwd user2

    # create user with home in /home directory
    ~: useradd -m user2

    # create user with home in custom directory
    ~: useradd -m -d /opt/user2 user2

    # create with specific user id
    ~: useradd -u 1500 user2

    # create user and assign multiple group
    ~: useradd -g users -G wheel,developers user2
    ```

    Remove user:

    ```bash
    ~: deluser user2
    ~: deluser --remove-home user2
    ```
