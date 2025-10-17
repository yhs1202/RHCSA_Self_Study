# Day1

- ## env setting
    Rocky Linux 9.5, VMWare 에서 진행

- ## dnf
    > Apache HTTPD 서버를 설치하고, `/var/www/html/index.html` 파일을 만들어 클라이언트에서 접근 가능하도록 설정하시오.

    ```bash
    sudo dnf install httpd -y
    sudo systemctl enable --now httpd
    echo "Hello RHCSA" | sudo tee /var/www/html/index.html   # or use vi editor
    # --permanent -> keep option after reboot, --add-service=http -> enable http (port 80) rule
    sudo firewall-cmd --permanent --add-service=http
    # --reload
    sudo firewall-cmd --reload

    # Verification
    systemctl status httpd
    curl http://localhost
    firewall-cmd --list-all

    ---
    httpd.service - The Apache HTTP Server
    Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
    Active: active (running) since Thu 2025-10-16 19:31:12 KST; 10s ago
    Main PID: 1234 (httpd)
    ---
    Hello RHCSA
    ---
    public (active)
    services: http https ssh

    ```

    1. dnf 하기 위해 VMWare 에서 iso 파일 불러오고 mount
        
        ```bash
        # iso check
        lsblk
        
        ---
        NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
        sr0          11:0    1 10.7G  0 rom  /run/media/hsyoon_linux/Rocky-9-5-x86_64-dvd
        nvme0n1     259:0    0  120G  0 disk 
        ├─nvme0n1p1 259:1    0    1G  0 part /boot
        └─nvme0n1p2 259:2    0  119G  0 part 
        ├─rl-root 253:0    0   70G  0 lvm  /
        ├─rl-swap 253:1    0  7.9G  0 lvm  [SWAP]
        └─rl-home 253:2    0 41.1G  0 lvm  /home
        ```
        
        ```bash
        # mount
        sudo mount /dev/cdrom /mnt
        
        ---
        mount: /mnt: WARNING: source write-protected, mounted read-only.
        
        # set local repo
        sudo tee /etc/yum.repos.d/local.repo <<EOF
        [BaseOS]
        name=RHEL9 BaseOS
        baseurl=file:///mnt/BaseOS
        enabled=1
        gpgcheck=0
        
        [AppStream]
        name=RHEL9 AppStream
        baseurl=file:///mnt/AppStream
        enabled=1
        gpgcheck=0
        EOF
        
        # clean cache
        sudo dnf clean all
        sudo dnf repolist
        ```
        
    2. dnf 해도 로컬 repo 로 못잡고 온라인 repo로 잡아서 네트워크 에러 발생.
        
        ```bash
        [hsyoon_linux@localhost ~]$ sudo dnf install httpd -y
        
        ---
        Rocky9 BaseOS                                                                                                                57 MB/s | 2.3 MB     00:00    
        Rocky9 AppStream                                                                                                             70 MB/s | 8.3 MB     00:00    
        Rocky Linux 9 - BaseOS                                                                                                      0.0  B/s |   0  B     00:00    
        Errors during downloading metadata for repository 'baseos':
        - Curl error (6): Couldn't resolve host name for https://mirrors.rockylinux.org/mirrorlist?arch=x86_64&repo=BaseOS-9 [Could not resolve host: mirrors.rockylinux.org]
        Error: Failed to download metadata for repo 'baseos': Cannot prepare internal mirrorlist: Curl error (6): Couldn't resolve host name for https://mirrors.rockylinux.org/mirrorlist?arch=x86_64&repo=BaseOS-9 [Could not resolve host: mirrors.rockylinux.org]
        ```
        
    3. 온라인 repo 비활성화
        
        ```bash
        sudo dnf config-manager --set-disabled baseos appstream extras
        ```
        
    4. Enable httpd (Working)
        
        ```bash
        [hsyoon_linux@localhost ~]$ sudo dnf install httpd -y
        ```

- ## nmcli, nmtui

    > 시스템의 ens160 인터페이스에 대해 다음 조건을 설정하시오.
    >
    > IP 주소: 192.168.100.50/24
    >
    > 게이트웨이: 192.168.100.1
    >
    > DNS: 8.8.8.8
    >
    > 부팅 시 자동 활성화

    ```bash
    # nmtui
    sudo nmtui
    “Edit a connection” → ens160 선택

    IPv4 Configuration → Manual 선택

    Address / Gateway / DNS 입력

    “Automatically connect” 체크 후 Save -> Back -> Active a Connection

    # Verification
    sudo nmcli con show ens160
    ip addr show ens160
    ping -c 3 8.8.8.8

    ---
    autoconnect=true, method=manual check
    ```

    ```bash
    # nmcli version
    # check current connection
    sudo nmcli con show

    ---
    NAME    UUID                                  TYPE      DEVICE
    ens33   a1b2c3d4-e5f6-7890-1112-131415161718  ethernet  ens33

    # setup IPv4
    sudo nmcli con mod ens33 ipv4.method manual
    sudo nmcli con mod ens33 ipv4.addresses 192.168.122.50/24
    sudo nmcli con mod ens33 ipv4.gateway 192.168.122.1
    sudo nmcli con mod ens33 ipv4.dns 8.8.8.8
    ---

    # set autoconnect
    sudo nmcli con mod ens33 connection.autoconnect yes

    ---

    # reset connection
    sudo nmcli con down ens33
    sudo nmcli con up ens33

    ---

    # check conditions
    sudo nmcli con show ens33 | egrep 'ipv4|autoconnect'
    ---
    ipv4.method:                            manual
    ipv4.addresses:                         192.168.122.50/24
    ipv4.gateway:                           192.168.122.1
    ipv4.dns:                               8.8.8.8
    connection.autoconnect:                 yes

    ---
    # reboot and Verification
    sudo reboot
    ip addr show ens33
    ping -c 3 8.8.8.8
    ```

- ## ps

    ```bash
    # ps aux (BSD Style)
    ps aux
    ps aux --sort=-%mem | head -n 2   # memory descending order -> top 2
    ps aux --sort=-%cpu           # cpu descending order

    # ps -ef (SysV Style) "every" "**f**ormat"
    ps -ef

    ps -ef | grep httpd

    # check PID
    pgrep httpd

    # ps -fp (detail based on PID)
    ps -fp $(pgrep ngnix)

    # ps -u (user)
    ps -u student
    ps -ef | grep student
    ```