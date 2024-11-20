# Setting up fail2ban with nginx proxy manager running via docker

## 1. install fail2ban

  ```sh
  sudo apt install fail2ban
  ```

## 2. make a copy of the jail config to edit

  ```sh
  sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
  ```

  edit your preferred defaults in here. e.g. bantime, ignoreip - **Remember to add remote ips like your inernet office, home, etc**

## 3. mount your log folder outside of nginx proxy manager

  ```yaml
      volumes:
      - /path/to/logs:/data/logs
  ```

## 4. create `/etc/fail2ban/filter.d/npm.conf`

  ```conf
  [INCLUDES]

  [Definition]

  failregex = ^<HOST>.+" (4\d\d|3\d\d) (\d\d\d|\d) .+$
              ^.+ 4\d\d \d\d\d - .+ \[Client <HOST>\] \[Length .+\] ".+" .+$
  ```

## 5. create `/etc/fail2ban/action.d/docker-action.conf`

  ```conf
  #https://www.the-lazy-dev.com/en/install-fail2ban-with-docker/
  [Definition]

  actionstart = iptables -N f2b-npm-docker
                iptables -A f2b-npm-docker -j RETURN
                iptables -I FORWARD -p tcp -m multiport --dports 0:65535 -j f2b-npm-docker

  actionstop = iptables -D FORWARD -p tcp -m multiport --dports 0:65535 -j f2b-npm-docker
               iptables -F f2b-npm-docker
               iptables -X f2b-npm-docker

  actioncheck = iptables -n -L FORWARD | grep -q 'f2b-npm-docker[ \t]'

  actionban = iptables -I f2b-npm-docker -s <ip> -j DROP

  actionunban = iptables -D f2b-npm-docker -s <ip> -j DROP
  ```

## 6. create `/etc/fail2ban/jail.d/npm.local`

  ```conf
  [npm]
  enabled = true
  chain=INPUT
  maxretry = 3
  bantime = 48h
  findtime = 60m
  logpath = /path/to/logs/default-host_*.log
            /path/to/logs/proxy-host-*.log
  action = docker-action
  ```
