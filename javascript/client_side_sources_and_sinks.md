## the following keywords can help you find the sinks and sources.
> [!IMPORTANT]
> These keywords are collected from [@fath3ad.22](https://medium.com/@fath3ad.22) writeup on [this link](https://medium.com/@fath3ad.22/understanding-dom-based-xss-sources-and-sinks-c17ae4bc7455)

**Sources**
```
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location.search
document.cookie
document.referrer
```

**Sinks**
```
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

> [!NOTE]
> If you can improve this list feel free to send pull request.💙
