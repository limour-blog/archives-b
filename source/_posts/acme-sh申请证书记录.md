---
title: ACME.SH申请证书记录
tags: []
id: '2061'
categories:
  - - uncategorized
date: 2022-10-20 10:45:09
---

*   curl https://get.acme.sh sh
*   alias acme.sh="/root/.acme.sh/acme.sh"
*   acme.sh --upgrade --auto-upgrade
*   export CF\_Key="babababababala"
*   export CF\_Email="youremail@mail.com"
*   acme.sh --register-account -m limour@limour.top
*   acme.sh --issue --dns dns\_cf -d quic.j11.fun --server https://acme-v02.api.letsencrypt.org/directory
*   acme.sh --installcert -d quic.j11.fun --fullchain-file /root/quic/1.pem --key-file /root/quic/1.key