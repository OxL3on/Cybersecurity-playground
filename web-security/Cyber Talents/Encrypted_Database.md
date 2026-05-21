## [Encrypted Database](https://cybertalents.com/challenges/web/encrypted-database)

Try to go to /admin. It gives 403 error. So we can use bypass tools like: 4-ZERO-3 or byp4xx

- **Installation of byp4xx:**
`
go install github.com/lobuhi/byp4xx@latest
`

- **Installation of 4-ZERO-3:**

`
git clone https://github.com/Dheerajmadhukar/4-ZERO-3 ~/TOOLS/4-ZERO-3
cd ~/TOOLS/4-ZERO-3
` 


After using one of the tools I got into /admin and then I viewed the page source. Then I saw secret-database/db.json.
So when I went there `http://cdcamxwl32pue3e6mxmdvww5c1v580wz0wkr5cwzo-web.cybertalentslabs.com/admin%2f/secret-database/db.json` I got the flag.
