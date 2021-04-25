---
title: Acme.sh
---

### Install

    # curl https://get.acme.sh | sh

### Prepare

1. 域名配置 A 记录：ray.kidlj.com -> server_ip.
2. 配置 Nginx ray.kidlj.com server.

### Issue

    # acme.sh --issue -d ray.kidlj.com  --nginx
    ...
    -----END CERTIFICATE-----
    [Tue Jun 11 09:28:54 EDT 2019] Your cert is in  /root/.acme.sh/ray.kidlj.com/ray.kidlj.com.cer 
    [Tue Jun 11 09:28:54 EDT 2019] Your cert key is in  /root/.acme.sh/ray.kidlj.com/ray.kidlj.com.key 
    [Tue Jun 11 09:28:54 EDT 2019] The intermediate CA cert is in  /root/.acme.sh/ray.kidlj.com/ca.cer 
    [Tue Jun 11 09:28:54 EDT 2019] And the full chain certs is there:  /root/.acme.sh/ray.kidlj.com/fullchain.cer 

Note: nginx 模式只能使用 80 端口验证。

### Install certs

    # acme.sh --install-cert -d ray.kidlj.com --cert-file /etc/nginx/ssl/ray.kidlj.com.cer --key-file /etc/nginx/ssl/ray.kidlj.com.key \
    --fullchain-file /etc/nginx/ssl/fullchain.cer \ 
    --reloadcmd "systemctl reload nginx"

### Renew

    # crontab -e  
    56 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null

### Ali DNS Mode

    # export Ali_Key="sdfsdfsdfljlbjkljlkjsdfoiwje"
    # export Ali_Secret="jlsdflanljkljlfdsaklkjflsa"

    # acme.sh --issue --dns dns_ali -d ray2.kidlj.com --force