After first development deploy I had endpoints for:
  - admin-client
  - dealers-service
  - identity-service
  - notifications-service
  - statistics-service
  - user-client
  - watchdog-client

And everything seemed to be there and working fine.

But on the first production deploy I ran into QUOTA_EXCEEDED exception because gcp has 'IN_USE_ADDRESSES' Quota: 8.

So from gcp's 'Network services -> Load balancing -> Advanced menu' I choose to remove the Forwarding rules for:
- dealers-service
- identity-service
- notifications-service
- statistics-service

Now I have:

- http://35.232.70.23/ - user-client (development)

- http://34.121.218.13:5000/ - admin-client (development)

- http://34.122.250.206:5500/healthchecks - watchdog-client (development)

- http://104.197.185.80/ - user-client (production)

- http://35.223.12.85:5000/ - admin-client (production)

- http://34.69.213.76:5500/healthchecks - watchdog-client (production)

But there are some features missing... That was the price... :D
