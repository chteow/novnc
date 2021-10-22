# Remote Desktop Over The Internet

sudo apt install tigervnc-standalone-server tigervnc-xorg-extension tigervnc-viewer

Go to /usr/local/share folder and clone here a GitHub repository with noVNC:
sudo git clone git://github.com/kanaka/noVNC


# vncserver@:1.service
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
User=<user>

ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/usr/bin/vncserver -geometry 1280x720 -localhost %i
ExecStop=/usr/bin/vncserver -kill %i

[Install]
WantedBy=multi-user.target


# novnc.service
[Unit]
Description=Remote desktop service (noVNC)
Requires=vncserver@:1.service
After=vncserver@:1.service


[Service]
Type=simple
User=chteow
WorkingDirectory=/home/<user>
ExecStart=/usr/local/share/noVNC/utils/novnc_proxy --vnc localhost:5901 --listen 6080

[Install]
WantedBy=multi-user.target


# Nginx conf

location /websockify {
       proxy_http_version 1.1;
       proxy_pass http://127.0.0.1:6080/;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_read_timeout 61s;
       proxy_buffering off;
       add_header X-Frame-Options SAMEORIGIN;
}

location /vncws {
        index vnc.html;
        alias /usr/local/share/noVNC/;
        try_files $uri $uri/ /vnc.html;
}

