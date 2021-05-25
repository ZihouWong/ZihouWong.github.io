---

title: "linux-iptables"
date: 2017-8-20
categories: Linux iptables Mac

layout: post
comments: true

---


	iptables -t nat -A PREROUTING -p tcp --dport 3306 -j REDIRECT --to-ports 3307