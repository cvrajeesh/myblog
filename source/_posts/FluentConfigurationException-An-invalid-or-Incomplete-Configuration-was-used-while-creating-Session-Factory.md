title: "FluentConfigurationException: An invalid or Incomplete Configuration was used while creating Session Factory"
date: 2010-01-01 11:04:13
tags:
---

I got this exception when I tried to run my Repository tests and couldn't find the exact reason why these tests failed.

All the unit tests where passing before I removed the default constructors from Domain Model, when I reverted the default constructors back, everything started working again..... !!!

Do you know the exact reason for this?

**Update:** In order to support lazy loading, NHibernate needs to create proxy objects. Those proxy classes derived from the actual domain model class itself, and they are instantiated using the default constructor. So the parent class also should have a default constructor.
