# Suspending and Waking of Plex Media Server on Ubuntu 22.04

## The following instructions will show how to setup the following:
* When nothing is playing on the Plex Media Server, the system will suspend itself
* When a Plex client requests access to the Plex Media Server, the system will wake up

## Why do this
* Save money on your power bill; lower power consumption
* Save disk lifespan of Plex Media Server, especially if using SSDs

## Requirements
* "Backend" system
  * Ubuntu 22.04 Server installed
  * Wake-on-LAN (WOL) enabled - usually via BIOS
  * Plex Media SErver installed
* "Frontend" system
  * Ubuntu 22.04 Server installed
  * Low power consumption

## Frontend setup
1. Install Ubuntu 22.04 Server
2. For our example, set IP to 10.0.0.1
3. `apt install wakeonlan`
4. Create script `/usr/local/bin/iptable_rules.sh`:
   ```
   #!/bin/bash -e

   IPTABLES="/sbin/iptables"
   BACKEND_IP="10.0.0.2"

   # start from scratch
   ${IPTABLES} --flush
   ${IPTABLES} -t nat --flush

   # log when forwarded ports are newly hit
   ${IPTABLES} -t nat -A PREROUTING -p tcp --match limit --limit 1/minute --limit-burst 1 --match state --state NEW --match multiport --dports 32400,3005,8324,32469 -j LOG --log-prefix='WOL_TRIGGER '

   # forward ports to plex server
   ${IPTABLES} -t nat -A PREROUTING -p tcp --match multiport --dports 32400,3005,8324,32469 -j DNAT --to-destination ${BACKEND_IP}
   ${IPTABLES} -t nat -A PREROUTING -p udp --match multiport --dports 1900,5353,32410,32412,32413,32414 -j DNAT --to-destination ${BACKEND_IP}
   ${IPTABLES} -t nat -A POSTROUTING ! -s 127.0.0.1 -j MASQUERADE
   ```
5. Add following line to `/etc/crontab`:
   ```
   @reboot	root	/usr/local/bin/iptables_rules.sh
   ```
6. Create script `/usr/local/bin/ping_wol.sh` changing `IP` and `MAC` variables:
   ```
   #!/bin/bash

   # If an IP is not pingable, it will send a wakeonlan to MAC.

   IP="10.0.0.2"
   MAC="00:11:22:33:44:55"

   if ! ping -c 1 -W 1 ${IP} > /dev/null; then
     logger "$0 - sending wakeonlan to ${MAC}"
     wakeonlan ${MAC} > /dev/null
   fi
   ```
7. Create `/etc/rsyslog.d/99-send-wol.conf`:
   ```
   :msg,contains,"WOL_TRIGGER" ^/usr/local/bin/ping_wol.sh
   ```

## Backend setup
1. Install Ubuntu 22.04 Server
2. For our example, set IP to 10.0.0.2
3. Install "Plex Media Server"
4. Create script `/usr/local/bin/iptable_rules.sh`:
   ```
   #!/bin/bash -e

   IPTABLES="/sbin/iptables"
   FRONTEND_IP="10.0.0.1"

   # start from scratch
   ${IPTABLES} --flush
   ${IPTABLES} -t nat --flush

   # only allow plexfront access to plex data
   # '-i eno1' is needed else it will block 127.0.0.1 connections
   ${IPTABLES} -A INPUT -p tcp -i eno1 ! -s ${FRONTEND_IP} --match multiport --dports 32400,3005,8324,32469 -j REJECT
   ${IPTABLES} -A INPUT -p udp -i eno1 ! -s ${FRONTEND_IP} --match multiport --dports 1900,5353,32410,32412,32413,32414 -j REJECT
   ```
5. Add following line to `/etc/crontab`:
   ```
   @reboot	root	/usr/local/bin/iptables_rules.sh
   ```
6. Create script `/usr/local/bin/plexserver_suspend.sh`
   ```
   #!/bin/bash -e

   # After MAX_SECONDS of nothing playing, suspend system

   MAX_SECONDS=900 # 15 minutes
   COUNT_FILE="/tmp/no_plex_playing"

   # Obtain 'PlexOnlineToken' value from "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Preferences.xml"
   PLEX_TOKEN="xxxxxxxxxxxxxxxxxxxx"

   # Obtain number of videos currently playing
   COUNT=$(curl -s -H "accept: application/json"  http://localhost:32400/status/sessions?X-Plex-Token=${PLEX_TOKEN} | jq .MediaContainer.size)

   # If something is playing remove count file
   if (( COUNT > 0 )); then
     logger "$0: $COUNT items playing"
     rm -f ${COUNT_FILE}
   else
     if [ -f ${COUNT_FILE} ]; then
       # check how old the file is
       create_time=$(stat -c"%W" ${COUNT_FILE})
       now=$(date +%s)
       difference=$(( now - create_time ))
       logger "$0: nothing playing for ${difference}/${MAX_SECONDS} seconds"
       if (( difference > MAX_SECONDS )); then
         (sleep 5 ; systemctl suspend) &
         rm -f ${COUNT_FILE}
         logger "$0: suspending system"
       fi
     else
       logger "$0: nothing currently playing"
       touch ${COUNT_FILE}
     fi
   fi
   ```
7. Add following to `/etc/crontab`:
   ```
   * *	* * *	root	/usr/local/bin/plexserver_suspend.sh
   ```
