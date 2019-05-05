# DOM XSS

HINT: DOM always appear at DOM object such as document or location

Sources: where the input go, DOM object (document or location)
- document.url
- document.referer
- location
- location.href
- location.search
- location.hash
- location.pathname


Sinks: where the output end and prompt XSS, javascript function that do operation work with DOM
- element.innerHTML()
- element.outerHTML()
- eval()
- setTimeout()
- setInterval()
- document.write()
- document.writeln()
