---
layout: post
title: Why Self-Hosting?
---

# Self-Hosting 
Last Fall, I started with a question. How hard would it be to self-host an open-source password manager? This question was quickly followed by another, how do I do this securely?

There are many passwords managers. I choose to focus on Vaultwarden and Passbolt. These were some of the first that I found. I looked at some others but these two caught my attention because of their differing origins and implementations.

My seemingly simple project took longer than I expected. I started by reading reviews, associated documentation, and installation requirements. I also learned that projects and scope can grow quickly when presented with interesting topics.

I found that Passbolt has installation instructions for various operating systems. I also found that an NTP service and SMTP service were required. Additionally, the docker setup seemed to be less refined than Vaultwarden. Passbolt contains a much larger code base and feature list.

I decided to spend more of my time focused on Vaultwarden. After setting up Vaultwarden I started poking around with some testing. My interest soon shifted. I had to use a self-signed certificate to enable HTTPS. It's not a big deal. Many locally-hosted services will use self-signed certificates to enable HTTPS. Although, I was starting to get annoyed with the warnings from browsers.

I decided to switch gears and build my own local certificate authority. I got to a stopping point in my password manager research. Then I brushed up on my PKI knowledge, found some of my short falls, and started learning more. I used Easy-RSA to build out a root and intermediate CA. 

I then went on a tangent and learned about mDNS, when I was looking up possible local top-level domains I could use.

## Conclusion
What I'm trying to say is that there is a lot to learn. Small projects can become extended periods of learning when you take interest in what you do. Many applications relay on other technologies. Taking time to self-host services can help you see how systems work together and expose you to aspects of infrastructure, development, data management, and security. Rushing a self-hosted project can leave a service vulnerable and limit personal growth. Even a simply project to test available services will help you learn a lot.
