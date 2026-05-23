After registering a new account and logging in, the assigned profile ID was:

```txt id="z4r8mv"
/profile/67
```

The application exposed user profiles through predictable numeric IDs:

```txt id="t7m2qp"
/profile/5
```

This indicated a possible Insecure Direct Object Reference (IDOR) vulnerability.

By modifying the profile ID manually and visiting:

```txt id="w1k9cx"
/profile/66
```

the flag belonging to admin user was revealed.


