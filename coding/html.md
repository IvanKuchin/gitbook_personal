# HTML

## CORS (Cross Origin Resource Sharing)

Resources (audio, images, scripts, link, video) belongs to original URL can be used w/o any hassle. Original URL includes: protocol, domain, port. If any of these is different browser will not send reply from remote server to the frontend. This is matter to prevent a cross-site request forgery attack.&#x20;

Attack: front-end from malicious web-site can send AJAX-requests to valid sites already been authenticated in the user's browser (for example: API call to fb.com will already contains authenticated session id)

Basic mechanism (w/o crossorigin attribute):

1. Front-end of https://my-domain/ sends AJAX/fetch to remote image https://remote/image
2. Browser sends request _without_ Origin-field in HTTP-request header
3. Any reply from https://remote/image will be dropped by browser with the CORS-exception

Basic mechanism (w/ [crossorigin attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin) `<img src="https://remote/image" crossorigin="anonymous">`):

1. Front-end of https://my-domain/ sends AJAX/fetch to remote image https://remote/image
2. Browser sends request _with_ Origin-field in HTTP-request header
   1. If remote server reply with `access-control-allow-origin: *` or `access-control-allow-origin: https://my-domain`, then front-end page will receive reply
   2. Any other reply from https://remote/image will be dropped by browser with the CORS-exception



