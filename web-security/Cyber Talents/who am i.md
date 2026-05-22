# who am i?

View Page Source. Found credentials: Guest:Guest

Login using those creds. Use Cookie editor to view the cookie.
It shows: `Authentication: bG9naW49R3Vlc3Q%3D`.

`bG9naW49R3Vlc3Q%3D` is just URL encoded + base64 encoded. So after decoding it looks like: `login=Guest`.
So make that `login=admin` and then base64 decode + url decode. Set `Authentication:bG9naW49YWRtaW4%3D`

Save and Reload.
Got the flag
