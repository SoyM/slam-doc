# linux

## swap

https://help.ubuntu.com/community/SwapFaq

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/installation_guide/s2-diskpartrecommend-ppc#id4394007

## disk io speed
dd count=1000 bs=1M if=/dev/zero of=./test.img

## Enable sudo without password 
`sudo visudo`

At the end of the /etc/sudoers file add this line:

All command
  `<username>  ALL=(ALL) NOPASSWD:ALL`

Custom command
  `<username> ALL=(ALL) NOPASSWD:/usr/bin/nmcli,/bin/cat,/bin/systemctl`
