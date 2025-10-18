Local Mail Server Setup Guide (Ubuntu)

#### 1. Network and Hostname Configuration
**Hosts file (/etc/hosts):**
127.0.0.1   localhost
192.168.127.137   mailserver.bitbank.local   mailserver
192.168.127.138   client1.bitbank.local      client1
192.168.127.139   client2.bitbank.local      client2

**Set hostname on each machine:**
hostnamectl set-hostname mailserver.bitbank.local   # Mail Server
hostnamectl set-hostname client1.bitbank.local      # Client1
hostnamectl set-hostname client2.bitbank.local      # Client2

**Test connectivity:**
ping mailserver
ping client1
ping client2

#### 2. Install Mail Server Components on Mail Server
sudo apt update
sudo apt install postfix dovecot-imapd dovecot-pop3d mailutils -y

**Postfix:** configure for "Internet Site" with system mail name: `bitbank.local`

**Dovecot:** 
vi /etc/dovecot/conf.d/10-mail.conf
mail_location = maildir:~/Maildir
sudo systemctl restart dovecot

#### 3. Create Users
adduser user1
adduser user2

**Create Maildir for each user:**
sudo mkdir -p /home/user1/Maildir/{cur,new,tmp}
sudo mkdir -p /home/user2/Maildir/{cur,new,tmp}
sudo chown -R user1:user1 /home/user1/Maildir
sudo chown -R user2:user2 /home/user2/Maildir
sudo chmod -R 700 /home/user1/Maildir
sudo chmod -R 700 /home/user2/Maildir

#### 4. Test Local Email
From Mail Server or any client with `mailutils`:
echo "Test message" | mail -s "Hello" user2
Check email in Maildir:
ls /home/user2/Maildir/new

#### 5. Optional: Roundcube Webmail
Install Apache + PHP + Roundcube:
sudo apt install apache2 php php-mysql php-intl php-mbstring php-xml php-gd php-curl roundcube roundcube-core roundcube-mysql -y
Edit Roundcube configuration `/etc/roundcube/config.inc.php`:

$config['mail_domain'] = 'bitbank.local';
$config['imap_host'] = ['localhost:143'];
$config['smtp_host'] = 'localhost:587';
$config['smtp_user'] = '%u';
$config['smtp_pass'] = '%p';

Restart Apache:
sudo systemctl restart apache2
Access Roundcube at: `http://mailserver.bitbank.local/roundcube`

#### 6. Add New Users
sudo adduser user3
sudo mkdir -p /home/user3/Maildir/{cur,new,tmp}
sudo chown -R user3:user3 /home/user3/Maildir
sudo chmod -R 700 /home/user3/Maildir

Test email delivery and login with Roundcube.

✅ **Notes:**

* Postfix delivers to Maildir (`~/Maildir`) for Dovecot.
* Roundcube reads emails via IMAP through Dovecot.
* Permissions are critical: ensure Maildir ownership and mode are correct.
* For multiple users, repeat user creation and Maildir setup steps.
