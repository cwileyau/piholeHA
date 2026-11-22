# piholeHA

INSTALL

    sudo apt update
    sudo apt install -y keepalived

ENABLE PIHOLE on virtual IP

    echo "net.ipv4.ip_nonlocal_bind=1" | sudo tee -a /etc/sysctl.d/99-keepalived.conf
    sudo sysctl --system

MASTER

EDIT CONF

    sudo nano /etc/keepalived/keepalived.conf

CONF Contents

    vrrp_script check_pihole {
        # Verify Pi-hole DNS service is answering queries locally
        script "dig +short +time=1 +tries=1 @127.0.0.1 google.com > /dev/null 2>&1"
        interval 2
        weight -20
    }
    
    vrrp_instance VI_PIHOLE {
        state MASTER
        interface eth0               # Replace with your interface if different (e.g., wlan0)
        virtual_router_id 51         # Must be identical on both Pis (1-255)
        priority 100                 # Higher priority = primary holder of VIP
        advert_int 1
    
        authentication {
            auth_type PASS
            auth_pass SecretPass123  # Must match on both Pis (max 8 chars)
        }
    
        virtual_ipaddress {
            192.168.1.99/24          # Your shared VIP
        }
    
        track_script {
            check_pihole
        }
    }

BACKUP

EDIT CONF
    sudo nano /etc/keepalived/keepalived.conf

CONF Contents

    vrrp_script check_pihole {
        script "dig +short +time=1 +tries=1 @127.0.0.1 google.com > /dev/null 2>&1"
        interval 2
        weight -20
    }
    
    vrrp_instance VI_PIHOLE {
        state BACKUP
        interface eth0               # Replace with your interface if different
        virtual_router_id 51         # Must match Pi-hole A
        priority 90                  # Lower priority than Master
        advert_int 1
    
        authentication {
            auth_type PASS
            auth_pass SecretPass123  # Must match Pi-hole A
        }
    
        virtual_ipaddress {
            192.168.1.99/24          # Same shared VIP
        }
    
        track_script {
            check_pihole
        }
    }


ENABLE Services

    sudo systemctl enable keepalived
    sudo systemctl restart keepalived

SHOW virt IP is running

    ip a show eth0

TEST

    sudo systemctl stop keepalived
    sudo systemctl start keepalived

